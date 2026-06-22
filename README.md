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

Claudeが書いたコードの脆弱性をその場でチェックし、同じ作業の流れの中で修正してくれます。SQLインジェクションや危険なコードの書き方などを、本番に反映される前に見つけてくれます。一度入れれば自動で動くので、毎回コマンドを打つ必要はありません。
（出典: [Anthropic公式ドキュメント](https://code.claude.com/docs/en/security-guidance)）

Anthropic社内では、このプラグインを使ったPRのセキュリティ関連の指摘が30〜40%減ったと報告されています。
（出典: [Help Net Security](https://www.helpnetsecurity.com/2026/05/27/anthropic-claude-code-security-guidance-plugin/)）

---

## ターミナルの開き方

**Mac の場合**
1. `command + スペース` を押す
2. `ターミナル` と入力してEnter

**Windows の場合**
1. `Windowsキー` を押す
2. `PowerShell` と入力してEnter

> **Pythonの仮想環境（.venv）に入っている場合**
> プロンプトの先頭に `(.venv)` などと表示されていることがあります。この状態でも以下の手順は問題なく進められますが、気になる場合は `deactivate` と入力すると仮想環境から抜けられます。

---

## Step 1：Claude Code をインストールする 〔ターミナルで〕

お使いのOSに合わせて、以下の**公式推奨**コマンドを**ターミナル**に入力してください。コマンドはいずれも公式ドメイン `claude.ai` および [Anthropic公式GitHub](https://github.com/anthropics/claude-code) に記載されているものです。

### Mac / Linux の場合

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

または [Homebrew](https://brew.sh) を使う場合：

```bash
brew install --cask claude-code
```

### Windows の場合

PowerShellで次を実行します。

```powershell
irm https://claude.ai/install.ps1 | iex
```

または [WinGet](https://learn.microsoft.com/windows/package-manager/) を使う場合：

```powershell
winget install Anthropic.ClaudeCode
```

> **補足**：以前はnpm（`npm install -g @anthropic-ai/claude-code`）でのインストールが案内されていましたが、現在は**非推奨**です。Node.js不要・自動アップデート対応の上記公式インストーラーを使ってください。すでにnpm版を使っていても、無理に消す必要はありません。
> （出典: [Anthropic公式GitHub](https://github.com/anthropics/claude-code)）

### インストールの確認 〔ターミナルで〕

```bash
claude --version
```

`2.1.185` のような数字が表示されればOKです。

> **必要なバージョン**：Claude Code 2.1.144 以降、Python 3.8 以降（プラグイン動作のため）
> （出典: [InventiveHQ](https://inventivehq.com/blog/claude-code-security-guidance-plugin)）

---

## Step 2：Claude Code を起動する 〔ターミナルで〕

```bash
claude
```

初回は画面の見た目（テーマ）などを聞かれます。矢印キーで好きなものを選んでEnterで進めてください。フォルダを信頼するか聞かれたら「**Yes, I trust this folder**」を選びます。

> ここから先はすべて **Claude Codeの中** での操作になります。

---

## Step 3：プラグインをインストールする 〔Claude Codeの中で〕

Claude Codeのプロンプト（入力待ちの状態）になったら、次を入力してEnterを押します。このプラグインはAnthropic公式マーケットプレイス `claude-plugins-official`（[GitHub](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/security-guidance)）で配布されています。

```
/plugin install security-guidance@claude-plugins-official
```

> **「マーケットプレイスが見つからない」と表示された場合**
> 先に次を入力してから、もう一度上のコマンドを入力してください。これはAnthropic公式リポジトリを登録するコマンドです。
> ```
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

### `command not found`（`claude` や `/plugin` などで出る）
入力する場所を間違えている可能性が高いです。
- `claude` `claude --version` → **ターミナル**に入力
- `/plugin` `/reload-plugins` `/exit` など `/` で始まるもの → **Claude Codeの中**に入力（先に `claude` で起動）

### `/plugin isn't available in this environment`
VS Code内のターミナルではこのコマンドが使えないことがあります。Mac標準のターミナル、またはWindowsのPowerShellから `claude` を起動してください。一度インストールすれば設定は共通なので、その後はVS Code内でも有効になります。

### Claude Codeを終了したい
ターミナルで `/exit` と打つのは間違いです。Claude Codeの中で `/exit` と入力するか、`Ctrl + C` を2回押すと終了できます。

### レビューが表示されない
プラグインは診断ログを `~/.claude/security/log.txt` に書き出します。なお、レビュー機能はgitで管理されたプロジェクト内でのみ完全に動作します。ホームディレクトリなどgit管理外の場所では一部の機能が動きません。
（出典: [Anthropic公式ドキュメント](https://code.claude.com/docs/en/security-guidance)）

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
```bash
brew install --cask claude-code
```

### Windows の場合

PowerShellで次を実行します。

```powershell
irm https://claude.ai/install.ps1 | iex
```

または [WinGet](https://learn.microsoft.com/windows/package-manager/) を使う場合：

```powershell
winget install Anthropic.ClaudeCode
```

> **補足**：以前はnpm（`npm install -g @anthropic-ai/claude-code`）でのインストールが案内されていましたが、現在は**非推奨**です。Node.js不要・自動アップデート対応の上記公式インストーラーを使ってください。
> （出典: [Anthropic公式GitHub](https://github.com/anthropics/claude-code)）

### インストールの確認

```bash
claude --version
```

`2.1.185` のような数字が表示されればOKです。

> **必要なバージョン**：Claude Code 2.1.144 以降、Python 3.8 以降（プラグイン動作のため）
> （出典: [InventiveHQ](https://inventivehq.com/blog/claude-code-security-guidance-plugin)）

---

## Step 2：Claude Code を起動する

```bash
claude
```

初回は画面の見た目（テーマ）などを聞かれます。矢印キーで好きなものを選んでEnterで進めてください。フォルダを信頼するか聞かれたら「**Yes**」を選びます。

---

## Step 3：プラグインをインストールする

Claude Codeのプロンプト（入力待ちの状態）になったら、次を入力してEnterを押します。このプラグインはAnthropic公式マーケットプレイス `claude-plugins-official`（[GitHub](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/security-guidance)）で配布されています。

```
/plugin install security-guidance@claude-plugins-official
```

> **「マーケットプレイスが見つからない」と表示された場合**
> 先に次を実行してから、もう一度上のコマンドを実行してください。これはAnthropic公式リポジトリを登録するコマンドです。
> ```
> /plugin marketplace add anthropics/claude-plugins-official
> ```

---

## Step 4：適用範囲（スコープ）を選ぶ

インストール時に適用範囲を聞かれます。**矢印キーで「user」を選んでEnter**を押してください。

> 「user」を選ぶと、このパソコンで起動するすべての作業でプラグインが有効になります。
> （出典: [Anthropic公式ドキュメント](https://code.claude.com/docs/en/security-guidance)）

---

## Step 5：プラグインを有効にする

次を入力してEnterを押します。

```
/reload-plugins
```

以下のように表示されれば成功です。

```
Reloaded: 1 plugin
```

おつかれさまでした。これで設定完了です。

---

## 動作確認

本当に動いているか確認したいときは、Claude Codeに次のように頼んでみてください。

```
eval() を使ったコードを書いて
```

セキュリティ上の注意や修正提案が表示されれば、正常に動作しています。

---

## うまく動かないときは

### `claude: command not found`
Step 1に戻ってClaude Codeをインストールしてください。インストール後はターミナルを開き直す必要があります。

### `/plugin isn't available in this environment`
VS Code内のターミナルではこのコマンドが使えないことがあります。Mac標準のターミナル、またはWindowsのPowerShellから `claude` を起動してください。一度インストールすれば設定は共通になります。

### レビューが表示されない
プラグインは診断ログを `~/.claude/security/log.txt` に書き出します。なお、レビュー機能はgitで管理されたプロジェクト内でのみ完全に動作します。
（出典: [Anthropic公式ドキュメント](https://code.claude.com/docs/en/security-guidance)）

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
