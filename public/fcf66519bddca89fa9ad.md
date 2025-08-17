---
title: Microsoftが提案する新しいプロンプト管理
tags:
  - VSCode
  - prompt_engineering
  - poml
private: true
updated_at: '2025-08-17T13:51:08+09:00'
id: fcf66519bddca89fa9ad
organization_url_name: null
slide: false
ignorePublish: false
---

## はじめに

### プロンプトエンジニアリングが直面する「管理の壁」

ChatGPTをはじめとする大規模言語モデル（LLM）の能力を最大限に引き出す「プロンプトエンジニアリング」。しかし、プロジェクトが進行するにつれて、プロンプトはどんどん長く、複雑になっていきます。

- 「あのタスクで使ったプロンプト、どこに保存したっけ？」
- 「このプロンプト、どの部分がどういう意図だっけ？」
- 「チームメンバーとプロンプトを共有・改善するのが大変...」

このような「プロンプト管理の壁」に突き当たっている方も多いのではないでしょうか。プロンプトが単なるテキストとして扱われる限り、再利用性や可読性、共同作業の効率には限界があります。

### この記事が提供する解決策とゴール

本記事では、この課題を解決する新しいアプローチとして、Microsoftが開発した**POML (Prompt Orchestration Markup Language)** を紹介します。POMLは、プロンプトを構造化し、再利用性を高めるために設計されたマークアップ言語です。

この記事を読み終える頃には、以下のことができるようになります。

- POMLの基本的な概念とメリットを理解する。
- POMLを使って、構造的で読みやすいプロンプトを作成する。
- VSCode拡張機能やPython/TypeScript SDKを使い、効率的にプロンプトを管理・活用する。

それでは、プロンプト開発の新しい世界を見ていきましょう。

## POMLとは？ - プロンプトのためのマークアップ言語

### Microsoftが提唱する新しいプロンプトの形

POMLは、Microsoft Researchによって開発が進められている、プロンプトを記述するための新しいマークアップ言語です。その最大の特徴は、プロンプトとその構成要素を「モデル化」し、明確な構造を与える点にあります。

HTMLがウェブページの構造を定義するように、POMLはプロンプトの構造を定義します。これにより、人間にとっても機械にとっても、プロンプトの意図が理解しやすくなります。

### POMLの3つのコアコンセプト：構造化、再利用性、可読性

POMLは、以下の3つのコンセプトを重視しています。

1.  **構造化 (Structuring):** `<role>`や`<examples>`といったタグを用いて、プロンプトの役割や構成要素を意味的に区別します。これにより、複雑な指示も論理的に整理できます。
2.  **再利用性 (Reusability):** プロンプトを部品（コンポーネント）として定義し、他のプロンプトから参照・再利用できます。これにより、共通の指示や設定を何度も書く手間が省けます。
3.  **可読性 (Readability):** マークアップによってプロンプトの意図が明確になるため、自分自身が後から見返したときや、他のチームメンバーが見たときにも、内容を素早く理解できます。

## セットアップ

POMLを使い始めるには、利用したい環境に応じてSDKをインストールします。

### Python

```bash
# 安定版
pip install --upgrade poml

# 最新のナイトリービルド
pip install --upgrade --pre --index-url https://test.pypi.org/simple/ --extra-index-url https://pypi.org/simple/ poml
```

### Node.js / TypeScript

```bash
# 安定版
npm install pomljs

# ナイトリービルド
npm install pomljs@nightly
```

## POMLの主要な構文と特徴

POMLでは、直感的なタグを使って様々なプロンプトテクニックを実装できます。

### 基本構造: `<poml>`, `<role>`, `<task>`

プロンプトは`<poml>`タグで囲み、`<role>`で役割を、`<task>`で具体的な指示を記述します。

```xml
<poml>
  <role>あなたは忍耐強い先生です。10歳の子供に概念を説明します。</role>
  <task>提供された画像を参考に、光合成の概念を説明してください。</task>
  
  <img src="photosynthesis_diagram.png" alt="光合成の図" />

  <output-format>
    説明はシンプルで、魅力的で、100語未満にしてください。
    「やあ、未来の科学者！」で始めてください。
  </output-format>
</poml>
```

### Few-shotプロンプト: `<examples>` と `<example>`

LLMに応答の例を示すFew-shotプロンプティングも簡単です。`<examples>`タグで複数の`<example>`をまとめることができます。

```xml
<examples>
  <example>
    <input>What is the capital of France?</input>
    <output>Paris</output>
  </example>
  <example>
    <input>What is the capital of Germany?</input>
    <output>Berlin</output>
  </example>
</examples>
```

### 思考連鎖 (CoT): `<stepwise-instructions>`

複雑なタスクは、`<stepwise-instructions>`とリストを使って段階的な指示に分解できます。

```xml
<stepwise-instructions>
  <list listStyle="decimal">
    <item>まず、テキストの主題を特定します。</item>
    <item>次に、主要な議論を要約します。</item>
    <item>最後に、結論となる文を書きます。</item>
  </list>
</stepwise-instructions>
```

### 外部リソースの参照: `<img>`, `<import>`

POMLの強力な機能の一つが、外部リソースの埋め込みです。ローカルの画像ファイルを`<img src="..." />`で直接参照したり、`<import from="..." />`で他のPOMLファイルやドキュメントを読み込むことができます。

## 【実践】VSCodeとPOMLで変わるプロンプト開発

POMLは、Visual Studio Codeの拡張機能と組み合わせることで、その真価を発揮します。

### VSCode拡張機能のセットアップ

1.  VSCodeのマーケットプレイスで「POML」と検索します。
2.  Microsoftが提供している「POML Language Service」をインストールします。

これにより、`.poml`ファイルに対するシンタックスハイライト、リアルタイムプレビュー、コード補完などが有効になります。

### LLMプロバイダーの設定

VSCodeの`settings.json`に設定を追加することで、POMLから直接LLMを呼び出してプロンプトをテストできます。

**基本的なOpenAIの設定:**
```json
{
  "poml.languageModel.provider": "openai",
  "poml.languageModel.model": "gpt-4o",
  "poml.languageModel.apiKey": "sk-your-api-key-here"
}
```

**他のプロバイダーの設定例 (Azure, Anthropic, Google):**
```json
// Azure OpenAI
{
  "poml.languageModel.provider": "microsoft",
  "poml.languageModel.model": "my-gpt4-deployment",
  "poml.languageModel.apiKey": "your-azure-api-key",
  "poml.languageModel.apiUrl": "https://your-resource.openai.azure.com/"
}

// Anthropic
{
  "poml.languageModel.provider": "anthropic",
  "poml.languageModel.model": "claude-3-5-sonnet-20240620",
  "poml.languageModel.apiKey": "your-anthropic-api-key"
}

// Google
{
  "poml.languageModel.provider": "google",
  "poml.languageModel.model": "gemini-1.5-pro",
  "poml.languageModel.apiKey": "your-google-api-key"
}
```

## 【応用】プログラムからPOMLを呼び出す

POMLはPythonやTypeScript/JavaScript用のSDKも提供されており、アプリケーションにプロンプト機能を組み込むことができます。

### Python SDK

```python
from poml.parser import PomlParser

def process_poml(file_path):
    parser = PomlParser()
    # ファイルをパースして中間表現を取得
    result = parser.parse_file(file_path)
    
    # レンダリングして最終的なプロンプト文字列を取得
    # (変数を埋め込む場合は render() の引数に渡す)
    return result.rendered_prompt

# 使用例
prompt_text = process_poml('./my-prompt.poml')
print(prompt_text)

# あとはこのプロンプトを任意のLLMクライアントに渡す
# response = your_llm_client.chat(prompt_text)
```

### TypeScript/JavaScript SDK (`pomljs`)

```typescript
import { PomlParser } from 'pomljs'; // or '@microsoft/poml-parser' depending on version
import * as fs from 'fs/promises';

async function process_poml(filePath: string): Promise<string> {
  const parser = new PomlParser();
  const fileContent = await fs.readFile(filePath, 'utf-8');
  
  // ファイル内容をパース
  const result = await parser.parse(fileContent);

  // レンダリングしてプロンプト文字列を取得
  return result.renderedPrompt;
}

// 使用例
process_poml('./my-prompt.poml').then(prompt => console.log(prompt));
```

## POMLの将来性と展望

### なぜ今、POMLに注目すべきなのか

POMLはまだ新しい技術ですが、以下の理由から大きな可能性を秘めています。

- **Microsoftによる強力なサポート:** 世界的なソフトウェア企業が開発を主導しており、今後の発展とエコシステムの拡大が期待できます。
- **標準化への期待:** プロンプトの記述方法が標準化されることで、異なるツールやプラットフォーム間でのプロンプトの共有が容易になります。
- **エンタープライズでの活用:** 構造化され管理しやすいプロンプトは、大規模な組織でのLLM活用において不可欠な要素となります。

## まとめ

本記事では、Microsoftが開発したプロンプト記述言語「POML」について、具体的なコードを交えて解説しました。

- POMLは、プロンプトを**構造化・再利用・可読性向上**させるマークアップ言語です。
- `pip`や`npm`で簡単にSDKをインストールできます。
- `<role>`, `<examples>`, `<img>`などのタグで、高度なプロンプトを直感的に記述できます。
- VSCode拡張機能と連携し、`settings.json`で設定することで、開発サイクルを劇的に効率化します。
- PythonやTypeScriptのアプリケーションから簡単に利用できます。

複雑化するプロンプトエンジニアリングの世界において、POMLは強力な味方となるでしょう。ぜひ一度、試してみてはいかがでしょうか。

## 参考資料

- [POML Documentation](https://microsoft.github.io/poml/latest/)
- [POML Components](https://microsoft.github.io/poml/latest/language/components/)
- [新しいプロンプト言語POMLでLLMのプロンプトを構造化してみる](https://azukiazusa.dev/blog/poml-prompt-structured-document/)
