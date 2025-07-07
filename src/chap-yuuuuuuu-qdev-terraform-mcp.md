---
class: chapter
---

# Amazon Q Developerの脆弱性診断(その② AWS Terraform MCP Server編)

<div class="flush-right">
Yuuuuuuu
</div>

## はじめに
皆さんTerraform書いてますか？(以下略)

本章は"その① /review編"の続編として設定しています。
この章では、Amazon Q Developerを拡張する機能であるMCPを、そして、AWS公式から提供されているAWS Terraform MCP Serverを用いて脆弱性診断を試してみます。

## AWS Terraform MCP Serverが提供している機能
- Terraformベストプラクティス
    - AWS上でアプリケーションを構築するための規範的なTerraformアドバイスを提示します。
- セキュリティファースト開発ワークフロー
    - セキュアなコードを作成するための構造化されたプロセスを提供します。
- **Checkov統合** ←今回のメイン
    - セキュリティとコンプライアンススキャンのためのCheckovと連携して脆弱性診断を行います。
- AWSプロバイダードキュメント
    - AWSおよびAWSCCプロバイダーリソースの検索してくれます。
- AWS-IA GenAIモジュール
    - AI/MLワークロード用の専用モジュールへのアクセスを行います。
- Terraform Registryモジュール分析
    - Terraform Registryモジュールの分析を行います。
- Terraformワークフロー実行
    - Terraformコマンドの直接実行してくれます。
- Terragruntワークフロー実行
    - Terragruntコマンドの直接実行してくれます。

AWS MCP Servers、並びにAWS Terraform MCP Serverの詳細は以下をご覧ください。

https://github.com/awslabs/mcp

https://github.com/awslabs/mcp/tree/main/src/terraform-mcp-server

### Checkovとは
今回メインで使用するCheckovについても触れておきます。
CheckovはInfrastructure as Code（IaC）とソフトウェア構成分析（SCA）のための静的コード解析ツールです。
今回のターゲットであるTerraformをはじめ、CloudFormation、Kubernetes、Helmなどのクラウドインフラストラクチャ設定をスキャンして、デプロイ前に設定ミスを発見することができます。
GitHub ActionsなどのCI/CDツールもサポートしており、CheckovでOKとなったコードのみをデプロイする、といった分岐に組み込むことができます。

今回は、MCPサーバの機能として利用してみます。

Checkovの公式ページ

https://www.checkov.io/

## 脆弱なコードの紹介

本章でも、以下のTerraformコードと、おまけのLambda用の脆弱なPythonコード(ZIP済み)を対象とします。

https://github.com/yuuuuuuu168/terraform-vulnerability-examples

## Let's Checkov
1. AWS Terraform MCP ServerをAmazon Q Developerに登録します。
    以下設定を使用しています。
    ![Amazon_Q_Developer-MCP-setting](./src/images/chap-yuuuuuuu-qdev-terraform-mcp/Amazon_Q_Developer-MCP-setting.png)

2. プロンプトで脆弱性チェックを依頼します。

    プロンプト
    ```
    ワークスペースに配置してあるファイルに脆弱性が含まれていないかチェックしてください
    ```
    

3. まだ途中のプロンプトですが...以下のように"Checkovを使用"という宣言を確認することができます。
    
    ![First_context](./src/images/chap-yuuuuuuu-qdev-terraform-mcp/First_context.png)

4. "多数の重大なセキュリティ脆弱性"が発見されました。
   なお、以下のようにこういう点でまずい！という解説はありますが、具体的にどの部分のコードがまずいのか？という説明はありませんでした。
   
    ![Scan1](./src/images/chap-yuuuuuuu-qdev-terraform-mcp/Scan1.png)
    ![Scan2](./src/images/chap-yuuuuuuu-qdev-terraform-mcp/Scan2.png)
    ![Scan3](./src/images/chap-yuuuuuuu-qdev-terraform-mcp/Scan3.png)

5. ということで、具体的にコードのどの部分がまずいのか？と聞いてみると、該当箇所とそこをどう修正したらよいかの提案をしてくれました。
    "+-"で前後の比較もわかりやすく、問題箇所やどう編集したら良いか判断がつきやすいようになっています。

    以下、S3のブロックパブリックアクセスと暗号化の例

    ![Diff](./src/images/chap-yuuuuuuu-qdev-terraform-mcp/Diff.png)

## Checkov結果について
前章で記載した通り、今回、25個の脆弱性を仕込んでいました。
このうち、AWS Terraform MCP Serverによって指摘を受けたのは22個でした。

Amazon Q Developerの"/review"と比較すると、例えば前章で指摘して欲しかったと記載したEC2インスタンスのIMDS V2の利用を指摘してくれました。
上記の他、Amazon Q Developerが指摘して、AWS Terraform MCP Serverの指摘漏れ...というものもありませんでした。
指摘できなかったものの例として、例えばSecrets Managerのシークレットのローテーションをしていない、というものです。
あまりSecrets Managerを使わないため、これがどの程度脆弱なのかを指摘することができないのですが、そこまで指摘するほどではない...ということなのでしょうか(まあ、ローテーションをしないのにSecrets Managerを使うなよ、とは思いますけど)。

また、Amazon Q Developer同様に、zip化されたLambdaコードに関しての指摘もありませんでした。
これは、**Terraform**のMCPサーバだから、というのもあるのでしょうか。

## おわりに
あくまで今回使用したTerraformコードに対して、また今回何度かAIを回してその平均値をとるetcはしていないため、確実なことは言えませんが以下の傾向を確認しました

Amazon Q Developerは、指摘事項がグラフィカルに表示されており、また、その脆弱性の危険度が4段階で示されていることから、何から対処すべきかの判断がしやすいように感じます。一方で、指摘事項としては今一歩な感覚でした。

AWS Terraform MCP Serverは脆弱性の指摘は的確で、個人的に必須と考えていた脆弱性は全て指摘することができていました。一方で、全ての脆弱性が一辺倒にリストされており、どれから対応すべきか(もちろん今回の脆弱性は割とどれも対処が必須級ではありますが)が分かりにくかったです。また、それがなぜ問題なのか、どう対処すべきかが追加の問いをしなければならなかった点もうーんでした(これも最初のコンテキストで依頼すれば含まれたかもしれません)。

上記のため、例えば以下の流れで脆弱性スキャンを行う、というのが良いかと考えました。
1. Amazon Q Developerの"/review"でおおまかな脆弱性を潰す
2. AWS Terraform MCP Serverでその他脆弱性が残っていないか最終チェックを行う

それぞれの機能を把握して、うまく、上手に使っていきましょう。
AIを使いこなす技量、大事ですよ！

#### 著者紹介
---

<div class="author-profile">
    <img src="src/images/chap-yuuuuuuu-qdev-review/yuuuuuuu_icon.png" width="30%">
    <div>
        <div>
            <b>Yuuuuuuu</b>
            @ec2_on_aws
        </div>
    </div>
</div>
<p style="margin-top: 0.5em; margin-bottom: 2em;">
趣味はAWS/Terraformに挑戦中 </br>
AWS15冠(2024-2025)、Terraform1つ、OCI2冠、Azure1つ、Databricks1つ </br>
目指せCommunity Builder & Top Engineer!! </br>
</p>