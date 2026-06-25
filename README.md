# Claude Code セキュリティガイダンス導入ガイド

Anthropic公式の **Security Guidance プラグイン** を、はじめての人でも導入できる手順書です。
Mac・Windows両方に対応しています。上から順番に進めれば完了します。

> このプラグインは無料で、すべてのClaude Codeプランで利用できます。
> （出典: [Anthropic公式ドキュメント](https://code.claude.com/docs/en/security-guidance)）

---

## このガイドでインストールするもの（すべて公式）

このガイドでは以下をインストールします。いずれも開発元が公式に配布しているもので、配布元リンクを併記しています。コマンドを実行する前に、リンク先で公式の配布元であることを確認してください。出所が確認できないソフトやコマンドは実行しないでください。

| ソフト | 配布元 | 公式リンク |
|---|---|---|
| Claude Code | Anthropic（公式） | [公式ドキュメント](https://code.claude.com/docs/en/setup) / [GitHub](https://github.com/anthropics/claude-code) |
| Security Guidance プラグイン | Anthropic（公式マーケットプレイス） | [GitHub](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/security-guidance) |

---

## ⚠️ 最初に必ず読んでください：コマンドを入力する「2つの場所」

このガイドには2種類のコマンドが出てきます。**どちらに入力するか**を間違えると動きません。これが一番つまずきやすいポイントです。

### 場所① ターミナル（パソコンの黒い画面）

パソコンを直接操作する画面です。プロンプトの先頭は次のような見た目です。

```
ユーザー名@パソコン名 ~ %      ← Mac
C:\Users\ユーザー名>           ← Windows
```

このガイドで **`$` マーク付き、または「ターミナルで」と書かれたコマンド**はここに入力します。
例：`claude --version`、`claude`

### 場所② Claude Code の中（claudeを起動した後の画面）

`claude` と打ってEnterを押すと、Claude Codeが起動します。起動後の入力欄が「Claude Codeの中」です。

このガイドで **`/`（スラッシュ）で始まるコマンド**はここに入力します。
例：`/plugin install ...`、`/reload-plugins`

> **よくある失敗**
> - `/plugin` や `/exit` をターミナルに直接打つ → `command not found` エラーになります。これらは必ず `claude` を起動した後に入力してください。
> - 逆に `claude --version` をClaude Codeの中に打つ → 動きません。これはターミナルに入力します。

### 補足：このガイドはチャット画面（claude.ai）とは別物です

ブラウザで使うClaudeのチャット画面や、Claudeのデスクトップアプリでは `/plugin` は使えません。このガイドの作業は、必ず**ターミナルで起動したClaude Code**で行ってください。

---

## このプラグインは何をするのか

このプラグインを入れると、**Claudeが自分で書いたコードを自分でセキュリティチェックして、危険な箇所をその場で直してくれる**ようになります。問題のあるコードが完成したり、本番環境に反映されたりする前に食い止めるのが目的です。一度入れれば自動で動くので、毎回コマンドを打つ必要はありません。
（出典: [Anthropic公式ドキュメント](https://code.claude.com/docs/en/security-guidance)）

### 具体的に何を見つけてくれるのか

公式ドキュメントによると、以下のような代表的な脆弱性を検出できます。

- **インジェクション攻撃**（SQLインジェクション、コマンドインジェクションなど）
  ユーザーの入力をそのままデータベースやコマンドに渡してしまい、攻撃者に不正な操作を許してしまう問題。例：ログイン画面から全ユーザーのデータを抜き取られる。
- **認可バイパス（authorization bypass）**
  ログインや権限チェックをすり抜けられてしまう問題。例：一般ユーザーが管理者画面に入れてしまう。
- **安全でない直接オブジェクト参照（IDOR）**
  URLのIDを書き換えるだけで他人のデータが見えてしまう問題。
- **SSRF（サーバーサイドリクエストフォージェリ）**
  サーバーを踏み台にして、本来アクセスできない内部ネットワークに侵入される問題。
- **安全でないデシリアライズ**
  外部から受け取ったデータを展開する際に、悪意あるコードを実行されてしまう問題。
- **危険なDOM API**（XSSなど）
  `innerHTML` などの使い方が原因で、他人のブラウザ上で勝手にスクリプトを実行される問題。
- **弱い暗号（weak cryptography）**
  古くて破られやすい暗号方式を使ってしまう問題。

（出典: [Help Net Security](https://www.helpnetsecurity.com/2026/05/27/anthropic-claude-code-security-guidance-plugin/)、[Anthropic公式ドキュメント](https://code.claude.com/docs/en/security-guidance)）

### どのタイミングでチェックするのか（3段階）

1. **ファイルを編集した瞬間** — 危険なコードの書き方を高速でパターンチェック（AIを使わないので一瞬・追加コストなし）
2. **Claudeが応答を終えた瞬間** — そのとき変更したコード全体をAIがバックグラウンドでレビュー
3. **コミット・プッシュする瞬間** — 周辺のファイルや関連するコードまで読み込んだ、より深いレビュー

すべて自動で動くため、開発者が別途ツールを立ち上げたり、コマンドを覚えたりする必要はありません。
（出典: [Anthropic公式ドキュメント](https://code.claude.com/docs/en/security-guidance)）

### どれくらい効果があるのか

Anthropic社内のロールアウトとベンチマークでは、このプラグインを使って開いたPR（プルリクエスト）のセキュリティ関連の指摘が30〜40%減ったと報告されています。本格的なコードレビューの前に、問題を先回りして潰してくれる「最初のふるい」として機能します。
（出典: [Help Net Security](https://www.helpnetsecurity.com/2026/05/27/anthropic-claude-code-security-guidance-plugin/)）

---

## ターミナルの開き方

**Mac の場合**
1. `command + スペース` を押す
2. `ターミナル` と入力してEnter

**Windows の場合**
1. `Windowsキー` を押す
2. `PowerShell` と入力してEnter

---

## Step 1：Claude Code をインストールする 〔ターミナルで〕

お使いのOSに合わせて、以下の**公式推奨**コマンドを**ターミナル**に入力してください。コマンドはいずれも公式ドメイン `claude.ai` および [Anthropic公式GitHub](https://github.com/anthropics/claude-code) に記載されているものです。

### Mac / Linux の場合

```bash
# claude.aiから公式インストーラーをダウンロードして実行する
curl -fsSL https://claude.ai/install.sh | bash
```

または [Homebrew](https://brew.sh) を使う場合：

```bash
# Homebrew経由でClaude Codeを入れる
brew install --cask claude-code
```

### Windows の場合

PowerShellで次を実行します。

```powershell
# claude.aiから公式インストーラーをダウンロードして実行する
irm https://claude.ai/install.ps1 | iex
```

または [WinGet](https://learn.microsoft.com/windows/package-manager/) を使う場合：

```powershell
# WinGet経由でClaude Codeを入れる
winget install Anthropic.ClaudeCode
```

> **補足**：以前はnpm（`npm install -g @anthropic-ai/claude-code`）でのインストールが案内されていましたが、現在は**非推奨**です。Node.js不要・自動アップデート対応の上記公式インストーラーを使ってください。すでにnpm版を使っていても、無理に消す必要はありません。
> （出典: [Anthropic公式GitHub](https://github.com/anthropics/claude-code)）

### インストールの確認 〔ターミナルで〕

```bash
# Claude Codeのバージョンを表示する（入っていれば数字が出る）
claude --version
```

`2.1.185` のような数字が表示されればOKです。

> **必要なバージョン**：Claude Code 2.1.144 以降、Python 3.8 以降（プラグイン動作のため）
> （出典: [InventiveHQ](https://inventivehq.com/blog/claude-code-security-guidance-plugin)）

---

## Step 2：Claude Code を起動する 〔ターミナルで〕

```bash
# Claude Codeを起動する
claude
```

初回は画面の見た目（テーマ）などを聞かれます。矢印キーで好きなものを選んでEnterで進めてください。フォルダを信頼するか聞かれたら「**Yes, I trust this folder**」を選びます。

> ここから先はすべて **Claude Codeの中** での操作になります。

---

## Step 3：プラグインをインストールする 〔Claude Codeの中で〕

Claude Codeのプロンプト（入力待ちの状態）になったら、次を入力してEnterを押します。このプラグインはAnthropic公式マーケットプレイス `claude-plugins-official`（[GitHub](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/security-guidance)）で配布されています。

```
# セキュリティガイダンスのプラグインをインストールする
/plugin install security-guidance@claude-plugins-official
```

> **「マーケットプレイスが見つからない」と表示された場合**
> 先に次を入力してから、もう一度上のコマンドを入力してください。これはAnthropic公式リポジトリを登録するコマンドです。
> ```
> # 公式のプラグイン配布元を登録する
> /plugin marketplace add anthropics/claude-plugins-official
> ```

---

## Step 4：適用範囲（スコープ）を選ぶ 〔Claude Codeの中で〕

インストール時に適用範囲を聞かれます。**矢印キーで「user」を選んでEnter**を押してください。

> 「user」を選ぶと、このパソコンで起動するすべての作業でプラグインが有効になります。
> （出典: [Anthropic公式ドキュメント](https://code.claude.com/docs/en/security-guidance)）

---

## Step 5：プラグインを有効にする 〔Claude Codeの中で〕

次を入力してEnterを押します。

```
# インストールしたプラグインを読み込み直して有効にする
/reload-plugins
```

以下のように表示されれば成功です。

```
Reloaded: 1 plugin
```

おつかれさまでした。これで設定完了です。

---

## 動作確認 〔Claude Codeの中で〕

本当に動いているか確認したいときは、Claude Codeに次のように頼んでみてください。

```
eval() を使ったコードを書いて
```

セキュリティ上の注意や修正提案が表示されれば、正常に動作しています。

---

## うまく動かないときは

### インストールは成功したのに `claude` が「見つからない / 認識されません」と出る（特にWindows）

インストール時に次のような警告が出ていた場合に起こります。

```
⚠ ... is not in your PATH.
```

これは **Claude Code本体のインストールは成功しているが、その置き場所をパソコンが「コマンドを探す場所（PATH）」として認識していない** 状態です。インストーラーがPATHへの登録に失敗すると起こります（インストール自体は壊れていません）。

**Windows（PowerShell）での解決方法**

警告に表示されたパス（通常 `C:\Users\ユーザー名\.local\bin`）をPATHに登録します。

```powershell
# 探す場所のリストに Claude Code の置き場所を追加する（ユーザー名は自分のものに置き換え）
[Environment]::SetEnvironmentVariable("PATH", $env:PATH + ";C:\Users\ユーザー名\.local\bin", "User")
```

実行したら、**PowerShellを完全に閉じて開き直してから** `claude --version` を試してください。

**Mac / Linux での解決方法**

警告に表示されたパス（通常 `~/.local/bin`）をPATHに追加します。お使いのシェルに合わせて実行してください。

```bash
# zsh の場合
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc && source ~/.zshrc

# bash の場合
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc && source ~/.bashrc
```

その後 `claude --version` を試してください。

### `command not found`（`claude` や `/plugin` などで出る）
入力する場所を間違えている可能性が高いです。
- `claude` `claude --version` → **ターミナル**に入力
- `/plugin` `/reload-plugins` `/exit` など `/` で始まるもの → **Claude Codeの中**に入力（先に `claude` で起動）

（`claude` をターミナルに打っても見つからない場合は、上の「PATHに登録されていない」問題の可能性が高いです。）

### `/plugin isn't available in this environment`
VS Code内のターミナルではこのコマンドが使えないことがあります。Mac標準のターミナル、またはWindowsのPowerShellから `claude` を起動してください。一度インストールすれば設定は共通なので、その後はVS Code内でも有効になります。なお、VS Code拡張のチャット欄で `/plugins` と打つとプラグイン管理画面が開く場合もあります。

### Claude Codeを終了したい
ターミナルで `/exit` と打つのは間違いです。Claude Codeの中で `/exit` と入力するか、`Ctrl + C` を2回押すと終了できます。

### レビューが表示されない
プラグインは診断ログを `~/.claude/security/log.txt` に書き出します。なお、レビュー機能はgitで管理されたプロジェクト内でのみ完全に動作します。ホームディレクトリなどgit管理外の場所では一部の機能が動きません。
（出典: [Anthropic公式ドキュメント](https://code.claude.com/docs/en/security-guidance)）

### 古いnpm版と新しい版が混在しているかも
以前 `npm install -g @anthropic-ai/claude-code` で入れたことがある場合、古い版と新しい版が両方残っていることがあります。どの `claude` が使われているかは次で確認できます。

```bash
# Mac / Linux
which -a claude
# Windows (PowerShell)
where.exe claude
```

複数表示された場合は古い版が優先されている可能性があります。動作が不安定なら、古いnpm版を `npm uninstall -g @anthropic-ai/claude-code` で削除すると整理できます。

### （Windows）インストールコマンドが「スクリプトの実行が無効」と出て止まる
`irm https://claude.ai/install.ps1 | iex` を実行したとき、次のようなエラーが出ることがあります。

```
... cannot be loaded because running scripts is disabled on this system.
（スクリプトの実行がこのシステムで無効になっているため、読み込めません）
```

これはWindowsのPowerShellが、安全のためスクリプト実行をデフォルトで制限しているために起こります（Windowsクライアントの初期設定は「Restricted」）。次のコマンドで自分のユーザーだけスクリプト実行を許可してから、もう一度インストールコマンドを実行してください。確認を求められたら `Y` を入力します。

```powershell
# 現在のユーザーだけスクリプト実行を許可する（管理者権限は不要）
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
```

> 会社や学校のPCで、この設定がグループポリシー（GPO）で管理されている場合は、上のコマンドでも変更できないことがあります。その場合は管理者・情報システム部門に相談してください。

### 会社・学校のPCでインストールや設定変更がブロックされる
管理されたPCでは、次のような制限で手順が進まないことがあります。
- ソフトのインストール自体が管理者権限でブロックされる
- PATHの変更が許可されていない
- 社内ネットワークのプロキシやファイアウォールで `claude.ai` などへの通信が遮断され、インストーラーのダウンロードが失敗する

これらは個人では解決できないことが多いため、情報システム部門や管理者に「Claude Code（開発ツール）を導入したい」と相談するのが確実です。

### それでも解決しないとき
Claude Code本体やプラグインは頻繁に更新されており、画面表示や手順が変わることがあります。うまくいかない場合は [Claude Code公式ドキュメント](https://code.claude.com/docs/en/setup) で最新の手順を確認してください。

---

## 注意点

このプラグインは補助ツールであり、完璧ではありません。脆弱性を見逃したり、誤検知することもあります。専用のセキュリティツールや人によるコードレビューの代わりにはならないため、他の対策と組み合わせて使ってください。
（出典: [GitHub - anthropics/claude-plugins-official](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/security-guidance)）

---

## 参照元・配布元リンク一覧

**公式ドキュメント**
- [Anthropic公式: Catch security issues as Claude writes code](https://code.claude.com/docs/en/security-guidance)
- [Anthropic公式: Claude Code セットアップ手順](https://code.claude.com/docs/en/setup)

**配布元（インストールするものの出所）**
- [GitHub: anthropics/claude-code（Claude Code公式）](https://github.com/anthropics/claude-code)
- [GitHub: anthropics/claude-plugins-official（公式プラグインマーケットプレイス）](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/security-guidance)
- [Homebrew 公式サイト](https://brew.sh)

**参考記事**
- [Help Net Security: Claude now reviews and fixes vulnerabilities as you write code](https://www.helpnetsecurity.com/2026/05/27/anthropic-claude-code-security-guidance-plugin/)
- [SecurityWeek: Anthropic Releases New Claude Sandbox, Security Guidance Plugin](https://www.securityweek.com/anthropic-releases-new-claude-sandbox-security-guidance-plugin/)
