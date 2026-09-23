# 社内ナレッジベース記事リスト（残り35本 / 締切: 2026-09-30）

進捗管理用。ステータスは `未着手` / `下書き` / `投稿済み` のいずれかで更新する。

`articles/` 配下はジャンル（軸）ごとにサブディレクトリを分けている。ファイル名の先頭番号はこの表のNoではなく、各サブディレクトリ内での連番。

| No | タイトル案 | 軸 | ステータス | 投稿日 | ファイル |
|---|---|---|---|---|---|
| 1 | git commit --amendで直前のコミットメッセージを修正する | ①ツール・コマンド | 未着手 | | |
| 2 | git rebase -iでコミット履歴を整理する | ①ツール・コマンド | 未着手 | | |
| 3 | git stashで作業を一時退避するテクニック | ①ツール・コマンド | 未着手 | | |
| 4 | cpコマンドで複数ファイルを一括コピーする方法 | ①ツール・コマンド | 未着手 | | |
| 5 | mvコマンドでリネームする時の注意点 | ①ツール・コマンド | 未着手 | | |
| 6 | grepで大量ログから特定パターンを高速検索する | ①ツール・コマンド | 下書き | | `01_tools-commands/01_grep-log-search.md` |
| 7 | エディタのマルチカーソル編集で置換作業を効率化する | ①ツール・コマンド | 未着手 | | |
| 8 | 見積もりに「調査・検証」の工数を入れ忘れて痛い目にあった話 | ②見積もり | 未着手 | | |
| 9 | 既存システム改修の見積もりで調査工数を別枠にする理由 | ②見積もり | 未着手 | | |
| 10 | 見積もり時に確認すべき非機能要件チェックリスト | ②見積もり | 未着手 | | |
| 11 | 工数見積もりでバッファを取る際の考え方（楽観・悲観・最頻値） | ②見積もり | 未着手 | | |
| 12 | 要望が曖昧な段階で見積もり精度を上げるための質問リスト | ②見積もり | 未着手 | | |
| 13 | テーブル設計で正規化・非正規化どちらを選ぶかの判断軸 | ③設計 | 未着手 | | |
| 14 | API設計でリソースの粒度に迷った時の考え方 | ③設計 | 未着手 | | |
| 15 | 画面設計で「一覧に何を出すか」を決める視点 | ③設計 | 未着手 | | |
| 16 | 排他制御（楽観ロック・悲観ロック）をどちらにするかの判断基準 | ③設計 | 未着手 | | |
| 17 | 設計レビューで指摘されがちなポイントまとめ | ③設計 | 未着手 | | |
| 18 | N+1問題に気づくためのログの見方 | ④製造 | 未着手 | | |
| 19 | printデバッグではなくログレベルを使い分ける理由 | ④製造 | 未着手 | | |
| 20 | Nullチェック漏れを防ぐためのコーディング習慣 | ④製造 | 未着手 | | |
| 21 | 命名で迷った時に使う語彙リスト（get/fetch/list等の使い分け） | ④製造 | 未着手 | | |
| 22 | コードレビューで指摘されやすいミスパターン集 | ④製造 | 未着手 | | |
| 23 | 正規表現でハマりやすい落とし穴 | ④製造 | 下書き | | `04_implementation/01_regex-pitfalls.md` |
| 24 | 境界値テストで見落としがちなケース | ⑤テスト | 下書き | | `05_testing/01_boundary-test-cases.md` |
| 25 | テストデータ作成を効率化するちょっとした工夫 | ⑤テスト | 未着手 | | |
| 26 | 手動テストのエビデンス取得を効率化する方法 | ⑤テスト | 下書き | | `05_testing/02_manual-test-evidence.md` |
| 27 | リグレッションテストの範囲をどう決めるか | ⑤テスト | 未着手 | | |
| 28 | バグ再現手順の書き方（再現性を上げるコツ） | ⑤テスト | 下書き | | `05_testing/03_bug-report-writing.md` |
| 29 | リリース前チェックリストの作り方 | ⑥リリース | 未着手 | | |
| 30 | 本番リリース時のロールバック手順を事前に用意する重要性 | ⑥リリース | 未着手 | | |
| 31 | DBマイグレーションを伴うリリースで気をつけること | ⑥リリース | 未着手 | | |
| 32 | リリース作業のタイムスケジュールをどう組むか | ⑥リリース | 未着手 | | |
| 33 | リリース後の動作確認で最低限見るべきポイント | ⑥リリース | 未着手 | | |
| 34 | Claude Codeにコードレビューを下読みしてもらう使い方 | ⑦AI/ツール活用 | 下書き | | `07_ai-tools/01_claude-code-review-first-pass.md` |
| 35 | AIにテストケース案を出してもらう時のプロンプトの工夫 | ⑦AI/ツール活用 | 下書き | | `07_ai-tools/02_ai-test-case-prompting.md` |
| 36 | AIで見積もり資料のたたき台を作らせる方法 | ⑦AI/ツール活用 | 下書き | | `07_ai-tools/03_ai-estimation-draft.md` |
| 37 | AIツールで設計ドキュメントの誤字脱字チェックをする | ⑦AI/ツール活用 | 下書き | | `07_ai-tools/04_ai-doc-typo-check.md` |
| 38 | MySQL 5.6→8.4: 文字コード・照合順序のデフォルト変更 | ⑧MySQLバージョンアップ | 下書き | | `08_mysql-upgrade/01_mysql-upgrade-charset.md` |
| 39 | MySQL 5.6→8.4: 認証プラグイン・認証方式の変更 | ⑧MySQLバージョンアップ | 下書き | | `08_mysql-upgrade/02_mysql-upgrade-auth.md` |
| 40 | MySQL 5.6→8.4: sql_modeとSQL挙動の厳格化 | ⑧MySQLバージョンアップ | 下書き | | `08_mysql-upgrade/03_mysql-upgrade-sqlmode.md` |
| 41 | MySQL 5.6→8.4: InnoDB関連パラメータのデフォルト値変化 | ⑧MySQLバージョンアップ | 下書き | | `08_mysql-upgrade/04_mysql-upgrade-innodb.md` |
| 42 | MySQL 5.6→8.4: レプリケーション用語・コマンドの刷新 | ⑧MySQLバージョンアップ | 下書き | | `08_mysql-upgrade/05_mysql-upgrade-replication-terms.md` |
| 43 | MySQL 5.6→8.4: 廃止・削除された主要パラメータ一覧 | ⑧MySQLバージョンアップ | 下書き | | `08_mysql-upgrade/06_mysql-upgrade-removed-params.md` |
| 44 | MySQL 5.6→8.4: その他の運用系パラメータのデフォルト変更 | ⑧MySQLバージョンアップ | 下書き | | `08_mysql-upgrade/07_mysql-upgrade-misc-defaults.md` |
| 45 | Claude Codeのスラッシュコマンド早見表（まず覚えたい10個） | ⑨Claude Codeスラッシュコマンド | 下書き | | `09_claude-slash-commands/01_claude-slash-cheatsheet.md` |
| 46 | /clear と /compact の使い分け | ⑨Claude Codeスラッシュコマンド | 下書き | | `09_claude-slash-commands/02_claude-clear-vs-compact.md` |
| 47 | /init と CLAUDE.md で、プロジェクトの前提を毎回説明しなくて済むようにする | ⑨Claude Codeスラッシュコマンド | 下書き | | `09_claude-slash-commands/03_claude-init-claude-md.md` |
| 48 | /resume で前回の会話の続きから作業する | ⑨Claude Codeスラッシュコマンド | 下書き | | `09_claude-slash-commands/04_claude-resume.md` |
| 49 | /code-review コマンドのオプションを使い分ける | ⑨Claude Codeスラッシュコマンド | 下書き | | `09_claude-slash-commands/05_claude-code-review-command.md` |
| 50 | /permissions で「毎回の確認」を減らしつつ、危険な操作は止める | ⑨Claude Codeスラッシュコマンド | 下書き | | `09_claude-slash-commands/06_claude-permissions.md` |
| 51 | そもそもSkillsとは何か | ⑨Claude Codeスラッシュコマンド | 下書き | | `09_claude-slash-commands/07_claude-skills-what.md` |
| 52 | Skillsは何のために使うのか | ⑨Claude Codeスラッシュコマンド | 下書き | | `09_claude-slash-commands/08_claude-skills-purpose.md` |
| 53 | Skillsの使い方いろいろ | ⑨Claude Codeスラッシュコマンド | 下書き | | `09_claude-slash-commands/09_claude-skills-usage.md` |
| 54 | Skillsを使う上での注意点 | ⑨Claude Codeスラッシュコマンド | 下書き | | `09_claude-slash-commands/10_claude-skills-cautions.md` |
