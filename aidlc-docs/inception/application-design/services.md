# サービス設計

## 目的

この文書は、コンポーネント間のオーケストレーション単位を定義する。
実装時の class/function 名は後続の Functional Design と Code Generation で確定する。

## サービス一覧

| ID | サービス | 主な責務 |
|---|---|---|
| S-01 | Access Service | 認証状態、owner identity、Chrome gate を統合する |
| S-02 | Intake Service | 音声/テキスト入力から prepared input を作る |
| S-03 | Card Creation Service | 安全判定、AI 構造化、保存を信頼境界内で実行する |
| S-04 | Card Query Service | owner-scoped active card list を提供する |
| S-05 | Safety Service | guardrail 判定と安全応答を提供する |
| S-06 | Observability Service | イベント、ログ、メトリクスを中央集約する |
| S-07 | Error Service | typed error と safe UI error の変換を提供する |

## S-01: Access Service

### 責務

- Amplify Authenticator のセッション状態を読み取る
- Cognito `sub` を owner identity として提供する
- Chrome support gate を適用する
- 未認証または非対応ブラウザ時にコア機能を止める

### オーケストレーション

1. Browser Support Gate が Chrome 対応を判定する
2. Auth Boundary が認証状態を判定する
3. App Shell が core input experience の表示可否を決める
4. Observability Service が support/auth 関連イベントを記録する

## S-02: Intake Service

### 責務

- 音声入力セッションを開始/停止する
- Transcribe Streaming の最終文字起こしを取得する
- テキスト入力を検証する
- prepared input を Card Creation Client に渡す
- 入力失敗時にテキストフォールバックを案内する

### オーケストレーション

1. ユーザーが voice または text の入力経路を選ぶ
2. voice の場合は Transcription Client が `ja-JP` で文字起こしする
3. text の場合は UI レベルで入力検証する
4. prepared input を Card Creation Client へ渡す
5. Error Service が復旧案内を生成する

## S-03: Card Creation Service

### 責務

- バックエンドの信頼境界内で Card 作成を完結させる
- owner identity を検証する
- Safety Service を先に呼ぶ
- safety allow の場合のみ Structuring Module を呼ぶ
- AI 応答を backend source-of-truth schema で検証する
- Card Persistence に保存を委譲する

### オーケストレーション

1. `CreateCardRequest` を schema validation する
2. owner identity を確認する
3. Safety Service が `SafetyDecision` を返す
4. `block` の場合は Card を保存せず safe response を返す
5. `allow` の場合は Structuring Module が Card 候補を生成する
6. Card Domain Contracts が候補を検証する
7. Card Persistence が owner-scoped Card として保存する
8. Observability Service が成功/失敗/guardrail イベントを記録する

## S-04: Card Query Service

### 責務

- 認証済み owner の active cards を取得する
- AppSync/Amplify Data の owner authorization に従う
- フロントエンド view model に必要な項目を返す

### オーケストレーション

1. Auth Boundary から owner identity を取得する
2. AppSync query を実行する
3. 結果を generated API types から local view model へ変換する
4. Error Service が取得失敗時の UI error を作る

## S-05: Safety Service

### 責務

- Bedrock Guardrails を呼び出す
- 健康・医療・服薬・依存症・自傷他害・法的義務・扶養/介護に関する入力を検出する
- 通常の肯定/構造化を止める
- Card を作らず safe response を返す
- ユーザーテキストや Card データを含まない最小限の audit/metric event を送る

### オーケストレーション

1. 入力テキストを guardrail 用に処理する
2. Bedrock Guardrails とアプリ側分類を実行する
3. 判定結果を `SafetyDecision` として返す
4. `block` の場合は `guardrail_triggered` を記録する

## S-06: Observability Service

### 責務

- 中央集約のイベントスキーマを管理する
- フロントエンド/バックエンド双方のイベント送信を提供する
- PII、email、token、secret、入力全文をログから除外する
- p95 latency と成功率を測定できるイベントを揃える

### 対象イベント

- `voice_input_started`
- `transcription_completed`
- `text_input_submitted`
- `card_structure_requested`
- `card_structure_succeeded`
- `card_structure_failed`
- `card_saved`
- `guardrail_triggered`

## S-07: Error Service

### 責務

- unknown error を `AppError` に分類する
- backend/system error を安全な UI メッセージに変換する
- retry/fallback 可否を表現する
- 本番で内部情報を露出しない

## Security Baseline 準拠

- Card Creation Service を信頼境界とし、ユーザーが直接 ownerId や保存内容を自由に操作できない設計にする。
- Safety Service は通常フローより前に実行され、fail closed を前提にする。
- Observability Service はログ抑制を中央集約する。
- Error Service は内部情報露出を抑止する。

## PBT 準拠

Application Design では PBT は N/A。
ただし Error Service、Card Domain Contracts、Observability event schema は後続の Functional Design と Code Generation で PBT 候補として扱う。
