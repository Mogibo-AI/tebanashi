# Technical Environment Document — Tebanashi（手放し）

## Project Technical Summary

- **Project Name**: Tebanashi（手放し）
- **Project Type**: Greenfield
- **Primary Runtime Environment**: Cloud（AWSサーバーレス）
- **Cloud Provider**: AWS
- **Target Deployment Model**: Serverless（Lambda / API Gateway / DynamoDB / Amplify Hosting）
- **Primary Region**: ap-northeast-1（東京）。Bedrockのモデル可用性に応じてクロスリージョン推論プロファイル経由で他リージョンを呼び出す可能性あり。
- **Team Size**: 想定 1〜3名（小規模）
- **Team Experience**: TypeScript／React／AWSサーバーレスの基本知識を前提

---

## Programming Languages

### Required Languages

| Language | Version | Purpose | Rationale |
|----------|---------|---------|-----------|
| TypeScript | 5.x | フロントエンド（React）、バックエンド（Lambda）、IaC（Amplify Gen 2 / CDK） | エンドツーエンドの型安全性。Amplify Gen 2 はTypeScriptファースト |
| JavaScript (ESM) | Node.js 24.x ランタイム互換 | Lambda 実行ランタイム | Lambda の最新 Node.js 24.x ランタイムを採用。新規プロジェクトのため最新LTSで開始する |

> Lambda Node.js ランタイムは **Node.js 24.x** を標準とする。Python は当面採用しない。

### Permitted Languages

| Language | Conditions for Use |
|----------|-------------------|
| Python 3.13+ | Bedrock SDK や音声処理ライブラリで TypeScript より優位なライブラリが必要になった場合のみ、当該 Lambda 関数単位で許可。新規採用時はテックリード承認 |

### Prohibited Languages

| Language | Reason |
|----------|--------|
| Java / Kotlin / Scala | Lambda コールドスタート遅延が UX 要件（音声入力後 5 秒以内）と整合しない |
| Go / Rust | チームの主要スキルセット外。MVP のスコープに対し過剰 |
| PHP / Ruby | プロジェクト方針（サーバーレス＋TypeScript 統一）と不整合 |

---

## Frameworks and Libraries

### Required Frameworks

| Framework/Library | Version | Domain | Rationale |
|-------------------|---------|--------|-----------|
| React | 18.x | フロントエンド UI | Amplify Gen 2 公式サポートフレームワーク |
| Vite | 5.x 以降 | フロントエンドビルドツール | Amplify React クイックスタート標準 |
| AWS Amplify Gen 2 | `@aws-amplify/backend` 最新安定版 | バックエンド定義／IaC／デプロイ | TypeScript-first、Auth/Data/Storage/Functions の統合定義が可能。AWS CDK の上に構築されている |
| AWS Amplify JavaScript Library | `aws-amplify` 最新安定版 | フロントエンドからの認証・データアクセス | Amplify Gen 2 と一体運用 |
| AWS SDK for JavaScript v3 | 最新安定版（モジュラーパッケージ） | Lambda 関数からの AWS サービス呼び出し | v2 はメンテナンス終了。v3 のモジュラー構造は Lambda バンドル最適化に有利 |
| Anthropic SDK for AWS Bedrock | `@anthropic-ai/bedrock-sdk` 最新版 | Bedrock 経由での Claude モデル呼出 | 公式SDK。`AnthropicBedrock` クライアント経由で `messages.create` を発行 |
| Amazon Transcribe Streaming Client | `@aws-sdk/client-transcribe-streaming` 最新版 | 音声リアルタイム文字起こし | Lambda またはブラウザ経由で WebSocket / HTTP/2 ストリームを利用 |

### Preferred Libraries

| Library | Purpose | Use When |
|---------|---------|----------|
| Zod | ランタイム型検証 | 外部入力（API リクエスト、Bedrock レスポンスのJSON）すべてに必須 |
| Pino | 構造化ロギング | すべての Lambda 関数 |
| Vitest | フロントエンド・バックエンド単体テスト | Vite と統合された Jest 互換のテストランナー。Amplify Gen 2 環境で推奨 |
| @aws-amplify/ui-react | 一部UIコンポーネント | Amplify が提供する標準コンポーネント（必要に応じて） |
| Tailwind CSS | スタイリング | UI 全般。祝祭演出のアニメーションは tailwindcss-animate 等を組み合わせて実装 |
| Framer Motion | アニメーション | 「手放す」ボタン押下時の祝祭演出 |
| nanoid | クライアントサイドID生成 | やめ候補IDの先発行など |

### Prohibited Libraries

| Library | Reason | Alternative |
|---------|--------|-------------|
| Moment.js | 非推奨、バンドルサイズ大 | date-fns または Day.js |
| Lodash（フル） | バンドルサイズ大 | ネイティブJSまたは lodash-es の個別 import |
| Request | 非推奨 | ネイティブ fetch または `@aws-sdk/*` クライアント |
| AWS SDK for JavaScript v2 (`aws-sdk`) | v3 がモジュラーで推奨。v2 はメンテナンス終了 | `@aws-sdk/client-*` 個別インポート |
| jQuery | React 環境で不要 | React 標準機能 |
| node-fetch | Node.js 18 以降は fetch がネイティブサポート | グローバル fetch |

### Library Approval Process

新規ライブラリ追加時は、PRに以下を記述する: (1) 用途、(2) ライセンス、(3) 直近6ヶ月のメンテナンス状況、(4) バンドルサイズへの影響、(5) 代替検討結果。テックリードのレビュー＆承認が必要。

---

## Cloud Environment

### Cloud Provider

- **Primary Provider**: AWS
- **Account Structure**: シングルアカウント（MVP）。Phase 2 以降で dev/stg/prod のマルチアカウント分離を検討
- **Regions**:
  - **Primary**: ap-northeast-1（東京）
  - **Secondary（Bedrock 推論プロファイル経由のみ）**: us-west-2 / us-east-1（Bedrockモデル可用性確保のため）
  - **DR**: MVPでは構築しない

### Service Allow List

| Service | Approved Use Cases | Constraints |
|---------|-------------------|-------------|
| AWS Amplify Hosting | フロントエンド（React + Vite）の CDN 配信、CI/CD | Git ブランチと環境を 1:1 マッピング |
| AWS Lambda | API ハンドラ／バックエンドロジック | Node.js 24.x ランタイム。タイムアウトは 30 秒以下を基本（Transcribe Streaming セッション中継用関数は例外として最大 15 分）。アーキテクチャは arm64 を優先 |
| Amazon API Gateway (HTTP API) | REST API 公開 | HTTP API を採用（REST API より低レイテンシ・低コスト） |
| AWS AppSync | GraphQL API（Amplify Data 経由） | Amplify Data 定義から生成。手放しイベントのリアルタイム購読（Phase 2想定） |
| Amazon DynamoDB | やめ候補／手放しイベントの永続化 | オンデマンドキャパシティ（MVP）。`PAY_PER_REQUEST`。テーブルは Amplify Data の自動生成を利用 |
| Amazon Cognito User Pool | Googleソーシャルログインのフェデレーション、ユーザーディレクトリ | ホストUIではなくAmplify Authenticator または独自UIでサインインフローを実装。ID/Access/Refresh トークンを発行 |
| Amazon Cognito Identity Pool | User Pool 認証済みユーザーへのAWS一時クレデンシャル払い出し（ブラウザから直接Transcribe Streaming等を呼ぶため） | Authenticated ロールのみ使用。Unauthenticated（Guest）ロールは無効化 |
| Amazon Bedrock | Claude モデル呼び出し（やめ候補の構造化／全肯定メッセージ生成） | モデルは `anthropic.claude-haiku-4-5` を MVP の標準とし、品質要件に応じて `global.anthropic.claude-sonnet-4-6-v1` 等に切替可能とする。クロスリージョン推論プロファイルを利用 |
| Amazon Transcribe Streaming | 日本語音声のリアルタイム文字起こし | 言語コード `ja-JP`。WebSocket/HTTP-2 ストリーミング |
| Amazon CloudWatch | メトリクス／ログ／アラーム | 全 Lambda は構造化ログを出力。主要 KPI のダッシュボード必須 |
| AWS Secrets Manager | 外部 API キー等のシークレット管理 | MVP範囲では使用最小化（Bedrock/Transcribe は IAM ロールで認証するため不要） |
| AWS Systems Manager Parameter Store | 環境変数・設定値（非シークレット） | Amplify Gen 2 の Branch 環境変数で代替可能な範囲は Amplify を優先 |
| AWS IAM | 権限管理 | 最小権限原則。Lambda ごとに専用ロール |
| Amazon CloudFront | Amplify Hosting 経由で利用 | Amplify Hosting に内包 |

### Service Disallow List

| Service | Reason | Alternative |
|---------|--------|-------------|
| Amazon EC2（直接） | サーバーレス方針と不整合 | Lambda |
| Amazon ECS / Fargate | MVPには過剰 | Lambda |
| Amazon Elastic Beanstalk | IaC ワークフロー（Amplify Gen 2 / CDK）と不整合 | Amplify Hosting + Lambda |
| Amazon RDS / Aurora | リレーショナル要件なし。運用負荷大 | DynamoDB |
| Amazon ElastiCache | 現在のスケールでは不要 | DynamoDB DAX もしくはアプリケーションキャッシュ（必要時） |
| Amazon Kinesis | 複雑度過剰 | EventBridge または DynamoDB Streams |
| Amazon SNS／SES（メール送信） | MVPはメール通知なし。ソーシャルログイン採用のためサインアップ確認メールも不要 | 不要 |
| Amazon Comprehend | 構造化は Bedrock の Claude で実施 | Bedrock |
| Amazon Polly | テキスト読み上げはMVPスコープ外 | 不要 |
| AWS Step Functions | MVPの複雑度では過剰 | Lambda 単発呼出 |
| Amazon S3（MVPでは） | MVPは音声ファイルを永続化しない（Transcribe Streamingで都度処理） | 不要。Phase 2で音声履歴保存検討時に再評価 |

### Service Approval Process

許可リスト外のサービスを使用する場合、PR で (1) 用途、(2) コスト見積、(3) セキュリティ評価、(4) 代替検討結果を提示し、テックリード承認を得る。

---

## Preferred Technologies and Patterns

### Architecture Patterns

| Pattern | When to Use | When Not to Use |
|---------|-------------|-----------------|
| Serverless-first | すべての新規バックエンドコンポーネント | 15分超の処理 |
| Event-driven | 将来の通知／集計／手放しストリーム購読 | MVPの単純な同期処理 |
| Frontend-Backend-for-Frontend (BFF) なし | クライアントから直接 AppSync / Bedrock / Transcribe を呼ぶ | Bedrock 呼出は Lambda 経由（プロンプト保護のため） |

### API Design Standards

- **Style**: GraphQL（AppSync via Amplify Data）を主軸。長時間ストリーミングや音声仲介が必要な箇所のみ HTTP API（API Gateway）+ Lambda
- **Versioning**: GraphQL スキーマ進化（後方互換維持）。HTTP API は URL パスバージョニング（`/v1/...`）
- **Documentation**: GraphQL スキーマは Amplify Data 定義から自動生成。HTTP API は OpenAPI 3.x を必須
- **Naming Convention**: GraphQL は camelCase、HTTP API は kebab-case
- **Error Format**: 統一エラー型（code, message, details）

### Data Patterns

- **Primary Data Store**: Amazon DynamoDB（Amplify Data モデルで定義）
- **データモデル例**:
  - `Card`（=やめ候補）: ownerId（Cognito User Pool の `sub` クレーム）、title、category、estimatedTimeCost、estimatedMoneyCost、status（active / withdrawn）、createdAt、withdrawnAt
  - `WithdrawalEvent`（=手放しイベント）: ownerId、cardId、affirmationMessage、suggestedReward、createdAt
- **データ所有**: ownerId による行レベル認可。Amplify Data の `allow.owner()` ルール準拠
- **キャッシュ**: なし（MVP）

### Messaging and Events

- **同期**: AppSync GraphQL（Query/Mutation）、API Gateway HTTP API
- **非同期**: MVPでは未採用。Phase 2 以降で EventBridge を検討
- **Event Schema**: 採用時は CloudEvents 互換を目指す

### Frontend Patterns

- **Component Library**: 自作コンポーネント中心。Amplify UI の `Authenticator` をソーシャルログイン画面（または独自スタイルでラップ）として利用
- **State Management**: React Context（軽量状態）+ TanStack Query（サーバ状態キャッシュ）
- **Routing**: React Router v6
- **Build Tool**: Vite
- **Styling**: Tailwind CSS + Framer Motion（祝祭演出）

### LLM 呼出パターン

- **モデル選定**: MVP標準は **Claude Haiku 4.5** （`anthropic.claude-haiku-4-5`）。低レイテンシ・低コストで音声駆動UXに適合
- **品質要求が高い場合**: **Claude Sonnet 4.6** （`global.anthropic.claude-sonnet-4-6-v1`）への切替を許可
- **呼出方法**: Lambda から `@anthropic-ai/bedrock-sdk` の `AnthropicBedrock` クライアントで `messages.create` を発行
- **クロスリージョン推論プロファイル**: モデルが東京リージョンで未提供の場合に使用
- **プロンプト管理**: プロンプトは Lambda コード内に定数化。Phase 2 でプロンプトカタログ／A/B 検証基盤を整備
- **ガードレール**: Bedrock Guardrails で「健康・医療・依存症」関連の過度な肯定をブロック
- **構造化出力**: JSON モード（`tool_use` または `response_format` 指定）を活用し、Zod でランタイム検証
- **トークン制限**: 1リクエストあたり max_tokens を 1024 以下に制限（コスト抑制）

### 音声入力パターン

- **取得**: ブラウザ Web Audio API + MediaRecorder（PCM 16kHz）
- **送信方式**: ブラウザから API Gateway WebSocket → Lambda → Transcribe Streaming に中継、またはブラウザから直接 Transcribe Streaming（Cognito Identity Pool が User Pool 認証済みユーザーへ払い出す一時クレデンシャルを利用）
- **採用方式**: **直接 Transcribe Streaming 接続**（Identity Pool の Authenticated ロールに付与した最小権限IAMで `transcribe:StartStreamTranscriptionWebSocket` を許可）。Lambda 中継より低レイテンシ
- **言語コード**: `ja-JP` 固定（MVP）
- **無音検出／自動停止**: クライアントサイドで実装

---

## Security Requirements

### Authentication and Authorization

- **Authentication Method**: Amazon Cognito User Pool 経由でGoogleのソーシャルログイン（OIDC フェデレーション）。User Pool ホストUIまたはAmplify Authenticator を使用
- **Authorization Model**:
  - AppSync: `allow.owner()` ルールで、Cognito User Pool の `sub` クレームをオーナーキーとし、データ行レベルで自身のレコードのみアクセス可能
  - AWS リソースへのブラウザからの直接呼出（Transcribe Streaming 等）: Cognito Identity Pool が Authenticated ロールの一時 AWS クレデンシャルを払い出し、IAM ポリシーで最小権限を付与
- **Token Format**: User Pool 発行の JWT（ID Token / Access Token / Refresh Token）。Identity Pool 経由では AWS STS 一時クレデンシャル
- **Session Management**: Refresh Token による自動更新。トークンは `aws-amplify` ライブラリが管理（既定では in-memory + 永続ストア）。ID Token 有効期限はデフォルトの1時間を採用、Refresh Token は30日間
- **IdP 設定**:
  - Google: Google Cloud Console で OAuth 2.0 クライアントID／シークレットを発行し、User Pool のIdPプロバイダーに登録。要求するスコープは `openid` `email` `profile` の基本のみ（Sensitive／Restricted scope は使用しない）
- **サインアウト**: User Pool グローバルサインアウトを使用し、すべてのリフレッシュトークンを無効化

### Data Protection

- **Encryption at Rest**: DynamoDB はAWS管理キー (SSE) によるデフォルト暗号化を有効化
- **Encryption in Transit**: TLS 1.2 以上を必須。Amplify Hosting / API Gateway / AppSync は標準で対応
- **PII Handling**: ユーザー入力の音声・テキストは個人的な悩みを含む可能性があるため、最小限保存。音声データは Transcribe ストリーミング処理後に保存しない（MVP）
- **Data Classification**: ユーザー入力 = Confidential、ユーザー識別子（Cognito sub、IdPメール） = Confidential、手放しイベントログ（識別子除去後の集計） = Internal

### Network Security

- **VPC Requirements**: なし（フルマネージドサービスのみ）
- **WAF**: Amplify Hosting / API Gateway 前段に AWS WAF を有効化（OWASP Core Rule Set ベース）
- **Private Endpoints**: MVPでは不要

### Secrets Management

- **Secrets Storage**: AWS Secrets Manager（必要時）
- **Rotation Policy**: 自動ローテーション設定（採用シークレットがある場合）
- **Access Policy**: 最小権限の IAM ポリシー
- **Prohibited Practices**:
  - ソースコード／ビルド時環境変数／設定ファイルへのシークレット埋め込み禁止
  - サービス間での認証情報共有禁止
  - 長期 Access Key の発行禁止（IAM ロールを使用）

### Compliance Requirements

- **Standards**: 特定法令適用なし。日本の個人情報保護法に準拠した取扱を行う
- **Audit Logging**: CloudTrail を全リージョンで有効化。CloudWatch Logs の保持期間は本番 90 日、開発 7 日
- **Vulnerability Scanning**: GitHub Dependabot を有効化。weekly スキャン

### Dependency Security

- **Dependency Scanning**: Dependabot weekly + PR 時の `npm audit`
- **License Policy**: 許容ライセンス: MIT, Apache-2.0, BSD-2/3-Clause, ISC。禁止: GPL, AGPL
- **Update Policy**: Critical 脆弱性は7日以内、High は30日以内にパッチ適用

### Security Compliance Framework

- **Framework chosen**: **OWASP Top 10 (2021)** および **OWASP API Security Top 10 (2023)**
- **Rationale**: Web アプリ + GraphQL API + HTTP API という構成に最適。MVPの規模に見合う粒度

#### OWASP Top 10 (2021) 対応マトリクス

| カテゴリ | 対応 |
|---------|------|
| A01 Broken Access Control | AppSync の `allow.owner()` ルール、IAM 最小権限。Cognito User Pool の `sub` クレームをオーナーキーとして所有データを分離 |
| A02 Cryptographic Failures | TLS 1.2+、DynamoDB SSE、Secrets Manager。平文保存禁止 |
| A03 Injection | GraphQL は Amplify Data の生成スキーマ準拠。LLMプロンプトは入力サニタイズ + システムプロンプトでガード |
| A04 Insecure Design | 設計レビュー必須。脅威モデリングを MVP 設計フェーズで実施 |
| A05 Security Misconfiguration | Amplify Gen 2 の IaC 化により再現性確保。WAFデフォルトルール適用 |
| A06 Vulnerable and Outdated Components | Dependabot weekly スキャン |
| A07 Identification and Authentication Failures | Cognito User Pool の管理されたソーシャルログインフロー（Google OIDC）を採用し、自前パスワード管理を回避。Refresh Token のローテーション、サインアウト時のトークン失効を実装 |
| A08 Software and Data Integrity Failures | npm パッケージは package-lock.json でロック。CI/CD は Amplify Hosting の管理パイプライン経由 |
| A09 Security Logging and Monitoring Failures | CloudTrail / CloudWatch Logs を全レイヤで有効化。主要メトリクスのアラーム設定 |
| A10 Server-Side Request Forgery | 外部 URL アクセスは Bedrock / Transcribe / AWS API のみ。任意URL fetch を行うコードは禁止 |

#### OWASP API Security Top 10 (2023) 対応マトリクス

| カテゴリ | 対応 |
|---------|------|
| API1 Broken Object Level Authorization | AppSync の owner-based authz ルール |
| API2 Broken Authentication | Cognito User Pool ソーシャルログイン（Google OIDC）+ JWT 検証 |
| API3 Broken Object Property Level Authorization | GraphQL スキーマでフィールドレベル認可 |
| API4 Unrestricted Resource Consumption | API Gateway / AppSync スロットリング、Bedrock 呼出回数のクライアント単位レートリミット |
| API5 Broken Function Level Authorization | IAM ロールで関数別最小権限 |
| API6 Unrestricted Access to Sensitive Business Flows | 「手放す」ボタンには CSRF 相当の冪等性キー（`Idempotency-Key`）を要求 |
| API7 Server Side Request Forgery | Not applicable（外部URL fetch を行わない） |
| API8 Security Misconfiguration | IaC（Amplify Gen 2）で構成を集中管理 |
| API9 Improper Inventory Management | 全エンドポイントを Amplify Data / OpenAPI で文書化 |
| API10 Unsafe Consumption of APIs | Bedrock/Transcribe レスポンスは Zod で型検証してから利用 |

---

## Testing Requirements

### Test Strategy Overview

| Test Type | Required | Coverage Target | Tooling |
|-----------|----------|----------------|---------|
| Unit Tests | Yes | line 80% / branch 70% 以上 | Vitest |
| Integration Tests | Yes | 全 Lambda、AppSync リゾルバ、Bedrock/Transcribe 呼出 | Vitest + AWS SDK モック (`aws-sdk-client-mock`) |
| End-to-End Tests | Yes | クリティカルパス（音声入力→手放し完了）のみ | Playwright |
| Contract Tests | No | MVPでは不要 | - |
| Performance Tests | Conditional | リリース前にスモーク負荷試験のみ | k6 |
| Security Tests | Yes | 公開エンドポイントに対する基本テスト | OWASP ZAP（リリース前1回） |

### Unit Testing Standards

- **Coverage Minimum**: line 80% / branch 70%
- **Mocking Policy**: AWS SDK 呼出と Bedrock/Transcribe はモック。内部ビジネスロジックはモックしない
- **Naming Convention**: `describe('CardService') > it('should structure voice input into card')`
- **Test Location**: ソースと同階層 `*.test.ts`

### Integration Testing Standards

- **Scope**: Lambda ハンドラ単位での入出力検証、AppSync リゾルバ、Amplify sandbox 環境での実 AWS 呼出
- **Environment**: Amplify cloud sandbox（開発者ごと）
- **Data Management**: テストごとに固有 ownerId を生成し、テスト後にクリーンアップ

### End-to-End Testing Standards

- **Scope**:
  1. 音声入力 → やめ候補の作成 → 一覧表示
  2. テキスト入力 → やめ候補の作成 → 手放し → 肯定メッセージ
  3. 再訪問時のやめ候補リスト復元
- **Environment**: ステージング Amplify ブランチ
- **Data-testid Requirements**: すべてのインタラクティブ要素に安定した `data-testid` 属性を付与

### Performance Testing Standards

- **Baseline Requirements**: 音声入力 → やめ候補表示 p95 5 秒以内、「手放す」ボタン → 肯定メッセージ p95 3 秒以内
- **Test Scenarios**: 同時接続 50 ユーザー（MVP想定）
- **Tooling**: k6

### CI/CD Testing Gates

| Pipeline Stage | Required Tests | Failure Action |
|---------------|---------------|----------------|
| Pre-commit | ESLint, Prettier, tsc --noEmit | Block commit |
| Pull Request | Unit tests, Integration tests | Block merge |
| Pre-deploy (staging) | E2E tests | Block deploy |
| Post-deploy (production) | スモークテスト（手放し1回分のE2E） | 自動ロールバック判断はマニュアル |

---

## Example and Template Code Guidance

### Purpose of Example Code

本プロジェクトでは AI-DLC によるコード生成時、`examples/` ディレクトリのサンプルを正典パターンとして従う。新たに発明しないこと。

### When to Provide Example Code

- Lambda + Bedrock 呼び出しパターン（Claude へのやめ候補の構造化リクエスト）
- AppSync / Amplify Data モデル定義パターン
- フロントエンドからの音声入力 → Transcribe Streaming パターン
- 「手放す」ボタンの祝祭演出コンポーネント
- 単体テストのテンプレート
- 統合テストのテンプレート

### Directory Structure

```
project-root/
  amplify/                         # Amplify Gen 2 バックエンド定義
    auth/resource.ts               # Cognito User Pool（Google ソーシャルログイン）+ Identity Pool
    data/resource.ts               # AppSync / DynamoDB データモデル
    functions/
      structure-card/
        handler.ts                 # 音声テキスト → Bedrock → やめ候補の構造化
        resource.ts
      affirm-withdrawal/
        handler.ts                 # 手放し時の肯定メッセージ生成
        resource.ts
    backend.ts
  src/                             # フロントエンド（React + Vite）
    components/
    hooks/
    pages/
    lib/
  examples/
    bedrock-claude-call/
      handler.ts                   # Bedrock 呼び出しの正典実装
      handler.test.ts
      README.md
    transcribe-streaming-client/
      useTranscribe.ts             # フロントエンドフック
      README.md
    appsync-data-model/
      resource.ts
      README.md
    celebration-button/
      CelebrationButton.tsx
      README.md
```

### Bedrock 呼び出しサンプル（最小例）

`examples/bedrock-claude-call/handler.ts`:

```typescript
import { AnthropicBedrock } from "@anthropic-ai/bedrock-sdk";
import { z } from "zod";

const client = new AnthropicBedrock({ awsRegion: "ap-northeast-1" });

const CardSchema = z.object({
  title: z.string(),
  category: z.string(),
  estimatedTimeCostHoursPerMonth: z.number(),
  estimatedMoneyCostJpyPerMonth: z.number(),
  withdrawalUpsideSummary: z.string(),
});

export const handler = async (event: { rawText: string; ownerId: string }) => {
  const message = await client.messages.create({
    model: "anthropic.claude-haiku-4-5",
    max_tokens: 1024,
    system:
      "あなたは『やめたいこと』を構造化するアシスタントです。健康・医療・依存症に関わる入力は肯定せず、専門家相談を促してください。出力は指定JSONスキーマに厳密準拠してください。",
    messages: [
      {
        role: "user",
        content: `次の独り言からやめ候補を生成: ${event.rawText}`,
      },
    ],
  });

  const text = message.content
    .filter((block) => block.type === "text")
    .map((block) => ("text" in block ? block.text : ""))
    .join("");

  const json = JSON.parse(text);
  const card = CardSchema.parse(json);

  return { ownerId: event.ownerId, ...card };
};
```

> 上記は最小例。本番コードでは構造化出力に `tool_use` を採用、エラーハンドリング、Pino ロギング、Idempotency-Key、Bedrock Guardrails の利用などを追加する。

### How AI-DLC Uses Example Code

1. コード生成前に `examples/` を読む
2. パターンに従う（独自発明しない）
3. ビジネスロジックのみカスタマイズし、共通構造（エラーハンドリング、ロギング、認可）はそのまま流用
4. コード生成プランで参照した example を明示する

### Maintaining Example Code

- 標準が変わったら example も更新する
- 新メンバーは合流時に全 example を読む
- example も本番コードと同じレビュープロセスを通す

---

## Open Technical Questions

- [ ] Amazon Bedrock の `anthropic.claude-haiku-4-5` および `claude-sonnet-4-6` の東京リージョンでの利用可否を、AWSアカウント有効化時に確認する。未提供の場合はクロスリージョン推論プロファイル（`global.` プレフィックス付きモデルID）を使用する
- [ ] フロントエンドから直接 Transcribe Streaming に WebSocket 接続する場合、Cognito Identity Pool Authenticated ロールに付与する IAM 権限の最小化粒度を要詳細設計（`transcribe:StartStreamTranscriptionWebSocket` 等）
- [ ] Google から取得した `email` クレームを DynamoDB に保存するか、`sub` のみ保存しメールはトークンから都度取得するかを確定する
- [ ] Bedrock Guardrails を MVP で採用するか、まずは LLM プロンプトのシステムメッセージのみで対応するかを決定する
- [ ] DynamoDB のテーブル設計（シングルテーブル設計 vs マルチテーブル設計）を Amplify Data の生成挙動を踏まえて決定する
- [ ] Lambda アーキテクチャを arm64 で統一するかどうか（コスト最適化）
- [ ] Chrome 以外のブラウザアクセス時の検出ロジック（User-Agent ベース）の置き場所（CloudFront 関数 vs フロント JS）
