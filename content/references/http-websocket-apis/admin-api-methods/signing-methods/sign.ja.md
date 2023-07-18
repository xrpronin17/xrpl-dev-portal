---
html: sign.html # watch for clashes w/ this filename
parent: signing-methods.html
blurb: トランザクションの署名済みバイナリー表現を返します。
labels:
  - トランザクション送信
---
# sign
[[ソース]](https://github.com/ripple/rippled/blob/master/src/ripple/rpc/handlers/SignHandler.cpp "ソース")

<span class="code-snippet">sign</span>メソッドは[JSONフォーマットのトランザクション](transaction-formats.html)と[シード値](cryptographic-keys.html)を受け取り、トランザクションの署名済みバイナリー表現を返します。[マルチシグトランザクション](multi-signing.html)に署名を付与する場合は、代わりに[sign_forメソッド][]を使用します。

{% include '_snippets/public-signing-note.md' %}
<!--_ -->

**注意:** 独自の<span class="code-snippet">rippled</span>サーバーを運用している場合を除き、このコマンドを使用するのではなく、[クライアントライブラリ](client-libraries.html)を実行してください。詳細については[安全な署名の設定](set-up-secure-signing.html)を参照してください。 

## 要求フォーマット
要求フォーマットの例:

<!-- MULTICODE_BLOCK_START -->

*WebSocket*

```json
{
  "id": 2,
  "command": "sign",
  "tx_json" : {
      "TransactionType" : "Payment",
      "Account" : "rf1BiGeXwwQoi8Z2ueFYTEXSwuJYfV2Jpn",
      "Destination" : "ra5nK24KXen9AHvsdFTKHSANinZseWnPcX",
      "Amount" : {
         "currency" : "USD",
         "value" : "1",
         "issuer" : "rf1BiGeXwwQoi8Z2ueFYTEXSwuJYfV2Jpn"
      }
   },
   "secret" : "s████████████████████████████",
   "offline": false,
   "fee_mult_max": 1000
}
```

*JSON-RPC*

```json
{
    "method": "sign",
    "params": [
        {
            "offline": false,
            "secret": "s████████████████████████████",
            "tx_json": {
                "Account": "rf1BiGeXwwQoi8Z2ueFYTEXSwuJYfV2Jpn",
                "Amount": {
                    "currency": "USD",
                    "issuer": "rf1BiGeXwwQoi8Z2ueFYTEXSwuJYfV2Jpn",
                    "value": "1"
                },
                "Destination": "ra5nK24KXen9AHvsdFTKHSANinZseWnPcX",
                "TransactionType": "Payment"
            },
            "fee_mult_max": 1000
        }
    ]
}
```

*コマンドライン*

```sh
#Syntax: sign secret tx_json [offline]
rippled sign s████████████████████████████ '{"TransactionType": "Payment", "Account": "rf1BiGeXwwQoi8Z2ueFYTEXSwuJYfV2Jpn", "Destination": "ra5nK24KXen9AHvsdFTKHSANinZseWnPcX", "Amount": { "currency": "USD", "value": "1", "issuer" : "rf1BiGeXwwQoi8Z2ueFYTEXSwuJYfV2Jpn" }, "Sequence": 360, "Fee": "10000"}' offline
```

<!-- MULTICODE_BLOCK_END -->

トランザクションに署名するには、[トランザクションを承認](transaction-basics.html#トランザクションの承認)できるシークレットキーを提供する必要があります。通常、サーバーが秘密鍵を取得するシード値を提供します。これを行うには、以下の方法があります。

* <span class="code-snippet">secret</span>フィールドにシードを指定し、<span class="code-snippet">key_type</span>フィールドを省略します。この値は、XRP Ledgerの[base58][]シード、RFC-1751、16進値のフォーマットで記述するか、文字列パスフレーズとして記述します（secp256k1キーのみ）。
* <span class="code-snippet">key_type</span>値と、<span class="code-snippet">seed</span>、<span class="code-snippet">seed_hex</span>、または<span class="code-snippet">passphrase</span>のいずれか1つを提供します。<span class="code-snippet">secret</span>フィールドは省略します。（コマンドライン構文ではサポートされません）。

要求には以下のパラメーターが含まれます。

| `Field`        | 型      | 説明                                              |
|:---------------|:--------|:--------------------------------------------------|
| <span class="code-snippet">tx_json</span> | オブジェクト | JSONフォーマットの[トランザクション定義](transaction-formats.html) |
| <span class="code-snippet">secret</span> | 文字列 | _（省略可）_ トランザクションを提供するアカウントの秘密シード。トランザクションへの署名に使用されます。信頼できないサーバーに対して、またはセキュリティが確保されていないネットワーク接続を通じて機密情報を送信しないでください。<span class="code-snippet">key_type</span>、<span class="code-snippet">seed</span>、<span class="code-snippet">seed_hex</span>、<span class="code-snippet">passphrase</span>と同時に使用することはできません。 |
| <span class="code-snippet">seed</span> | 文字列 | _（省略可）_ トランザクションを提供するアカウントの秘密シード。トランザクションへの署名に使用されます。XRP Ledgerの[base58][]フォーマットにする必要があります。指定する場合は、<span class="code-snippet">key_type</span>も指定する必要があります。<span class="code-snippet">secret</span>、<span class="code-snippet">seed_hex</span>、<span class="code-snippet">passphrase</span>と同時に使用することはできません。 |
| <span class="code-snippet">seed_hex</span> | 文字列 | _（省略可）_ トランザクションを提供するアカウントの秘密シード。トランザクションへの署名に使用されます。16進フォーマットにする必要があります。指定する場合は、<span class="code-snippet">key_type</span>も指定する必要があります。<span class="code-snippet">secret</span>、<span class="code-snippet">seed</span>、<span class="code-snippet">passphrase</span>と同時に使用することはできません。 |
| <span class="code-snippet">passphrase</span> | 文字列 | _（省略可）_ トランザクションを提供するアカウントの秘密シード。文字列パスフレーズとして、トランザクションへの署名に使用されます。指定する場合は、<span class="code-snippet">key_type</span>も指定する必要があります。<span class="code-snippet">secret</span>、<span class="code-snippet">seed</span>、<span class="code-snippet">seed_hex</span>と同時に使用することはできません。 |
| <span class="code-snippet">key_type</span> | 文字列 | _（省略可）_ 指定された暗号化キーペアの[署名アルゴリズム](cryptographic-keys.html#署名アルゴリズム)。有効な種類は、<span class="code-snippet">secp256k1</span>または<span class="code-snippet">ed25519</span>です。デフォルトは<span class="code-snippet">secp256k1</span>です。<span class="code-snippet">secret</span>と同時に使用することはできません。 |
| <span class="code-snippet">offline</span> | ブール値 | _（省略可）_ <span class="code-snippet">true</span>の場合は、トランザクションの生成時に、トランザクションの詳細を[自動入力](#自動入力可能なフィールド)しないでください。デフォルトは<span class="code-snippet">false</span>です。 |
| <span class="code-snippet">build_path</span> | ブール値 | _（省略可）_ Payment型のトランザクションに対して指定した場合、署名前に<span class="code-snippet">Paths</span>フィールドが自動で入力されます。**注意:** サーバーは、このフィールドの値ではなく、このフィールドが存在するかどうかを調べます。この動作は変更される可能性があります。 |
| <span class="code-snippet">fee_mult_max</span> | 整数 | _（省略可）_[自動的に提供される<span class="code-snippet">Fee</span>フィールド](transaction-common-fields.html#自動入力可能なフィールド)の上限値を設定します。現在の[トランザクションコストの負荷の乗数](transaction-cost.html#ローカル負荷コスト)が（<span class="code-snippet">fee_mult_max</span> ÷ <span class="code-snippet">fee_div_max</span>）よりも大きい場合、署名は<span class="code-snippet">rpcHIGH_FEE</span>エラーで失敗します。トランザクションの<span class="code-snippet">Fee</span>フィールド（[トランザクションコスト](transaction-cost.html)）を指定した場合は無視されます。デフォルトは<span class="code-snippet">10</span>です。 |
| <span class="code-snippet">fee_div_max</span> | 整数 | _（省略可）_ 現在の[トランザクションコストの負荷の乗数](transaction-cost.html#ローカル負荷コスト)が（<span class="code-snippet">fee_mult_max</span> ÷ <span class="code-snippet">fee_div_max</span>）よりも大きい場合、署名は<span class="code-snippet">rpcHIGH_FEE</span>エラーで失敗します。トランザクションの<span class="code-snippet">Fee</span>フィールド（[トランザクションコスト](transaction-cost.html)）を指定した場合は無視されます。デフォルトは<span class="code-snippet">1</span>です。[新規: rippled 0.30.1][] |

### 自動入力可能なフィールド

<span class="code-snippet">tx_json</span>（[トランザクションオブジェクト](transaction-formats.html)）の特定のフィールドを省略すると、サーバーは自動的に入力しようとします。要求の<span class="code-snippet">offline</span>を<span class="code-snippet">true</span>と指定しない限り、サーバーは署名前に以下のフィールドを提供します。

* <span class="code-snippet">Sequence</span> - サーバーは、送信者のアカウント情報にある次のシーケンス番号を自動的に使用します。
  * **注意:** アカウントの次のシーケンス番号は、このトランザクションが適用されるまで増分されません。トランザクションの送信および個々のトランザクションへの応答を待たずに複数のトランザクションに署名する場合は、最初のトランザクション以降の各トランザクションについて、正しいシーケンス番号を手動で提供する必要があります。
* <span class="code-snippet">Fee</span> - <span class="code-snippet">Fee</span>パラメーターを省略した場合、サーバーは適切なトランザクションコストを自動的に入力しようとします。本番環境のXRP Ledgerでは、適切な<span class="code-snippet">fee_mult_max</span>値を提供しない限り、この処理は<span class="code-snippet">rpcHIGH_FEE</span>エラーで失敗します。
  * <span class="code-snippet">fee_mult_max</span>パラメーターと<span class="code-snippet">fee_div_max</span>パラメーターは、[リファレンストランザクションコスト](transaction-cost.html#referenceトランザクションコスト)に適用される負荷スケーリング乗数によって、自動的に提供されるトランザクションコストの上限値を設定します。デフォルト設定では、自動的に提供される値が10×の乗数より大きい場合、エラーが返されます。ただし、本番環境のXRP Ledgerでは、[1000×の負荷乗数を使用することが一般的](transaction-cost.html#現在のトランザクションコスト)です。
  * コマンドライン構文では、<span class="code-snippet">fee_mult_max</span>および<span class="code-snippet">fee_div_max</span>はサポートされません。本番環境のXRP Ledgerの場合は、<span class="code-snippet">Fee</span>値を提供する必要があります。
  * **注意:** 悪意のあるサーバーは、<span class="code-snippet">fee_mult_max</span>や<span class="code-snippet">fee_div_max</span>の値を無視して、きわめて大きなトランザクションコストを指定するおそれがあります。
* <span class="code-snippet">Paths</span> - Payment型のトランザクションの場合（XRP間の移動を除く）、Pathsフィールドは、[ripple_path_findメソッド][]を使用した場合と同様に自動的に入力できます。<span class="code-snippet">build_path</span>を指定した場合のみ入力されます。

## 応答フォーマット

処理が成功した応答の例:

<!-- MULTICODE_BLOCK_START -->

*WebSocket*

```json
{
  "id": 2,
  "status": "success",
  "type": "response",
  "result": {
    "tx_blob": "1200002280000000240000016861D4838D7EA4C6800000000000000000000000000055534400000000004B4E9C06F24296074F7BC48F92A97916C6DC5EA9684000000000002710732103AB40A0490F9B7ED8DF29D246BF2D6269820A0EE7742ACDD457BEA7C7D0931EDB7446304402200E5C2DD81FDF0BE9AB2A8D797885ED49E804DBF28E806604D878756410CA98B102203349581946B0DDA06B36B35DBC20EDA27552C1F167BCF5C6ECFF49C6A46F858081144B4E9C06F24296074F7BC48F92A97916C6DC5EA983143E9D4A2B8AA0780F682D136F7A56D6724EF53754",
    "tx_json": {
      "Account": "rf1BiGeXwwQoi8Z2ueFYTEXSwuJYfV2Jpn",
      "Amount": {
        "currency": "USD",
        "issuer": "rf1BiGeXwwQoi8Z2ueFYTEXSwuJYfV2Jpn",
        "value": "1"
      },
      "Destination": "ra5nK24KXen9AHvsdFTKHSANinZseWnPcX",
      "Fee": "10000",
      "Flags": 2147483648,
      "Sequence": 360,
      "SigningPubKey": "03AB40A0490F9B7ED8DF29D246BF2D6269820A0EE7742ACDD457BEA7C7D0931EDB",
      "TransactionType": "Payment",
      "TxnSignature": "304402200E5C2DD81FDF0BE9AB2A8D797885ED49E804DBF28E806604D878756410CA98B102203349581946B0DDA06B36B35DBC20EDA27552C1F167BCF5C6ECFF49C6A46F8580",
      "hash": "4D5D90890F8D49519E4151938601EF3D0B30B16CD6A519D9C99102C9FA77F7E0"
    }
  }
}
```

*JSON-RPC*

```json
200 OK

{
    "result": {
        "status": "success",
        "tx_blob": "1200002280000000240000016861D4838D7EA4C6800000000000000000000000000055534400000000004B4E9C06F24296074F7BC48F92A97916C6DC5EA9684000000000002710732103AB40A0490F9B7ED8DF29D246BF2D6269820A0EE7742ACDD457BEA7C7D0931EDB7446304402200E5C2DD81FDF0BE9AB2A8D797885ED49E804DBF28E806604D878756410CA98B102203349581946B0DDA06B36B35DBC20EDA27552C1F167BCF5C6ECFF49C6A46F858081144B4E9C06F24296074F7BC48F92A97916C6DC5EA983143E9D4A2B8AA0780F682D136F7A56D6724EF53754",
        "tx_json": {
            "Account": "rf1BiGeXwwQoi8Z2ueFYTEXSwuJYfV2Jpn",
            "Amount": {
                "currency": "USD",
                "issuer": "rf1BiGeXwwQoi8Z2ueFYTEXSwuJYfV2Jpn",
                "value": "1"
            },
            "Destination": "ra5nK24KXen9AHvsdFTKHSANinZseWnPcX",
            "Fee": "10000",
            "Flags": 2147483648,
            "Sequence": 360,
            "SigningPubKey": "03AB40A0490F9B7ED8DF29D246BF2D6269820A0EE7742ACDD457BEA7C7D0931EDB",
            "TransactionType": "Payment",
            "TxnSignature": "304402200E5C2DD81FDF0BE9AB2A8D797885ED49E804DBF28E806604D878756410CA98B102203349581946B0DDA06B36B35DBC20EDA27552C1F167BCF5C6ECFF49C6A46F8580",
            "hash": "4D5D90890F8D49519E4151938601EF3D0B30B16CD6A519D9C99102C9FA77F7E0"
        }
    }
}
```

*コマンドライン*

```json
Loading: "/etc/rippled.cfg"
Connecting to 127.0.0.1:5005

{
   "result" : {
      "status" : "success",
      "tx_blob" : "1200002280000000240000016861D4838D7EA4C6800000000000000000000000000055534400000000004B4E9C06F24296074F7BC48F92A97916C6DC5EA9684000000000002710732103AB40A0490F9B7ED8DF29D246BF2D6269820A0EE7742ACDD457BEA7C7D0931EDB7447304502210094D24C795CFFA8E46FE338AF63421DA5CE5E171ED56F8E4CE70FFABA15D3CFA2022063994C52BF0393C8157EBFFCDE6A7E7EDC7B16A462CA53214F64CC8FCBB5E54A81144B4E9C06F24296074F7BC48F92A97916C6DC5EA983143E9D4A2B8AA0780F682D136F7A56D6724EF53754",
      "tx_json" : {
         "Account" : "rf1BiGeXwwQoi8Z2ueFYTEXSwuJYfV2Jpn",
         "Amount" : {
            "currency" : "USD",
            "issuer" : "rf1BiGeXwwQoi8Z2ueFYTEXSwuJYfV2Jpn",
            "value" : "1"
         },
         "Destination" : "ra5nK24KXen9AHvsdFTKHSANinZseWnPcX",
         "Fee" : "10000",
         "Flags" : 2147483648,
         "Sequence" : 360,
         "SigningPubKey" : "03AB40A0490F9B7ED8DF29D246BF2D6269820A0EE7742ACDD457BEA7C7D0931EDB",
         "TransactionType" : "Payment",
         "TxnSignature" : "304502210094D24C795CFFA8E46FE338AF63421DA5CE5E171ED56F8E4CE70FFABA15D3CFA2022063994C52BF0393C8157EBFFCDE6A7E7EDC7B16A462CA53214F64CC8FCBB5E54A",
         "hash" : "DE80DA6FF9F93FE4CE87C99441F403E0290E35867FF48382204CB89975BF343E"
      }
   }
}
```

<!-- MULTICODE_BLOCK_END -->

この応答は[標準フォーマット][]に従っており、正常に完了した場合は結果に次のフィールドが含まれます。

| `Field`   | 型     | 説明                                                    |
|:----------|:-------|:--------------------------------------------------------|
| <span class="code-snippet">tx_blob</span> | 文字列 | 正しく作成された署名済みトランザクションの16進バイナリー表現 |
| <span class="code-snippet">tx_json</span> | オブジェクト | 自動的に入力されたフィールドを含む、署名済み[トランザクション全体](transaction-formats.html)のJSON仕様 |

**注意:** このコマンドの結果としてエラーメッセージが表示された場合は、要求で指定されたシークレット値がメッセージの中に含まれている可能性があります。これらのエラーが他者から見えない状態であることを確認してください。

* このエラーは、複数の人物が参照できるログファイルに書き込まないでください。
* デバッグを目的として、誰でも参照できる場所にこのエラーを貼り付けないでください。
* このエラーメッセージは、誤ってWebサイトに表示しないようにしてください。

## 考えられるエラー

* いずれかの[汎用エラータイプ][]。
* <span class="code-snippet">invalidParams</span> - 1つ以上のフィールドの指定が正しくないか、1つ以上の必須フィールドが指定されていません。
* <span class="code-snippet">highFee</span> - トランザクションコストに適用される現在の負荷乗数が、自動的に提供されるトランザクションコストの上限を超えています。要求で指定する<span class="code-snippet">fee_mult_max</span>を大きくするか（1000以上）、<span class="code-snippet">tx_json</span>の<span class="code-snippet">Fee</span>フィールドに値を手動で指定します。
* <span class="code-snippet">tooBusy</span> - トランザクションにパスが含まれていませんが、サーバーがビジーであるため、パス検出処理をすぐに実行できません。管理者として接続している場合は発生しません。
* <span class="code-snippet">noPath</span> - トランザクションにパスが含まれておらず、サーバーは、このペイメントの発生経路となるパスを検出できませんでした。


{% include '_snippets/rippled_versions.md' %}
{% include '_snippets/rippled-api-links.md' %}
