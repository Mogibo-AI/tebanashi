# ユニット・ストーリーマップ

## 1. 目的

この文書は、承認済みユーザーストーリー US-001 から US-012 を Units Generation で確定した Unit of Work に対応付ける。
各ストーリーは少なくとも 1 つの primary unit を持つ。
横断 story は supporting unit も明記する。

## 2. Unit Legend

| Unit ID | Unit of Work |
|---|---|
| UOW-01 | Foundation, Access, and Shared Observability Foundation |
| UOW-02 | Intake and Transcription |
| UOW-03 | Card Creation, Safety, Persistence, and Display |
| UOW-04 | Observability and Quality Gate |

## 3. Story to Unit Map

| Story | Priority | Primary Unit | Supporting Units | Assignment Rationale |
|---|---|---|---|---|
| US-001 Google sign-in precondition | Must | UOW-01 | UOW-04 | Auth Boundary と app shell の責務。UOW-04 で E2E/quality gate を確認する |
| US-002 User-owned card data | Must | UOW-03 | UOW-01, UOW-04 | UOW-01 が owner identity を提供し、UOW-03 が owner-scoped read/write を実装する |
| US-003 Chrome support gate | Must | UOW-01 | UOW-04 | Browser Support Gate と app shell の責務。UOW-04 で E2E/accessibility を確認する |
| US-004 Accessible responsive input shell | Must | UOW-01 | UOW-02, UOW-04 | Shell は UOW-01。入力操作と最終 accessibility gate は UOW-02/UOW-04 で補完する |
| US-005 Voice intake | Must | UOW-02 | UOW-01, UOW-04 | Transcribe browser client と voice state は UOW-02。Identity Pool 前提は UOW-01 |
| US-006 Text fallback intake | Must | UOW-02 | UOW-03, UOW-04 | text fallback は thin vertical slice の入口として UOW-02 が所有する |
| US-007 Recoverable input and processing errors | Must | UOW-04 | UOW-02, UOW-03 | 各ユニットが自範囲の typed error を出し、UOW-04 が safe/recoverable UX を横断確認する |
| US-008 Structured card generation | Must | UOW-03 | UOW-04 | AI structuring、schema validation、latency event は UOW-03 が所有する |
| US-009 Save and list active cards | Must | UOW-03 | UOW-01, UOW-04 | owner-scoped persistence、query、Card list UI を UOW-03 が統合する |
| US-010 Guardrail-safe response | Must | UOW-03 | UOW-04 | Safety は Card creation flow 内に統合し、UOW-04 が safe response scenario を確認する |
| US-011 Guardrails inside normal structuring flow | Must | UOW-03 | UOW-02, UOW-04 | voice/text 由来を問わず UOW-03 の safety-first flow を通す |
| US-012 Core observability and latency measurement | Must | UOW-04 | UOW-01, UOW-02, UOW-03 | UOW-01 が薄い基盤を置き、UOW-02/UOW-03 がイベントを発行し、UOW-04 が完了判定する |

## 4. Unit to Story Map

### UOW-01: Foundation, Access, and Shared Observability Foundation

| Role | Stories |
|---|---|
| Primary | US-001, US-003, US-004 |
| Supporting | US-002, US-005, US-009, US-012 |

主な受け入れ観点:

- Google sign-in が core input experience の前提になる。
- Cognito `sub` を owner identity として扱える。
- Chrome 以外では core input experience を止める。
- mobile/desktop Chrome で基本 shell が破綻しない。
- 後続ユニットが使う error/observability contract がある。

### UOW-02: Intake and Transcription

| Role | Stories |
|---|---|
| Primary | US-005, US-006 |
| Supporting | US-004, US-007, US-011, US-012 |

主な受け入れ観点:

- 日本語音声から transcription result を得られる。
- text fallback が必ず利用できる。
- 空文字や過剰長入力を抑止できる。
- voice failure が text fallback に復帰できる。
- voice/text のどちらも同じ `PreparedInputText` contract になる。

### UOW-03: Card Creation, Safety, Persistence, and Display

| Role | Stories |
|---|---|
| Primary | US-002, US-008, US-009, US-010, US-011 |
| Supporting | US-006, US-007, US-012 |

主な受け入れ観点:

- Card Creation API が owner identity と input を検証する。
- Safety が Structuring より前に実行される。
- guardrail 発火時に normal Card を保存しない。
- AI response は schema validation 後にのみ保存される。
- Card は owner-scoped で保存・取得される。
- active card list に保存済み Card が表示される。
- voice/text の入力由来にかかわらず同じ safety-first flow を通る。

### UOW-04: Observability and Quality Gate

| Role | Stories |
|---|---|
| Primary | US-007, US-012 |
| Supporting | US-001, US-003, US-004, US-005, US-006, US-008, US-009, US-010, US-011 |

主な受け入れ観点:

- 必須イベントがすべて発行可能である。
- p95 structuring latency と voice-to-structure success rate を算出できる。
- logs/events が PII、email、token、secret、入力全文を含まない。
- critical path E2E が auth、Chrome gate、text/voice input、Card creation、safe response、active card list を確認する。
- PBT Partial の対象 pure function と serialization/validation round-trip が明確である。
- recoverable error UX が入力、AI、保存、認証、network の主要失敗をカバーする。

## 5. Traceability by Epic

| Epic | Stories | Primary Units |
|---|---|---|
| E-01 Access and Ownership | US-001, US-002 | UOW-01, UOW-03 |
| E-02 Platform Support | US-003, US-004 | UOW-01 |
| E-03 Intake Journey | US-005, US-006, US-007 | UOW-02, UOW-04 |
| E-04 AI Structuring | US-008, US-009 | UOW-03 |
| E-05 Safety | US-010, US-011 | UOW-03 |
| E-06 Observability | US-012 | UOW-04 |

## 6. Story Coverage Validation

| Check | Status |
|---|---|
| US-001 assigned | Complete |
| US-002 assigned | Complete |
| US-003 assigned | Complete |
| US-004 assigned | Complete |
| US-005 assigned | Complete |
| US-006 assigned | Complete |
| US-007 assigned | Complete |
| US-008 assigned | Complete |
| US-009 assigned | Complete |
| US-010 assigned | Complete |
| US-011 assigned | Complete |
| US-012 assigned | Complete |

すべての approved user stories は primary unit を持つ。
横断要素は supporting unit として明示した。

## 7. Initial Development Completion Milestone

ユーザー回答 Q10 に従い、初期開発の「やめ候補作成まで」は UOW-01、UOW-02、UOW-03 が統合され、safe response を含む Card 作成フローが成立した時点で機能成立とみなす。
UOW-04 は release readiness と品質ゲートを完成させるための必須後続ユニットである。
