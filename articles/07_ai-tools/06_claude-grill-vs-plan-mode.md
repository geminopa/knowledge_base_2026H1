# /grill-me と Planモードの違い

Planモードは「AIが調べて計画を作り、人が承認する」、`/grill-me` は「AIが質問し、人が判断する」。複数の記事で共通して言われているのは、**設計上の判断を誰が下すか**の違い。

## 比較表

| | Planモード | `/grill-me` |
|---|---|---|
| 種類 | Claude Code標準の権限モード | 外部Skill（mattpocock/skills） |
| 判断を下すのは | AI（計画として一括で提示し、人は承認する） | 人（AIの質問に答えて決める） |
| 出てくるもの | 完成した計画 | 質問（推奨回答つき） |
| 編集の制限 | 承認までファイル編集をブロック（権限による強制） | ブロックなし（Skillの指示として、確認が取れるまで実行しない） |
| 起動方法 | `Shift+Tab` / `/plan` / `claude --permission-mode plan` | `/grill-me 計画の説明` |

## 複数の出典で共通して言えること

1. **Planモードでは判断がAI側に寄り、`/grill-me` では人側に寄る。**
   Planモードは調査して計画を一括で出し、人は承認する側になる。`/grill-me` は質問に答える形で、人が判断を下す側になる（出典 3・4・5・6）
2. **一括で出た計画は「なんとなく承認」しやすい。**
   もっともらしい計画が一度に出るため、AIの暗黙の前提に気づかないまま承認しがち、という指摘が複数ある（出典 4・5・6）
3. **計画の一部を変えたいとき、Planモードは計画の書き直しになりやすい。**
   `/grill-me` は対話のやり取り自体が計画になるため、実装へ移りやすいという体験談がある（出典 3・5）
4. **Planモードの目的は「勝手に実装させない」こと、`/grill-me` の目的は「勝手に決めさせない」こと。**
   前者は編集をブロックする安全装置、後者は判断を人に戻す対話（出典 1・3・5）
5. **`/grill-me` の作者は、使うときはPlanモードをオフにするよう書いている。**
   理由は「Planモードは計画づくりへ急がせ、質問を続ける姿勢と逆になるため」（出典 2）

## 注意点

- **出典 3〜6は個人の体験・意見。** 特に「Planモードは質問なしに実装まで進んだ」（出典 4）は1回の比較の記述で、Planモードが必ず質問しないという意味ではない。公式ドキュメント（出典 1）にも、Planモードが質問を重ねる仕組みの記載はない
- **`/grill-me` は「1問ずつ聞く」と書く記事が多いが、現行の定義（出典 2の `grilling`）はラウンドごとに複数質問。** 記事の執筆時期や版で差がある
- 速度の観点では、Planモードの方が速く、`/grill-me` は時間がかかるという指摘がある（出典 4）。仕様の曖昧さや影響の大きさに応じて選ぶ

## 出典

| No | 種別 | 内容 | URL |
|---|---|---|---|
| 1 | 公式 | Claude Code公式ドキュメント「Choose a permission mode」（Planモードの仕様） | https://code.claude.com/docs/en/permission-modes |
| 2 | 作者 | Matt Pocock「The /grill-me Skill」、およびリポジトリ内 `docs/productivity/grill-me.md`（どちらも「Leave plan mode off」の記載） | https://www.aihero.dev/skills-grill-me / https://github.com/mattpocock/skills/blob/main/docs/productivity/grill-me.md |
| 3 | 非公式 | Ryo Nakae「grill-me スキルがめちゃ良いので布教したい」（Zenn、2026-04-09） | https://zenn.dev/ryonakae/articles/8783c6b3ead2cb |
| 4 | 非公式 | Alex Rusin「Claude Code Planning Tools Compared: Plan Mode vs Grill Me vs Superpowers」（2026-05-25） | https://blog.alexrusin.com/claude-code-planning-tools-plan-mode-vs-grill-me-vs-superpowers/ |
| 5 | 非公式 | 竹内 一真「Plan modeを見直す 〜grill-meスキルで設計を固め、アプリを作る〜」（cloud.config Tech Blog、2026-05-28） | https://tech-blog.cloud-config.jp/2026-05-27-plan-mode-vs-grill-me |
| 6 | 非公式 | Hack-Log「I stopped using Claude Code's Plan mode. The grill-me skill completely changed my design process」（note、2026-06-05） | https://note.com/hacklog_stealth/n/n7044b69f0390 |
