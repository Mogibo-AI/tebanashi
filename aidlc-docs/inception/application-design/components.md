# コンポーネント定義

## 設計方針

Tebanashi 初回開発スコープは、音声/テキスト入力から AI によるやめ候補作成・保存・表示までである。
フロントエンドは機能別フォルダを基本にし、再利用 UI、hooks、lib、types は共通レイヤーに置く。
バックエンドは信頼境界として Card 作成、AI 構造化、安全判定、永続化を担当する。

## コンポーネント一覧

| ID | コンポーネント | 層 | 目的 |
|---|---|---|---|
| C-01 | App Shell and Routing | Frontend | ルーティング、認証ゲート、レイアウト、ページ構成を提供する |
| C-02 | Auth Boundary | Frontend/AWS | Amplify UI Authenticator と Cognito/Google ログイン境界を提供する |
| C-03 | Browser Support Gate | Frontend | Chrome 対応判定と非対応ブラウザ案内を提供する |
| C-04 | Intake Feature | Frontend | 音声入力、テキストフォールバック、送信状態、ユーザー向けエラーを扱う |
| C-05 | Transcription Client | Frontend/AWS | Cognito Identity Pool の認証済み一時クレデンシャルで Transcribe Streaming に直接接続する |
| C-06 | Card Creation Client | Frontend | 入力テキストをバックエンドの Card 作成 API に送信し、UX 状態へ変換する |
| C-07 | Card Domain Contracts | Backend/API | Card、AI 応答、エラー、API 入出力の source-of-truth スキーマを所有する |
| C-08 | Card Creation API | Backend | 安全判定、AI 構造化、スキーマ検証、Card 永続化を信頼境界内でオーケストレーションする |
| C-09 | Safety Guardrail Module | Backend/AWS | Bedrock Guardrails とアプリ側分類により、安全でない手放し肯定を防ぐ |
| C-10 | Structuring Module | Backend/AWS | Bedrock Claude に入力テキストを渡し、Card 候補の構造化結果を得る |
| C-11 | Card Persistence | Backend/AWS | AppSync/Amplify Data/DynamoDB に owner-scoped Card を保存・取得する |
| C-12 | Observability Module | Frontend/Backend | イベントスキーマ、構造化ログ、メトリクス送信を中央集約する |
| C-13 | Error Mapping | Frontend/Backend | 型付きエラーを安全な UI メッセージへ変換する |

## C-01: App Shell and Routing

### 目的

React Router v6 を使い、アプリ全体のページ構成、認証済み/未認証の分岐、Chrome サポート判定、レスポンシブレイアウトを統括する。

### 責務

- ルート定義とページ遷移を管理する
- Auth Boundary の状態に応じて保護された画面を表示する
- Browser Support Gate をコア画面より前に適用する
- モバイルファーストのレイアウトシェルを提供する

### インターフェース

- 入力: `AuthState`, `BrowserSupportResult`
- 出力: 表示すべき page/shell state
- 依存: C-02, C-03, C-04, C-12, C-13

## C-02: Auth Boundary

### 目的

Amplify UI Authenticator を認証境界として使い、Google ログインと Cognito セッションをアプリへ提供する。

### 責務

- Google ログイン UI を提供する
- Cognito セッション状態を取得する
- owner identity として使用する Cognito `sub` を提供する
- 未認証時にコア入力画面を表示しない

### インターフェース

- 入力: Google OIDC/Cognito session
- 出力: `AuthState`, `OwnerIdentity`
- 依存: AWS Cognito User Pool, Amplify Auth

## C-03: Browser Support Gate

### 目的

MVP のサポート対象である Chrome をフロントエンドで判定し、非対応ブラウザでは Chrome 利用案内を表示する。

### 責務

- ブラウザ環境を判定する
- 非 Chrome の場合はコア入力体験を止める
- 案内 UI をアクセシブルに表示する

### インターフェース

- 入力: `navigator.userAgent` などのブラウザ情報
- 出力: `BrowserSupportResult`
- 依存: C-13

## C-04: Intake Feature

### 目的

音声入力とテキストフォールバックの UX 状態を管理し、入力テキストが準備できたら Card Creation Client に渡す。

### 責務

- マイク開始/停止、録音中、文字起こし中、送信中の状態を表示する
- テキスト入力を提供する
- 空文字や明らかに過剰な入力を UI レベルで抑止する
- 入力失敗時にテキストフォールバックへ誘導する

### インターフェース

- 入力: user voice, user text, auth state
- 出力: `PreparedInputText`, UI state
- 依存: C-05, C-06, C-12, C-13

## C-05: Transcription Client

### 目的

Cognito Identity Pool の認証済み一時クレデンシャルを使い、ブラウザから Amazon Transcribe Streaming に直接接続して日本語音声を文字起こしする。

### 責務

- 認証済み AWS 一時クレデンシャルを取得する
- `ja-JP` の Transcribe Streaming セッションを開始する
- 部分/最終文字起こし結果を Intake Feature に返す
- エラー時に安全なエラー型へ変換する

### インターフェース

- 入力: audio stream, `OwnerIdentity`
- 出力: `TranscriptionState`, `TranscriptionResult`
- 依存: Cognito Identity Pool, Amazon Transcribe Streaming, C-12, C-13

## C-06: Card Creation Client

### 目的

フロントエンドの UX 状態を維持しつつ、信頼境界内の Card 作成処理をバックエンド API に委譲する。

### 責務

- 入力テキストを Card Creation API に送信する
- 成功時に保存済み Card を UI 用 view model に変換する
- 安全応答時に Card を表示せず安全案内を表示する
- エラーを UI メッセージへ変換する

### インターフェース

- 入力: `PreparedInputText`
- 出力: `CreateCardViewResult`
- 依存: C-08, C-12, C-13

## C-07: Card Domain Contracts

### 目的

バックエンドが source-of-truth として Card、API 入出力、AI 応答、エラー型のスキーマを所有する。

### 責務

- Card schema と status enum を定義する
- AI 構造化応答 schema を定義する
- API request/response schema を定義する
- フロントエンド向けには生成 API 型と local view model の利用を前提にする

### インターフェース

- 入力: schema definitions
- 出力: backend validation schemas, generated API types
- 依存: Zod, Amplify generated types

## C-08: Card Creation API

### 目的

入力テキストから Card 作成までの信頼境界内処理を担う単一バックエンド API として設計する。
内部では Safety Guardrail Module と Structuring Module を分離する。

### 責務

- 認証済み owner identity を検証する
- 入力テキストを検証する
- Safety Guardrail Module を実行する
- 通常入力のみ Structuring Module へ進める
- AI 応答を Card schema で検証する
- Card Persistence に保存を委譲する
- Card または safe response を返す

### インターフェース

- 入力: `CreateCardRequest`, `OwnerIdentity`
- 出力: `CreateCardResponse`
- 依存: C-07, C-09, C-10, C-11, C-12, C-13

## C-09: Safety Guardrail Module

### 目的

健康・医療・服薬・依存症・自傷他害・法的義務・扶養/介護など、通常の全肯定が不適切な入力を検出する。

### 責務

- Bedrock Guardrails を呼び出す
- アプリ側の unsafe-domain 分類を実行する
- guardrail 発火時は Card を作らず安全応答へ誘導する
- ユーザーテキストや Card データを含まない最小限の監査/メトリクスイベントを発行する

### インターフェース

- 入力: sanitized input text, request context
- 出力: `SafetyDecision`
- 依存: Bedrock Guardrails, C-12, C-13

## C-10: Structuring Module

### 目的

Bedrock Claude を使い、日本語入力から Card 候補を構造化する。

### 責務

- システムプロンプトと入力テキストを組み立てる
- Bedrock Claude を呼び出す
- AI 応答を取り出して C-07 の schema で検証する
- max_tokens などのコスト/安全制約を守る

### インターフェース

- 入力: `StructuringRequest`
- 出力: `StructuredCardCandidate`
- 依存: Amazon Bedrock, Anthropic Bedrock SDK, C-07, C-12, C-13

## C-11: Card Persistence

### 目的

owner-scoped Card の保存と一覧取得を提供する。

### 責務

- Card を DynamoDB/AppSync/Amplify Data に保存する
- `allow.owner()` 相当の owner-based authorization を前提にする
- active cards の一覧を取得する
- 他ユーザーの Card を返さない

### インターフェース

- 入力: `ValidatedCard`, `OwnerIdentity`
- 出力: `SavedCard`, `CardList`
- 依存: AppSync, Amplify Data, DynamoDB, C-07, C-12, C-13

## C-12: Observability Module

### 目的

フロントエンドとバックエンド全体で使う中央集約の観測性モジュールを提供する。

### 責務

- イベント名とイベント payload schema を定義する
- 構造化ログ形式を定義する
- PII、email、token、secret、入力全文を記録しない
- p95 latency、成功率、guardrail 発火率の計測を支える

### インターフェース

- 入力: `ObservabilityEvent`, `LogContext`, `MetricSample`
- 出力: structured logs, metrics, audit-safe events
- 依存: CloudWatch, frontend telemetry adapter, backend logger

## C-13: Error Mapping

### 目的

バックエンド/システムエラーを型付きエラーとして扱い、安全な UI メッセージと復旧案内に変換する。

### 責務

- backend/system error を `AppError` へ分類する
- ユーザー向けメッセージから内部情報を除去する
- retry/fallback 可否を表現する
- fail closed の原則を支援する

### インターフェース

- 入力: unknown error, backend error response
- 出力: `AppError`, `SafeUiError`
- 依存: C-12

## Security Baseline 準拠

- 認証境界、owner identity、入力検証、安全応答、ログ抑制、最小権限 IAM、CORS、fail closed を設計上のコンポーネント責務に含めた。
- 具体的な IAM、CORS、暗号化、ログ保持、HTTP security headers は NFR Design と Infrastructure Design で詳細化する。

## PBT 準拠

Application Design では PBT は N/A。
ただし C-07、C-13、C-12 は後続の PBT Partial 対象になりうる pure function / serialization round-trip を含むため、Functional Design と Code Generation に引き継ぐ。
