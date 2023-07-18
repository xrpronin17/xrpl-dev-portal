---
html: paths.html
parent: tokens.html
blurb: 発行済み通貨の支払いは、接続されているユーザーのパスとオーダーブックを通す必要があります。
labels:
  - 支払い
  - 複数通貨間
---
# パス

XRP Ledgerでは、[発行済み通貨](tokens.html)の支払いが送金元から受取人に届くまでにたどる中間ステップの道筋をパスによって定義します。パスは、XRP Ledgerの[分散型取引所](decentralized-exchange.html)のオーダーを介して送金元と受取人を結び付けることで、[複数通貨間の支払い](cross-currency-payments.html)を可能にします。また、負債を相殺するような複雑な決済もパスにより可能になります。

XRP Ledgerでは1つのPaymentトランザクションは複数のパスを使用でき、複数のソースの流動性を組み合わせて必要な額を送金することができます。そのため、トランザクションには使用可能なパスをまとめた _パスセット_ が含まれます。パスセットの中のパスでは開始時と終了時には同一通貨が使用される必要があります。

XRPは任意のアドレスに直接送金できるため、[XRP間のトランザクション](direct-xrp-payments.html)ではパスは使用されません。

## パスのステップ

パスは、支払いの送金元と受取人を結ぶステップで構成されています。すべてのステップは次のいずれかを行います。

* 同一通貨の別のアドレスを通じたRippling
* オーダーブックでの通貨の取引

別のアドレスを通じたRipplingは、負債を移動するプロセスです。一般的なケースでは、ある当事者に対するイシュアーの債務が削減され、別の当事者に対する債務が増加します。Ripplingは、トラストラインで結ばれているすべてのアドレスの間で発生させることができます。Ripplingのその他の例については、[NoRippleフラグについて](rippling.html)を参照してください。

通貨取引ステップの場合、パスステップにより変換先の通貨が指定されますが、オーダーブックにはオファーの状態は記録されません。レジャーが検証されるまではトランザクションの正規の順序は最終的ではないため、トランザクションの検証が完了するまでは、トランザクションがどのオファーをとるかは不明です。（各トランザクションは最終レジャーでの実行時に利用可能なオファーの中から最適なオファーをとるため、経験に基づいて推測することができます。） <!-- STYLE_OVERRIDE: will -->

いずれのタイプのステップでも、中間アドレスでは取得する価値と失う価値はほぼ同等です。トラストラインから同じ通貨の別のトラストラインへ残高がripplingするか、または以前に出されたオーダーに基づいて通貨が交換されます。場合によっては、[送金手数料](transfer-fees.html)、トラストラインクオリティ、または数値の丸め方が原因で、取得する価値と失われる価値が厳密に同等ではないことがあります。

[![3つのパスの例を示す図](img/paths-examples.ja.png)](img/paths-examples.ja.png)



# 技術詳細

## Pathfinding

<span class="code-snippet">rippled</span> APIではPathfindingに使用できるメソッドが2つあります。[ripple_path_findメソッド][]は、1回限りのパスセットの検索を実行します。[path_findメソッド][]（WebSocketのみ）は、レジャーが閉鎖するか、またはサーバーがより適切なパスを検出するたびに、フォローアップ応答によって検索を拡大します。

署名時に<span class="code-snippet">rippled</span>によりパスが自動的に入力されるようにするには、[signメソッド][]または[<span class="code-snippet">submit</span>コマンド（署名と送信モード）](submit.html#署名と送信モード)への要求に<span class="code-snippet">build_path</span>フィールドを指定します。ただし、トラブルを回避するために、署名前にPathfindingを個別に実行し、結果を確認することが推奨されます。

**注意:** <span class="code-snippet">rippled</span>は可能な限り低コストのパスを検出するように設計されていますが、常にこのようなパスを検出できるわけではありません。信頼できない<span class="code-snippet">rippled</span>インスタンスが改ざんされ、利益目的でこの動作が変更される可能性もあります。パスに沿った支払いの実行にかかる実際のコストは、送信時とトランザクション実行時で異なることがあります。

パスの検出は、新しいレジャーが検証されるたびに数秒ごとに変化する非常に難しい課題であるため、<span class="code-snippet">rippled</span>は完全に最適なパスを検出するようには設計されていません。ただし、いくつかの有効なパスを検出し、特定額の送金コストを推定することができます。


## 暗黙のステップ

規約として、パスのステップのいくつかは[Paymentトランザクションのフィールド](payment.html)により黙示的に示されます。これらのフィールドは、<span class="code-snippet">Account</span>（送金元）、<span class="code-snippet">Destination</span>（受取人）、<span class="code-snippet">Amount</span>（通貨と納入額）、<span class="code-snippet">SendMax</span>（通貨と送金額（指定されている場合））です。暗黙のステップは次のとおりです。

* パスの1番目のステップは、トランザクションの<span class="code-snippet">Account</span>フィールドに定義されるとおり、トランザクションの送信者であると常に黙示されます。
* トランザクションに、そのトランザクションの送信者ではない<span class="code-snippet">issuer</span>が指定されている<span class="code-snippet">SendMax</span>フィールドが含まれている場合、そのイシュアーはパスの2番目のパスとして黙示されます。
  * <span class="code-snippet">SendMax</span>の<span class="code-snippet">issuer</span>が送信側アドレス _である_ 場合、パスはその送信側アドレスから始まり、そのアドレスの特定の通貨のトラストラインのいずれかを使用できます。詳細は、[SendMaxおよびAmountの特殊な値](payment.html#sendmaxおよびamountで使用する特殊なissuerの値)を参照してください。
* トランザクションの<span class="code-snippet">Amount</span>フィールドに、トランザクションの<span class="code-snippet">Destination</span>とは異なる<span class="code-snippet">issuer</span>が指定されている場合、そのイシュアーはパスの最後から2番目のステップであると黙示されます。
* 最後に、トランザクションの<span class="code-snippet">Destination</span>フィールドに定義されるとおり、パスの最終ステップはトランザクションの受信者であることが常に黙示されます。


## デフォルトパス

明示的に指定されたパスの他に、トランザクションは _デフォルトパス_ に沿って実行できます。デフォルトパスは、トランザクションの[暗黙のステップ](#暗黙のステップ)を接続する最も簡単な方法です。

デフォルトパスは次のいずれかになります。

* トランザクションで（イシュアーに関係なく）1種類の通貨のみが使用される場合、デフォルトパスでは支払いが、関連するアドレスを通じてRipplingされると想定されます。このパスは、これらのアドレスがトラストラインで接続されている場合にのみ機能します。
  * <span class="code-snippet">SendMax</span>が省略されているか、または<span class="code-snippet">SendMax</span>の<span class="code-snippet">issuer</span>が送金元の場合、デフォルトパスが機能するためには送金元<span class="code-snippet">Account</span>から宛先<span class="code-snippet">Amount</span>の<span class="code-snippet">issuer</span>へのトラストラインが必要です。
  * <span class="code-snippet">SendMax</span>と<span class="code-snippet">Amount</span>に異なる<span class="code-snippet">issuer</span>値が指定されており、そのいずれも送金元または受取人ではない場合、これらの2つのイシュアー間のトラストラインでRipplingが必要となるため、デフォルトパスは有用ではない可能性があります。一般にイシュアーが互いに直接信頼し合うことはお勧めしません。
* 複数通貨間のトランザクションの場合、デフォルトパスは支払元通貨（<span class="code-snippet">SendMax</span>フィールドで指定）と宛先通貨（<span class="code-snippet">Amount</span>フィールドで指定）の間でオーダーブックを使用します。

有効なすべてのデフォルトパスを次の図に示します。
[![デフォルトパスの図](img/paths-default_paths.ja.png)](img/paths-default_paths.ja.png)

デフォルトパスを無効にするには[<span class="code-snippet">tfNoDirectRipple</span>フラグ](payment.html#paymentのフラグ)を使用します。このケースでは、トランザクションに明示的に指定されたパスを使用してトランザクションを実行することだけが可能です。トレーダーはこのオプションを使用して裁定取引を実行できます。


## パスの仕様

パスセットは配列です。パスセットの各要素は、個々の _パス_ を表す別の配列です。パスの各要素は、ステップを指定するオブジェクトです。ステップのフィールドを次に示します。

| フィールド  | 値                     | 説明                                   |
|:-----------|:-----------------------|:---------------------------------------|
| <span class="code-snippet">account</span>  | 文字列 - アドレス       | _（省略可）_ 指定されている場合、このパスステップは指定されたアドレスを通じたRipplingを表します。このステップに<span class="code-snippet">currency</span>フィールドまたは<span class="code-snippet">issuer</span>フィールドが指定されている場合は、このフィールドを指定しないでください。 |
| <span class="code-snippet">currency</span> | 文字列 - 通貨コード | _（省略可）_ 指定されている場合、このパスステップはオーダーブックを通じた通貨の変換を表します。指定される通貨は新しい通貨を表します。このステップに<span class="code-snippet">account</span>フィールドが指定されている場合は、このフィールドを指定しないでください。 |
| <span class="code-snippet">issuer</span>   | 文字列 - アドレス       | _（省略可）_ 指定されている場合、このパスステップは通貨の変換を表し、このアドレスは新しい通貨のイシュアーを定義します。XRP以外の<span class="code-snippet">currency</span>のステップでこのフィールドが省略されている場合、パスの直前のステップがイシュアーを定義します。<span class="code-snippet">currency</span>が省略され、このフィールドが指定されている場合、イシュアーが異なる同名の通貨間でオーダーブックを使用するパスステップを示します。<span class="code-snippet">currency</span>がXRPの場合は省略する必要があります。このステップに<span class="code-snippet">account</span>フィールドが指定されている場合は、このフィールドを指定しないでください。 |
| <span class="code-snippet">type</span>     | 整数                | **廃止予定**_（省略可）_ 他にどのフィールドが指定されているかを示します。 |
| <span class="code-snippet">type_hex</span> | 文字列                 | **廃止予定**: _（省略可）_<span class="code-snippet">type</span>フィールドの16進数表現です。 |

要約すると、以下のフィールドの組み合わせが有効であり、またオプションで<span class="code-snippet">type</span>、<span class="code-snippet">type_hex</span>のいずれかまたは両方を指定できます。

- <span class="code-snippet">account</span>のみ
- <span class="code-snippet">currency</span>のみ
- <span class="code-snippet">currency</span>と<span class="code-snippet">issuer</span>（<span class="code-snippet">currency</span>がXRP以外の場合）
- <span class="code-snippet">issuer</span>のみ

パスステップで<span class="code-snippet">account</span>、<span class="code-snippet">currency</span>、<span class="code-snippet">issuer</span>の各フィールドを上記以外の方法で指定することは無効です。

パスセットのバイナリシリアル化に使用される<span class="code-snippet">type</span>フィールドは、実際には1つの整数上でビット演算により作成されます。ビットの定義は次のとおりです。

| 値（16進数） | 値（10進数）     | 説明        |
|-------------|-----------------|-------------|
| 0x01        | 1               | アドレスの変更（Rippling）:<span class="code-snippet">account</span>フィールドが指定されています。 |
| 0x10        | 16              | 通貨の変更:<span class="code-snippet">currency</span>フィールドが指定されています。 |
| 0x20        | 32              | イシュアーの変更:<span class="code-snippet">issuer</span>フィールドが指定されています。 |

## 関連項目

- **コンセプト:**
  - [複数通貨間の支払い](cross-currency-payments.html)
  - [分散型取引所](decentralized-exchange.html)
  - [Partial Payments](partial-payments.html)
- **リファレンス:**
  - [Paymentトランザクション][]
  - [path_findメソッド][]（WebSocketのみ）
  - [ripple_path_findメソッド][]

<!--{# common link defs #}-->
{% include '_snippets/rippled-api-links.md' %}			 
{% include '_snippets/tx-type-links.md' %}			 
{% include '_snippets/rippled_versions.md' %}
