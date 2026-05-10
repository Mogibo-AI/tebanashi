# ユニット定義

## 1. 分割方針

承認済みのユニット分割計画に従い、Tebanashi 初期開発はユーザー体験の journey を主軸に分解する。
独立した microservice 分割ではなく、Amplify Gen 2 + React の単一アプリケーション内に論理ユニットを置く。

採用方針は次の通り。

- 最初に薄い end-to-end vertical slice を通し、その後ユニットごとに厚くする。
- Safety and Guardrails は独立ユニットにせず、Card 作成バックエンドの信頼境界内に統合する。
- Observability は最初に薄い shared foundation を作り、各ユニットで必要な計測を追加し、最後に品質ゲートで横断確認する。
- Greenfield のコード構成は、Amplify テンプレートに近い root `src` + `amplify` を基本にし、必要最小限の feature folder を追加する。
- 1 人で短命ブランチ運用しやすい単位を優先する。

## 2. ユニット一覧

| Unit ID | Unit of Work | 主な目的 | Primary Stories |
|---|---|---|---|
| UOW-01 | Foundation, Access, and Shared Observability Foundation | アプリ基盤、認証、owner identity、Chrome gate、共通観測性の薄い基盤 | US-001, US-003, US-004 |
| UOW-02 | Intake and Transcription | 音声/テキスト入力、Transcribe browser client、入力検証、入力復旧 | US-005, US-006 |
| UOW-03 | Card Creation, Safety, Persistence, and Display | Card 作成 API、AI 構造化、Safety、保存、一覧表示を 1 つの作成フローとして完成させる | US-002, US-008, US-009, US-010, US-011 |
| UOW-04 | Observability and Quality Gate | p95 latency、成功率、ログ抑制、E2E、PBT、Security/NFR 横断確認 | US-007, US-012 |

## 3. 実装順序

| Order | Unit | 方針 |
|---|---|---|
| 1 | UOW-01 | root `src` + `amplify` の最小構成、auth/browser gate、共通 error/observability の薄い型を先に置く |
| 2 | UOW-02 | text fallback を先に通し、voice/Transcribe を同じ `PreparedInputText` contract に接続する |
| 3 | UOW-03 | prepared input から safety-first Card creation、保存、表示までを統合し、初期開発の「やめ候補作成まで」を満たす |
| 4 | UOW-04 | 全体計測、品質ゲート、E2E、PBT、Security/NFR の横断確認で release readiness を作る |

薄い vertical slice は UOW-01 から UOW-03 の最初の pass で形成する。
具体的には、認証済み Chrome user が text fallback から prepared input を作り、Card 作成 API を呼び、保存済み Card を一覧に表示する最小経路を先に通す。
その後、音声入力、Bedrock 構造化、guardrail 分岐、観測性、エラー復旧を厚くする。

## 4. Greenfield コード構成方針

アプリケーションコードは workspace root に置き、`aidlc-docs/` には置かない。
初期構成は monorepo workspace package ではなく、Amplify テンプレートに近い単一アプリ構成とする。

```text
.
├── amplify/
│   ├── auth/
│   ├── data/
│   └── functions/
│       └── create-card/
├── src/
│   ├── app/
│   ├── features/
│   │   ├── access/
│   │   ├── intake/
│   │   └── cards/
│   ├── shared/
│   │   ├── errors/
│   │   ├── observability/
│   │   ├── types/
│   │   └── ui/
│   ├── lib/
│   │   └── amplify/
│   └── test/
└── tests/
    ├── e2e/
    └── property/
```

### 配置ルール

- `src/app/`: routing、app shell、provider composition。
- `src/features/access/`: auth boundary、browser support gate、owner identity 取得。
- `src/features/intake/`: voice/text input、Transcribe client adapter、prepared input state。
- `src/features/cards/`: Card creation client、Card result/list UI、view model mapping。
- `src/shared/errors/`: typed error、safe UI error mapping。
- `src/shared/observability/`: event schema、frontend telemetry adapter、PII 抑制ルール。
- `src/shared/types/`: generated API type を直接置き換えない範囲の local view model と UI contract。
- `amplify/auth/`: Cognito User Pool、Google provider、Identity Pool 前提。
- `amplify/data/`: Card model、owner authorization、AppSync/Amplify Data schema。
- `amplify/functions/create-card/`: Card Creation API、Safety、Structuring、Persistence orchestration。

## 5. UOW-01: Foundation, Access, and Shared Observability Foundation

### 目的

React/Vite/Amplify の最小基盤を作り、認証済み Chrome user だけが core experience に到達できる状態を作る。
以降のユニットが使う owner identity、typed error、薄い observability event schema もここで準備する。

### 主な責務

- React/Vite/Tailwind/Amplify の初期構成を作る。
- Amplify UI Authenticator で Google ログイン境界を作る。
- Cognito `sub` を owner identity として扱う contract を定義する。
- Chrome support gate を app shell より前に適用する。
- モバイルファーストの accessible responsive shell を作る。
- PII、email、token、secret、入力全文を受け取らない observability event の薄い型を定義する。
- typed error と safe UI error の基本 contract を定義する。

### 関連コンポーネント/サービス

- C-01 App Shell and Routing
- C-02 Auth Boundary
- C-03 Browser Support Gate
- C-12 Observability Module の薄い foundation
- C-13 Error Mapping の基本 contract
- S-01 Access Service
- S-06 Observability Service の薄い foundation
- S-07 Error Service の基本 contract

### 境界

- 実際の Transcribe Streaming 実装は UOW-02 が所有する。
- Card 作成 API、Bedrock、DynamoDB 保存は UOW-03 が所有する。
- p95 latency 集計、成功率集計、E2E/PBT の横断完了判定は UOW-04 が所有する。

### 完了条件

- 未認証 user は core input experience に到達できない。
- Google ログイン後に owner identity として Cognito `sub` を扱える。
- 非 Chrome user には core experience の代わりに案内が表示される。
- responsive shell が mobile/desktop Chrome の主要表示に耐える。
- 後続ユニットが使う error/observability の基本型が存在する。

## 6. UOW-02: Intake and Transcription

### 目的

ユーザーが音声またはテキストで「やめたいこと」を入力し、backend に渡せる `PreparedInputText` を作る。
音声が失敗しても text fallback で継続できる入力体験を作る。

### 主な責務

- text fallback の入力、空文字/過剰長の UI レベル検証を実装する。
- voice recording の開始/停止 state を管理する。
- Cognito Identity Pool の authenticated role 前提で Transcribe Streaming browser client を実装する。
- `ja-JP` の最終文字起こしを `PreparedInputText` に変換する。
- microphone、network、Transcribe 失敗時に text fallback へ戻す。
- `voice_input_started`、`transcription_completed`、`text_input_submitted` を発行する。

### 関連コンポーネント/サービス

- C-04 Intake Feature
- C-05 Transcription Client
- C-12 Observability Module
- C-13 Error Mapping
- S-02 Intake Service
- S-06 Observability Service
- S-07 Error Service

### 境界

- Cognito Identity Pool/IAM の前提定義は UOW-01 が所有する。
- Card 作成 API への最終処理、AI 構造化、安全判定、保存は UOW-03 が所有する。
- latency/success-rate の release 判定は UOW-04 が所有する。

### 完了条件

- text fallback で valid な prepared input を作れる。
- voice input が成功した場合、日本語文字起こし結果から prepared input を作れる。
- voice input が失敗しても text fallback が利用できる。
- 入力由来にかかわらず、後続の Card 作成フローへ同一 contract で渡せる。

## 7. UOW-03: Card Creation, Safety, Persistence, and Display

### 目的

`PreparedInputText` から安全判定、AI 構造化、schema validation、owner-scoped 保存、active card 一覧表示までを backend 信頼境界中心に完成させる。
Safety and Guardrails はこのユニットに統合し、通常の Card 作成フローから分離した別モードにはしない。

### 主な責務

- Card、AI response、API request/response、error の backend source-of-truth schema を定義する。
- Card Creation API を単一 backend entry point として実装する。
- API 入力と owner identity を検証する。
- Structuring より前に Safety Guardrail Module を必ず実行する。
- Bedrock Guardrails とアプリ側 unsafe-domain 分類を実行する。
- block の場合は Card を保存せず、安全応答と最小限の audit/metric event を返す。
- allow の場合は Bedrock Claude で Card 候補を構造化する。
- AI 応答を schema validation し、不正な結果は保存しない。
- owner-scoped Card として保存し、active cards を取得する。
- frontend の Card Creation Client、Card result UI、Card list UI を実装する。
- `card_structure_requested`、`card_structure_succeeded`、`card_structure_failed`、`card_saved`、`guardrail_triggered` を発行する。

### 関連コンポーネント/サービス

- C-06 Card Creation Client
- C-07 Card Domain Contracts
- C-08 Card Creation API
- C-09 Safety Guardrail Module
- C-10 Structuring Module
- C-11 Card Persistence
- C-12 Observability Module
- C-13 Error Mapping
- S-03 Card Creation Service
- S-04 Card Query Service
- S-05 Safety Service
- S-06 Observability Service
- S-07 Error Service

### 境界

- 入力 state と Transcribe browser client は UOW-02 が所有する。
- 共通 auth/owner/browser gate は UOW-01 が所有する。
- 横断的な p95/success-rate 集計、E2E、PBT 完了判定は UOW-04 が所有する。

### 完了条件

- valid non-guardrail input から、title、category、estimated cost、withdrawal upside summary、status、ownerId、createdAt を持つ Card が作られる。
- malformed AI output は保存されない。
- guardrail 対象入力では通常 Card を保存せず、安全応答を返す。
- voice/text どちらの入力由来でも同じ safety-first flow を通る。
- 保存済み active card が同一 owner の一覧に表示される。
- 他 owner の card は参照できない。
- このユニット完了時点で、初期開発の「やめ候補作成まで」は機能成立とみなす。

## 8. UOW-04: Observability and Quality Gate

### 目的

全ユニットで追加された観測性、NFR、Security Baseline、PBT Partial、E2E を横断的に検証し、release readiness を作る。

### 主な責務

- 必須イベントが PII、email、token、secret、入力全文を含まないことを検証する。
- p95 structuring latency と voice-to-structure success rate を算出できる event/metric を揃える。
- `detectBrowserSupport`、`validateTextInput`、schema parsing、view model mapping、typed error mapping、observability event schema を PBT 候補として具体化する。
- Vitest coverage、Playwright critical path、security-related checks の実行方針をまとめる。
- 主要エラーが safe UI error に変換されることを確認する。
- accessibility の主要操作、keyboard focus、contrast、reduced motion を確認する。

### 関連コンポーネント/サービス

- C-12 Observability Module
- C-13 Error Mapping
- All frontend/backend components
- S-06 Observability Service
- S-07 Error Service

### 境界

- 新しい product feature は追加しない。
- 不足している instrumentation、test hook、quality gate の補完を行う。
- Infrastructure/NFR Design で確定すべき IAM、CORS、暗号化、security headers、ログ保持は、このユニットの検証対象として引き継ぐ。

### 完了条件

- US-012 の必須イベントが全て発行可能である。
- p95 8 秒基準と 5 秒 stretch target を評価できる。
- voice-to-structure success rate を評価できる。
- Security Baseline と PBT Partial の後続 stage 引き継ぎが明確である。
- E2E critical path が、auth、Chrome gate、text/voice input、Card creation、safe response、active card list をカバーする。

## 9. 横断ルール

### Security Baseline

- 各ユニットは自分の範囲の Security Baseline 責務を所有する。
- UOW-03 は owner authorization、schema validation、guardrail-before-structuring、safe failure を強く所有する。
- UOW-04 は横断確認と release gate を所有する。

### Property-Based Testing

- PBT Partial は後続の NFR/Code/Build stages で具体化する。
- UOW-01 は browser detection と typed error contract を候補にする。
- UOW-02 は text input validation と prepared input mapping を候補にする。
- UOW-03 は schema parsing、AI response validation、view model mapping を候補にする。
- UOW-04 は observability event schema と serialization/validation round-trip を横断確認する。

### Commit/PR 粒度

- 各ユニットはさらに小さい logical commits に分けてよい。
- 1 commit は 1 つの設計変更、契約追加、実装 slice、テスト追加などの単位に限定する。
- architecture、contract、behavior を変える場合は AI-DLC 文書も同じ logical change に含める。
