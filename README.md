# RAG + LLM Serverless API

　
## 📘 概要
XX

　
## 🏗 アーキテクチャ構成・処理フロー
本システムは AWS サーバーレスを用いて構築しており、以下の２つの要素で構成される。
- API Gateway → Lambda → Bedrock → LLM による推論パイプライン
- S3 → Bedrock Knowledge Bases による RAG 検索パイプライン
 
【アーキテクチャ】
1. Lambdaの配置
・Lambda は Private Subnet A / B の 2つのサブネットに配置し、冗長構成を確保。
・外部インターネットへ直接出ない閉域構成としてセキュアな実行環境を構築。

2. Bedrock / S3 へのプライベート接続
- Lambda から Bedrockにアクセスする際は、PrivateLinkを経由することでインターネット非公開の安全な通信を実現。
- S3へのデータのアップロードはGateway VPC Endpoint を通じて実施。

3. Bedrock Knowledge Bases による RAG 検索基盤
- S3 をデータソースとし、KB が自動でチャンク化・Embedding 生成・インデックス化を実行。
- RAGの検索処理は Bedrock Agent Runtime により提供され、サーバーレスで運用可能。

4. Bedrock Runtimeによる推論処理
- RAG 検索結果（context）と query を基にLLMへプロンプトを送信。
- Claude / Llama / Titan / Cohere など 任意のモデルを可変利用できる設計

<img width="826" height="498" alt="image" src="https://github.com/user-attachments/assets/c5907f52-d783-440f-a40a-280101635bd2" />

【処理フロー】
- API Gateway で受け付けたリクエストは、VPC 内に配置されたLambdaに引き渡され、処理内容に応じて 「文書の登録（ingest API） 」 または 「質問への回答生成（query API）」 を実行する。
- Ingest APIで連携されたデータはS3に格納される。Bedrock Knowledge Bases は S3をデータソースとして自動的に同期・インデックス化を行い、質問応答に利用するためのRAG基盤を構築する。
- Query API では、Lambda が Knowledge Bases に対して関連文書の検索を行い、取得したcontextとユーザーから受け取ったqueryを基にプロンプトを生成する。生成したプロンプトはBedrockのLLMに送信され、LLMが最終的な回答を生成する。





## 📡 API Endpoints
### POST /ingest
XX

### POST /query
XX

　
## 🧩 工夫点
<img width="871" height="483" alt="image" src="https://github.com/user-attachments/assets/176adcc7-6d45-4853-b5ee-93d13d282f7f" />
