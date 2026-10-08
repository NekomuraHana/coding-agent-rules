# C# Coding Rules

この文書は、C# で AI エージェントが実装する際の追加ルールを定義します。  
`common.md` と `c.md` の考え方を基礎とし、この文書には C# 固有の差分だけを記載します。

`c.md` のうち、所有権、境界確認、エラー処理、状態管理、テスト容易性等の考え方は C# でも適用します。  
一方、`malloc`、生ポインタ、C 配列、ISR 等の C 固有ルールは、unsafe code、native interop、組み込み等で該当する場合を除き、そのまま適用しません。

## 1. null と型安全性

- Nullable Reference Types が有効なプロジェクトでは、その警告を無視しない。
- Nullable Reference Types が無効な既存プロジェクトへ、依頼なくプロジェクト全体の設定変更を行わない。
- `null!` による警告抑制は、初期化順序等の理由が明確な場合に限定する。
- `object`、dynamic、不要な型変換を乱用しない。
- 値の種類が限定される場合は、文字列定数の乱用より enum 等を検討する。

## 2. リソースと IDisposable

- `IDisposable` / `IAsyncDisposable` を実装するリソースは、所有者が明確に解放する。
- ローカルで所有する disposable resource は、原則として `using` / `await using` を使用する。
- 自分が所有していない `IDisposable` を勝手に Dispose しない。
- Stream、Socket、Timer、CancellationTokenSource、native handle 等の寿命を曖昧にしない。
- finalizer は、native resource を直接所有する等の必要がある場合に限定する。

## 3. 例外処理

- 例外を通常の分岐処理の代用にしない。
- 外部 I/O、framework、library 等が例外で失敗を通知する場合は、回復・記録・変換を行える適切な境界で処理する。
- ファイル、network、database 等の API 呼び出しで例外が発生し得る場合でも、タイムアウト、not found、入力不正、再試行可能な通信失敗等をアプリケーション上の通常結果として扱う設計なら、上位層では Result / status / Try pattern 等へ変換する。
- `TryParse`、`TryGetValue` 等、失敗が通常ケースとして想定された API が提供されている場合は、例外を発生させる方法よりそれらを優先する。
- catch は、回復、エラー形式の変換、必要なログ記録等を実際に行える箇所に置き、理由なく握り潰さない。
- 元の stack trace を維持して再送出する場合は `throw;` を使用し、`throw ex;` を使用しない。
- 外部 API や I/O の例外を変換する場合は、呼び出し側が必要とする情報を失わない。
- `OperationCanceledException` / `TaskCanceledException` 等、framework が制御状態を例外として表現するものは、通常の障害と同一視せず既存の cancellation 方針に従う。
- 独自例外型は、呼び出し側が種類を識別する実益がある場合だけ追加する。

## 4. async / await

- 非同期 API では、可能な限り async/await を end-to-end で維持する。
- `.Result`、`.Wait()` 等による同期ブロックは、deadlock や応答停止の原因になるため原則として避ける。
- `async void` は、イベントハンドラ等の必要な場合に限定する。
- 長時間処理やキャンセル可能な処理では、必要に応じて `CancellationToken` を受け渡す。
- fire-and-forget を行う場合は、例外処理と寿命を明確にする。
- ライブラリコードでの `ConfigureAwait` は、既存プロジェクトの方針に従う。

## 5. コレクションと LINQ

- 公開 API では、必要以上に変更可能なコレクションを公開しない。
- 呼び出し側に変更させる必要がない場合は、read-only interface 等を検討する。
- LINQ は可読性が向上する場合に使用し、単純な処理を過度に連結して読みにくくしない。
- 同じ `IEnumerable<T>` を意図せず複数回列挙しない。
- 大量データや高頻度処理では、不要な allocation、boxing、LINQ の中間生成を意識する。

## 6. クラスと API 設計

- 不要な継承より、interface と合成を優先する。
- interface は、実際に差し替え・境界分離が必要な責務に対して定義する。
- 1実装しかなく将来性も不明な箇所へ、形式だけの interface を大量に追加しない。
- public API を内部実装の都合だけで増やさない。
- setter を公開する必要がないプロパティは、read-only または限定された setter を検討する。
- record、init-only property 等の採用は、対象言語バージョンと既存コードの方針に従う。

## 7. イベントとコールバック

- event を購読したオブジェクトの寿命を確認し、不要になった購読を解除する。
- 長寿命オブジェクトから短寿命オブジェクトへの購読によるメモリリークに注意する。
- callback 内で重い処理や同期ブロックを行わない。
- イベント発火元と受信側の thread / SynchronizationContext の前提を曖昧にしない。

## 8. native interop / unsafe

- `unsafe` は、性能上または native interop 上必要な場合に限定する。
- unsafe code や P/Invoke は、可能な限り限定された境界へ集約する。
- native handle の所有権は明確にし、可能であれば `SafeHandle` を使用する。
- P/Invoke では calling convention、文字列エンコーディング、構造体レイアウト、サイズ、寿命を確認する。
- native buffer を扱う場合は、`c.md` のポインタ・境界・所有権ルールも適用する。

## 9. スレッドと共有状態

- 共有される可変状態を必要最小限にする。
- lock、SemaphoreSlim、Interlocked、concurrent collection 等を用途に応じて使い分ける。
- lock 内で長時間処理や await を行わない。
- static mutable state を増やさない。
- UI thread に関する追加ルールは、Windows デスクトップアプリでは `windows-desktop.md` に従う。

## 10. スタイルと解析

- 既存の `.editorconfig`、analyzer、formatter、IDE 設定を優先する。
- 機能変更に不要な全体整形や style-only diff を行わない。
- analyzer warning を抑制する場合は、修正できない理由を明確にする。
- warning suppression を問題解決の代わりに使用しない。

## 11. テスト

- `c.md` のテスト方針のうち、責務分離、外部依存の Mock / Stub、境界値確認、未実行テストの明示等を適用する。
- `c.md` に記載された C 向け Unity の指定は C# には適用しない。
- 既存の MSTest / NUnit / xUnit 等がある場合は、それを優先する。
- テストのためだけに production code の public API を増やさない。
- 時刻、乱数、I/O、通信等に依存する処理は、必要に応じて差し替え可能な境界へ分離する。
