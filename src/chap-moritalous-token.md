---
class: chapter
---

# トークンとは

<div class="flush-right">
森田 和明
</div>

LLMは与えられたテキストの「もっともらしい続き」を生成することで文章を生成しています。チャットでのやり取りもコード生成も、直前までのテキストの続きを生成しているに過ぎません。
続きを予想する過程で、テキストを小さな単位に分割して処理を行っており、この分割したテキストが「トークン」です。

## トークンに分割してみる

トークンへの分割はトークナイザーと呼ばれるライブラリーで行います。代表的なトークナイザーであるTokenizer（名称がそのまんまですねw）を使用して、実際にテキストをトークンに分割してみましょう。

まずTokenizerライブラリーを取得します。

```shell
pip install tokenizers
```

続いてトークンに分割するスクリプトを作成します。
トークン`from_pretrained`のパラメーターとして使用するLLMを指定します。これは、LLMごとにトークンへの分割ロジックが異なるためです。

テキストを"|"で区切って出力するように実装しました。

```shell
from tokenizers import Tokenizer

tokenizer = Tokenizer.from_pretrained("Cohere/rerank-v3.5")

output = tokenizer.encode("ぼくのAWSの推しのポイント100本ノック")

for token in output.tokens:
    print(token, end=" | ")
```

このコードを実行すると、このように出力されます。

```
[CLS] | ▁ | ぼく | の | AWS | の | 推 | し | のポイント | 100 | 本 | ノ | ック | [SEP] |
```

おおよそ単語ごとに区切られていますが、そうでないものもありますね。ひと文字で1トークンのものもあれば、5文字で1トークンのものもあります。「[CLS]」「▁」「[SEP]」などは制御文字です。

英語の文章でも試してみました。

```
[CLS] | ▁Amazon | ▁Web | ▁Services | ▁ | offers | ▁reli | able | , | ▁scal | able | , | ▁and | ▁in | expensive | ▁cloud | ▁ | computing | ▁services | . | ▁Free | ▁to | ▁join | , | ▁pay | ▁only | ▁for | ▁what | ▁you | ▁use | . | [SEP] |
```

英語の場合は日本語と比べ、1トークンあたりの文字数が多くなっていることがわかります。

## どうしてトークンに分割するの？

もともとは、1文字ごとに分割していたようなのですが、処理の効率化のため複数の文字をまとめて処理をするようになったようです。LLMの精度向上にも寄与しているようで、「ぼくのAWSの推しのポイント100本ノック」は、GPT-3.5では17トークンに分割されますが、GPT-4oでは13トークンと、トークン数が削減（＝1トークンあたりの文字数が増加）されています。
API利用のLLMの場合、消費したトークン量での課金になることが多いので、単価が同じでもトータルで安くなることもあります。


#### 著者紹介
---

<div class="author-profile">
    <img src="images/chap-moritalous-token/moritalous1.jpg" width="30%">
    <div>
        <div>
            <b>森田 和明</b>
            @moritalous
        </div>
    </div>
</div>
<p style="margin-top: 0.5em; margin-bottom: 2em;">
Amazon Bedrock 生成AIアプリ開発入門 [AWS深掘りガイド]の共著者のひとりです。Qiitaに毎週投稿継続中。いいねください。
</p>
