# AskRAG API

　
## 📘 概要
- Amazon Bedrock Knowledge Bases と AWS Serverless を活用したドキュメント検索・質問応答 API プラットフォーム

| 用語 | 説明 |
|------|------|
| AWS Bedrock | AWS が提供する サーバーレスの生成AIサービス |
| Bedrock Knowledge Bases(KB) | Amazon Bedrockサービスが提供するフルマネージドの RAG 検索基盤 |
| AWS Serverless | アプリケーションを動かすための サーバー管理が不要になる AWS のアーキテクチャモデル のこと |

## 🏗 アーキテクチャ構成・処理フロー
本システムは AWS のサーバーレスアーキテクチャを用いて構築している。
構成は、「LLMによる推論パイプライン」と、Bedrock Knowledge Bases を用いた 「RAGパイプライン」の2つから成る。
なお、本プロジェクトでは PoC（個人開発）のため Lambda は VPC 外で実装する。
実運用ではVPC内に配置し閉域化構成となる想定。

### アーキテクチャ構成の説明
| 項目 | 内容 |
|------|------|
| **Bedrock Knowledge Bases(KB)** | S3 を自動同期し、チャンク化・Embedding・インデックス化を自動実行。 |
| **Bedrock Runtime** | RAG context + query を LLM に送信し、回答を生成する。 |
| **Vector Store(DynamoDB Vector Index)** | Bedrock Knowledge Base が生成した文書 Embedding（ベクトル）を格納し、類似度検索を高速に行うためのベクトルストア。 |
 
<img width="720" height="486" alt="image" src="https://github.com/user-attachments/assets/c85d568e-2ea4-4540-a578-6ad94e5ecebb" />





### 処理フローの説明
- API Gateway で受け付けたリクエストは、Lambdaに引き渡され、処理内容に応じて 「文書の登録（ingest API） 」 または 「質問への回答生成（query API）」 を実行する。
- Ingest APIで連携されたデータはS3に格納される。Bedrock Knowledge Bases は S3をデータソースとして自動的に同期・インデックス化を行い、質問応答に利用するためのRAG基盤を構築する。
- Query API では、Lambda が Knowledge Bases に対して関連文書の検索を行い、取得したcontextとユーザーから受け取ったqueryを基にプロンプトを生成する。生成したプロンプトはBedrockのLLMに送信され、LLMが最終的な回答を生成する。

## 📡 API概要
本システムでは以下の 2 種類の API を提供している。
### POST /ingest
XX

### POST /query
XX

本プロジェクトの API 仕様は Swagger（OpenAPI）で定義しており、  
リポジトリ内の Swagger UI から確認できる。
👉 **[Swagger UI を開く（index.html）](./index.html)**  
👉 Swagger 定義ファイル: `./swagger.yaml`

## 📡 デモ
curl -X POST https:XXXXX 
　
## 🧩 工夫点
<img width="871" height="483" alt="image" src="https://github.com/user-attachments/assets/176adcc7-6d45-4853-b5ee-93d13d282f7f" />
