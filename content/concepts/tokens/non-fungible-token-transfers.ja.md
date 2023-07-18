---
name: NFTの取引
html: non-fungible-token-transfers.html
parent: non-fungible-tokens.html
blurb: NFTokenをダイレクトモードまたはブローカーモードで取引する。
labels:
 - 非代替性トークン, NFT
---

# XRP Ledger上でNFTokenを売買する

XRP Ledger上のアカウント間で<span class="code-snippet">NFToken</span>オブジェクトを転送することができます。<span class="code-snippet">NFToken</span>の売買をオファーしたり、他のアカウントから自分が保有するNFTokenへの売買オファーを受け入れることができます。<span class="code-snippet">NFToken</span>を無料(価格が0)で売却することで、<span class="code-snippet">NFToken</span>を配布することもできます。すべてのオファーは[NFTokenCreateOfferトランザクション][]を使って作成されます。

_([NonFungibleTokensV1_1 amendment][]が必要です)_

## 売却オファー


### 売却オファーの作成

<span class="code-snippet">NFToken</span>オブジェクトの所有者であれば、<span class="code-snippet">tfSellToken</span>フラグを指定して[NFTokenCreateOfferトランザクション][]を使用して売却オファーを作成することができます。<span class="code-snippet">NFTokenID</span>と、対価として受け取る金額<span class="code-snippet">Amount</span>を指定します。オプションで、そのオファーが無効になる<span class="code-snippet">Expiration</span>と、その<span class="code-snippet">NFToken</span>を購入することができる唯一のアカウントである<span class="code-snippet">Destination</span>を指定することができます。


### 売却オファーを受け入れる

販売されている<span class="code-snippet">NFToken</span>を購入するには、<span class="code-snippet">NFTokenAcceptOffer</span>トランザクションを使用します。<span class="code-snippet">NFTokenOffer</span>オブジェクトの所有者アカウントと<span class="code-snippet">NFTokenOfferID</span>を指定し、受け入れることを決定します。


## 購入オファー


### 購入オファーの作成

どのアカウントでも<span class="code-snippet">NFToken</span>の購入オファーを作成することができます。<span class="code-snippet">tfSellToken</span>のフラグを指定せずに、[NFTokenCreateOffer][]を使用することで、購入オファーを作成することが可能です。<span class="code-snippet">Owner</span>アカウント、<span class="code-snippet">NFTokenID</span>、オファーの<span class="code-snippet">Amount</span>を指定します。


### 購入オファーを受け入れる

<span class="code-snippet">NFTokenAcceptOffer</span>トランザクションを使用して<span class="code-snippet">NFToken</span>を転送します。<span class="code-snippet">NFTokenOfferID</span>と所有者アカウントを指定して、トランザクションを完了させてください。


## 取引モード

<span class="code-snippet">NFToken</span>を取引する場合、購入者と販売者の間で直接取引を行う、 _ダイレクト_ 取引と、第三者の口座が売りと買いのオファーをマッチングして取引を仲介する、 _ブローカー_ 取引を選択することができます。

ダイレクトモードでの取引では、販売者が転送をコントロールすることができます。販売者は誰でも購入できるように<span class="code-snippet">NFToken</span>を出品するか、特定の取引先アカウントに<span class="code-snippet">NFToken</span>を販売することができます。販売者はNFTokenの販売価格全額を受け取ります。

ブローカーモードでは、販売者は第三者のアカウントに<span class="code-snippet">NFToken</span>の販売を仲介させます。ブローカーアカウントは、合意したレートで仲介手数料を徴収し、転送を行います。購入はリアルタイムで完了し、ブローカーと販売者には購入資金から支払われ、ブローカーによる前払いは必要ありません。


### ブローカーモードを使用する場合

<span class="code-snippet">NFToken</span>の作成者が適切な購入者を探す時間と忍耐力がある場合、作成者は販売から得たすべての収益を得ることができます。これは、少数の<span class="code-snippet">NFToken</span>オブジェクトを様々な価格で販売するクリエイターにとって、非常に有効な方法です。

一方、クリエイターは、創作に時間を割くことができるのに、販売に時間を割くのは抵抗があるのではないでしょうか。そのような場合、個別に対応するのではなく、第三者であるブローカーのアカウントに販売業務を委託することが可能です。

ブローカーを利用すると、いくつかの利点があります。例えば

* ブローカーは仲介者として、<span class="code-snippet">NFToken</span>の販売価格を最大化するために活動することができます。ブローカーが販売価格の何割かを受け取る場合、価格が高ければ高いほど、ブローカーの収入も増えます。
* ブローカーは、ニッチな市場や価格帯などの基準に基づいて<span class="code-snippet">NFToken</span>オブジェクトの管理を行う管理者として活動することができます。これによって、クリエイターの作品を発見できないような購入者のグループを呼び込むことができるでしょう。
* ブローカーは、Opensea.ioのようなマーケットプレイスとして機能し、アプリケーション層でオークション機能を提供することもできます。


### ブローカー販売のワークフロー

最も単純なワークフローでは、クリエイターが新しい<span class="code-snippet">NFToken</span>を発行します。クリエイターは売却オファーを作成する際、最低売却価格を入力し、売却先にブローカーを設定します。購入希望者はブローカーを経由して<span class="code-snippet">NFToken</span>に入札を行います。ブローカーは落札者を選び、取引を完了させ、ブローカー手数料を受け取ります。ベストプラクティスとして、ブローカーは<span class="code-snippet">NFToken</span>に対して残っている購入オファーをすべてキャンセルします。


![Brokered Mode with Reserve](img/nft-brokered-mode-with-reserve.png)


もう1つのワークフローは、クリエイターが販売をよりコントロールできるようにするものです。このワークフローでは、クリエイターが新しい<span class="code-snippet">NFToken</span>を発行します。入札者はオファーを作成し、ブローカーを宛先として設定します。ブローカーは落札者を選び、仲介手数料を差し引き、<span class="code-snippet">NFTokenCreateOffer</span>を使用してクリエイターに署名の依頼をします。クリエーターは要求されたオファーに署名し、ブローカーを宛先として設定します。ブローカーは<span class="code-snippet">NFTokenAcceptOffer</span>を使って売却を完了し、仲介手数料を保持します。ブローカーは<span class="code-snippet">NFTokenCancelOffer</span>を使用して<span class="code-snippet">NFToken</span>に対する残りの入札をキャンセルします。


![Brokered Mode without Reserve](img/nft-brokered-mode-without-reserve.png)


所有者が他のアカウントで作成した<span class="code-snippet">NFToken</span>をリセールする場合にも、同じワークフローを使用することができます。


<!--{# common link defs #}-->
{% include '_snippets/rippled-api-links.md' %}			
{% include '_snippets/tx-type-links.md' %}			
{% include '_snippets/rippled_versions.md' %}
