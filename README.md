# coding-agent-rules

AI コーディングエージェントに実装や技術文書の作成・変更を任せる前に、コードの構造・責務・変更方針・文書作成方針を固定するためのルール集です。

このリポジトリの目的は、機能仕様そのものを定義することではありません。  
「何を作るか」ではなく、「どのようなコードとして実装するか」「どのような文書として記録するか」を先に決め、エージェントの設計・編集上の自由度を適切に制限することを目的とします。

## 構成

- `rules/common.md` : 言語に依存しない共通ルール
- `rules/c.md` : C 言語向けルール
- `rules/c-plus.md` : C++ 固有の追加ルール
- `rules/c-sharp.md` : C# 固有の追加ルール
- `rules/python.md` : Python 向けルール
- `rules/javascript.md` : JavaScript 向けルール
- `rules/documentation.md` : README、仕様書、設計資料等の技術文書向けルール
- `rules/windows-desktop.md` : MFC / Win32 / Windows Forms / WPF / WinUI 等の Windows デスクトップ GUI アプリケーション向けルール

## 使い方

各プロジェクトでは、まず `rules/common.md` を基本ルールとして適用し、使用言語に応じて言語別ルールを追加します。

C++ では `rules/c.md` を基礎として `rules/c-plus.md` を追加適用します。  
C# では `rules/c.md` の設計・安全性に関する考え方を基礎として `rules/c-sharp.md` を追加適用し、C 固有の低レベル規則の適用範囲は `c-sharp.md` の記載に従います。

README、仕様書、設計資料等の技術文書を作成・変更する場合は、`rules/documentation.md` も適用します。

Windows デスクトップ GUI アプリケーションでは、言語別ルールとは別に `rules/windows-desktop.md` を明示的に適用します。  
特に MFC プロジェクトでは、C++ であることだけを理由にこのルールが自動的に適用されることを前提とせず、プロジェクト側の `AGENTS.md` 等から参照先を明記します。

例えば MFC プロジェクトでは、次のように指定します。

```text
このプロジェクトは MFC を使用する Windows GUI アプリケーションです。
実装・変更時は、次のルールを必ず参照してください。

- coding-agent-rules/rules/common.md
- coding-agent-rules/rules/c.md
- coding-agent-rules/rules/c-plus.md
- coding-agent-rules/rules/windows-desktop.md
```

C# の Windows Forms / WPF / WinUI 等では、次のように指定します。

```text
このプロジェクトは C# を使用する Windows GUI アプリケーションです。
実装・変更時は、次のルールを必ず参照してください。

- coding-agent-rules/rules/common.md
- coding-agent-rules/rules/c.md
- coding-agent-rules/rules/c-sharp.md
- coding-agent-rules/rules/windows-desktop.md
```

プロジェクト固有の制約がある場合は、このリポジトリの内容を直接増やし続けるのではなく、各プロジェクト側に追加ルールを置きます。

優先順位は次のとおりです。

1. プロジェクト固有ルール
2. 言語別ルール / Windows デスクトップアプリ向けルール / 文書ルール
3. 共通ルール

ルール同士が矛盾する場合は、上位のルールを優先します。

## 基本方針

AI は実装担当または文書編集担当であり、プロジェクト全体の設計方針や仕様を勝手に変更してはいけません。

変更前に既存コード、既存文書、既存ルールを確認し、必要最小限の差分で変更します。  
新しい抽象化、依存ライブラリ、公開 API、ディレクトリを導入する場合は、その必要性を説明できる状態にします。

## 今後追加したいもの

- テスト方針
- Git / PR の運用ルール
- 組み込み向け追加ルール
- Node.js / Browser JavaScript の差分
- MicroPython 向け追加ルール
