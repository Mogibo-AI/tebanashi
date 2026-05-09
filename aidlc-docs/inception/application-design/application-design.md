# アプリケーション設計

## 1. 設計概要

Tebanashi 初回開発スコープは、音声/テキスト入力から AI によるやめ候補作成・保存・表示までである。
設計上の主要方針は次の通り。

- フロントエンドは React/Vite/Amplify を使い、機能別フォルダと共通レイヤーのハイブリッド構成にする。
- 認証 UI は Amplify UI Authenticator を境界とし、Google ログインを Cognito User Pool 経由で扱う。
- 音声文字起こしは Cognito Identity Pool の認証済み一時クレデンシャルを使い、ブラウザから Transcribe Streaming へ直接接続する。
- Card 作成はバックエンドの信頼境界内で実行し、frontend は UX 状態を担当する。
- AI 構造化と safety は単一 backend API の内部モジュールとして分離する。
- Guardrail 発火時は Card を保存せず、ユーザーテキストや Card データを含まない最小限の audit/metric event だけ残す。
- 観測性とエラーマッピングは中央集約する。
- 共有コントラクトはバックエンド source-of-truth とし、frontend は生成 API 型と local view model を使う。

## 2. 生成成果物

| Artifact | Purpose |
|---|---|
| `components.md` | コンポーネント名、目的、責務、インターフェースを定義する |
| `component-methods.md` | 高レベルのメソッドシグネチャと入出力型を定義する |
| `services.md` | サービス定義とオーケストレーションパターンを定義する |
| `component-dependency.md` | 依存関係、通信パターン、データフローを定義する |
| `application-design.md` | Application Design の統合サマリー |

## 3. コンポーネント構成

| ID | コンポーネント | 要約 |
|---|---|---|
| C-01 | App Shell and Routing | ルーティング、認証ゲート、Chrome gate、レイアウトを統括する |
| C-02 | Auth Boundary | Amplify UI Authenticator と Cognito/Google ログインを扱う |
| C-03 | Browser Support Gate | Chrome 対応判定と非対応ブラウザ案内を扱う |
| C-04 | Intake Feature | 音声入力、テキスト入力、送信 UX、入力エラーを扱う |
| C-05 | Transcription Client | ブラウザから Transcribe Streaming へ直接接続する |
| C-06 | Card Creation Client | frontend から backend Card creation API を呼び出す |
| C-07 | Card Domain Contracts | backend source-of-truth schema を所有する |
| C-08 | Card Creation API | safety、structuring、validation、persistence を統括する |
| C-09 | Safety Guardrail Module | Bedrock Guardrails と unsafe-domain 分類を扱う |
| C-10 | Structuring Module | Bedrock Claude による Card 候補生成を扱う |
| C-11 | Card Persistence | owner-scoped Card の保存・一覧取得を扱う |
| C-12 | Observability Module | イベント、ログ、メトリクスを中央集約する |
| C-13 | Error Mapping | typed error と safe UI error の変換を扱う |

## 4. 主要サービス

| ID | サービス | 要約 |
|---|---|---|
| S-01 | Access Service | 認証状態、owner identity、Chrome gate を統合する |
| S-02 | Intake Service | 音声/テキスト入力から prepared input を作る |
| S-03 | Card Creation Service | backend 信頼境界内で安全判定、AI 構造化、保存を実行する |
| S-04 | Card Query Service | owner-scoped active cards を取得する |
| S-05 | Safety Service | guardrail 判定と安全応答を提供する |
| S-06 | Observability Service | イベント、ログ、メトリクスを中央集約する |
| S-07 | Error Service | typed error と safe UI error の変換を提供する |

## 5. 主要フロー

### 5.1 通常 Card 作成フロー

1. App Shell が Chrome gate と auth gate を適用する。
2. ユーザーが音声またはテキストで入力する。
3. 音声の場合、Transcription Client が Transcribe Streaming から文字起こし結果を得る。
4. Intake Feature が prepared input を Card Creation Client に渡す。
5. Card Creation API が入力と owner identity を検証する。
6. Safety Guardrail Module が safety allow/block を判定する。
7. allow の場合、Structuring Module が Bedrock Claude で Card 候補を生成する。
8. Card Domain Contracts が AI 応答を検証する。
9. Card Persistence が owner-scoped Card として保存する。
10. frontend が保存済み Card を一覧に表示する。

### 5.2 Guardrail 発火フロー

1. 入力テキストが Card Creation API に届く。
2. Safety Guardrail Module が Bedrock Guardrails とアプリ側分類を実行する。
3. block の場合、Structuring Module と Card Persistence には進まない。
4. ユーザーテキストや Card データを含まない最小限の audit/metric event を送る。
5. frontend には安全な応答を表示する。

## 6. 後続ステージへの引き継ぎ

### Functional Design

- Card 作成の詳細 business rules
- unsafe-domain 分類の入力/出力ルール
- 入力長、カテゴリ、数値範囲、status 遷移
- typed error mapping の詳細

### NFR Requirements / NFR Design

- p95 8 秒要件と 5 秒ストレッチ目標
- Security Baseline 各ルールの具体化
- PBT Partial 対象の特定
- アクセシビリティ、ログ抑制、メトリクス、rate limit

### Infrastructure Design

- Amplify Gen 2 構成
- Cognito User Pool / Identity Pool
- AppSync / DynamoDB
- Lambda / Bedrock / Guardrails
- Transcribe Streaming IAM
- CloudWatch logs/metrics/alarms

### Code Generation

- TDD で探索、Red、Green、Refactoring を実行する
- backend source-of-truth schema と generated frontend types の連携を実装する
- PBT Partial は pure function と serialization/validation round-trip に適用する

## 7. Extension Compliance

### Security Baseline

Status: Application Design では準拠。

認証境界、owner identity、backend trust boundary、guardrail-before-structuring、safe error mapping、central observability、PII ログ抑制を設計に含めた。
具体的な IAM、暗号化、CORS、HTTP security headers、ログ保持、アラームは後続の NFR/Infrastructure/Code stages で検証する。

### Property-Based Testing

Status: Application Design では N/A。

PBT Partial enforcement は後続ステージへ引き継ぐ。
候補領域は schema parsing、view model mapping、typed error mapping、observability event schema である。
