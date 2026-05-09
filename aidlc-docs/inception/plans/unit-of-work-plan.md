# ユニット分割計画

## 1. 目的

この計画は、承認済みの要件、ユーザーストーリー、実行計画、アプリケーション設計をもとに、初期開発を論理的な Unit of Work に分解するためのものである。

Units Generation は 2 段階で進める。

1. 計画: 分割方針を確認し、必要な質問に回答してから承認する。
2. 生成: 承認済み計画に従って、ユニット定義、依存関係、ストーリーマップを作成する。

現時点では計画段階であり、このファイルの `[Answer]:` をすべて埋めるまでユニット成果物は生成しない。

## 2. 入力コンテキスト

| 種別 | 参照 |
|---|---|
| 要件 | `aidlc-docs/inception/requirements/requirements.md` |
| ユーザーストーリー | `aidlc-docs/inception/user-stories/stories.md` |
| ペルソナ | `aidlc-docs/inception/user-stories/personas.md` |
| 実行計画 | `aidlc-docs/inception/plans/execution-plan.md` |
| アプリケーション設計 | `aidlc-docs/inception/application-design/` |

初期開発スコープは「音声/テキスト入力から AI によるやめ候補作成・保存・表示まで」である。

## 3. 計画チェックリスト

- [x] Units Generation ルールを確認する
- [x] 承認済み要件を確認する
- [x] 承認済みユーザーストーリーとペルソナを確認する
- [x] 承認済み実行計画を確認する
- [x] 承認済みアプリケーション設計を確認する
- [x] ユニット分割計画を作成する
- [x] すべての `[Answer]:` を収集する
- [x] 回答の曖昧さ、矛盾、未定義語を分析する
- [x] 必要であればフォローアップ質問を追加する
- [ ] ユニット分割計画の明示承認を得る

## 4. 生成対象成果物チェックリスト

計画承認後、次の成果物を生成する。

- [ ] `aidlc-docs/inception/application-design/unit-of-work.md` にユニット定義、責務、境界、Greenfield のコード構成方針を記述する
- [ ] `aidlc-docs/inception/application-design/unit-of-work-dependency.md` にユニット依存関係と依存マトリクスを記述する
- [ ] `aidlc-docs/inception/application-design/unit-of-work-story-map.md` にユーザーストーリーからユニットへの対応を記述する
- [ ] ユニット境界と依存関係を検証する
- [ ] すべてのストーリーがいずれかのユニットに割り当てられていることを検証する
- [ ] Security Baseline と PBT Partial の責務が後続ステージに引き継がれていることを確認する

## 5. 現時点の分割仮説

これは最終決定ではなく、回答後に確定する。

| 候補ユニット | 主な範囲 | 関連ストーリーの初期仮説 |
|---|---|---|
| Foundation and Access | React/Vite/Amplify 基盤、Google ログイン、owner identity、Chrome gate、レスポンシブ shell | US-001, US-002, US-003, US-004 |
| Intake and Transcription | 音声入力、テキストフォールバック、文字起こし状態、入力検証、回復可能な intake error | US-005, US-006, US-007 |
| AI Card Structuring and Persistence | Card 作成 API、AI 構造化、Card schema、保存、active card 一覧 | US-008, US-009 |
| Safety and Guardrails | Bedrock Guardrails、unsafe-domain 分類、安全応答、`needs_attention` 系の扱い | US-010, US-011 |
| Observability and Quality Gates | イベント、構造化ログ、p95 latency、成功率、E2E hooks、PBT/NFR 引き継ぎ | US-012 と各ユニット横断 |

## 6. 回答が必要な質問

次の質問に対し、選択肢の記号または具体案を `[Answer]:` の後に入力する。

### Q1. ストーリーのグルーピング方針

ユーザーストーリーをどの粒度でユニット化しますか。

A) 実行計画の 5 ユニット案を基本にする
B) Safety and Guardrails を AI Card Structuring and Persistence に統合し、バックエンド作成フローを 1 ユニットに寄せる
C) Observability and Quality Gates を独立ユニットにせず、各ユニットの受け入れ条件として扱う
D) Access、Platform、Intake、AI、Persistence、Safety、Observability のように、より小さい story-based unit に分ける
X) その他。具体的に記入する

[Answer]: B

### Q2. 実装順序の優先順位

後続の construction で、どの順序を前提にユニット依存を整理しますか。

A) Foundation and Access -> AI Card Structuring and Persistence -> Intake and Transcription -> Safety and Guardrails -> Observability and Quality Gates
B) Foundation and Access -> Intake and Transcription -> AI Card Structuring and Persistence -> Safety and Guardrails -> Observability and Quality Gates
C) Foundation and Access -> Safety/AI contract -> Intake/Persistence -> Observability and Quality Gates
D) 最初に end-to-end の薄い vertical slice を作り、その後ユニットごとに厚くする
X) その他。具体的に記入する

[Answer]: D

### Q3. Greenfield のコード構成方針

`unit-of-work.md` に記載する初期コード構成はどれを優先しますか。

A) 単一アプリ構成。`src/features/*`、`src/shared/*`、`amplify/*` を基本にする
B) workspace 構成。`apps/web`、`packages/domain`、`packages/ui`、`amplify` を分ける
C) Amplify テンプレートに近い root `src` + `amplify` から始め、必要最小限の feature folder だけ追加する
D) frontend/backend/domain/contracts を明示的に分けるが、workspace package までは導入しない
X) その他。具体的に記入する

[Answer]: C

### Q4. バックエンドのユニット境界

Card 作成 API、safety、structuring、persistence の境界をどう扱いますか。

A) Card Creation API、safety、structuring、persistence を 1 つのバックエンド vertical unit として扱う
B) AI structuring/persistence と safety を別ユニットに分ける
C) Foundation が data schema/persistence を先に持ち、AI unit が backend orchestration を持つ
D) Card domain contracts を最初の独立ユニットとして切り出し、後続ユニットがそれに依存する
X) その他。具体的に記入する

[Answer]: A

### Q5. Transcribe と認可の境界

ブラウザ直接 Transcribe Streaming と Cognito Identity Pool/IAM の責務をどのユニットに置きますか。

A) Intake and Transcription が Identity Pool 利用、IAM 前提、browser client 実装までまとめて所有する
B) Foundation and Access が Identity Pool/IAM の前提を所有し、Intake and Transcription は browser client と UX を所有する
C) Infrastructure/NFR 側で IAM と Identity Pool を扱い、Units Generation では Intake の論理責務だけ定義する
D) 音声入力はリスクが高いため、text fallback vertical slice を先行ユニットにする
X) その他。具体的に記入する

[Answer]: B

### Q6. 観測性と品質ゲートの扱い

US-012、p95 latency、成功率、guardrail 発火率、E2E hooks、PBT Partial をどのようにユニット化しますか。

A) Observability and Quality Gates を独立した最終ユニットにする
B) 観測性と品質ゲートを全ユニットの受け入れ条件に埋め込み、独立ユニットは作らない
C) 最初に薄い shared observability foundation を置き、各ユニットで必要な計測を追加する
D) NFR Requirements/NFR Design で主に扱い、Units Generation では最小限の依存だけ表現する
X) その他。具体的に記入する

[Answer]: C

### Q7. チームや PR サイズの前提

construction の単位はどの進め方を想定しますか。

A) 小さな sequential unit を作り、1 ユニット 1 PR を基本にする
B) integration overhead を減らすため、やや大きい vertical slice 単位にする
C) 並行作業しやすいように interface/contract を先に固め、複数ユニットを並行可能にする
D) まず 1 人で短命ブランチ運用しやすい粒度を優先する
X) その他。具体的に記入する

[Answer]: D

### Q8. ビジネスドメイン境界

Tebanashi の初期スコープで、どの業務能力を最も強い境界として扱いますか。

A) Access/Ownership、Intake、Card Creation、Safety、Observability をそれぞれ強い境界にする
B) ユーザー体験の journey を優先し、Access -> Intake -> Card 作成/表示を主軸に Safety/Observability を横断関心にする
C) Card domain を中心に置き、入力手段や AI/safety は Card 作成の手段として扱う
D) Safety を独立した業務境界として強く扱い、通常フローより先に契約を固める
X) その他。具体的に記入する

[Answer]: B

### Q9. NFR と Security Baseline の所有

Security Baseline、ログ抑制、owner authorization、rate limit、accessibility、PBT Partial の責務をどう割り当てますか。

A) 各ユニットが自分の範囲の NFR/Security/PBT を所有する
B) Dedicated NFR/Quality unit が横断的に所有し、各ユニットは機能実装に集中する
C) 各ユニットに最低限の責務を持たせ、最後に品質ゲート unit で横断確認する
D) NFR Requirements/NFR Design で所有関係を確定し、Units Generation では暫定マッピングに留める
X) その他。具体的に記入する

[Answer]: C

### Q10. 初期開発の完了判定

初期開発の「やめ候補作成まで」をどのユニットの完了で満たしたとみなしますか。

A) AI Card Structuring and Persistence まで完了し、音声/テキストから Card 保存・表示まで動く状態
B) Safety and Guardrails まで完了し、安全応答を含めて Card 作成フローが成立する状態
C) Observability and Quality Gates まで完了し、p95 latency と成功率を測れる状態
D) 全ユニット完了後のみ初期開発完了とみなす
X) その他。具体的に記入する

[Answer]: B

## 7. 回答分析結果

**分析日時**: 2026-05-09T14:16:33Z

### 回答確認

- Q1-Q10 の `[Answer]:` はすべて入力済みである。
- 回答はすべて選択肢の記号で示されており、未定義の追加用語はない。
- 「Safety and Guardrails」は独立ユニットではなく、Card 作成バックエンドの責務に統合する方針として扱う。
- Q10 の完了判定は、統合された safety/guardrails を含む Card 作成フローが成立した状態を指す。
- Q6 と Q9 は、薄い shared observability foundation を先に置き、各ユニットで最低限の NFR/Security/PBT 責務を持ち、最後に品質ゲートで横断確認する方針として扱う。

### 採用する分割方針

- 実装順序は、最初に end-to-end の薄い vertical slice を作り、その後ユニットごとに厚くする。
- Greenfield のコード構成は、Amplify テンプレートに近い root `src` + `amplify` から始め、必要最小限の feature folder を追加する。
- Cognito Identity Pool/IAM の前提は Foundation and Access が所有し、Transcribe browser client と UX は Intake and Transcription が所有する。
- チーム/PR サイズは、1 人で短命ブランチ運用しやすい粒度を優先する。
- 業務境界はユーザー体験の journey を優先し、Access -> Intake -> Card 作成/表示を主軸に Safety/Observability を横断関心として扱う。

### フォローアップ判定

追加質問は不要である。

## 8. 回答後の分析手順

- [x] すべての `[Answer]:` が空でないことを確認する
- [x] 複数案の混在がある場合、どの条件で使い分けるかを確認する
- [x] 未定義の用語がある場合、フォローアップ質問を追加する
- [x] 依存順序とユニット境界が矛盾していないか確認する
- [x] 全ストーリーが割り当て可能か確認する
- [x] Greenfield コード構成方針が生成成果物に反映可能か確認する
- [x] 計画承認プロンプトを提示する

## 9. 計画承認後の生成手順

- [ ] 承認済み回答を読み込む
- [ ] 最終ユニット一覧を確定する
- [ ] `unit-of-work.md` を生成する
- [ ] `unit-of-work-dependency.md` を生成する
- [ ] `unit-of-work-story-map.md` を生成する
- [ ] 依存関係、ストーリー割り当て、NFR/Security/PBT 引き継ぎを検証する
- [ ] Units Generation 完了レビューを提示する

## 10. 現在の停止条件

この計画が明示承認されるまで、次の成果物は生成しない。

- `aidlc-docs/inception/application-design/unit-of-work.md`
- `aidlc-docs/inception/application-design/unit-of-work-dependency.md`
- `aidlc-docs/inception/application-design/unit-of-work-story-map.md`
