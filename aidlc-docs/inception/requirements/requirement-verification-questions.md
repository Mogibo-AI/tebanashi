# Requirements Verification Questions

Tebanashi MVP の要求を確定するため、以下の質問に回答してください。
各質問の `[Answer]:` の後に選択肢の文字を記入してください。
選択肢に合わない場合は `X` を選び、同じ行または次の行に内容を記入してください。

## Intent Analysis Summary

- **User Request**: AI-DLC ワークフローで開発を開始する
- **Request Type**: New Project
- **Scope Estimate**: System-wide
- **Complexity Estimate**: Complex
- **Requirements Depth**: Comprehensive
- **Primary Inputs**:
  - `.aidlc/inputs/vision-document.md`
  - `.aidlc/inputs/technical-environment-document.md`

## Question 1: 今回の AI-DLC 対象スコープ
今回の AI-DLC ワークフローでは、どこまでを初回の開発対象にしますか？

A) MVP 全体を対象にする
B) 認証・データモデル・基本UIまでの foundation を初回対象にする
C) 音声/テキスト入力からやめ候補作成までを初回対象にする
D) まず AI-DLC ドキュメント整備のみを対象にする
X) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 2: 健康・医療・依存症などのガードレール
「手放し」を全肯定すべきでないドメインは、MVPでどの扱いにしますか？

A) 健康・医療・服薬・依存症・自傷他害・法的義務・扶養/介護などは肯定せず、専門家や公的窓口への相談を促す
B) 健康・医療・依存症のみを対象外にし、それ以外は通常の肯定トーンにする
C) LLM の判断に任せ、明示的な禁止カテゴリは最小限にする
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 3: AI ご褒美提案の具体性
AI が生成する「ご褒美提案」には、どの程度具体的な金額や商品名を含めますか？

A) 金額の範囲と一般カテゴリのみを提示し、具体商品名や特定店舗名は出さない
B) ユーザー文脈に合う一般的な商品例は出すが、広告的表現や購入誘導は避ける
C) 具体商品名・サービス名まで許可する
X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 4: やめ候補と手放し履歴の保存期間
ユーザーに紐づくやめ候補と手放し履歴は、MVPでどの保存ポリシーにしますか？

A) ユーザーが削除またはアカウント削除するまで保存する
B) 一定期間で自動削除する
C) やめ候補は保存するが、手放し後の詳細履歴は短期間で削除する
X) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 5: モデル改善へのユーザー入力利用
ユーザー入力テキストや生成結果を、モデル改善やプロンプト改善に利用する前提をMVPに含めますか？

A) 含めない。MVPではサービス提供目的の処理と保存に限定する
B) 明示同意したユーザーの入力のみ、匿名化して改善に利用する
C) 利用規約で明示したうえで、匿名化・集計した範囲で改善に利用する
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 6: アカウント削除時のデータ削除
ユーザーがアカウント削除を要求した場合、関連データはどう扱いますか？

A) Cognito ユーザーと DynamoDB 上のやめ候補・手放し履歴を削除する
B) ユーザー識別子を削除/匿名化し、統計用途の集計データのみ残す
C) MVPではアカウント削除導線を提供せず、運用手続きで対応する
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 7: 祝祭演出の音声
「手放す」ボタン押下後の効果音は、MVPでどの扱いにしますか？

A) デフォルト ON。ただしユーザー操作後に再生し、ミュート切替を提供する
B) デフォルト OFF。ユーザーが明示的にONにした場合のみ再生する
C) MVPでは音声なし。視覚演出のみ提供する
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 8: アクセシビリティ範囲
MVPで求めるアクセシビリティ対応の範囲はどれですか？

A) WCAG 2.2 AA を目標に、キーボード操作・スクリーンリーダー・色コントラスト・モーション低減を設計に含める
B) 主要操作のキーボード操作、ラベル、色コントラスト、モーション低減のみ必須にする
C) MVPでは基本的なHTMLセマンティクスとラベル付けに限定する
X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 9: iOS Chrome の音声入力フォールバック
iOS Chrome で Web Audio / マイク権限が想定通り動作しない場合、MVPでどこまで代替フローを実装しますか？

A) テキスト入力フォールバックのみを必須にする
B) 録音ファイルアップロードによる文字起こしも実装する
C) iOS Chrome はMVP対応対象から除外し、デスクトップChromeとAndroid Chromeに限定する
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 10: レスポンシブ UI 方針
MVP のレスポンシブ UI はどちらを基準に設計しますか？

A) モバイルファーストで設計し、デスクトップへ拡張する
B) デスクトップファーストで設計し、モバイルへ最適化する
C) コア操作画面だけモバイルファースト、管理/設定画面はデスクトップ優先にする
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 11: Bedrock Guardrails の採用
健康・医療・依存症などの過度な肯定を防ぐため、MVPで Bedrock Guardrails を採用しますか？

A) 採用する。プロンプト制御に加えて Bedrock Guardrails を必須にする
B) MVPではプロンプトとアプリ側バリデーションで対応し、Guardrails は Phase 2 で検討する
C) 初期リリース前に評価して、レイテンシと運用コスト次第で採否を決める
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 12: Google email クレームの保存
Google ログインで取得した email は、アプリの永続データとして保存しますか？

A) 保存しない。永続データは Cognito `sub` を主識別子にし、email は認証トークンから必要時に参照する
B) ユーザー設定や問い合わせ対応のため、email を DynamoDB に保存する
C) ハッシュ化または暗号化した email のみ保存する
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 13: DynamoDB モデリング方針
MVP の永続データはどのモデリング方針にしますか？

A) Amplify Data の自然なモデル定義に従い、Card と WithdrawalEvent を別モデルとして定義する
B) 将来の集計や拡張を見越し、単一テーブル設計を前提に設計する
C) まず Card のみを永続化し、WithdrawalEvent は後続フェーズに回す
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 14: 非 Chrome ブラウザの扱い
Chrome 以外のブラウザアクセス時、MVPではどのように案内しますか？

A) フロントエンドで検出し、Chrome 利用を促す画面を表示する
B) CloudFront Function などエッジ側で検出し、案内ページへ誘導する
C) 明示的なブロックはせず、サポート外表示のみ行う
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 15: Security Extensions
このプロジェクトで security extension rules をブロッキング制約として適用しますか？

A) Yes - すべての SECURITY ルールをブロッキング制約として適用する
B) No - SECURITY ルールをスキップする
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 16: Property-Based Testing Extension
このプロジェクトで property-based testing (PBT) rules を適用しますか？

A) Yes - PBT ルールをブロッキング制約として適用する
B) Partial - pure function と serialization round-trip に限定して PBT ルールを適用する
C) No - PBT ルールをスキップする
X) Other (please describe after [Answer]: tag below)

[Answer]: B
