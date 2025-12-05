# RAG + LLM Serverless API

　
## 📘 Overview
XX

　
## 🏗 Architecture
- API Gateway で受け付けたリクエストは、VPC 内に配置されたLambdaに引き渡され、処理内容に応じて 「文書の登録（ingest API） 」 または 「質問への回答生成（query API）」 を実行する。
- Ingest APIで連携されたデータはS3に格納される。Bedrock Knowledge Bases は S3をデータソースとして自動的に同期・インデックス化を行い、質問応答に利用するためのRAG基盤を構築する。
- Query API では、Lambda が Knowledge Bases に対して関連文書の検索を行い、取得したcontextとユーザーから受け取ったqueryを基にプロンプトを生成する。生成したプロンプトはBedrockのLLMに送信され、LLMが最終的な回答を生成する。

<img width="826" height="498" alt="image" src="https://github.com/user-attachments/assets/c5907f52-d783-440f-a40a-280101635bd2" />



## 📡 API Endpoints
### POST /ingest
XX

### POST /query
XX

　
## 🧩 工夫点（Highlights）
<img width="871" height="483" alt="image" src="https://github.com/user-attachments/assets/176adcc7-6d45-4853-b5ee-93d13d282f7f" />
