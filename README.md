# AskRAG API

　
## 本プロジェクトについて
- 本プロジェクトでは、Amazon Bedrockを用いて、企業内ドキュメントを活用した RAG（Retrieval-Augmented Generation）型のQAシステム を構築した。
- RAG を “最小構成” で実現しつつ、AWS サーバーレスを活用してコスト最小化を意識した API ベースの設計と実装を行うことをテーマとしている。
- API 設計・Lambda 実装・RAG/LLM プロンプト設計・SAM による IaC まで一貫して実施した。

| 用語 | 説明 |
|------|------|
| AWS Bedrock | AWS が提供する フルマネージドの生成AIプラットフォーム。 |

## 機能概要
本システムは、「Ingest API」と「Query API」の 2 つの API で構成されている。
### Ingest API（テキストデータ登録API）
- RAG に利用するテキストデータを登録するための API
- クライアントから送信されたテキストデータを受け取り、システム内部のストレージに保管する
- 保存されたテキストデータは、後にRAGの検索対象（ナレッジ）として利用される
  
### Query API（質問応答API）
- クライアントの質問に対し、RAG による検索結果をコンテキストとして LLM が回答を生成する API
- 見つかったナレッジをもとに AI が最適な回答を返す（見つからない場合はハルシネーション防止のため一般知識で回答しない）

※本プロジェクトの API 仕様は Swagger（OpenAPI）で定義しており、  リポジトリ内の Swagger UI から確認できる。
**[Swagger UI を開く](https://yuta640karu-source.github.io/Portfolio/index.html)**  


## アーキテクチャ構成・処理フロー
本システムは AWS のサーバーレスアーキテクチャを用いて構築している。
なお、本プロジェクトでは PoC（個人開発）のため Lambda は VPC 外で実装する。
実運用ではVPC内に配置し閉域化構成となる想定。

<img width="725" height="495" alt="image" src="https://github.com/user-attachments/assets/736e1deb-182c-4425-9ec4-b9f12d3d7ca7" />

### 説明
| 項目 | 内容 |
|------|------|
| **Bedrock Runtime** | RAG context と query を LLM に送信し、回答生成を行う推論用エンドポイント。 |
| **Bedrock Knowledge Bases(KB)** | S3 を自動同期し、チャンク化・Embedding・インデックス化を自動実行することで、RAG の前処理をすべてマネージド化。 |
| **S3 Vectors** | Bedrock Knowledge Base が生成した文書 Embedding（ベクトル）を格納し、類似度検索を高速に行うためのベクトルストア。 |
 
- API Gateway で受け付けたリクエストは、Lambdaに引き渡され、処理内容に応じて 「Ingest API 」 または 「Query API」 を実行する。
- Ingest APIで連携されたデータはS3に格納される。Bedrock Knowledge Bases は S3をデータソースとして自動的に同期・インデックス化を行い、質問応答に利用するためのRAG基盤を構築する。
- Query API では、Lambda が Knowledge Bases に対して関連文書の検索を行い、取得したcontextとユーザーから受け取ったqueryを基にプロンプトを生成する。生成したプロンプトはBedrockのLLMに送信され、LLMが最終的な回答を生成する。

## 📡 デモ

### テストデータ
以下のテキストデータを Ingest API を通じて Bedrock Knowledge Base に登録している。
| No | タイトル | テキスト内容 |
|---|------|------|
| 1 | 会社概要 | 当社は、情報システムの企画・設計・開発を行うテクノロジー企業です。業務アプリケーションの開発、クラウド環境の構築、データ活用基盤の整備など、企業のIT活用を支援するサービスを提供しています。また、近年は生成AIや自然言語処理を活用した業務効率化ソリューションにも取り組み、組織の生産性向上とデジタル化の推進に貢献しています。|
| 2 | サービス概要 | 社内ドキュメントを安全に取り込み、LLM を活用して自然な対話形式で検索できる RAG 基盤を提供しています。就業規則や規程類、手順書、設計書などをナレッジベースに登録することで、従業員は「有給休暇の付与条件を教えて」「このシステムの問い合わせ窓口は？」といった曖昧な質問に対しても、関連ドキュメントを根拠とした回答を得ることができます。API ベースの提供のため、既存システムや社内ポータルとの連携も容易です。 |

### デモ用画面
API実行のデモ用画面を用意した。
**[デモ用画面 を開く](https://yuta640karu-source.github.io/Portfolio/demo.html)**  
※セキュリティの都合上、開発者のローカルPCにてのみ実行可能

### 実行結果
<img width="633" height="581" alt="image" src="https://github.com/user-attachments/assets/597a40fe-f4d3-4c2c-8417-eed062e166ff" width="40%" />


<img width="612" height="638" alt="image" src="https://github.com/user-attachments/assets/5f663600-53bd-4e2c-b4b1-e84b120a6c75" width="40%"/>



<img width="624" height="607" alt="image" src="https://github.com/user-attachments/assets/e52645d4-f704-4de7-8bc3-2eaf77d76b1c" width="40%"/>




 
## 🧩 工夫点
| 分類 | 工夫点 | 説明 |
|------|--------|------|
| アーキテクチャ | Bedrock Knowledge Basesを活用したRAGの構築 | RAG の検索基盤として Bedrock Knowledge Bases（KB）を採用。文書のチャンク化、Embedding 生成、インデックス管理をすべてフルマネージドで実行できるため、自前で OpenSearch などの複雑な検索基盤を構築・運用する必要がない。S3に文書を置くだけで KB が自動同期し、検索可能な状態を維持できるため、**RAG の実装を大幅に簡素化し、運用コストも削減**した。 |
| アーキテクチャ | Vector StoreとしてS3 Vectorsを採用 |小規模・個人開発では OpenSearch Serverless の最低構成でもランニングコストが高くなるため、S3 Vectorsを採用した。S3 はサーバレスかつストレージ従量課金でアイドルコストが発生しないため、**RAG に必要なベクトル検索基盤を極めて低コストで運用できる**。|
| インフラ構築 | IaC（Infrastructure as Code）によるインフラ自動構築 | VPC、API Gateway、Lambda などの主要な AWS リソースは **AWS SAM によりコード化して自動で構築**し、再現性の高い環境構築と変更管理を可能にした。 |
| API仕様 | LLMモデルの可変対応（モデル切り替え機能） | API リクエストでモデル ID を指定することで、用途に応じて任意の LLM を利用可能。Claude、Llama、Titan、Cohere など Bedrock 上の任意モデルを柔軟に選択でき、**精度・速度・コスト要件に応じて最適なモデルを切り替えられる設計**とした。 |
| API仕様 | Explainable AIのためのAPIレスポンス設計 | RAG の透明性を高めるため、APIレスポンスに以下の項目を設けた：<br>・rag_used：RAGを利用したか否か<br>・hit_count：RAGが返却したチャンク数<br>・context[]：LLM が返却したチャンク本文とそのメタ情報<br>LLMが**どの文書を参照して回答したかを可視化することで、業務システムで求められる説明責任と信頼性を確保**している。 |

## 🚀実運用（商用環境）に向けた拡張ポイント

| 項目 | 内容 |
|------|-------|
| **セキュリティ強化（閉域化）** | 実運用では Lambda を VPC 内に配置し、Bedrock・S3 へのアクセスは VPC Endpoint 経由で完結させる構成を想定。インターネットを経由しないことで情報漏洩リスクを低減する。 |
| **認証・認可** | デモでは API Key を利用。実運用では Amazon Cognito または Azure AD などの IdP と OIDC/OAuth2.0 連携を行い、より安全で運用しやすい認証基盤へ拡張する。 |
| **RAG 精度向上** | 現状はシンプルなチャンク分割。商用環境では Parent-Child Indexing、構造化チャンク、ハイブリッド検索、メタデータ検索などを用い、ドキュメント増加時でも検索精度を維持できる構成を想定。 |
