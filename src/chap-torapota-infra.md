---
class: chapter
---

# インフラエンジニアとして早く簡単に勉強できる！

<div class="flush-right">
    torano satoshi
</div>

## 簡単に始められる

AWS でインフラの勉強を始めるのは簡単です。

AWS アカウントを作成して、支払いの情報を設定するだけで、すぐに AWS のサービスを利用することができます。

「AWS アカウント作成の流れ」とインターネットで検索すると、AWS アカウントの作成方法が AWS さんの公式含めていっぱいでてきます。

### なぜ簡単だと、思うのか？

クラウドが選択できなかった（もしくはそんなにメジャーじゃなかった）時は、サーバーを購入して、電源を入れて、ネットワークの設定をして、OS をインストールして、ミドルウェアをインストールして、という手順が必要でした。
仮にサーバーを用意しなかったとしても、手元のパソコンをデュアルブートにして、OS をインストールする、使わないパソコンをサーバーとして構築する必要がありました。

パソコン壊れるかもしれないし、インストール OS などの準備が必要だったり、サーバーなんか置いたら、場所・音・電気代 etc 大変ですよね。

AWS は、簡単です。ブラウザが動くパソコンとインターネットがあれば、AWS アカウントを作成して、すぐに AWS のサービスを利用することができます。

### 意味を理解して作成する

AWS（インフラエンジニア向け） 初心者勉強会で良く質問するのが「どんな準備をしたら、パブリックサブネットを作成できますか？」という質問です。

この質問に完全回答できる人は、それほど多くいません。多少 AWS を触った人だったとしても、なんとなくインターネットに公開されている手順を実行していたり、インフラのベーシックな部分って既に用意されていて、その上に環境やアプリを構築することが多いからです。

パブリックサブネットを作成するために必要な準備は以下の通りです。

* VPC の作成
* インターネットゲートウェイの作成
* インターネットゲートウェイをルートに持つルートテーブルの作成
* サブネットの作成
* サブネットに上記ルートテーブルを関連付け

たったこれだけのことをするだけでも、意味を理解して進めると、ネットワークの基礎を理解することができます。

### 難しいところを後回しにできる

例えばデータベース１つとっても、詳細なパラメーターの理解をして、使い方に沿って設計をして、設計をした値を設定して、運用していく必要があります。

でも、最初からそんなことできないですよね。AWS のほとんどのサービスは、デフォルトの取り敢えずの値で起動できるようになっています。

起動しないって、ほんとに出足をくじかれます。勉強を辞めるきっかけになります。
取り敢えず起動できるから、AWS のサービスを触ってみて、どんなことができるのか？を知ることができます。

その後、必要に応じてパラメーターを理解して、設計をして、設定をして、運用して、ちょっとずつ、専門家に近づいていくことができます。

## スタンダードなインフラを作ってみる

難しいところを後回ししながらも、ウェブサービスの基盤を作ってみると、きっと以下のような構成ができると思います。

* ウェブサーバー（API Gateway（静的コンテンツを置くための S3 ））
* アプリケーションサーバー（Lambda）
* データベース（DynamoDB）

<br>

とりあえず動くので、セキュリティや拡張性を考えて構成を追加します。

* ファイヤーウォール（WAF）
* CDN（CloudFront）
* IdP（Cognito）

<br>

運用を考えて監視・バックアップ・通知系を追加します。

* 監視（CloudWatch）
* バックアップ（AWS Backup）
* 通知（SNSやQ Developer）

これだけのサービスで構成しても、モノの調達が無いから早くできます。テストで大量のトラフィックを流すとかしなければ、料金も比較的安価かなと思います。

## まとめ

例にあげただけでも、９つのジャンルについて知見をつけることができます。実際には、暗号化（KMS）やシークレット情報の管理（IAM）、権限（IAM）などもできると思います。

モノの調達もなく、AWS であれば、ハンズオンや解説ブログもインターネットを検索ると大量にでてきます。早く簡単に勉強はできるようになりました。

### 最後に

有効化するだけで高額なサービスも極わずかはあるので必ず課金のされ方や単価は調べましょう。支払い情報を登録することになるので、MFA 設定を必ず行うなど、不正利用に十分注意しましょう。

#### 著者紹介
---

<div class="author-profile">
    <img src="images/chap-torapota-infra/torano1.jpg" width="30%">
    <div>
        <div>
            <b>torano satoshi</b>
        </div>
    </div>
</div>
<p style="margin-top: 0.5em; margin-bottom: 2em;">
アプリケーションエンジニアからキャリアをはじめ、PMO、インフラエンジニア、コンサルタントなどIT業界経験20年超<br>
現在は、クラウドアーキテクトとして、困っているお客様の企画から構築まで全般的に支援しています。<br>
2024 Japan AWS All Certifications Engineer<br>
</p>
