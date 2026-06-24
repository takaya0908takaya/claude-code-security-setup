# AIコーディングを安全に使うためのセキュリティ基礎知識

Claude Code や claude.ai を使うときに、APIキーや個人情報などの機密情報をうっかり漏らさないための基礎知識をまとめたページです。専門知識がなくても読めるように、用語の説明から始めます。

> このページは「設定の手順書」ではなく「なぜそうするのか」を理解するための解説です。具体的なプラグイン導入手順は別途READMEを参照してください。

---

## 目次

1. [そもそも「秘密情報（シークレット）」とは](#1-そもそも秘密情報シークレットとは)
2. [APIキーは「値」ではなく「参照」で扱う（.envの基本）](#2-apiキーは値ではなく参照で扱うenvの基本)
3. [.gitignore で秘密情報をコミットしない](#3-gitignore-で秘密情報をコミットしない)
4. [Claude Code に機密情報を読ませない設定](#4-claude-code-に機密情報を読ませない設定)
5. [CLAUDE.md にルールを書く](#5-claudemd-にルールを書く)
6. [もしAPIキーが漏れてしまったら](#6-もしapiキーが漏れてしまったら)
7. [claude.ai（ブラウザ版）のセキュリティ対策](#7-claudeaiブラウザ版のセキュリティ対策)
8. [全体まとめ：多層防御の考え方](#8-全体まとめ多層防御の考え方)

---

## 1. そもそも「秘密情報（シークレット）」とは

外部に漏れると悪用される情報のことです。代表例は次の通りです。

- **APIキー / アクセストークン**：外部サービス（OpenAI、Stripe、AWSなど）を使うための「鍵」。漏れると他人に使われ、高額請求やデータ流出につながります。
- **パスワード / データベース接続情報**
- **秘密鍵（.pem / .key ファイルなど）**：SSH接続や暗号化に使う鍵。
- **個人情報**：氏名、住所、電話番号、メールアドレスなど。

これらをコードの中に直接書いたり、AIにそのまま渡したり、GitHubに公開してしまうのが最も危険なパターンです。

---

## 2. APIキーは「値」ではなく「参照」で扱う（.envの基本）

### 悪い例：コードに直接書く（ハードコード）

```python
# 絶対にやってはいけない書き方
api_key = "sk-abc123def456realkey"
```

これだと、コードを共有したりGitHubに上げたりした瞬間にキーが世界中に公開されます。

### 良い例：.env ファイルに分離し、コードからは「参照」する

**手順1：`.env` という名前のファイルを作り、キーの「値」だけを書く**

```bash
# .env ファイルの中身
OPENAI_API_KEY=sk-abc123def456realkey
DATABASE_URL=postgres://user:pass@localhost/mydb
```

**手順2：コードからは「名前」で読み込む（値は書かない）**

```python
# Python の例
import os
api_key = os.environ.get("OPENAI_API_KEY")  # 値ではなく名前で参照
```

```javascript
// JavaScript (Node.js) の例
const apiKey = process.env.OPENAI_API_KEY;  // 値ではなく名前で参照
```

こうすると、コード本体には「OPENAI_API_KEY という名前のものを使う」という情報しか残らず、実際のキーの値はコードに含まれません。コードを共有してもキーは漏れません。

> **ポイント**：コードに必要なのは「APIキーの使い方」であって「キーの値そのもの」ではありません。値は `.env` に隔離し、コードは名前で参照する。これが基本形です。
> （出典: [Hack-Log](https://note.com/hacklog_stealth/n/n9a482ef86561?hl=en)）

> **ただし `.env` は第一歩にすぎません**：`.env` は暗号化されていない普通のテキストファイルなので、そのPCを使える人やそのファイルを読めるツールは中身を見られます。`.env` への分離は「コード本体やGitHubへの漏洩」を防ぐためのものであり、それだけで万全ではありません。次の3つの追加対策と組み合わせて初めて実用的な安全性になります。
> - **GitHubにコミットしない**（→ セクション3）
> - **AIに読ませない設定**（→ セクション4）
> - **本物の鍵はシークレットマネージャーに置く**（→ セクション4末尾）

---

## 3. .gitignore で秘密情報をコミットしない

`.env` を作っても、それをGitHubにアップしてしまっては意味がありません。`.gitignore` というファイルに `.env` を書いておくと、Gitがそのファイルを「追跡しない（コミットに含めない）」ようになります。

**`.gitignore` の中身の例**

```bash
# 秘密情報
.env
.env.*
*.pem
*.key

# 依存ファイルやビルド成果物（おまけ）
node_modules/
.venv/
dist/
build/
.DS_Store
```

これで `git add` してもこれらのファイルは除外されます。
（出典: [Hack-Log](https://note.com/hacklog_stealth/n/n9a482ef86561?hl=en)）

> **重要な注意**：`.gitignore` はあくまで「Gitにコミットされないようにする」だけの仕組みです。**Claude Code が `.env` を読むのを防ぐ機能ではありません。** 次のセクションの設定が別途必要です。

---

## 4. Claude Code に機密情報を読ませない設定

ここは特に注意が必要なポイントです。

### よくある誤解

「`.gitignore` や `.claudeignore` に `.env` を書いておけば、Claude Codeは読まないだろう」と思いがちですが、**これは正しくありません。** 報道によると、Claude Codeは `.gitignore` や `.claudeignore` に `.env` が書かれていても、その内容を読み取ってコンソールに表示してしまうケースが確認されています。
（出典: [The Register](https://www.theregister.com/2026/01/28/claude_code_ai_secrets_files/)）

### 正しい方法：settings.json の permissions.deny を使う

公式ドキュメントが案内している確実な方法は、`settings.json` の `permissions.deny` に「読み取り禁止」ルールを書くことです。

**`~/.claude/settings.json`（全プロジェクトに適用する場合）**

```json
{
  "permissions": {
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)",
      "Read(*.pem)",
      "Read(*.key)",
      "Read(~/.ssh/**)",
      "Read(./config/credentials.*)"
    ]
  }
}
```

これを設定すると、Claude Codeはこれらのファイルの読み取りを拒否します。`~/.claude/settings.json` に書けば全プロジェクトに適用され、特定のプロジェクトだけにしたい場合はそのプロジェクトの `.claude/settings.json` に書きます。
（出典: [Claude Code公式ドキュメント](https://code.claude.com/docs/en/settings)、[Jad Joubran](https://jadjoubran.io/blog/prevent-claude-code-env)）

### さらに注意：Read を止めても Bash で読めてしまう

`Read(./.env)` で読み取りツールを禁止しても、Claudeは `cat .env` のようなBashコマンドで中身を表示できてしまいます。そのため、Bash経由のアクセスも合わせて塞ぐとより安全です。

```json
{
  "permissions": {
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Bash(cat .env*)",
      "Bash(curl *)",
      "Bash(wget *)"
    ]
  }
}
```

> permission ルールはアプリケーション層の制御であり、回避される可能性が残ります。OSレベルで隔離したい場合は、macOS/Linuxの**サンドボックス機能**を有効にすると、Bashの抜け道も塞げます。
> （出典: [DEV Community](https://dev.to/klement_gunndu/lock-down-claude-code-with-5-permission-patterns-4gcn)）

### 最も確実な方法：そもそも秘密をファイルに置かない

根本的に安全なのは、`.env` ファイルに値を書かず、**シークレットマネージャー**（AWS Secrets Manager、HashiCorp Vault、1Passwordなど）を使う方法です。秘密が実行時にだけ読み込まれてディスク上に残らなければ、ツールや攻撃者が読み取るものがそもそも存在しません。
（出典: [Jad Joubran](https://jadjoubran.io/blog/prevent-claude-code-env)、[Martin Paul Eve](https://eve.gd/2026/04/19/claude-code-can-consume-transmit-and-compromise-your-env-files-even-if-you-tell-it-not-to/)）

---

## 5. CLAUDE.md にルールを書く

`CLAUDE.md` はClaude Codeが毎回読み込む「指示書」です。秘密情報の扱い方を自然言語で書いておくと、Claudeがそれを踏まえて判断します。

```markdown
# セキュリティルール

- .env ファイルの内容をログやコンソールに出力しない
- APIキー・パスワードをコードにハードコードしない
- 環境変数は必ず process.env / os.environ から読み取る
- シークレットを含むファイルを git add しない
```

（出典: [Hack-Log](https://note.com/hacklog_stealth/n/n9a482ef86561?hl=en)）

> **注意**：`CLAUDE.md` はプロジェクトにコミットされるファイルです。**ここにAPIキーそのものを書かないでください。** あくまで「ルール（how-to）」を書く場所です。
>
> また、`CLAUDE.md` の指示はあくまでAIへのお願いであり、強制力は `permissions.deny` ほど高くありません。重要な防御は必ず `settings.json` 側で行い、`CLAUDE.md` は補助として併用してください。

---

## 6. もしAPIキーが漏れてしまったら

うっかりGitHubに上げてしまった、ログに出してしまった、という場合は、迷わず次の対応をします。

1. **すぐにキーを無効化（revoke）する**：漏れたキーを各サービスの管理画面（AWS IAM、Stripe、GitHubなど）で失効させます。
2. **新しいキーを発行（rotate）する**：新しいキーを作り直し、`.env` を更新します。
3. **アクセスログを確認する**：不審な利用がなかったか確認します。

「たぶん大丈夫」と思っても、必ずローテーションしてください。キーの作り直しは安価ですが、漏洩による被害は高額になり得ます。迷ったら作り直す、が鉄則です。
（出典: [Jad Joubran](https://jadjoubran.io/blog/prevent-claude-code-env)）

> **補足**：一度コミットした秘密情報は、ファイルを消してもGitの履歴に残り続けます。履歴からの削除は手間がかかるため、「履歴を消す」より「キーを無効化して作り直す」方が確実で速いです。

---

## 7. claude.ai（ブラウザ版）のセキュリティ対策

claude.ai はコードを実行しない代わりに、入力した内容のプライバシーが主な論点になります。

### ① 会話を学習に使わせない設定

2025年9月のポリシー変更で、消費者向けプラン（Free / Pro / Max）では設定によって会話が将来のモデル学習に使われるようになりました。気になる場合は自分でオフにします。

**設定方法**：左下のプロフィールアイコン → 設定 → プライバシー → 「Claudeの改善に協力する」をオフ
（出典: [Anthropic プライバシーセンター](https://privacy.claude.com/en/articles/12109829-how-do-i-change-my-model-improvement-privacy-settings)）

### ② 機密情報・個人情報をそのまま入力しない

入力内容はAnthropicのサーバーで処理されます。パスワードや会社の機密情報、個人情報はそのまま貼らず、必要ならダミーデータに置き換えます。

### ③ コネクター（外部アプリ連携）を見直す

GitHubやGoogleカレンダーなどとの連携が多いほど、Claudeがアクセスできるデータの範囲も広がります。使っていない連携は定期的に解除します。

### ④ 会話の共有リンクに注意

会話をURLで共有する機能は、リンクが流出すると第三者に見られる可能性があります。共有前に機密情報が含まれていないか確認します。

> **補足**：最高レベルのプライバシーを求める場合、Anthropic APIはWeb版より厳格で、データが学習に使われず、ログ保持期間も短く設定されています。

---

## 8. 全体まとめ：多層防御の考え方

セキュリティは「1つの完璧な対策」ではなく「複数の層で守る」のが基本です。どれか1つが破られても、別の層が守ってくれます。

| 層 | 対策 | 防ぐもの |
|---|---|---|
| 1 | `.env` に秘密を分離する | コードへのハードコード漏洩 |
| 2 | `.gitignore` に `.env` を追加 | GitHubへの誤コミット |
| 3 | `settings.json` の `permissions.deny` | Claude Codeによる読み取り |
| 4 | Bashコマンドのブロック / サンドボックス | 抜け道（cat等）からの読み取り |
| 5 | `CLAUDE.md` のルール | AIの判断レベルでの予防 |
| 6 | シークレットマネージャー | そもそもファイルに置かない |
| 7 | 漏洩時のキー無効化・再発行 | 漏れた後の被害拡大 |

すべてを完璧にやる必要はありませんが、最低でも **1・2・3** はどのプロジェクトでもやっておくことを推奨します。

---

## 参照元

**公式ドキュメント**
- [Claude Code Settings（permissions.deny）](https://code.claude.com/docs/en/settings)
- [Anthropic プライバシーセンター](https://privacy.claude.com/en/articles/12109829-how-do-i-change-my-model-improvement-privacy-settings)

**解説記事**
- [Jad Joubran: Prevent Claude Code from accessing .env](https://jadjoubran.io/blog/prevent-claude-code-env)
- [DEV Community: Lock Down Claude Code With 5 Permission Patterns](https://dev.to/klement_gunndu/lock-down-claude-code-with-5-permission-patterns-4gcn)
- [Hack-Log: AIコーディングエージェントでのシークレット管理](https://note.com/hacklog_stealth/n/n9a482ef86561?hl=en)

**注意喚起記事**
- [The Register: Claude Code ignores ignore rules meant to block secrets](https://www.theregister.com/2026/01/28/claude_code_ai_secrets_files/)
- [Martin Paul Eve: Claude Code can consume, transmit, and compromise your .env files](https://eve.gd/2026/04/19/claude-code-can-consume-transmit-and-compromise-your-env-files-even-if-you-tell-it-not-to/)
