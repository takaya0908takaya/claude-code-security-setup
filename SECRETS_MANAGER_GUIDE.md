# シークレットマネージャー導入ガイド

APIキーやパスワードを `.env` に平文で置くのをやめ、**シークレットマネージャー**で安全に管理するための実践的な導入手順です。

> このガイドは、別途用意した「[セキュリティ基礎知識](./SECURITY_BASICS.md)」の発展編です。基礎（`.env`への分離・`.gitignore`・Claude Codeへの読み取り禁止）を理解している前提で、「本物の鍵をそもそもファイルに置かない」ための具体的な方法を扱います。

---

## 目次

1. [なぜシークレットマネージャーが必要か](#1-なぜシークレットマネージャーが必要か)
2. [どれを選ぶべきか](#2-どれを選ぶべきか)
3. [1Password CLI の導入（ローカル開発向け・おすすめ）](#3-1password-cli-の導入ローカル開発向けおすすめ)
4. [AWS Secrets Manager の導入（AWS本番向け）](#4-aws-secrets-manager-の導入aws本番向け)
5. [Google Secret Manager の導入（GCP本番向け）](#5-google-secret-manager-の導入gcp本番向け)
6. [HashiCorp Vault の導入（チーム・大規模向け）](#6-hashicorp-vault-の導入チーム大規模向け)
7. [導入後のチェックリスト](#7-導入後のチェックリスト)

---

## 1. なぜシークレットマネージャーが必要か

`.env` ファイルは暗号化されていない平文のテキストです。そのため次のリスクが残ります。

- うっかり `git add .` してGitHubに漏れる
- PCを触れる人やツール（AIエージェント含む）に中身を読まれる
- バックアップやログに紛れ込む

シークレットマネージャーを使うと、**本物の鍵は専用の金庫に保管され、`.env` には「金庫のどこにあるか」という参照（住所）だけを書きます。** 実行時にだけ金庫から値を取り出してメモリに展開するため、ディスク上に本物の鍵が残りません。
（出典: [Jakob Osterberger](https://www.jakobosterberger.com/posts/1password-cli-ddev-env/)）

---

## 2. どれを選ぶべきか

| 状況 | おすすめ | 理由 |
|---|---|---|
| 個人開発・ローカル中心 | **1Password CLI** | 導入が簡単。既に1Passwordを使っていれば即始められる |
| 小〜中規模チーム | 1Password CLI | 共有vaultでチーム配布が楽 |
| AWS上の本番アプリ | **AWS Secrets Manager** | AWSと統合され自動ローテーション対応 |
| Google Cloud上の本番 | **Google Secret Manager** | GCPと統合 |
| 大規模・複数クラウド・厳格な要件 | **HashiCorp Vault** | 最も高機能だが運用コスト高 |

迷ったら、ローカル開発は **1Password CLI**、本番は使っているクラウドのマネージャー、という組み合わせが標準的です。

---

## 3. 1Password CLI の導入（ローカル開発向け・おすすめ）

### 前提
- 1Passwordアカウント（個人またはチーム）
- [1Password CLI](https://developer.1password.com/docs/cli/get-started/) のインストール

### 手順1：CLIをインストールしてサインインする

**Mac の場合（Homebrew）**

```bash
# 1Password CLI をインストール
brew install 1password-cli
```

**Windows の場合（winget）**

```powershell
# 1Password CLI をインストール
winget install 1password-cli
```

**Linux の場合（Debian / Ubuntu）**

公式リポジトリを登録してインストールします。コマンドは [1Password公式ドキュメント](https://developer.1password.com/docs/cli/get-started/) に記載のものです。

```bash
curl -sS https://downloads.1password.com/linux/keys/1password.asc | \
  sudo gpg --dearmor --output /usr/share/keyrings/1password-archive-keyring.gpg && \
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/1password-archive-keyring.gpg] https://downloads.1password.com/linux/debian/$(dpkg --print-architecture) stable main" | \
  sudo tee /etc/apt/sources.list.d/1password.list && \
sudo apt update && sudo apt install 1password-cli
```

> Linuxのその他のディストリビューション（Fedora/RHEL系など）や手動インストールの方法は[公式ドキュメント](https://developer.1password.com/docs/cli/get-started/)を参照してください。

インストールできたか確認します（全OS共通）。

```bash
# バージョンが表示されればOK
op --version
```

アカウントを追加してサインインします（全OS共通）。

```bash
# アカウントを登録
op account add

# サインイン（このセッションで有効なトークンが発行される）
eval $(op signin)
```

> `eval $(op signin)` は現在のシェルに認証トークンをセットします。セッションごとに再実行が必要ですが、Touch IDなどの生体認証を設定すると自動化できます。Windowsの場合は `op signin` を実行します。
> （出典: [Jakob Osterberger](https://www.jakobosterberger.com/posts/1password-cli-ddev-env/)、[1Password公式ドキュメント](https://developer.1password.com/docs/cli/get-started/)）

### 手順2：1Passwordに秘密を保存する

1Passwordアプリで、APIキーをアイテムとして保存します（例：vault名「Private」、アイテム名「OpenAI」、フィールド「credential」）。

### 手順3：.env の値を「参照」に書き換える

```bash
# 書き換え前（危険：本物の値）
OPENAI_API_KEY=sk-abc123realkey

# 書き換え後（安全：参照だけ）
OPENAI_API_KEY=op://Private/OpenAI/credential
```

参照は `op://vault名/アイテム名/フィールド名` の形式です。正確なフィールド名は、1Passwordデスクトップアプリでフィールドを右クリックして「シークレットリファレンスをコピー」で取得できます。
（出典: [NSHipster](https://nshipster.com/1password-cli/)）

### 手順4：実行時に値を注入する

```bash
# .env の参照を実際の値に置き換えてアプリを起動する
op run --env-file=.env -- python main.py
```

`op run` は `.env` の `op://` 参照を解決し、本物の値をそのプロセスの環境変数に注入してからアプリを起動します。値はそのプロセスのメモリ上にだけ存在し、ディスクには書き込まれません。
（出典: [Chainlink Documentation](https://docs.chain.link/cre/guides/workflow/secrets/managing-secrets-1password)）

### おまけ：参照だけになった .env はコミットできる

`op://` 参照は秘密情報を含まない「ただの住所」なので、この `.env` はGitにコミットしても安全です。チームメンバーは同じ参照ファイルを共有し、各自の1Passwordから値を取得できます。ただしグローバルな `.gitignore` で `.env` が除外されている場合は、明示的に許可する必要があります。

```bash
# グローバル .gitignore の .env 除外を上書きして、参照版だけ許可する
!.env
```

（出典: [NSHipster](https://nshipster.com/1password-cli/)）

> **注意**：これは「参照だけが書かれた `.env`」に限った話です。本物の値が書かれた `.env` は絶対にコミットしないでください。

---

## 4. AWS Secrets Manager の導入（AWS本番向け）

### 考え方
AWS上で動くアプリなら、`.env` を使わず、コードが起動時にAWSから直接秘密を取得します。AWSの認証（IAMロール）でアクセス制御するため、鍵を持ち歩く必要がありません。

### 手順0：AWS CLI をインストールする

操作には AWS CLI が必要です。コマンドは [AWS公式ドキュメント](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) に記載のものです。

**Mac の場合**

```bash
# 公式インストーラーを取得して実行
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
sudo installer -pkg AWSCLIV2.pkg -target /
```

または Homebrew：

```bash
brew install awscli
```

**Windows の場合（winget）**

```powershell
winget install Amazon.AWSCLI
```

**Linux の場合**

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

インストール後、認証情報を設定します（全OS共通）。

```bash
# AWSアカウントの認証情報を設定する
aws configure
```

### 手順1：秘密を登録する

```bash
# シークレットを作成する
aws secretsmanager create-secret \
  --name prod/myapp/openai \
  --secret-string '{"OPENAI_API_KEY":"sk-abc123realkey"}'
```

### 手順2：コードから取得する

```python
import boto3
import json

# 起動時にAWSから取得
client = boto3.client("secretsmanager")
secret = client.get_secret_value(SecretId="prod/myapp/openai")
api_key = json.loads(secret["SecretString"])["OPENAI_API_KEY"]
```

### メリット
- **自動ローテーション**：定期的にキーを自動で再発行できる
- **アクセス制御**：IAMで「誰が・どの秘密に」アクセスできるか細かく制御できる
- **監査ログ**：CloudTrailで誰がいつアクセスしたか記録される

---

## 5. Google Secret Manager の導入（GCP本番向け）

### 手順0：gcloud CLI をインストールする

操作には gcloud CLI が必要です。コマンドは [Google Cloud公式ドキュメント](https://cloud.google.com/sdk/docs/install) に記載のものです。

**Mac の場合（Homebrew）**

```bash
brew install --cask google-cloud-sdk
```

**Windows の場合**

公式インストーラー（[GoogleCloudSDKInstaller.exe](https://cloud.google.com/sdk/docs/install)）をダウンロードして実行するのが確実です。または winget：

```powershell
winget install Google.CloudSDK
```

**Linux の場合（Debian / Ubuntu）**

```bash
curl https://sdk.cloud.google.com | bash
exec -l $SHELL
```

インストール後、ログインします（全OS共通）。

```bash
# Googleアカウントでログインする
gcloud auth login
```

### 手順1：秘密を登録する

```bash
# シークレットを作成し、値を登録する
echo -n "sk-abc123realkey" | gcloud secrets create openai-api-key --data-file=-
```

### 手順2：コードから取得する

```python
from google.cloud import secretmanager

client = secretmanager.SecretManagerServiceClient()
name = "projects/PROJECT_ID/secrets/openai-api-key/versions/latest"
response = client.access_secret_version(request={"name": name})
api_key = response.payload.data.decode("UTF-8")
```

GCPもAWS同様、IAMによるアクセス制御と監査ログに対応しています。

---

## 6. HashiCorp Vault の導入（チーム・大規模向け）

最も高機能ですが、Vaultサーバー自体を運用する必要があるため、導入コストは高めです。複数のクラウドをまたぐ、厳格なコンプライアンス要件がある、といった場合に検討します。

### CLIのインストール

コマンドは [HashiCorp公式ドキュメント](https://developer.hashicorp.com/vault/install) に記載のものです。

**Mac の場合（Homebrew）**

```bash
brew tap hashicorp/tap
brew install hashicorp/tap/vault
```

**Windows の場合（winget）**

```powershell
winget install Hashicorp.Vault
```

**Linux の場合（Debian / Ubuntu）**

```bash
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install vault
```

### 基本的な流れ

```bash
# 秘密を保存する
vault kv put secret/myapp openai_api_key=sk-abc123realkey

# 秘密を取得する
vault kv get -field=openai_api_key secret/myapp
```

アプリからは公式SDK（各言語向け）やAPIを通じて取得します。動的シークレット（使うたびに有効期限付きの一時的な認証情報を発行する）など、高度な機能が特徴です。小〜中規模であれば、まずは1Passwordやクラウドのマネージャーで十分なことが多いです。

---

## 7. 導入後のチェックリスト

- [ ] 本物の鍵がシークレットマネージャーに保存されている
- [ ] `.env` には参照（住所）だけ、または `.env` 自体を廃止した
- [ ] 古い「本物の値が書かれた `.env`」を削除した
- [ ] その鍵が過去にGitにコミットされていないか確認した（されていたらキーを再発行）
- [ ] チーム開発の場合、メンバー全員が金庫へのアクセス権を持っている
- [ ] 本番環境では自動ローテーションを検討した

---

## 参照元

**公式ドキュメント**
- [1Password CLI: run コマンド](https://developer.1password.com/docs/cli/reference/commands/run/)
- [1Password CLI: Get started](https://developer.1password.com/docs/cli/get-started/)
- [AWS Secrets Manager ドキュメント](https://docs.aws.amazon.com/secretsmanager/)
- [Google Secret Manager ドキュメント](https://cloud.google.com/secret-manager/docs)
- [HashiCorp Vault ドキュメント](https://developer.hashicorp.com/vault/docs)

**解説記事**
- [NSHipster: op run](https://nshipster.com/1password-cli/)
- [Jakob Osterberger: Secrets-free .env with 1Password CLI](https://www.jakobosterberger.com/posts/1password-cli-ddev-env/)
- [Chainlink: Managing Secrets with 1Password CLI](https://docs.chain.link/cre/guides/workflow/secrets/managing-secrets-1password)
