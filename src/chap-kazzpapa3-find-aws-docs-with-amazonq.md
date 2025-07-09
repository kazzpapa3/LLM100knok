---
class: chapter
---

# 生成 AI そして MCP 時代の AWS ドキュメントの探し方

一説には 6,000 ページを超えるボリュームが言われている AWS 公式ドキュメント。
膨大な情報の海から望む情報を入手することが難解なので、Amazon Q Developer for CLI と MCP というアプローチでドキュメント探しが楽になるのではないかという話です。
なお、MCP 自体についてはどなたか猛者が書いてくれると信じているのでバッサリ割愛します。

<div class="flush-right">市野 和明 @kazzpapa3</div>

## AWS 公式ドキュメントの難しさ

他の米国産パブリッククラウドと同様に AWS ドキュメントにおいては英語が公式とされています。
ただでさえ、英語圏にない日本人エンジニアにとって英語ドキュメントはハードルが高く、難解なものだと思っています。さらに 240 を超えると言われる AWS サービスにおいて個別にドキュメントが存在しており、一説には総ページ数が 6,000 ページを超えるとさえ言われています。

このようなことから難しいと言われている AWS ですが、私自身は英語であるとかドキュメント数が多いこと自体は特段難しさの原因ではないように感じています。
個人的に難しさを感じている点は「調べたい内容が公式ドキュメントにあるかどうか自体わからない」という点です。

個人的な勉強不足や知見が足りていないことの証左ですが、問い合わせいただいた内容に対して、そもそもお客様が実装しようとしていることが、その AWS サービスで実現できるのかすらわからない時があります。

そのような時に「完全一致検索」「あいまい検索」での検索ではヒットしないことがあります。このような時、検索キーワードが悪いのか、元々合致する情報がないためにヒットしないのかわからない点が文書検索で最も難しいポイントだと個人的に考えています。

## Amazon Q Developer for CLI + MCP というアプローチ

ここで世は生成 AI 時代、MCP 時代です。

冒頭で述べた通り、MCP（Model Context Protocol）自体については他の猛者が描いてくれると信じているのでバッサリ割愛します。ただ概要は Claude という LLM（Large Language Model）で有名な Anthropic 社が提唱している生成 AI と他のシステムを双方向に繋ぐためのプロトコルです。

そのプロトコルを誰がしゃべるのかというと AI エージェントと呼ばれる存在がその主語になるようです。AI エージェントは特定のタスクの遂行において人間の介在なく、自律的に行動をしてくれるものを指しています。「検索してください」と命令を与えて検索した結果を単に返すのではなく、その結果を受けて再度検索をしてくれるなど、行動と結果に応じて再度考えて（いるように振る舞って）行動する特徴があるとされています。

## やってみる

以降の操作は macOS で実施しています。

### 環境の確認

```bash
% sw_vers
ProductName:		macOS
ProductVersion:		15.3.2
BuildVersion:		24D81

% zsh --version
zsh 5.9 (arm64-apple-darwin24.0)
```

### Amazon Q Developer for CLI のインストール

執筆時点で AWS 公式ドキュメント内の URL から .dmg ファイルをダウンロードしインストールする方法、Homebrew を利用する方法の 2 つが用意されています。<span class="footnote">[Installing Amazon Q for command line](https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/command-line-installing.html) https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/command-line-installing.html </span> 

私の端末は Homebrew が導入済みの環境でしたので、以降は Homebrew を用いた方法で進めます。

```bash
% brew install amazon-q
〜中略〜
==> Downloading https://desktop-release.q.us-east-1.amazonaws.com/1.12.3/Amazon%20Q.dmg
################################################################################# 100.0%
==> Installing Cask amazon-q
==> Moving App 'Amazon Q.app' to '/Applications/Amazon Q.app'
🍺  amazon-q was successfully installed!
```

上記のように `successfully installed!` と表示されていれば正しくインストールができています。

### インストール状況の確認

```bash
% which q
/Users/work/.local/bin/q

% q --version
q 1.12.3
```

### Amazon Q の実行

インストール直後の状態では、認証が行われていないためエラーが発生します。

```bash
% q
error: You are not logged in, please log in with q login
```

`q login` コマンドで Builder ID か IAM Identity Center を用いたログインを行う必要があります。

```bash
% q login
? Select login method ›
❯ Use for Free with Builder ID
  Use with Pro license
```

`Use for Free with Builder ID` を選択します。

```bash
✔ Select login method · Use for Free with Builder ID
▰▰▰▰▱▱▱ Logging in...

# ここで Web ブラウザが開きます
# Web ブラウザで Builder ID によるログイン成功後、CLI では以下の通りの表示となります。

Device authorized
Logged in successfully
```

これでようやく Amazon Q Developer for CLI の利用準備が整いました。

### MCP の設定

AWS MCP Servers というページで AWS が用意している MCP サーバの一覧が確認できます。<span class="footnote">[AWS MCP Servers](https://awslabs.github.io/mcp/) https://awslabs.github.io/mcp/ </span> 

IAM や CloudFormation など特定の AWS サービスに特化した MCP サーバが用意されています。さらに AWS Diagram MCP Server や AWS Terraform MCP Server など特定の目的に特化した MCP サーバも用意されています。<span class="footnote">[AWS Diagram MCP Server](https://awslabs.github.io/mcp/servers/aws-diagram-mcp-server/) https://awslabs.github.io/mcp/servers/aws-diagram-mcp-server/ </span>  <span class="footnote">[AWS Terraform MCP Server](https://awslabs.github.io/mcp/servers/terraform-mcp-server/) https://awslabs.github.io/mcp/servers/terraform-mcp-server/ </span> 

今回は、この中から [AWS Documentation MCP Server](https://awslabs.github.io/mcp/servers/aws-documentation-mcp-server/) を利用します。

### MCP の設定方法

せっかくなので CLI でやってみます。`qchat` コマンドに `mcp add` というサブコマンドが用意されていますので help を活用して、必要なパラメータなどを確認してみます。

```bash
qchat mcp add --help
Add or replace a configured server

Usage: qchat mcp add [OPTIONS] --name <NAME> --command <COMMAND>

Options:
      --name <NAME>        Name for the server
      --command <COMMAND>  The command used to launch the server
      --args <ARGS>        Arguments to pass to the command
      --scope <SCOPE>      Where to add the server to [possible values: workspace, global]
      --env <ENV>          Environment variables to use when launching the server
      --timeout <TIMEOUT>  Server launch timeout, in milliseconds
      --disabled           Whether the server should be disabled (not loaded)
      --force              Overwrite an existing server with the same name
  -v, --verbose...         Increase logging verbosity
  -h, --help               Print help
```

`--scope` オプションにある通り、追加する MCP サーバをグローバルに適用するか、ワークスペース（カレントディレクトリ）に設定するかの選択ができます。カレントディレクトリの違いによって挙動が変わるか見てみたいので、`workspace` としてみます。

```bash
% mkdir -p ${HOME}/Documents/with-mcp && cd $_
% qchat mcp add \
  --name awslabs.aws-documentation-mcp-server \
  --command uvx \
  --args awslabs.aws-documentation-mcp-server@latest \
  --scope workspace

# 実行後、以下のように返却されれば正しく設定が完了しています。  
To learn more about MCP safety, see https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/command-line-mcp-security.html

✓ Added MCP server 'awslabs.aws-documentation-mcp-server' to 📄 workspace

# scope を workspace とした場合は、カレントディレクトリに .amazonq/mcp.json ファイルが作成されています
% ls -la .amazonq/
total 8
drwxr-xr-x@ 3 work  staff   96  7  9 10:53 .
drwxr-xr-x@ 3 work  staff   96  7  9 10:53 ..
-rw-r--r--@ 1 work  staff  246  7  9 14:12 mcp.json

# 前述の qchat mcp add を実行した場合 mcp.json の中身は以下のようになっています
% cat .amazonq/mcp.json
{
  "mcpServers": {
    "awslabs.aws-documentation-mcp-server": {
      "command": "uvx",
      "args": [
        "awslabs.aws-documentation-mcp-server@latest"
      ],
      "env": {},
      "timeout": 120000,
      "disabled": false
    }
  }
}
```

<div class="column">
<div class="column-title">mcp.json ファイルとして作成し MCP サーバ設定を行う方法</div>
<p>今回、CLI で MCP サーバの設定を行いましたが、mcp.json ファイルを適切に配置し、設定を行うことも可能です。</p>
<p>その際、AWS 公式ドキュメント <a href="https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/command-line-mcp-understanding-config.html">Understanding MCP configuration files</a>（ https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/command-line-mcp-understanding-config.html ） にもあるとおり、適用範囲を全てのワークスペース（Global）とするか、特定のワークスペース・ディレクトリ（Workspace）とするかを定義することが可能です。</p>

<dl>
  <dt>Global</dt><dd><code>~/.aws/amazonq/mcp.json</code> として設定・保存する</dd>
  <dt>Workspace </dt><dd>目的のディレクトリで <code>.amazonq/mcp.json</code> として設定・保存する</dd>
</dl>

<p>Global で設定されている場合、<code>qchat</code> コマンドあるいは <code>q</code> コマンドを実行して、Amazon Q Developer for CLI を呼び出した際に、設定されている全ての MCP サーバとの通信を行い利用可能とするプロセスが発生します。</p>

<p>そのため特定のプロジェクトでしか利用しない MCP サーバ設定は Workspace にするなどの使い分けをすると、よりストレスの少ない MCP 利用ができるのではないかと考えられます。</p>
</div>

### 利用してみる

#### MCP サーバの設定をしているワークスペース（ディレクトリ）で実行した場合

```bash
% pwd 
/Users/work/Documents/with-mcp
% q
✓ awslabsaws_documentation_mcp_server loaded in 1.85 s


    ⢠⣶⣶⣦⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣤⣶⣿⣿⣿⣶⣦⡀⠀
 ⠀⠀⠀⣾⡿⢻⣿⡆⠀⠀⠀⢀⣄⡄⢀⣠⣤⣤⡀⢀⣠⣤⣤⡀⠀⠀⢀⣠⣤⣤⣤⣄⠀⠀⢀⣤⣤⣤⣤⣤⣤⡀⠀⠀⣀⣤⣤⣤⣀⠀⠀⠀⢠⣤⡀⣀⣤⣤⣄⡀⠀⠀⠀⠀⠀⠀⢠⣿⣿⠋⠀⠀⠀⠙⣿⣿⡆
 ⠀⠀⣼⣿⠇⠀⣿⣿⡄⠀⠀⢸⣿⣿⠛⠉⠻⣿⣿⠛⠉⠛⣿⣿⠀⠀⠘⠛⠉⠉⠻⣿⣧⠀⠈⠛⠛⠛⣻⣿⡿⠀⢀⣾⣿⠛⠉⠻⣿⣷⡀⠀⢸⣿⡟⠛⠉⢻⣿⣷⠀⠀⠀⠀⠀⠀⣼⣿⡏⠀⠀⠀⠀⠀⢸⣿⣿
 ⠀⢰⣿⣿⣤⣤⣼⣿⣷⠀⠀⢸⣿⣿⠀⠀⠀⣿⣿⠀⠀⠀⣿⣿⠀⠀⢀⣴⣶⣶⣶⣿⣿⠀⠀⠀⣠⣾⡿⠋⠀⠀⢸⣿⣿⠀⠀⠀⣿⣿⡇⠀⢸⣿⡇⠀⠀⢸⣿⣿⠀⠀⠀⠀⠀⠀⢹⣿⣇⠀⠀⠀⠀⠀⢸⣿⡿
 ⢀⣿⣿⠋⠉⠉⠉⢻⣿⣇⠀⢸⣿⣿⠀⠀⠀⣿⣿⠀⠀⠀⣿⣿⠀⠀⣿⣿⡀⠀⣠⣿⣿⠀⢀⣴⣿⣋⣀⣀⣀⡀⠘⣿⣿⣄⣀⣠⣿⣿⠃⠀⢸⣿⡇⠀⠀⢸⣿⣿⠀⠀⠀⠀⠀⠀⠈⢿⣿⣦⣀⣀⣀⣴⣿⡿⠃
 ⠚⠛⠋⠀⠀⠀⠀⠘⠛⠛⠀⠘⠛⠛⠀⠀⠀⠛⠛⠀⠀⠀⠛⠛⠀⠀⠙⠻⠿⠟⠋⠛⠛⠀⠘⠛⠛⠛⠛⠛⠛⠃⠀⠈⠛⠿⠿⠿⠛⠁⠀⠀⠘⠛⠃⠀⠀⠘⠛⠛⠀⠀⠀⠀⠀⠀⠀⠀⠙⠛⠿⢿⣿⣿⣋⠀⠀
 ⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⠛⠿⢿⡧

╭─────────────────────────────── Did you know? ────────────────────────────────╮
│                                                                              │
│     You can resume the last conversation from your current directory by      │
│                        launching with q chat --resume                        │
│                                                                              │
╰──────────────────────────────────────────────────────────────────────────────╯

/help all commands  •  ctrl + j new lines  •  ctrl + s fuzzy search
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🤖 You are chatting with claude-4-sonnet

〜中略〜

> EC2 で StatusCheckFailed_System となる原因を解説した公式ドキュメントがあれば提示してください。

> EC2のStatusCheckFailed_Systemに関する公式ドキュメントを検索してみます。

# この後、以下のような表示がされ、数回、ドキュメント検索を自律的に行なっています
🛠️  Using tool: search_documentation from mcp server awslabsaws_documentation_mcp_server

〜中略〜

# 数度、search_documentation の操作が行われ、実行時点では以下のような回答を得ました

> EC2のStatusCheckFailed_Systemに関する公式ドキュメントを見つけました。以下が関連する主要なドキュメントです：

## EC2のStatusCheckFailed_Systemに関する公式ドキュメント

### 1. ステータスチェックの概要
URL: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/monitoring-system-instance-status-check.html

このドキュメントでは、EC2インスタンスのステータスチェックについて詳しく説明されており、StatusCheckFailed_Systemの原因について以下のように記載されています：

システムステータスチェック失敗の原因：
• ネットワーク接続の喪失
• システム電源の喪失
• 物理ホスト上のソフトウェア問題
• ネットワーク到達性に影響する物理ホスト上のハードウェア問題

### 2. トラブルシューティングガイド
URL: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/TroubleshootingInstances.html

このドキュメントでは、ステータスチェックが失敗した場合の具体的なトラブルシューティング手順が詳しく説明されています。以下のような内容が含まれています：

• システムログの取得方法
• 一般的なエラーパターンとその対処法
• メモリエラー、デバイスエラー、カーネルエラー、ファイルシステムエラーなどの具体的な解決策

### 3. ステータスチェックの確認方法
URL: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/viewing_status.html

このドキュメントでは、EC2コンソールやAWS CLIを使用してステータスチェックの結果を確認する方法が説明されています。

### 主な対処法
StatusCheckFailed_Systemが発生した場合の推奨対処法：

1. EBSバックアップインスタンスの場合：インスタンスを停止して再起動する
2. インスタンスストアバックアップインスタンスの場合：インスタンスを終了して新しいインスタンスを起動する
3. AWSによる問題解決を待つ
4. CloudWatchアラームを設定して自動復旧を行う

これらのドキュメントは、StatusCheckFailed_Systemの原因と対処法について包括的な情報を提供しており、トラブルシューティングに役立ちます。
```

途中、省略している部分がありますが、実行時、以下のような順序で `search_documentation`、`read_documentation` を繰り返して、前述の回答を出してきています。

| 呼び出された機能 | パラメータ・引数 |
| --- | --- |
| search_documentation | "search_phrase": "EC2 StatusCheckFailed_System system status check failed" |
| search_documentation | "search_phrase": "EC2 status checks system instance monitoring" |
| read_documentation | "url": "https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/viewing_status.html" |
| search_documentation | "search_phrase": "EC2 troubleshoot failed status checks system instance" |
| search_documentation | "search_phrase": "\\"TroubleshootingInstances\\" EC2 status check failed" |
| search_documentation | "search_phrase": "Troubleshoot Amazon EC2 Linux instances with failed status checks" |
| read_documentation | "url": "https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/TroubleshootingInstances.html" |
| read_documentation	| "url": "https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/TroubleshootingInstances.html" |
| search_documentation | "search_phrase": "\"Status checks for Amazon EC2 instances\" monitoring system instance" |
| search_documentation | "search_phrase": "monitoring-system-instance-status-check EC2" |
| read_documentation | "url": "https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/monitoring-system-instance-status-check.html" |

#### MCP サーバの設定をしていないワークスペース（ディレクトリ）で実行した場合

```bash
% pwd 
/Users/work/Documents/without-mcp
% q

    ⢠⣶⣶⣦⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣤⣶⣿⣿⣿⣶⣦⡀⠀
 ⠀⠀⠀⣾⡿⢻⣿⡆⠀⠀⠀⢀⣄⡄⢀⣠⣤⣤⡀⢀⣠⣤⣤⡀⠀⠀⢀⣠⣤⣤⣤⣄⠀⠀⢀⣤⣤⣤⣤⣤⣤⡀⠀⠀⣀⣤⣤⣤⣀⠀⠀⠀⢠⣤⡀⣀⣤⣤⣄⡀⠀⠀⠀⠀⠀⠀⢠⣿⣿⠋⠀⠀⠀⠙⣿⣿⡆
 ⠀⠀⣼⣿⠇⠀⣿⣿⡄⠀⠀⢸⣿⣿⠛⠉⠻⣿⣿⠛⠉⠛⣿⣿⠀⠀⠘⠛⠉⠉⠻⣿⣧⠀⠈⠛⠛⠛⣻⣿⡿⠀⢀⣾⣿⠛⠉⠻⣿⣷⡀⠀⢸⣿⡟⠛⠉⢻⣿⣷⠀⠀⠀⠀⠀⠀⣼⣿⡏⠀⠀⠀⠀⠀⢸⣿⣿
 ⠀⢰⣿⣿⣤⣤⣼⣿⣷⠀⠀⢸⣿⣿⠀⠀⠀⣿⣿⠀⠀⠀⣿⣿⠀⠀⢀⣴⣶⣶⣶⣿⣿⠀⠀⠀⣠⣾⡿⠋⠀⠀⢸⣿⣿⠀⠀⠀⣿⣿⡇⠀⢸⣿⡇⠀⠀⢸⣿⣿⠀⠀⠀⠀⠀⠀⢹⣿⣇⠀⠀⠀⠀⠀⢸⣿⡿
 ⢀⣿⣿⠋⠉⠉⠉⢻⣿⣇⠀⢸⣿⣿⠀⠀⠀⣿⣿⠀⠀⠀⣿⣿⠀⠀⣿⣿⡀⠀⣠⣿⣿⠀⢀⣴⣿⣋⣀⣀⣀⡀⠘⣿⣿⣄⣀⣠⣿⣿⠃⠀⢸⣿⡇⠀⠀⢸⣿⣿⠀⠀⠀⠀⠀⠀⠈⢿⣿⣦⣀⣀⣀⣴⣿⡿⠃
 ⠚⠛⠋⠀⠀⠀⠀⠘⠛⠛⠀⠘⠛⠛⠀⠀⠀⠛⠛⠀⠀⠀⠛⠛⠀⠀⠙⠻⠿⠟⠋⠛⠛⠀⠘⠛⠛⠛⠛⠛⠛⠃⠀⠈⠛⠿⠿⠿⠛⠁⠀⠀⠘⠛⠃⠀⠀⠘⠛⠛⠀⠀⠀⠀⠀⠀⠀⠀⠙⠛⠿⢿⣿⣿⣋⠀⠀
 ⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⠛⠿⢿⡧

╭─────────────────────────────── Did you know? ────────────────────────────────╮
│                                                                              │
│      Set a default model by running q settings chat.defaultModel MODEL.      │
│                          Run /model to learn more.                           │
│                                                                              │
╰──────────────────────────────────────────────────────────────────────────────╯

/help all commands  •  ctrl + j new lines  •  ctrl + s fuzzy search
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🤖 You are chatting with claude-4-sonnet

〜中略〜

> EC2 で StatusCheckFailed_System となる原因を解説した公式ドキュメントがあれば提示してください。

> EC2のStatusCheckFailed_Systemに関する公式ドキュメントをご案内します。

AWSの公式ドキュメントでは、以下のリソースでStatusCheckFailed_Systemについて詳しく解説されています：

主要な公式ドキュメント：

1. EC2 インスタンスのステータスチェック
   • URL: https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/monitoring-system-instance-status-
check.html
   • システムステータスチェックの詳細な説明と原因が記載されています

2. CloudWatch メトリクス - EC2
   • URL: https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/ec2-metricscollected.html
   • StatusCheckFailed_Systemメトリクスの詳細が説明されています

StatusCheckFailed_Systemの主な原因：
• ハードウェアの問題
• ネットワーク接続の問題
• システムの電源問題
• ソフトウェアの問題（ホストシステム側）

対処方法：
• インスタンスの停止と開始（再起動ではなく）
• インスタンスの別のハードウェアへの移行
• AWSサポートへの連絡

これらの公式ドキュメントで、StatusCheckFailed_Systemの詳細な原因と対処方法を確認できます。特に最初のリンクが最も包括的な情報を提供しています。
```

まず、冒頭で MCP サーバと通信しておらず定義されていないことがわかります。

MCP サーバとの対話をしないので、回答は素早く得られましたが、2 つ目の URL は CloudWatch メトリクスを発行する AWS サービスの一覧ページとなっています。そのページでは、Amazon EC2 が CloudWatch メトリクスを発行すること、そして詳細ページへのリンクは書かれています。ただ、ページ内で StatusCheckFailed_System メトリクスの詳細までは言及されていません。

## まとめ

Amazon Q Developer for CLI と AWS Documentation MCP Server を連携する様子を見ました。
自律的な検索をした上でより適切な AWS ドキュメントを返却してくれる例から、よりふさわしそうな検索キーワードを数回試してドキュメントを探り当ててきてくれている様子が伝わったのではないでしょうか。

冒頭、文書検索において「調べたい内容が公式ドキュメントにあるかどうか自体わからない」という課題があると述べました。
生成 AI の回答を 100％ 鵜呑みにすることはまだまだハードルが高いですが、AWS ドキュメント探しの良い伴走者になってくれるのではないかと期待できます。

#### 著者紹介

---

<div class="author-profile">
    <img src="images/kazzpapa3.jpg">
    <div>
        <div>
            <b>市野 和明</b> <a href="https://x.com/kazzpapa3">@kazzpapa3</a>
        </div>
        <div>
            所属：<a href="https://jawsug-kobe.connpass.com/">JAWS-UG 神戸</a>
        </div>
    </div>
</div>
<p style="margin-top: 0.5em; margin-bottom: 2em;">
とある AWS パートナー企業のテクニカルサポート部門で働いているテクニカルサポートエンジニアです。<br>
AWS パートナーで利用者と AWS テクニカルサポートの中間地点に立って、いただく問い合わせに向かう際に、AWS テクニカルサポートが当たり前と思っていることが利用者にはそう捉えられていないと感じる点や、それを踏まえて少し問い合わせ方法を改善いただくことで課題解決が早まりそうだな、などと思いながら仕事をしている気づきを JAWS を中心にお話ししていたりします。
</p>