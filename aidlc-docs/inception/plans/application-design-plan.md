# アプリケーション設計計画

## 目的

承認済み要求、ユーザーストーリー、実行計画をもとに、Tebanashi 初回開発スコープの高レベルアプリケーション設計を作成する。
この計画の `[Answer]:` がすべて埋まり、回答の曖昧さが解消されるまで、アプリケーション設計成果物は生成しない。

## コンテキスト

- **プロジェクト**: Tebanashi
- **ステージ**: INCEPTION - アプリケーション設計
- **承認済み初回スコープ**: 音声/テキスト入力から AI によるやめ候補作成・保存・表示まで
- **主な入力**:
  - `aidlc-docs/inception/requirements/requirements.md`
  - `aidlc-docs/inception/user-stories/stories.md`
  - `aidlc-docs/inception/user-stories/personas.md`
  - `aidlc-docs/inception/plans/execution-plan.md`

## 設計計画の進捗

- [x] アプリケーション設計ルールを読む
- [x] 承認済み要求を読む
- [x] 承認済みユーザーストーリーとペルソナを読む
- [x] 承認済み実行計画を読む
- [x] アプリケーション設計計画を作成する
- [x] この計画内のすべての `[Answer]:` タグに対する回答を収集する
- [x] 回答の曖昧さや矛盾を分析する
- [x] 必要に応じて追加質問を作成する（不要。ブロッキングな曖昧さなし）
- [ ] このアプリケーション設計計画の明示的な承認を得る

## 設計生成チェックリスト

- [ ] `aidlc-docs/inception/application-design/components.md` を生成する
- [ ] `aidlc-docs/inception/application-design/component-methods.md` を生成する
- [ ] `aidlc-docs/inception/application-design/services.md` を生成する
- [ ] `aidlc-docs/inception/application-design/component-dependency.md` を生成する
- [ ] `aidlc-docs/inception/application-design/application-design.md` を生成する
- [ ] コンポーネント名、目的、責務、インターフェースを定義する
- [ ] 高レベルのメソッドシグネチャと入出力型を定義する
- [ ] サービスのオーケストレーション境界を定義する
- [ ] 依存関係と通信パターンを定義する
- [ ] Security Baseline との整合性を検証する
- [ ] PBT Partial の義務を該当する後続ステージへ引き継ぐ

## 想定コンポーネント領域

以下の質問回答で別方針が示されない限り、次のコンポーネント領域を想定する。

| コンポーネント領域 | 目的 |
|---|---|
| アプリシェルとルーティング | React ルーティング、認証ゲート、レイアウトシェル、レスポンシブなページ構成 |
| 認証とユーザー識別 | Cognito/Google ログインのセッション処理と owner identity の提供 |
| ブラウザサポートゲート | フロントエンドでの Chrome 対応判定と非対応ブラウザ案内 |
| 入力 UI | 音声入力状態、テキストフォールバック、送信フロー、ユーザー向けエラー状態 |
| 文字起こしクライアント | ブラウザ側の Transcribe Streaming クライアント統合 |
| カード構造化 API クライアント | Lambda/AppSync に支えられた構造化ワークフロー用フロントエンドクライアント |
| カードドメインモデル | Card スキーマ、状態値、検証契約、owner マッピング |
| カード永続化 | owner スコープの Card 保存・一覧表示のための AppSync/Amplify Data アクセス |
| AI 構造化関数 | Bedrock Claude による構造化を担う Lambda 境界 |
| 安全ガードレール | ガードレール分類、Bedrock Guardrails 応答処理、安全な応答整形 |
| 観測性 | イベント送信、構造化ログ、p95/成功率メトリクスの支援 |

## 質問

各質問の `[Answer]:` の後に選択肢の文字を記入してください。
選択肢に合わない場合は `X` を選び、希望する内容を記入してください。

### Question 1: フロントエンドのコンポーネント境界

React フロントエンドのコンポーネントは、主にどの境界で整理しますか？

A) 機能別フォルダ: `auth`, `platform`, `intake`, `cards`, `safety`, `observability`
B) レイヤー別フォルダ: `pages`, `components`, `hooks`, `lib`, `types`
C) ハイブリッド: ドメイン固有コードは機能別、再利用 UI/hooks/lib は共通レイヤー別
X) Other (please describe after [Answer]: tag below)

[Answer]: C

### Question 2: 認証 UI の境界

Google ログイン UI は、設計上どのコンポーネント境界として扱いますか？

A) Amplify UI Authenticator を認証境界として使い、アプリ固有の軽いラップだけ行う
B) Amplify Auth API を直接呼び出す独自ログインページを設計する
C) 初回スコープでは認証 UI を外部扱いにし、認証済みアプリシェルのみ設計する
X) Other (please describe after [Answer]: tag below)

[Answer]: A

### Question 3: AI 構造化と安全制御の境界

AI 構造化と安全ガードレールの責務はどのように分離しますか？

A) 単一 Lambda がガードレール確認、Bedrock 構造化、スキーマ検証、応答整形をまとめてオーケストレーションする
B) Lambda を分ける: ガードレール/安全分類用と Card 構造化用を別関数にする
C) フロントエンドは単一バックエンド API を呼ぶが、バックエンド内部では safety と structuring を別モジュールとして設計する
X) Other (please describe after [Answer]: tag below)

[Answer]: C

### Question 4: ガードレール判定結果の永続化

健康・医療・依存症・法的義務・扶養/介護などのガードレールが発火した入力について、記録を永続化しますか？

A) Card は永続化せず、安全な応答のみ表示する
B) `needs_attention` ステータスの Card として永続化し、active cards とは明確に分離する
C) ユーザーテキストや Card データを含まない最小限の監査/メトリクスイベントだけ永続化する
X) Other (please describe after [Answer]: tag below)

[Answer]: C

### Question 5: Card 作成フローのオーケストレーション

入力テキストが準備できた後、Card 作成までの一連のフローはどのコンポーネントがオーケストレーションしますか？

A) フロントエンド主導: UI が構造化 Lambda を呼び、その後 AppSync mutation で Card を保存する
B) バックエンド主導: 1 つのバックエンド処理が構造化と永続化を行い、保存済み Card を返す
C) 分割: フロントエンドは UX 状態を担当し、バックエンドは信頼境界内の変更処理と永続化をすべて担当する
X) Other (please describe after [Answer]: tag below)

[Answer]: C

### Question 6: Transcribe Streaming の境界

Transcribe Streaming は設計上どのように表現しますか？

A) Cognito Identity Pool の認証済み一時クレデンシャルを使い、ブラウザから Transcribe Streaming へ直接接続する
B) API Gateway WebSocket + Lambda 中継で Transcribe Streaming に接続する
C) 両方を設計し、ブラウザ直接接続を主方式、Lambda 中継を文書化されたフォールバックとする
X) Other (please describe after [Answer]: tag below)

[Answer]: A

### Question 7: 観測性コンポーネントの境界

観測性はコンポーネント横断でどのように扱いますか？

A) 全コンポーネントが利用する中央集約の client/server observability module を置く
B) 各機能が自分のイベント送信とログ呼び出しを所有する
C) イベントスキーマは中央集約し、イベント送信用アダプターは各機能が所有する
X) Other (please describe after [Answer]: tag below)

[Answer]: A

### Question 8: 共有コントラクト定義

Card、API、AI 応答の共有コントラクトは概念上どこで定義しますか？

A) フロントエンドとバックエンドが利用する共有 TypeScript domain package/module に定義する
B) フロントエンドとバックエンドで別々に定義し、生成された Amplify 型と Zod スキーマで同期する
C) バックエンドが source-of-truth のスキーマを所有し、フロントエンドは生成 API 型とローカル view model を利用する
X) Other (please describe after [Answer]: tag below)

[Answer]: C

### Question 9: エラーハンドリング設計の境界

ユーザーに表示するエラーは、アプリケーション設計上どのようにモデル化しますか？

A) 共有の型付きエラーモデルで、バックエンド/システムエラーを安全な UI メッセージへマッピングする
B) 各機能が自分のユーザー向けエラーメッセージを定義する
C) バックエンドは汎用メッセージのみ返し、詳細な復旧案内はすべてフロントエンドが決める
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## 承認ゲート

## 回答分析

- **Q1 フロントエンド境界**: C。ドメイン固有コードは機能別、再利用 UI/hooks/lib は共通レイヤー別にする。
- **Q2 認証 UI 境界**: A。Amplify UI Authenticator を認証境界として使い、アプリ固有の軽いラップだけ行う。
- **Q3 AI 構造化と安全制御**: C。フロントエンドは単一バックエンド API を呼ぶが、バックエンド内部では safety と structuring を別モジュールとして設計する。
- **Q4 ガードレール結果永続化**: C。ユーザーテキストや Card データを含まない最小限の監査/メトリクスイベントだけ永続化する。
- **Q5 Card 作成オーケストレーション**: C。フロントエンドは UX 状態を担当し、バックエンドは信頼境界内の変更処理と永続化を担当する。
- **Q6 Transcribe Streaming 境界**: A。Cognito Identity Pool の認証済み一時クレデンシャルを使い、ブラウザから直接接続する。
- **Q7 観測性境界**: A。全コンポーネントが利用する中央集約の client/server observability module を置く。
- **Q8 共有コントラクト定義**: C。バックエンドが source-of-truth のスキーマを所有し、フロントエンドは生成 API 型とローカル view model を利用する。
- **Q9 エラーハンドリング境界**: A。共有の型付きエラーモデルで、バックエンド/システムエラーを安全な UI メッセージへマッピングする。

### 曖昧さレビュー

ブロッキングな曖昧さや矛盾は見つからなかった。
Q4 と Q5 は、通常の Card 作成はバックエンドが信頼境界内で永続化し、ガードレール発火時は Card を作らず最小限の監査/メトリクスイベントのみ残す、という扱いで整合する。
Q3、Q5、Q8、Q9 は、バックエンドを信頼境界としつつ内部モジュール分離と型付きエラー/スキーマ境界を明示する方針として整合する。

すべての `[Answer]:` タグが記入され、検証された後、この計画を明示的な承認に回す。
アプリケーション設計成果物は、計画承認後にのみ生成する。

## 拡張ルール準拠

### Security Baseline

Status: 計画段階では準拠。

認証、ガードレール、永続化、オーケストレーション、Transcribe クレデンシャル、観測性、共有コントラクト、安全なエラーハンドリングに関する質問により、セキュリティ関連の境界を明示的に扱っている。

### Property-Based Testing

Status: Application Design planning では N/A。

PBT Partial enforcement は、後続の NFR Requirements、Code Generation、Build and Test ステージで適用する。
