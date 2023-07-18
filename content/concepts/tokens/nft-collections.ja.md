---
html: nft-collections.html
parent: non-fungible-tokens.html
blurb: NFTのTaxonフィールドを使用して、NFTをコレクションとしてミントすることができます。
labels:
 - 非代替性トークン, NFT
---
# NFTのコレクション化

<span class="code-snippet">NFTokenTaxon</span>フィールドを使用すると、NFTをコレクションにグループ化することができます。ミント担当者は、<span class="code-snippet">0x0</span>から<span class="code-snippet">0xFFFFFFF</span>までの任意の数値を選択し、NFTを作成する際にそれを割り当てることができます。Taxon(分類群)の定義付けは完全に自由です。

例えば、最初のコレクションでは、<span class="code-snippet">NFTokenTaxon</span>を<span class="code-snippet">1</span>に設定します。NFTのコレクションで、Taxonの値が<span class="code-snippet">316</span>、<span class="code-snippet">420</span>、<span class="code-snippet">911</span>であるものがあるかもしれません。NFTの種類を示すために、数字で始まるタクソンを使用することもできます（たとえば、すべての不動産NFTは<span class="code-snippet">2</span>で始まるTaxonを持っているなど）。

<span class="code-snippet">NFTokenTaxon</span>フィールドは必須ですが、コレクションを作成するつもりがなければ<span class="code-snippet">0</span>を設定するのもよいでしょう。

[NFTokenの分類群](nftoken.html#分類群)を参照してください。
