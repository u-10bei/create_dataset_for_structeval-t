# StructEval 学習データセット生成ツール

構造化データ（JSON, XML, YAML, TOML, CSV）の生成・変換タスクに対応するLLMファインチューニング用データセットを作成するツールです。

## 概要

`create_dataset_0108.ipynb` は、以下の2種類のタスクを含む学習データセットをJSONL形式で生成します。

- **Generation**: 自然言語プロンプトから構造化データを生成
- **Conversion**: ある形式から別の形式への変換（例: JSON → XML）

各出力にはChain-of-Thought（CoT）推論が付加されており、モデルがステップバイステップで正確な構造化データを生成することを学習できます。

## 対応スキーマ（17種類）

| 分野 | スキーマ |
|------|---------|
| 学術 | research_paper, experiment_result |
| 医療 | clinical_note, lab_result, prescription |
| ビジネス | contract, financial_report, sales_analytics, transaction_record |
| テクノロジー | api_specification, error_log |
| EC | product_listing, customer_review |
| 教育 | course_syllabus, student_assessment |
| メディア | news_article, social_media_post |

## 出力形式

OpenAIチャット形式のJSONL（system / user / assistant メッセージ）で出力されます。

```json
{
  "messages": [
    {"role": "system", "content": "You are an expert in JSON format..."},
    {"role": "user", "content": "Generate research paper data in JSON format."},
    {"role": "assistant", "content": "Approach:\n1. Task: Create research paper in JSON\n..."}
  ],
  "metadata": {
    "format": "json",
    "complexity": "medium",
    "schema": "research_paper",
    "type": "generation",
    "estimated_tokens": 280
  }
}
```

## 使い方

### 必要なライブラリ

```bash
pip install faker pyyaml toml tqdm datasets huggingface_hub
```

### データセット生成

ノートブック内の `generate_dataset` 関数を実行します。

```python
generate_dataset(
    num_base_examples=500,   # ベース例の数
    output_file="output.jsonl",  # 出力ファイル名
    max_tokens=512           # アシスタント応答のトークン上限
)
```

500個のベース例から約3,900件のトレーニング例が生成されます（Generation + Conversion）。

### Hugging Face Hubへのアップロード

ノートブック後半のセルで、生成したデータセットをHugging Face Hubにプッシュできます。事前に`HF_TOKEN`の設定が必要です。

## パラメータ

| パラメータ | デフォルト値 | 説明 |
|-----------|------------|------|
| `num_base_examples` | 500 | 生成するベースデータの数 |
| `output_file` | `struct_eval_train_cot.jsonl` | 出力ファイルパス |
| `max_tokens` | 1200 | アシスタント応答の最大推定トークン数 |

## 複雑度の分布

- **simple** (20%): 3-5フィールド、最小限のネスト
- **medium** (40%): 5-8フィールド、2-3レベルのネスト
- **complex** (40%): 8-10フィールド、3-4レベルのネスト
