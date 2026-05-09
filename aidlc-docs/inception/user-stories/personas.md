# Personas

## Overview

Tebanashi の初回開発スコープは、音声/テキスト入力から AI によるやめ候補作成・保存・表示までである。
Personas は Vision Document の主要 3 類型に、ガードレール対象入力を行う注意喚起ユーザーを加えた 4 類型とする。

## Persona P-01: 継続疲れ層

| Item | Description |
|---|---|
| Archetype | 複数の習慣・サブスク・学習コミットメントを惰性で抱える個人ユーザー |
| Typical Context | 20〜40代。ジム、オンライン講座、習い事、SNS、サブスクなどを複数抱える |
| Primary Need | 罪悪感なく、やめたいことを言語化して整理したい |
| Motivation | 手間をかけずに「これはやめてもよい」と判断する材料がほしい |
| Friction | まとめる前に疲れる。ログインや入力が重いと離脱する |
| Success Signal | Chrome で開いて数分以内に、やめ候補が構造化されて保存される |

### Relevant Stories

- US-001: Google sign-in precondition
- US-003: Chrome support gate
- US-005: Voice intake
- US-006: Text fallback
- US-008: AI card structuring
- US-009: Save and list cards

## Persona P-02: 自己肯定感強化希望層

| Item | Description |
|---|---|
| Archetype | 自分の決断を肯定してくれる外部装置を求めるユーザー |
| Typical Context | 「続けられない自分」を責めがちで、やめる判断に心理的後押しが必要 |
| Primary Need | 自分の言葉を受け止めてもらい、やめ候補として見える形にしたい |
| Motivation | 手放しを失敗ではなくリソース最適化として捉えたい |
| Friction | AI の応答が冷たい、または入力内容を雑に扱うと信頼しない |
| Success Signal | 入力内容が自然なタイトル・カテゴリ・解放リソースとして返ってくる |

### Relevant Stories

- US-005: Voice intake
- US-006: Text fallback
- US-007: Recoverable input and processing errors
- US-008: AI card structuring
- US-012: Core observability and latency measurement

## Persona P-03: サンクコスト捕囚層

| Item | Description |
|---|---|
| Archetype | 過去に払った金額や時間を惜しみ、惰性で継続しているユーザー |
| Typical Context | 月額費、過去の教材費、すでに費やした時間を理由にやめられない |
| Primary Need | 月あたりの時間・金銭コストを整理し、続ける負荷を可視化したい |
| Motivation | サンクコストを切り離し、今後のリソースを取り戻したい |
| Friction | 推定コストが表示されない、または不自然だと納得しにくい |
| Success Signal | 推定時間コスト・推定金銭コスト・解放リソース要約がカードに表示される |

### Relevant Stories

- US-006: Text fallback
- US-008: AI card structuring
- US-009: Save and list cards
- US-012: Core observability and latency measurement

## Persona P-04: 注意喚起が必要なユーザー

| Item | Description |
|---|---|
| Archetype | 健康・医療・服薬・依存症・自傷他害・法的義務・扶養/介護など、通常の全肯定が不適切な内容を入力するユーザー |
| Typical Context | 「薬をやめたい」「治療をやめたい」「家族の介護を放棄したい」など、安全配慮が必要な入力を行う |
| Primary Need | 安易に肯定されず、専門家や公的窓口への相談を促される |
| Motivation | 目の前の負担を減らしたいが、自己判断での手放しが危険な可能性がある |
| Friction | アプリが危険な意思決定を祝福したり、通常カードとして保存したりすると害がある |
| Success Signal | 通常の全肯定ではなく、安全な案内が表示され、必要なら `needs_attention` として区別される |

### Relevant Stories

- US-007: Recoverable input and processing errors
- US-010: Guardrail-safe response
- US-011: Guardrail behavior inside normal structuring flow
- US-012: Core observability and latency measurement

## Persona-to-Story Coverage

| Persona | Primary Stories |
|---|---|
| P-01 継続疲れ層 | US-001, US-003, US-005, US-006, US-008, US-009 |
| P-02 自己肯定感強化希望層 | US-005, US-006, US-007, US-008, US-012 |
| P-03 サンクコスト捕囚層 | US-006, US-008, US-009, US-012 |
| P-04 注意喚起が必要なユーザー | US-007, US-010, US-011, US-012 |

## Out of Scope Personas for Initial Development

- コミュニティ閲覧・投稿ユーザー
- 月次振り返り利用ユーザー
- 管理者・運用担当者
- 外部連携サービス利用ユーザー

## Compliance Notes

### Security Baseline

Personas include data ownership, authentication, safe error handling, and guardrail-sensitive behavior needed for user-visible security and safety stories.

### Property-Based Testing

PBT is not applicable to persona documentation.
PBT obligations remain deferred to design, code generation, and test stages.
