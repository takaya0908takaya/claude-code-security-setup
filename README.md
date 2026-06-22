# claude-code-security-setup
# Claude Code セキュリティガイダンス導入ガイド

Anthropic公式の **Security Guidance プラグイン** を、誰でも1から導入できる手順書です。
画面の指示に沿って選択肢を選んでいくだけで完了します。

> このプラグインは無料で、すべてのClaude Codeプランで利用できます。
> （出典: [Anthropic公式ドキュメント](https://code.claude.com/docs/en/security-guidance)）

---

## このプラグインは何をするのか

Claudeが自分の書いたコードの脆弱性をその場でレビューし、同じセッション内で修正します。インジェクション、安全でないデシリアライズ、危険なDOM APIなどを、プルリクエストに到達する前に検出します。一度入れれば自動で動作し、別途コマンドを実行する必要はありません。
（出典: [Anthropic公式ドキュメント](https://code.claude.com/docs/en/security-guidance)）

### 3段階の自動レビュー

| 段階 | タイミング | 内容 | モデル使用 |
|---|---|---|---|
| 1 | ファイル編集時 | 危険なコードパターンを高速チェック | なし（コスト0） |
| 2 | ターン終了時 | 変更されたコード全体をレビュー | あり |
| 3 | コミット・プッシュ時 | 周辺コードも含めた深いレビュー | あり（最大20回/時） |

Anthropic社内のロールアウトとベンチマークでは、このプラグインを使って開いたPRのセキュリティ関連コメントが30〜40%減少したと報告されています。
（出典: [Help Net Security](https://www.helpnetsecurity.com/2026/05/27/anthropic-claude-code-security-guidance-plugin/)）

---

## 前提条件

- **Claude Code CLI 2.1.144 以降**
- **Python 3.8 以降**（PATH上にあること）

（出典: [InventiveHQ](https://inventivehq.com/blog/claude-code-security-guidance-plugin)）

確認方法：

```bash
claude --version
python3 --version
```

Claude Code がまだ入っていない場合は先に以下を実行してください。

```bash
npm install -g @anthropic-ai/claude-code
```

---

## インストール手順

### 1. Claude Code を起動する

ターミナルで次を実行します。

```bash
claude
```

> Mac標準のターミナルから起動するのが最も確実です。VS Code内のターミナルでも動作しますが、`/plugin` が使えない場合はMacのターミナルを使ってください。

---

### 2. プラグインをインストールする

Claude Codeのプロンプトが表示されたら、次を入力してEnterを押します。

```
/plugin install security-guidance@claude-plugins-official
```

> **マーケットプレイスが見つからないと表示された場合**
> 先に次を実行してから、もう一度上のコマンドを実行してください。
> ```
> /plugin marketplace add anthropics/claude-plugins-official
> ```

---

### 3. スコープ（適用範囲）を選ぶ

インストール時に適用範囲を聞かれます。**矢印キーで「user」を選んでEnter**を押してください。

> user スコープを選ぶと、このPCで起動するすべての新規セッションでプラグインが読み込まれます。
> （出典: [Anthropic公式ドキュメント](https://code.claude.com/docs/en/security-guidance)）

---

### 4. プラグインを有効化する

再起動せずに反映するため、次を入力します。

```
/reload-plugins
```

以下のように表示されれば成功です。

```
Reloaded: 1 plugin
```

---

## 動作確認

Claude Codeに危険なコードを書くよう頼んで、警告や修正提案が出るか確認します。

```
eval() を使ったコードを書いて
```

セキュリティ上の指摘が表示されれば正常に動作しています。

---

## チーム全体・リポジトリ単位で有効にする

リポジトリをクローンした全員に自動適用したい場合は、プロジェクトの `.claude/settings.json` に記述します。

```json
{
  "enabledPlugins": {
    "security-guidance@claude-plugins-official": true
  }
}
```

> 管理者は managed settings の `enabledPlugins` を設定することで、組織全体に強制適用できます。
> （出典: [Anthropic公式ドキュメント](https://code.claude.com/docs/en/security-guidance)）

---

## うまく動かないときは

プラグインは診断ログを `~/.claude/security/log.txt` に書き出します。レビューが表示されないときはまずここを確認してください。レビューが静かにスキップされる主な原因は次の通りです。

- **gitリポジトリではない**：2段階目と3段階目のレビューはgitの状態を必要とするため、リポジトリ外ではスキップされます
- **Anthropic認証がない**：モデルを使うレビューがスキップされ、1段階目のパターンチェックのみ動作します
- **`/plugin` が使えない**：VS Code拡張内ではなくMac標準のターミナルから起動してください

（出典: [Anthropic公式ドキュメント](https://code.claude.com/docs/en/security-guidance)）

---

## データの取り扱いについて

レビューの際、変更されたファイルパス・差分・関連するファイル内容がモデルのエンドポイントに送信されます。デフォルト（Anthropic API / サブスクリプション）では `api.anthropic.com` に送信され、Anthropicの商用利用規約とプライバシーポリシーの下で扱われます。
（出典: [GitHub - anthropics/claude-plugins-official](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/security-guidance)）

---

## 注意点

このプラグインは「最善の努力による補助ツール」であり、保証ではありません。検出結果は提案として扱い、人間によるコードレビュー、SAST/DAST、依存関係スキャン、ペネトレーションテストの代替とはしないでください。脆弱性を見逃したり、誤検知を出したりすることがあります。
（出典: [GitHub - anthropics/claude-plugins-official](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/security-guidance)）

---

## 参照元

- [Anthropic公式ドキュメント: Catch security issues as Claude writes code](https://code.claude.com/docs/en/security-guidance)
- [GitHub: anthropics/claude-plugins-official](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/security-guidance)
- [Help Net Security: Claude now reviews and fixes vulnerabilities as you write code](https://www.helpnetsecurity.com/2026/05/27/anthropic-claude-code-security-guidance-plugin/)
- [SecurityWeek: Anthropic Releases New Claude Sandbox, Security Guidance Plugin](https://www.securityweek.com/anthropic-releases-new-claude-sandbox-security-guidance-plugin/)
