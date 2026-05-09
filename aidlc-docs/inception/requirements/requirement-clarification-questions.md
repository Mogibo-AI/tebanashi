# Requirements Clarification Questions

回答内容は有効でしたが、入力文書内に現在スコープへ直接影響する不一致が 1 件あります。
以下の `[Answer]:` に選択肢の文字を記入してください。

## Ambiguity 1: 構造化完了までの p95 レイテンシ

`.aidlc/inputs/vision-document.md` では、Success Metrics に「音声入力後の構造化完了まで p95 で 8 秒以内」とあり、MVP Success Criteria には「Chrome ブラウザで音声入力を行い、5 秒以内に構造化されたやめ候補が表示」とあります。

### Clarification Question 1
今回の「音声/テキスト入力からやめ候補作成まで」のリリース判定では、どのレイテンシ目標を採用しますか？

A) p95 5 秒以内を必須のリリース基準にする
B) p95 8 秒以内を必須のリリース基準にし、5 秒以内はストレッチ目標にする
C) テキスト入力は p95 5 秒以内、音声入力は p95 8 秒以内に分ける
X) Other (please describe after [Answer]: tag below)

[Answer]: B
