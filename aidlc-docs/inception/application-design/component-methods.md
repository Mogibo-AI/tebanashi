# コンポーネントメソッド

## 目的

この文書は高レベルのメソッドシグネチャと入出力型を定義する。
詳細な business rules、validation の境界値、テスト性は後続の Functional Design で定義する。

## 共通型

```typescript
type OwnerIdentity = {
  sub: string;
};

type AuthState =
  | { status: "authenticated"; owner: OwnerIdentity }
  | { status: "unauthenticated" }
  | { status: "loading" };

type BrowserSupportResult =
  | { supported: true; browser: "chrome" }
  | { supported: false; reason: "non_chrome" | "unknown" };

type PreparedInputText = {
  text: string;
  source: "voice" | "text";
};

type CardStatus = "active" | "withdrawn" | "needs_attention";

type Card = {
  id: string;
  ownerId: string;
  title: string;
  category: string;
  estimatedTimeCostHoursPerMonth: number;
  estimatedMoneyCostJpyPerMonth: number;
  withdrawalUpsideSummary: string;
  status: CardStatus;
  createdAt: string;
  updatedAt: string;
};

type SafetyDecision =
  | { kind: "allow" }
  | { kind: "block"; category: string; safeMessage: string };

type AppErrorCode =
  | "AUTH_REQUIRED"
  | "UNSUPPORTED_BROWSER"
  | "INPUT_INVALID"
  | "TRANSCRIPTION_FAILED"
  | "AI_STRUCTURING_FAILED"
  | "GUARDRAIL_TRIGGERED"
  | "PERSISTENCE_FAILED"
  | "UNKNOWN";

type AppError = {
  code: AppErrorCode;
  retryable: boolean;
  fallbackAvailable: boolean;
  correlationId: string;
};
```

## C-01: App Shell and Routing

```typescript
function renderAppShell(auth: AuthState, browser: BrowserSupportResult): AppShellState;
function resolveInitialRoute(auth: AuthState, browser: BrowserSupportResult): RouteName;
```

- `renderAppShell`: 認証状態とブラウザ判定に応じて表示する shell state を決める。
- `resolveInitialRoute`: 初期表示ルートを決定する。

## C-02: Auth Boundary

```typescript
function getAuthState(): Promise<AuthState>;
function requireOwnerIdentity(auth: AuthState): OwnerIdentity;
function signOut(): Promise<void>;
```

- `getAuthState`: Amplify/Cognito の現在セッションを取得する。
- `requireOwnerIdentity`: 認証済み状態から Cognito `sub` を取り出す。未認証なら typed error を返す。
- `signOut`: Cognito セッションを終了する。

## C-03: Browser Support Gate

```typescript
function detectBrowserSupport(userAgent: string): BrowserSupportResult;
function shouldShowUnsupportedBrowserNotice(result: BrowserSupportResult): boolean;
```

- `detectBrowserSupport`: Chrome サポート対象かを判定する。
- `shouldShowUnsupportedBrowserNotice`: 非対応ブラウザ案内の表示可否を決める。

## C-04: Intake Feature

```typescript
function validateTextInput(rawText: string): PreparedInputText | AppError;
function submitPreparedInput(input: PreparedInputText): Promise<CreateCardViewResult>;
function recoverFromVoiceFailure(error: AppError): IntakeRecoveryAction;
```

- `validateTextInput`: 空文字、過剰長、不正形式を UI レベルで抑止する。
- `submitPreparedInput`: Card Creation Client に入力を渡す。
- `recoverFromVoiceFailure`: 音声失敗時のテキストフォールバック案内を決める。

## C-05: Transcription Client

```typescript
function startTranscription(owner: OwnerIdentity): Promise<TranscriptionSession>;
function stopTranscription(session: TranscriptionSession): Promise<TranscriptionResult>;
function handleTranscriptionError(error: unknown): AppError;
```

- `startTranscription`: 認証済み一時クレデンシャルを前提に Transcribe Streaming を開始する。
- `stopTranscription`: セッションを終了し、最終文字起こしを返す。
- `handleTranscriptionError`: AWS/ブラウザエラーを typed error に変換する。

## C-06: Card Creation Client

```typescript
function createCardFromInput(input: PreparedInputText): Promise<CreateCardViewResult>;
function mapCreateCardResponse(response: CreateCardResponse): CreateCardViewResult;
```

- `createCardFromInput`: バックエンドの Card Creation API を呼び出す。
- `mapCreateCardResponse`: 保存済み Card、安全応答、エラーを UI 表示向けに変換する。

## C-07: Card Domain Contracts

```typescript
function parseCreateCardRequest(input: unknown): CreateCardRequest;
function parseStructuredCardCandidate(input: unknown): StructuredCardCandidate;
function parseSavedCard(input: unknown): Card;
```

- `parseCreateCardRequest`: API request を backend source-of-truth schema で検証する。
- `parseStructuredCardCandidate`: AI 応答を保存前候補として検証する。
- `parseSavedCard`: 永続化済み Card を検証する。

## C-08: Card Creation API

```typescript
async function handleCreateCard(request: CreateCardRequest, owner: OwnerIdentity): Promise<CreateCardResponse>;
async function createAndPersistCard(input: PreparedInputText, owner: OwnerIdentity): Promise<Card>;
```

- `handleCreateCard`: 安全判定、構造化、検証、永続化を信頼境界内で統括する。
- `createAndPersistCard`: 通常入力のみ Card を生成して保存する。

## C-09: Safety Guardrail Module

```typescript
async function evaluateSafety(input: PreparedInputText, context: RequestContext): Promise<SafetyDecision>;
function toSafeResponse(decision: Extract<SafetyDecision, { kind: "block" }>): SafeResponse;
```

- `evaluateSafety`: Bedrock Guardrails とアプリ側分類を実行する。
- `toSafeResponse`: Card を作らず安全な応答を生成する。

## C-10: Structuring Module

```typescript
async function structureCard(input: PreparedInputText, owner: OwnerIdentity): Promise<StructuredCardCandidate>;
function buildStructuringPrompt(input: PreparedInputText): BedrockPrompt;
```

- `structureCard`: Bedrock Claude を呼び、Card 候補を返す。
- `buildStructuringPrompt`: 構造化用 prompt を構築する。

## C-11: Card Persistence

```typescript
async function saveCard(candidate: StructuredCardCandidate, owner: OwnerIdentity): Promise<Card>;
async function listActiveCards(owner: OwnerIdentity): Promise<Card[]>;
```

- `saveCard`: owner-scoped Card として保存する。
- `listActiveCards`: 認証済み owner の active cards のみ返す。

## C-12: Observability Module

```typescript
function emitEvent(event: ObservabilityEvent): void;
function logInfo(message: string, context: LogContext): void;
function logError(error: AppError, context: LogContext): void;
function recordMetric(sample: MetricSample): void;
```

- `emitEvent`: 定義済みイベントを送信する。
- `logInfo`: PII を含まない構造化ログを出力する。
- `logError`: typed error を安全にログ化する。
- `recordMetric`: p95 latency や成功率の計測材料を送る。

## C-13: Error Mapping

```typescript
function toAppError(error: unknown, context: ErrorContext): AppError;
function toSafeUiError(error: AppError): SafeUiError;
```

- `toAppError`: unknown error を安全な typed error に分類する。
- `toSafeUiError`: ユーザー向けメッセージと復旧案内を生成する。

## 後続ステージへの引き継ぎ

- `detectBrowserSupport`, `validateTextInput`, `parse*`, `toAppError`, `toSafeUiError` は pure function として PBT Partial の候補になる。
- `parse*` と API response/view model の変換は serialization/validation round-trip の候補になる。
