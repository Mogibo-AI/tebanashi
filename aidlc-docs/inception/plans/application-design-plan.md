# Application Design Plan

## Purpose

承認済み Requirements、User Stories、Execution Plan をもとに、Tebanashi 初回開発スコープの高レベルアプリケーション設計を作成する。
この plan の `[Answer]:` がすべて埋まり、回答の曖昧さが解消されるまで、Application Design artifacts は生成しない。

## Context

- **Project**: Tebanashi
- **Stage**: INCEPTION - Application Design
- **Approved Initial Scope**: 音声/テキスト入力から AI によるやめ候補作成・保存・表示まで
- **Primary Inputs**:
  - `aidlc-docs/inception/requirements/requirements.md`
  - `aidlc-docs/inception/user-stories/stories.md`
  - `aidlc-docs/inception/user-stories/personas.md`
  - `aidlc-docs/inception/plans/execution-plan.md`

## Design Planning Progress

- [x] Read Application Design rule file
- [x] Read approved requirements
- [x] Read approved user stories and personas
- [x] Read approved execution plan
- [x] Create application design plan
- [ ] Collect answers for all `[Answer]:` tags in this plan
- [ ] Analyze answers for ambiguity or contradiction
- [ ] Create follow-up questions if needed
- [ ] Obtain explicit approval of this application design plan

## Design Generation Checklist

- [ ] Generate `aidlc-docs/inception/application-design/components.md`
- [ ] Generate `aidlc-docs/inception/application-design/component-methods.md`
- [ ] Generate `aidlc-docs/inception/application-design/services.md`
- [ ] Generate `aidlc-docs/inception/application-design/component-dependency.md`
- [ ] Generate `aidlc-docs/inception/application-design/application-design.md`
- [ ] Define component names, purposes, responsibilities, and interfaces
- [ ] Define high-level method signatures and input/output types
- [ ] Define service orchestration boundaries
- [ ] Define dependency relationships and communication patterns
- [ ] Validate consistency with Security Baseline
- [ ] Carry PBT Partial obligations forward to later stages where applicable

## Proposed Component Areas

The following component areas are expected unless answers below direct otherwise:

| Component Area | Purpose |
|---|---|
| App Shell and Routing | React routing, auth gating, layout shell, responsive page composition |
| Auth and Identity | Cognito/Google sign-in session handling and owner identity exposure |
| Browser Support Gate | Frontend Chrome support detection and unsupported-browser notice |
| Intake UI | Voice input state, text fallback, submission flow, user-visible error states |
| Transcription Client | Browser-side Transcribe Streaming client integration |
| Card Structuring API Client | Frontend client for Lambda/AppSync-backed structuring workflow |
| Card Domain Model | Card schema, status values, validation contracts, owner mapping |
| Card Persistence | AppSync/Amplify Data access for saving and listing owner-scoped cards |
| AI Structuring Function | Lambda boundary for Bedrock Claude structuring |
| Safety Guardrail | Guardrail classification, Bedrock Guardrails response handling, safe response shaping |
| Observability | Event emission, structured logging, p95/success-rate metric support |

## Questions

Please answer every question by filling in the letter choice after each `[Answer]:` tag.
If none of the options match, choose `X` and describe your preference.

### Question 1: Frontend Component Boundary

React frontend components should be organized primarily by which boundary?

A) Feature folders: `auth`, `platform`, `intake`, `cards`, `safety`, `observability`
B) Layer folders: `pages`, `components`, `hooks`, `lib`, `types`
C) Hybrid: feature folders for domain-specific code, shared layer folders for reusable UI/hooks/lib
X) Other (please describe after [Answer]: tag below)

[Answer]: 

### Question 2: Authentication UI Boundary

Google sign-in UI should be represented in design as which component boundary?

A) Use Amplify UI Authenticator as the auth boundary with light app-specific wrapping
B) Design a custom sign-in page that calls Amplify Auth APIs directly
C) Treat auth UI as external for initial scope and design only the authenticated app shell
X) Other (please describe after [Answer]: tag below)

[Answer]: 

### Question 3: AI Structuring and Safety Boundary

How should AI structuring and safety guardrail responsibilities be separated?

A) Single Lambda orchestrates guardrail check, Bedrock structuring, schema validation, and response shaping
B) Separate Lambda functions: one for guardrail/safety classification and one for card structuring
C) Frontend calls a single backend API, but backend design separates safety and structuring as internal modules
X) Other (please describe after [Answer]: tag below)

[Answer]: 

### Question 4: Guardrail Result Persistence

When an input triggers health/medical/addiction/legal/caregiving guardrails, should a record be persisted?

A) Do not persist a Card; show safe response only
B) Persist a Card with `needs_attention` status and clearly separate it from active cards
C) Persist only a minimal audit/metric event without user text or Card data
X) Other (please describe after [Answer]: tag below)

[Answer]: 

### Question 5: Card Creation Orchestration

Which component should orchestrate the end-to-end card creation flow after input text is ready?

A) Frontend orchestration: UI calls structuring Lambda, then AppSync mutation to save Card
B) Backend orchestration: one backend operation structures and persists Card, returning saved Card
C) Split orchestration: frontend owns UX state, backend owns all trusted mutation and persistence steps
X) Other (please describe after [Answer]: tag below)

[Answer]: 

### Question 6: Transcribe Streaming Boundary

How should Transcribe Streaming be represented in the design?

A) Browser direct Transcribe Streaming client using Cognito Identity Pool credentials
B) API Gateway WebSocket + Lambda relay to Transcribe Streaming
C) Design both, with browser direct as primary and relay as documented fallback
X) Other (please describe after [Answer]: tag below)

[Answer]: 

### Question 7: Observability Component Boundary

How should observability be handled across components?

A) Central client/server observability module used by all components
B) Each feature owns its own event emission and logging calls
C) Central event schema with feature-owned adapters for emitting events
X) Other (please describe after [Answer]: tag below)

[Answer]: 

### Question 8: Shared Contract Definition

Where should shared Card, API, and AI response contracts be defined conceptually?

A) In a shared TypeScript domain package/module used by frontend and backend
B) Separately in frontend and backend, synchronized through generated Amplify types and Zod schemas
C) Backend owns source-of-truth schemas; frontend consumes generated API types and local view models
X) Other (please describe after [Answer]: tag below)

[Answer]: 

### Question 9: Error Handling Design Boundary

How should user-visible errors be modeled in the application design?

A) A shared typed error model maps backend/system errors to safe UI messages
B) Each feature defines its own user-visible error messages
C) Backend returns only generic messages; frontend decides all detailed recovery guidance
X) Other (please describe after [Answer]: tag below)

[Answer]: 

## Approval Gate

After all `[Answer]:` tags are completed and validated, this plan will be submitted for explicit approval.
Application Design artifacts are generated only after plan approval.

## Extension Compliance

### Security Baseline

Status: Compliant for planning.

Security-relevant boundaries are explicitly covered by questions about auth, guardrails, persistence, orchestration, Transcribe credentials, observability, shared contracts, and safe error handling.

### Property-Based Testing

Status: N/A for Application Design planning.

PBT Partial enforcement applies in later NFR Requirements, Code Generation, and Build and Test stages.
