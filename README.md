# AskRAG API

　
## 📘 概要
- 本プロジェクトでは、Amazon Bedrockを用いて、企業内ドキュメントを活用した RAG（Retrieval-Augmented Generation）型のQAシステム を構築した。

| 用語 | 説明 |
|------|------|
| AWS Bedrock | AWS が提供する フルマネージドの生成AIプラットフォーム。 |

- 本システムは、RAG に登録するテキストデータを S3 経由でナレッジベースへ投入する「Ingest API」と、LLM を用いて質問に対する最適な回答を生成する「Query API」の 2 つの API で構成されている。これにより、企業型ドキュメントの取り込みから質問応答までを、シンプルな構成で実現している。

## 🏗 アーキテクチャ構成・処理フロー
本システムは AWS のサーバーレスアーキテクチャを用いて構築している。
構成は、「LLMによる推論パイプライン」と、Bedrock Knowledge Bases を用いた 「RAGパイプライン」の2つから成る。
なお、本プロジェクトでは PoC（個人開発）のため Lambda は VPC 外で実装する。
実運用ではVPC内に配置し閉域化構成となる想定。

### アーキテクチャ構成の説明
| 項目 | 内容 |
|------|------|
| **Bedrock Runtime** | RAG context と query を LLM に送信し、回答生成を行う推論用エンドポイント。 |
| **Bedrock Knowledge Bases(KB)** | S3 を自動同期し、チャンク化・Embedding・インデックス化を自動実行することで、RAG の前処理をすべてマネージド化。 |
| **S3 Vectors** | Bedrock Knowledge Base が生成した文書 Embedding（ベクトル）を格納し、類似度検索を高速に行うためのベクトルストア。 |
 
<img width="725" height="495" alt="image" src="https://github.com/user-attachments/assets/736e1deb-182c-4425-9ec4-b9f12d3d7ca7" />






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
| 分類 | 工夫点 | 説明 |
|------|--------|------|
| アーキテクチャ | Bedrock Knowledge Basesを活用したRAGの構築 | RAG の検索基盤として Bedrock Knowledge Bases（KB）を採用。文書のチャンク化、Embedding 生成、インデックス管理をすべてフルマネージドで実行できるため、自前で OpenSearch などの複雑な検索基盤を構築・運用する必要がない。S3に文書を置くだけで KB が自動同期し、検索可能な状態を維持できるため、**RAG の実装を大幅に簡素化し、運用コストも削減**した。 |
| アーキテクチャ | Vector StoreとしてS3 Vectorsを採用 |小規模・個人開発では OpenSearch Serverless の最低構成でもランニングコストが高くなるため、S3 Vectorsを採用した。S3 はサーバレスかつストレージ従量課金でアイドルコストが発生しないため、**RAG に必要なベクトル検索基盤を極めて低コストで運用できる**。|
| インフラ構築 | IaC（Infrastructure as Code）によるインフラ自動構築 | VPC、API Gateway、Lambda などの主要な AWS リソースは **AWS SAM によりコード化して自動で構築**し、再現性の高い環境構築と変更管理を可能にした。 |
| API仕様 | LLMモデルの可変対応（モデル切り替え機能） | API リクエストでモデル ID を指定することで、用途に応じて任意の LLM を利用可能。Claude、Llama、Titan、Cohere など Bedrock 上の任意モデルを柔軟に選択でき、**精度・速度・コスト要件に応じて最適なモデルを切り替えられる設計**とした。 |
| API仕様 | Explainable AIのためのAPIレスポンス設計 | RAG の透明性を高めるため、APIレスポンスに以下の項目を設けた：<br>・rag_used：RAGを利用したか否か<br>・hit_count：RAGが返却したチャンク数<br>・context[]：LLM が返却したチャンク本文とそのメタ情報<br>LLMが**どの文書を参照して回答したかを可視化することで、業務システムで求められる説明責任と信頼性を確保**している。 |
| API仕様 | LLMのプロンプトおよび回答の加工 | XX |

