---
html: ticksize.html
parent: decentralized-exchange.html
blurb: 発行者は、為替レートのごくわずかな差を超えて、頻繁な取引を抑制するためにオーダーブックで通貨のカスタムチックサイズを設定することができます。
labels:
  - 分散型取引所
  - トークン
---
# ティックサイズ

_（[TickSize Amendment][]が必要です。）_

オファーがオーダーブックに対して発行されると、そのオファーに関係する通貨のイシュアーによって設定された<span class="code-snippet">TickSize</span>の値に基づいて、為替レートが切り捨てられます。トレーダーがXRPと発行済み通貨を交換するオファーを出した場合は、その発行済み通貨のイシュアーからの<span class="code-snippet">TickSize</span>が適用されます。トレーダーが2種類の発行済み通貨を交換するオファーを出した場合は、小さい方の<span class="code-snippet">TickSize</span>の値（有効数字の桁数が少ない値）がこのオファーに適用されます。いずれの通貨にも<span class="code-snippet">TickSize</span>が設定されていない場合、デフォルトが適用されます。

オーダーブックにオファーが発行されると、<span class="code-snippet">TickSize</span> によりオファーの為替レートの _有効数字_ の桁数が切り捨てられます。イシュアーは[AccountSetトランザクション][]を使用して<span class="code-snippet">TickSize</span>を<span class="code-snippet">3</span>～<span class="code-snippet">15</span>の整数に設定できます。為替レートは有効数字と指数で表されますが、<span class="code-snippet">TickSize</span>は指数には影響しません。これにより、XRP Ledgerでは価値が大きく異なる資産（ハイパーインフレ通貨と希少通貨など）間の為替レートを表せます。イシュアーが設定する<span class="code-snippet">TickSize</span>が小さいほど、トレーダーはより多くの増分をオファーして、既存のオファーよりも高い為替レートと見えるようにする必要があります。

<span class="code-snippet">TickSize</span>は、オファーの即時に実行可能な部分には影響しません。（この理由から、<span class="code-snippet">tfImmediateOrCancel</span>が指定されたOfferCreateトランザクションは<span class="code-snippet">TickSize</span> の値の影響を受けません。）オファーを完全に実行できない場合、トランザクション処理エンジンは<span class="code-snippet">TickSize</span>に基づいて為替レートを計算して切り捨てを行います。次にエンジンは、切り捨てた後の為替レートに一致するように、「重要性が低い」側からのオファーの残額を丸めます。デフォルトのOfferCreateトランザクション（「買い」オファー）の場合、<span class="code-snippet">TakerPays</span>の額（購入額）が丸められます。<span class="code-snippet">tfSell</span>フラグが有効な場合（「売り」オファー）<span class="code-snippet">TakerGets</span>の額（売却額）が丸められます。

イシュアーが<span class="code-snippet">TickSize</span>を有効化、無効化、または変更する場合、以前の設定で発行されたオファーはその影響を受けません。

## 参照項目

- [Dev Blog: Introducing the TickSize Amendment](https://ripple.com/dev-blog/ticksize-amendment-open-voting/#ticksize-amendment-overview)

- [AccountSetトランザクション][]


<!--{# common link defs #}-->
{% include '_snippets/rippled-api-links.md' %}
{% include '_snippets/tx-type-links.md' %}
{% include '_snippets/rippled_versions.md' %}
