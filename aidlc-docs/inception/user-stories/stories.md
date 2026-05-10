# User Stories

## Story Strategy

- **Breakdown Approach**: Hybrid Journey + Safety
- **Granularity**: Epic and sub-story two-level structure
- **Acceptance Criteria Format**: Given/When/Then plus checklist
- **Priority Labels**: Must, Should, Could
- **Scope**: 音声/テキスト入力から AI によるやめ候補作成・保存・表示まで

## Story Index

| ID | Epic | Story | Priority | Personas |
|---|---|---|---|---|
| US-001 | E-01 Access and Ownership | Google sign-in precondition | Must | P-01 |
| US-002 | E-01 Access and Ownership | User-owned card data | Must | P-01, P-03 |
| US-003 | E-02 Platform Support | Chrome support gate | Must | P-01 |
| US-004 | E-02 Platform Support | Accessible responsive input shell | Must | P-01, P-02 |
| US-005 | E-03 Intake Journey | Voice intake | Must | P-01, P-02 |
| US-006 | E-03 Intake Journey | Text fallback intake | Must | P-01, P-02, P-03 |
| US-007 | E-03 Intake Journey | Recoverable input and processing errors | Must | P-02, P-04 |
| US-008 | E-04 AI Structuring | Structured card generation | Must | P-01, P-02, P-03 |
| US-009 | E-04 AI Structuring | Save and list active cards | Must | P-01, P-03 |
| US-010 | E-05 Safety | Guardrail-safe response | Must | P-04 |
| US-011 | E-05 Safety | Guardrails inside normal structuring flow | Must | P-02, P-04 |
| US-012 | E-06 Observability | Core observability and latency measurement | Must | P-01, P-02, P-03, P-04 |

## Epic E-01: Access and Ownership

### US-001: Google Sign-In Precondition

**Priority**: Must

**User Story**:
As a first-time user, I want to sign in with my Google account before using Tebanashi, so that my generated cards can be restored across visits and devices.

**Personas**: P-01

**Requirement Trace**: FR-001, FR-002, SEC-013

**Acceptance Criteria**

Given a user opens Tebanashi without an authenticated session,  
When they attempt to access the core input experience,  
Then the app requires Google sign-in before allowing card creation.

Given a user completes Google sign-in,  
When the app receives a valid Cognito session,  
Then the user reaches the core input experience without additional profile entry.

**Checklist**

- [ ] Google sign-in is the only MVP authentication path.
- [ ] The user is not asked to create a password.
- [ ] The core input experience is unavailable without authentication.
- [ ] Sign-in failure shows a generic recoverable error.

### US-002: User-Owned Card Data

**Priority**: Must

**User Story**:
As an authenticated user, I want my generated cards to belong only to me, so that private concerns and spending details are not exposed to other users.

**Personas**: P-01, P-03

**Requirement Trace**: FR-002, FR-011, AC-005, SEC-007

**Acceptance Criteria**

Given an authenticated user creates a card,  
When the card is saved,  
Then it is associated with the user's Cognito `sub`.

Given two authenticated users exist,  
When either user views their card list,  
Then they only see cards owned by their own `sub`.

**Checklist**

- [ ] Google email is not persisted as application data.
- [ ] Owner-based authorization is required for card reads.
- [ ] Owner-based authorization is required for card writes.
- [ ] Authorization failures do not reveal whether another user's card exists.

## Epic E-02: Platform Support

### US-003: Chrome Support Gate

**Priority**: Must

**User Story**:
As a user on an unsupported browser, I want clear guidance that Tebanashi MVP supports Chrome, so that I understand why the core input experience is unavailable.

**Personas**: P-01

**Requirement Trace**: FR-003, AC-007

**Acceptance Criteria**

Given a user opens Tebanashi in a non-Chrome browser,  
When the app initializes,  
Then the app shows a Chrome support notice instead of the core input experience.

Given a user opens Tebanashi in Chrome,  
When the app initializes,  
Then the app allows the normal authenticated flow to continue.

**Checklist**

- [ ] Browser support detection runs in the frontend.
- [ ] The notice does not imply non-Chrome browsers are broken permanently.
- [ ] The notice is accessible by keyboard and screen reader.
- [ ] The notice avoids exposing implementation details.

### US-004: Accessible Responsive Input Shell

**Priority**: Must

**User Story**:
As a mobile or desktop Chrome user, I want the input screen to be readable and operable, so that I can create a card without layout or accessibility barriers.

**Personas**: P-01, P-02

**Requirement Trace**: NFR-007, NFR-008, AC-008

**Acceptance Criteria**

Given a user opens the app on mobile Chrome,  
When they view the input screen,  
Then primary controls fit the viewport and remain usable without horizontal scrolling.

Given a user navigates by keyboard,  
When they move through the core controls,  
Then focus order is predictable and each interactive control has an accessible label.

**Checklist**

- [ ] Layout is mobile-first and expands to desktop.
- [ ] Main controls have visible focus states.
- [ ] Primary text meets contrast expectations.
- [ ] Motion-heavy UI respects reduced motion preferences where applicable.

## Epic E-03: Intake Journey

### US-005: Voice Intake

**Priority**: Must

**User Story**:
As a user who wants low-friction input, I want to speak my "やめたいこと" in Japanese, so that I can create a card without first organizing my thoughts.

**Personas**: P-01, P-02

**Requirement Trace**: FR-004, AC-001, NFR-006

**Acceptance Criteria**

Given an authenticated Chrome user grants microphone permission,  
When they speak Japanese into the app,  
Then the app captures speech and obtains a transcription result.

Given voice transcription succeeds,  
When the transcription is ready,  
Then the user can submit it for AI structuring.

**Checklist**

- [ ] Voice input uses Japanese transcription.
- [ ] The user can see whether recording or transcription is in progress.
- [ ] Transcription failure does not discard the user's ability to continue.
- [ ] Relevant observability events are emitted: `voice_input_started`, `transcription_completed`.
- [ ] Guardrail-sensitive phrases remain subject to safety handling after transcription.

### US-006: Text Fallback Intake

**Priority**: Must

**User Story**:
As a user whose microphone is unavailable or unreliable, I want to type my "やめたいこと" directly, so that I can still create a card.

**Personas**: P-01, P-02, P-03

**Requirement Trace**: FR-005, AC-002

**Acceptance Criteria**

Given voice input is unavailable, denied, or fails,  
When the user enters Japanese text manually,  
Then they can submit the text for AI structuring.

Given a user is on iOS Chrome and voice behavior is unreliable,  
When they use the text input,  
Then the text path remains the required fallback.

**Checklist**

- [ ] Text input is available in the core input experience.
- [ ] Empty input cannot be submitted.
- [ ] Overly long input is rejected with a user-facing explanation.
- [ ] Relevant observability event is emitted: `text_input_submitted`.
- [ ] Guardrail-sensitive typed input follows the same safety path as voice input.

### US-007: Recoverable Input and Processing Errors

**Priority**: Must

**User Story**:
As a user encountering microphone, network, AI, or save failures, I want safe and understandable recovery options, so that I can continue without losing trust.

**Personas**: P-02, P-04

**Requirement Trace**: FR-013, SEC-009, SEC-016

**Acceptance Criteria**

Given a processing failure occurs,  
When the app reports the error to the user,  
Then the message is generic, recoverable, and free of internal implementation details.

Given a voice input failure occurs,  
When recovery options are shown,  
Then text input remains available.

**Checklist**

- [ ] User-facing errors do not include stack traces, AWS resource names, tokens, or model internals.
- [ ] The app fails closed when authorization, guardrail, or schema validation fails.
- [ ] Retry or fallback is available where safe.
- [ ] Relevant observability event is emitted: `card_structure_failed` when structuring fails.

## Epic E-04: AI Structuring

### US-008: Structured Card Generation

**Priority**: Must

**User Story**:
As a user who has submitted a voice transcript or typed text, I want Tebanashi to turn it into a structured card, so that I can understand what I may be ready to stop doing.

**Personas**: P-01, P-02, P-03

**Requirement Trace**: FR-007, FR-008, AC-003, AC-004

**Acceptance Criteria**

Given a valid non-guardrail input is submitted,  
When AI structuring succeeds,  
Then the result contains title, category, estimated monthly time cost, estimated monthly money cost, withdrawal upside summary, status, ownerId, and createdAt.

Given AI returns malformed or incomplete output,  
When the app validates the response,  
Then the app rejects the result and does not save an invalid card.

Given card structuring is measured,  
When p95 latency is evaluated for release,  
Then the required threshold is 8 seconds and 5 seconds is tracked as a stretch target.

**Checklist**

- [ ] Structured output is schema-validated before saving.
- [ ] Estimated time and money costs are non-negative.
- [ ] Category is constrained to allowed values or a safe fallback category.
- [ ] Relevant observability events are emitted: `card_structure_requested`, `card_structure_succeeded`, `card_structure_failed`.
- [ ] Guardrail-sensitive input does not bypass safety handling.

### US-009: Save and List Active Cards

**Priority**: Must

**User Story**:
As a returning user, I want my active generated cards to be saved and listed, so that I can revisit the things I may choose to let go later.

**Personas**: P-01, P-03

**Requirement Trace**: FR-011, FR-012, AC-005

**Acceptance Criteria**

Given a valid structured card is generated,  
When the app saves it,  
Then it appears in the authenticated user's active card list.

Given the user returns with the same account,  
When the card list loads,  
Then previously saved active cards are visible to that user.

**Checklist**

- [ ] Saved cards have `active` status by default.
- [ ] Card list shows title, category, estimated time cost, estimated money cost, and withdrawal upside summary.
- [ ] Card list excludes cards owned by other users.
- [ ] Relevant observability event is emitted: `card_saved`.

## Epic E-05: Safety

### US-010: Guardrail-Safe Response

**Priority**: Must

**User Story**:
As a user entering health, medical, addiction, legal obligation, self-harm, or care-related content, I want Tebanashi to avoid unsafe affirmation, so that I am not encouraged toward harmful decisions.

**Personas**: P-04

**Requirement Trace**: FR-009, FR-010, AC-006, SEC-012

**Acceptance Criteria**

Given a user submits content involving health, medical care, medication, addiction, self-harm, legal obligation, support, or caregiving,  
When the guardrail path is triggered,  
Then the app does not produce a normal affirmative card.

Given the guardrail path is triggered,  
When the app responds,  
Then it uses a safe tone and encourages consultation with an appropriate expert or public resource.

**Checklist**

- [ ] Bedrock Guardrails is part of the MVP safety path.
- [ ] App-side validation or classification also participates in safety handling.
- [ ] Guardrail events avoid logging PII or full user input.
- [ ] Relevant observability event is emitted: `guardrail_triggered`.

### US-011: Guardrails Inside Normal Structuring Flow

**Priority**: Must

**User Story**:
As a user using the normal voice or text flow, I want safety checks to happen without needing a separate mode, so that risky inputs are handled consistently.

**Personas**: P-02, P-04

**Requirement Trace**: FR-009, FR-010, US-005, US-006, US-008

**Acceptance Criteria**

Given a user submits input through voice transcription,  
When the text is sent for structuring,  
Then the same guardrail checks apply before a normal card can be saved.

Given a user submits input through text fallback,  
When the text is sent for structuring,  
Then the same guardrail checks apply before a normal card can be saved.

**Checklist**

- [ ] Voice and text inputs share equivalent safety handling.
- [ ] Safety handling runs before saving a normal `active` card.
- [ ] A `needs_attention` state may be used only when clearly distinguished from normal cards.
- [ ] Safety criteria are represented in AI structuring tests and E2E scenarios.

## Epic E-06: Observability

### US-012: Core Observability and Latency Measurement

**Priority**: Must

**User Story**:
As a project owner, I want the intake and structuring journey to emit meaningful metrics, so that MVP success criteria and failures can be measured.

**Personas**: P-01, P-02, P-03, P-04

**Requirement Trace**: FR-014, NFR-005, NFR-006, SEC-003, SEC-015

**Acceptance Criteria**

Given a user completes or fails part of the intake journey,  
When the relevant step occurs,  
Then the app emits the expected event without logging PII, tokens, secrets, or full user input.

Given the system is evaluated for release,  
When latency and success metrics are reviewed,  
Then p95 structuring latency and voice-to-structure success rate can be calculated.

**Checklist**

- [ ] Events include `voice_input_started`.
- [ ] Events include `transcription_completed`.
- [ ] Events include `text_input_submitted`.
- [ ] Events include `card_structure_requested`.
- [ ] Events include `card_structure_succeeded`.
- [ ] Events include `card_structure_failed`.
- [ ] Events include `card_saved`.
- [ ] Events include `guardrail_triggered`.
- [ ] Logs avoid PII, email, tokens, secrets, and full user input.

## INVEST Review

| Story | Independent | Negotiable | Valuable | Estimable | Small | Testable |
|---|---|---|---|---|---|---|
| US-001 | Yes | Yes | Yes | Yes | Yes | Yes |
| US-002 | Yes | Yes | Yes | Yes | Yes | Yes |
| US-003 | Yes | Yes | Yes | Yes | Yes | Yes |
| US-004 | Yes | Yes | Yes | Yes | Yes | Yes |
| US-005 | Yes | Yes | Yes | Yes | Yes | Yes |
| US-006 | Yes | Yes | Yes | Yes | Yes | Yes |
| US-007 | Yes | Yes | Yes | Yes | Yes | Yes |
| US-008 | Yes | Yes | Yes | Yes | Yes | Yes |
| US-009 | Yes | Yes | Yes | Yes | Yes | Yes |
| US-010 | Yes | Yes | Yes | Yes | Yes | Yes |
| US-011 | Yes | Yes | Yes | Yes | Yes | Yes |
| US-012 | Yes | Yes | Yes | Yes | Yes | Yes |

## Persona Mapping

| Persona | Stories |
|---|---|
| P-01 継続疲れ層 | US-001, US-002, US-003, US-004, US-005, US-006, US-008, US-009, US-012 |
| P-02 自己肯定感強化希望層 | US-004, US-005, US-006, US-007, US-008, US-011, US-012 |
| P-03 サンクコスト捕囚層 | US-002, US-006, US-008, US-009, US-012 |
| P-04 注意喚起が必要なユーザー | US-007, US-010, US-011, US-012 |

## Out of Scope for Initial Stories

- ワンタップ「手放す」実行
- 手放しタイムスタンプ記録
- AI 全肯定メッセージ生成
- ご褒美提案
- 祝祭演出と効果音
- リソース解放ダッシュボード
- 月次振り返り
- コミュニティ機能

## Extension Compliance

### Security Baseline

Status: Compliant for User Stories generation.

Security-relevant user-visible requirements are represented in US-001, US-002, US-003, US-007, US-010, US-011, and US-012.
Rules that require concrete architecture, infrastructure, or code verification remain deferred to later AI-DLC stages.

### Property-Based Testing

Status: N/A for User Stories generation.

PBT Partial enforcement applies to later design, code generation, and build/test stages.
