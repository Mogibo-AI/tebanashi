# Requirements Document

## 1. Intent Analysis

### User Request

AI-DLC ワークフローで Tebanashi の開発を開始する。
ワークフローの進め方は `.aidlc/aidlc-rules/` 配下、入力情報は `.aidlc/inputs/` 配下の Markdown ファイルを正とする。

### Request Classification

| Item | Assessment |
|---|---|
| Request Type | New Project |
| Project Type | Greenfield |
| Initial Scope | 音声/テキスト入力からやめ候補作成まで |
| Overall Product Scope | Tebanashi MVP |
| Complexity | Complex |
| Requirements Depth | Comprehensive |

### Primary Inputs

- `.aidlc/inputs/vision-document.md`
- `.aidlc/inputs/technical-environment-document.md`
- `aidlc-docs/inception/requirements/requirement-verification-questions.md`
- `aidlc-docs/inception/requirements/requirement-clarification-questions.md`

## 2. Product Context

Tebanashi は、合わない習慣・サブスク・継続コミットメントを「賢明な損切り」として手放せるよう支援する、日本語ユーザー向け Web アプリケーションである。

MVP 全体の体験は「独り言レベルの音声入力、AI 構造化、ワンタップ手放し、AI 全肯定」である。
ただし今回の初回開発対象は、音声/テキスト入力から AI によるやめ候補作成までに限定する。

## 3. Scope

### 3.1 In Scope for Initial Development

- Google ログイン済みユーザーが、Chrome でやめたいことを音声入力できる
- 音声が使えない場合、テキスト入力で同じ構造化処理を実行できる
- 日本語の入力テキストを AI がやめ候補として構造化する
- 構造化結果には、少なくともタイトル、カテゴリ、推定時間コスト、推定金銭コスト、手放した場合の解放リソース要約を含める
- 作成されたやめ候補をユーザーに紐づけて保存できる
- 作成されたやめ候補を一覧で表示できる
- 健康・医療・服薬・依存症・自傷他害・法的義務・扶養/介護など、全肯定が不適切な入力を検出し、通常の肯定/構造化とは異なる安全な応答にする
- Chrome 以外のブラウザでは、フロントエンドで検出して Chrome 利用を促す

### 3.2 Deferred from Initial Development

- ワンタップ「手放す」実行
- 手放しタイムスタンプ記録
- AI 全肯定メッセージ生成
- ご褒美提案の表示
- 祝祭演出と効果音
- 手放し履歴の短期保存・削除ポリシー実装
- リソース解放ダッシュボード
- 月次振り返り
- コミュニティ機能
- Apple/LINE/X/GitHub など Google 以外の IdP
- Chrome 以外のブラウザ正式対応

### 3.3 MVP-Level Decisions That Affect Later Units

- ご褒美提案は、ユーザー文脈に合う一般的な商品例までは許可するが、広告的表現や購入誘導を避ける
- やめ候補は保存し、手放し後の詳細履歴は短期間で削除する方針とする
- ユーザー入力テキストや生成結果は、MVP ではモデル改善に利用しない
- アカウント削除時は Cognito ユーザーと DynamoDB 上のやめ候補・手放し履歴を削除する
- 効果音は、手放しボタン押下後のユーザー操作をトリガーにデフォルト ON とし、ミュート切替を提供する

## 4. Functional Requirements

### FR-001: Google Login Requirement

ユーザーは Google アカウントでログインできなければならない。
認証は Amazon Cognito User Pool 経由の Google ソーシャルログインを使用する。

### FR-002: Authenticated User Data Ownership

やめ候補は Cognito User Pool の `sub` を主識別子としてユーザーに紐づけなければならない。
Google から取得した email は、アプリの永続データとして DynamoDB に保存しない。

### FR-003: Chrome Support Gate

アプリはフロントエンドでブラウザを検出し、Chrome 以外のブラウザではサポート対象外であることと Chrome 利用を促す案内を表示しなければならない。

### FR-004: Voice Input

ユーザーは Chrome のマイク権限を許可し、日本語音声でやめたいことを入力できなければならない。
音声入力は Amazon Transcribe Streaming の `ja-JP` を使用してテキスト化する。

### FR-005: Text Input Fallback

ユーザーは音声入力が使えない場合でも、日本語テキストを直接入力してやめ候補作成を実行できなければならない。
iOS Chrome で Web Audio またはマイク権限が期待通り動作しない場合も、MVP ではテキスト入力フォールバックを必須代替手段とする。

### FR-006: Input Submission

ユーザーは音声認識結果またはテキスト入力を送信し、やめ候補の構造化を要求できなければならない。
送信時には空文字、過剰に長い文字列、不正な形式の入力を拒否しなければならない。

### FR-007: AI Card Structuring

システムは入力テキストを Amazon Bedrock 経由の Claude モデルに渡し、やめ候補を構造化しなければならない。
構造化結果は Zod などのランタイムスキーマで検証し、スキーマ不一致時は保存せず安全に失敗しなければならない。

### FR-008: Card Fields

やめ候補には以下の項目を含めなければならない。

| Field | Requirement |
|---|---|
| title | やめ候補の短いタイトル |
| category | サブスク、習慣、学習、SNS、その他などの分類 |
| estimatedTimeCostHoursPerMonth | 月あたり推定時間コスト |
| estimatedMoneyCostJpyPerMonth | 月あたり推定金銭コスト |
| withdrawalUpsideSummary | 手放した場合に解放されるリソースの要約 |
| status | 初期値は `active` |
| ownerId | Cognito `sub` |
| createdAt | 作成日時 |

### FR-009: Guardrail Classification

システムは健康・医療・服薬・依存症・自傷他害・法的義務・扶養/介護など、通常の手放し肯定が不適切な入力を検出しなければならない。
該当する場合は、通常のやめ候補として全肯定せず、専門家や公的窓口への相談を促す安全な応答にしなければならない。

### FR-010: Bedrock Guardrails

MVP ではプロンプト制御とアプリ側バリデーションに加え、Bedrock Guardrails を必須とする。
Guardrails の判定結果がブロックまたは注意喚起を要求する場合、システムは通常カード保存を行わない、または安全状態のカードとして明確に区別しなければならない。

### FR-011: Card Persistence

構造化されたやめ候補は DynamoDB に保存し、再訪問時に同一ユーザーが参照できなければならない。
データモデルは Amplify Data の自然なモデル定義に従い、Card と WithdrawalEvent を別モデルとして定義する。
初回開発では Card を主対象とし、WithdrawalEvent は後続ユニットで使用する。

### FR-012: Card List Display

ユーザーは自分が作成した `active` なやめ候補一覧を表示できなければならない。
一覧にはタイトル、カテゴリ、推定時間コスト、推定金銭コスト、解放リソース要約を表示する。

### FR-013: Error Handling

音声認識、AI 構造化、保存、認証、ネットワークの各失敗時、ユーザーには内部情報を含まない分かりやすいエラーを表示しなければならない。
失敗時には可能な限りテキスト入力への復帰や再試行を案内する。

### FR-014: Instrumentation

システムは少なくとも以下のイベントを計測できなければならない。

| Event | Purpose |
|---|---|
| voice_input_started | 音声入力開始 |
| transcription_completed | 文字起こし完了 |
| text_input_submitted | テキスト入力送信 |
| card_structure_requested | 構造化要求 |
| card_structure_succeeded | 構造化成功 |
| card_structure_failed | 構造化失敗 |
| card_saved | 保存成功 |
| guardrail_triggered | 安全制御発火 |

## 5. Non-Functional Requirements

### NFR-001: Runtime and Language

実装言語は TypeScript を標準とする。
Lambda ランタイムは Node.js 24.x を標準とする。
Python は、TypeScript より明確に優位な音声処理または Bedrock SDK 要件がある Lambda 関数に限り、承認後に使用できる。

### NFR-002: Frontend Stack

フロントエンドは React 18.x、Vite、Tailwind CSS を使用する。
状態管理は React Context と TanStack Query を基本とする。
ルーティングは React Router v6 を使用する。

### NFR-003: AWS Serverless Architecture

バックエンドとインフラは AWS サーバーレス構成を標準とする。
AWS Amplify Gen 2、Cognito、AppSync、DynamoDB、Lambda、API Gateway、Bedrock、Transcribe Streaming、CloudWatch を許可サービスとして使用する。
EC2、ECS/Fargate、RDS、ElastiCache、Kinesis、Step Functions は MVP では使用しない。

### NFR-004: Region

主要リージョンは `ap-northeast-1` とする。
Bedrock モデル可用性に応じて、Bedrock のみクロスリージョン推論プロファイルを利用できる。

### NFR-005: Latency

音声またはテキスト入力から構造化されたやめ候補が表示されるまでの必須リリース基準は p95 8 秒以内とする。
p95 5 秒以内はストレッチ目標とする。

### NFR-006: Success Rate

音声入力からテキスト構造化までの成功率は 90% 以上を目標とする。
失敗時はテキスト入力フォールバックでコア体験を継続できなければならない。

### NFR-007: Responsive UI

UI はモバイルファーストで設計し、デスクトップへ拡張する。
デスクトップ Chrome とモバイル Chrome で主要操作が破綻なく利用できなければならない。

### NFR-008: Accessibility

MVP では主要操作のキーボード操作、フォームラベル、色コントラスト、モーション低減対応を必須とする。
祝祭演出など動きの強い UI は `prefers-reduced-motion` を尊重する。

### NFR-009: Privacy

音声データは Transcribe Streaming 処理後に保存しない。
ユーザー入力テキストおよび生成結果は、MVP ではモデル改善に利用しない。
サービス提供に必要な最小限のデータのみ保存する。

### NFR-010: Account Deletion

アカウント削除時には Cognito ユーザーと DynamoDB 上の関連データを削除する。
削除対象には Card と WithdrawalEvent を含める。

### NFR-011: Testing

Vitest による unit test を必須とし、line coverage 80% 以上、branch coverage 70% 以上を目標とする。
AWS SDK、Bedrock、Transcribe は単体テストでモックする。
クリティカルパスは Playwright による E2E テスト対象とする。

### NFR-012: Property-Based Testing

PBT extension は Partial enforcement とする。
pure function と serialization round-trip に限定して、PBT-02、PBT-03、PBT-07、PBT-08、PBT-09 をブロッキング制約として適用する。
TypeScript では fast-check を PBT フレームワーク候補とする。

## 6. Security Requirements

Security Baseline extension は Full blocking enforcement として有効である。
以下は初回開発スコープに適用される必須要求である。

### SEC-001: Encryption

DynamoDB、AppSync、API Gateway、Amplify Hosting、Lambda と外部通信は、保存時暗号化と TLS 1.2 以上の通信暗号化を使用しなければならない。

### SEC-002: Access Logging

外部通信を扱う API Gateway、AppSync、Amplify Hosting/CDN 相当のレイヤーはアクセスログまたは相当する CloudWatch ログを有効化しなければならない。

### SEC-003: Structured Logging

Lambda と主要フロントエンド計装は構造化ログを使用する。
ログには timestamp、request/correlation ID、level、message を含め、PII、トークン、シークレット、ユーザー入力全文を記録してはならない。

### SEC-004: HTTP Security Headers

HTML を返すエンドポイントでは、少なくとも Content-Security-Policy、Strict-Transport-Security、X-Content-Type-Options、X-Frame-Options、Referrer-Policy を設定しなければならない。

### SEC-005: Input Validation

すべての API 入力、LLM 入出力、GraphQL mutation 入力は Zod などのスキーマで検証しなければならない。
文字列長、形式、列挙値、数値範囲、HTML/script 混入を検証する。

### SEC-006: Least Privilege

Lambda ロール、Cognito Identity Pool authenticated role、AppSync、Bedrock、Transcribe、DynamoDB の IAM 権限は最小権限で定義する。
ワイルドカード action/resource は、AWS API がリソースレベル制御を提供しない場合のみ例外として文書化する。

### SEC-007: Application Authorization

Card と WithdrawalEvent は owner-based authorization を必須とし、他ユーザーのデータを参照または変更できてはならない。
AppSync は `allow.owner()` に準拠し、HTTP API がある場合は JWT 検証と ownerId 検証をサーバー側で実施する。

### SEC-008: CORS

認証済みエンドポイントの CORS は許可オリジンを明示し、ワイルドカードを使用してはならない。

### SEC-009: Secure Error Handling

本番のユーザー向けエラーにはスタックトレース、内部パス、AWS リソース名、モデル詳細、認可判定の内部情報を含めてはならない。

### SEC-010: Supply Chain

依存関係は lock file で固定し、Dependabot または同等の脆弱性スキャンを CI に組み込む。
GPL/AGPL ライセンスの依存関係は禁止する。

### SEC-011: Rate Limiting and Abuse Prevention

公開 API と Bedrock/Transcribe 呼び出しは、ユーザー単位またはセッション単位のレート制限を設ける。
LLM 呼び出しの max_tokens は 1024 以下を標準とする。

### SEC-012: Guardrail Auditability

Guardrails が発火した場合は、PII や入力全文を避けた形で guardrail category、request ID、timestamp を記録する。

### SEC-013: Authentication and Session Management

MVP は自前のパスワード認証を実装せず、Cognito User Pool と Google OIDC に認証を委譲する。
セッションは Amplify/Cognito のトークン管理を使用し、ログアウト時は有効なセッションを失効させる。
管理者アカウントが存在する場合は MFA を必須とする。

### SEC-014: Software and Data Integrity

外部 CDN スクリプトは原則使用しない。
使用が必要な場合は Subresource Integrity を設定する。
依存関係と CI/CD 設定はレビュー対象とし、重要なデータ変更は actor、timestamp、対象 ID を追跡できる形で監査可能にする。

### SEC-015: Monitoring and Alerting

認証失敗、認可失敗、Guardrails 発火、Bedrock/Transcribe エラー率、構造化失敗率、p95 レイテンシに対して CloudWatch メトリクスまたはアラームを定義する。
本番ログ保持期間は 90 日以上とし、アプリケーション実行ロールが自分の監査ログを削除できないようにする。

### SEC-016: Fail-Safe Defaults

外部サービス呼び出し、DB 操作、認可判定、LLM 構造化、Guardrails 判定は明示的にエラーハンドリングする。
例外時は fail closed とし、保存・表示・権限付与を継続しない。
ユーザー向けには内部情報を含まない安全なエラーを返す。

## 7. Data Requirements

### 7.1 Card

| Field | Type | Notes |
|---|---|---|
| id | string | システム生成 ID |
| ownerId | string | Cognito `sub` |
| title | string | 最大長を定義する |
| category | string | 許可カテゴリに制限する |
| estimatedTimeCostHoursPerMonth | number | 0 以上 |
| estimatedMoneyCostJpyPerMonth | number | 0 以上 |
| withdrawalUpsideSummary | string | 最大長を定義する |
| status | enum | `active`, `withdrawn`, `needs_attention` |
| createdAt | datetime | ISO 8601 |
| updatedAt | datetime | ISO 8601 |

### 7.2 WithdrawalEvent

WithdrawalEvent は MVP 全体では必要だが、初回開発では後続ユニット対象とする。
設計上は Card と別モデルとして定義できるようにする。

## 8. User Scenarios

### Scenario 1: 音声入力からやめ候補作成

1. ユーザーが Chrome でログイン済みのアプリを開く
2. ユーザーがマイクボタンを押す
3. ユーザーが日本語でやめたいことを話す
4. システムが音声を文字起こしする
5. システムが文字起こし結果を AI で構造化する
6. システムがやめ候補を保存して表示する

### Scenario 2: テキスト入力からやめ候補作成

1. ユーザーがテキスト入力欄にやめたいことを入力する
2. ユーザーが送信する
3. システムが入力を AI で構造化する
4. システムがやめ候補を保存して表示する

### Scenario 3: ガードレール対象入力

1. ユーザーが健康・医療・依存症などに関わる内容を入力する
2. システムが Guardrails またはアプリ側分類で検出する
3. システムは通常の全肯定トーンを避ける
4. システムは専門家や公的窓口への相談を促す安全な応答を表示する
5. 通常カードとして保存しない、または `needs_attention` として明確に区別する

### Scenario 4: 非 Chrome ブラウザアクセス

1. ユーザーが Chrome 以外のブラウザでアクセスする
2. システムがフロントエンドでブラウザを検出する
3. システムが Chrome 利用を促す案内を表示する

## 9. Acceptance Criteria

### AC-001: Voice Intake

Chrome でマイク権限が許可されている場合、日本語音声入力から文字起こし結果を取得できる。

### AC-002: Text Fallback

音声入力が失敗または利用不可の場合でも、テキスト入力からやめ候補作成を完了できる。

### AC-003: Structured Card

有効な入力に対し、システムは必須 Card fields を満たす構造化結果を生成し、スキーマ検証後に保存する。

### AC-004: Latency

音声またはテキスト入力送信から構造化カード表示までの p95 が 8 秒以内である。
5 秒以内はストレッチ目標として計測する。

### AC-005: Owner Isolation

ユーザーは自分の Card のみを一覧表示でき、他ユーザーの Card を参照できない。

### AC-006: Guardrails

健康・医療・服薬・依存症・自傷他害・法的義務・扶養/介護などの入力では、通常の手放し肯定を行わず、安全な案内を表示する。

### AC-007: Chrome Gate

Chrome 以外のブラウザでは、コア入力画面ではなく Chrome 利用案内が表示される。

### AC-008: Accessibility

主要操作はキーボードで操作でき、フォームにはラベルがあり、主要テキストは必要な色コントラストを満たす。
モーション低減設定が有効な場合、強いアニメーションを抑制する。

### AC-009: Logging Safety

ログに email、トークン、入力全文、シークレットが出力されない。

## 10. Assumptions

- ターゲットユーザーは日本語 UI と Google ログインを受け入れる
- Amazon Transcribe Streaming は Chrome の Web Audio 経由で MVP のレイテンシ要件に収まる
- Bedrock Claude モデルは日本語の独り言から実用的な構造化結果を生成できる
- Bedrock Guardrails は UX を破綻させないレイテンシで利用できる
- iOS Chrome の音声制約はテキスト入力フォールバックで許容する

## 11. Risks

| Risk | Impact | Mitigation |
|---|---|---|
| Bedrock モデルまたは Guardrails のレイテンシが大きい | p95 8 秒を超える | モデル選定、プロンプト短縮、ストリーミング以外の処理削減 |
| iOS Chrome の音声入力が不安定 | 音声体験が使えない | テキスト入力フォールバックを必須にする |
| ガードレール分類の誤判定 | 有害な肯定または過剰ブロック | Bedrock Guardrails、プロンプト、アプリ側分類、テストケースで多層化 |
| Google OAuth 設定が未完了 | ログインできない | 早期に Google Cloud Console と Cognito 設定を行う |
| IAM 権限が広くなりすぎる | セキュリティリスク | Security Baseline の SECURITY-06 をブロッキング制約にする |

## 12. Traceability

| Source | Requirement Coverage |
|---|---|
| Vision: Voice Intake | FR-004, FR-005, FR-007, AC-001, AC-002 |
| Vision: やめ候補構造化 | FR-007, FR-008, FR-011, AC-003 |
| Vision: Chrome MVP | FR-003, AC-007 |
| Vision: Privacy Risks | NFR-009, SEC-003, SEC-012 |
| Technical Environment: AWS Serverless | NFR-003, NFR-004 |
| Technical Environment: Security | Security Requirements |
| User Answer Q1 | Initial scope definition |
| User Answer Q2/Q11 | FR-009, FR-010, AC-006 |
| User Answer Q9 | FR-005 |
| Clarification Answer | NFR-005, AC-004 |

## 13. Extension Compliance

### Security Baseline

Status: Compliant for Requirements Analysis.

The enabled Security Baseline rules are represented as requirements in section 6.
Rules that require concrete infrastructure or code verification will be re-evaluated during Application Design, Infrastructure Design, Code Generation, and Build and Test.

### Property-Based Testing

Status: N/A for Requirements Analysis, configured for later stages.

PBT Partial mode is recorded as a non-functional testing requirement.
Enforced PBT rules are PBT-02, PBT-03, PBT-07, PBT-08, and PBT-09.
PBT-09 will be evaluated during NFR Requirements, and PBT test obligations will be evaluated during Code Generation and Build and Test.
