---
html: validators.html
parent: status-and-debugging-methods.html
blurb: サーバーが使用する公開済みの信頼できるバリデータの最新リストに関する情報を返します。
labels:
  - ブロックチェーン
  - コアサーバー
---
# validators
[[ソース]](https://github.com/ripple/rippled/blob/master/src/ripple/rpc/handlers/Validators.cpp "Source")

<span class="code-snippet">validators</span>コマンドは、サーバーが使用する公開済みの信頼できるバリデータの最新リストに関する情報を、人間が読み取れる形式で返します。[新規: rippled 0.80.1][]

*<span class="code-snippet">validators</span>要求は、権限のないユーザーは実行できない[管理メソッド](admin-api-methods.html)です。*

### 要求フォーマット
要求フォーマットの例:

<!-- MULTICODE_BLOCK_START -->

*WebSocket*

```json
{
   "id": 1,
   "command": "validators"
}
```

*JSON-RPC*

```json
{
   "method": "validators",
   "params": [
       {}
   ]
}
```

*コマンドライン*

```sh
#Syntax: validators
rippled validators
```

<!-- MULTICODE_BLOCK_END -->

要求にはパラメーターが含まれていません。

### 応答フォーマット

処理が成功した応答の例:

<!-- MULTICODE_BLOCK_START -->

*WebSocket*

```json
{
   "id":5,
   "status":"success",
   "type":"response",
   "result":{
       "local_static_keys": [],
       "publisher_lists":[
           {
               "available":true,
               "expiration":"2017-Oct-13 14:56:00",
               "list":[
                   "nHBtBkHGfL4NpB54H1AwBaaSJkSJLUSPvnUNAcuNpuffYB51VjH6",
                   "nHBe4vqSAzjpPRLKwSFzRFtmvzXaf5wPPmuVrQCAoJoS1zskgDA4"
               ],
               "pubkey_publisher":"ED58ED4AA543B524F16771F6E1367BAA220D99DCF22CD8CF7A11309E9EAB1B647B",
               "seq":1,
               "version":1
           }
       ],
       "signing_keys":{},
       "status":"success",
       "trusted_validator_keys":[
           "nHBe4vqSAzjpPRLKwSFzRFtmvzXaf5wPPmuVrQCAoJoS1zskgDA4",
           "nHBtBkHGfL4NpB54H1AwBaaSJkSJLUSPvnUNAcuNpuffYB51VjH6"
       ],
       "validation_quorum":2,
       "validator_list_expires":"2017-Oct-13 14:56:00"
   }
}
```

*JSON-RPC*

```json
200 OK

{
   "result":{
       "local_static_keys": [],
       "publisher_lists":[
           {
               "available":true,
               "expiration":"2017-Oct-13 14:56:00",
               "list":[
                   "nHBtBkHGfL4NpB54H1AwBaaSJkSJLUSPvnUNAcuNpuffYB51VjH6",
                   "nHBe4vqSAzjpPRLKwSFzRFtmvzXaf5wPPmuVrQCAoJoS1zskgDA4"
               ],
               "pubkey_publisher":"ED58ED4AA543B524F16771F6E1367BAA220D99DCF22CD8CF7A11309E9EAB1B647B",
               "seq":1,
               "version":1
           }
       ],
       "signing_keys":{},
       "status":"success",
       "trusted_validator_keys":[
           "nHBe4vqSAzjpPRLKwSFzRFtmvzXaf5wPPmuVrQCAoJoS1zskgDA4",
           "nHBtBkHGfL4NpB54H1AwBaaSJkSJLUSPvnUNAcuNpuffYB51VjH6"
       ],
       "validation_quorum":2,
       "validator_list_expires":"2017-Oct-13 14:56:00"
   },
   "status":"success"
}
```

*コマンドライン*

```json
Loading: "/etc/rippled.cfg"
Connecting to 127.0.0.1:5005

{
   "result":{
       "local_static_keys": [],
       "publisher_lists":[
           {
               "available":true,
               "expiration":"2017-Oct-13 14:56:00",
               "list":[
                   "nHBtBkHGfL4NpB54H1AwBaaSJkSJLUSPvnUNAcuNpuffYB51VjH6",
                   "nHBe4vqSAzjpPRLKwSFzRFtmvzXaf5wPPmuVrQCAoJoS1zskgDA4"
               ],
               "pubkey_publisher":"ED58ED4AA543B524F16771F6E1367BAA220D99DCF22CD8CF7A11309E9EAB1B647B",
               "seq":1,
               "version":1
           }
       ],
       "signing_keys":{},
       "status":"success",
       "trusted_validator_keys":[
           "nHBe4vqSAzjpPRLKwSFzRFtmvzXaf5wPPmuVrQCAoJoS1zskgDA4",
           "nHBtBkHGfL4NpB54H1AwBaaSJkSJLUSPvnUNAcuNpuffYB51VjH6"
       ],
       "validation_quorum":2,
       "validator_list_expires":"2017-Oct-13 14:56:00"
   },
   "status":"success"
}
```

<!-- MULTICODE_BLOCK_END -->

応答は[標準フォーマット][]に従っており、正常に完了した場合は結果に次のフィールドが含まれています。

| `Field`                  | 型   | 説明                              |
|:-------------------------|:-------|:-----------------------------------------|
| <span class="code-snippet">listed_static_keys</span>     | 配列  | 信頼リストに常に追加可能なバリデータの公開鍵の配列。 |
| <span class="code-snippet">publisher_lists</span>        | 配列  | パブリッシャーリストオブジェクトの配列。         |
| <span class="code-snippet">signing_keys</span>           | オブジェクト | バリデータマニフェストを使用している登録済みバリデータのマスター公開鍵から、現在の署名キーへのマッピング。 |
| <span class="code-snippet">trusted_validator_keys</span> | 配列  | 現在信頼されているバリデータの公開鍵の配列。 |
| <span class="code-snippet">validation_quorum</span>      | 数値 | 1つのレジャーバージョンの検証に最低限必要となる信頼できる検証の数。状況によっては、サーバーがさらに検証を要求する場合があります。 |
| <span class="code-snippet">validator_list_expires</span> | 文字列 | 人間が読み取れる形式での現在のバリデータリストの有効期限、文字列<span class="code-snippet">unknown</span>（サーバーが公開済みバリデータリストを読み込む必要がある場合）、または文字列<span class="code-snippet">never</span>（サーバーが静的なバリデータリストを使用している場合）のいずれか。 |

`publisher_lists`配列の各メンバーは、以下のフィールドを有するオブジェクトです。

| <span class="code-snippet">Field</span>            | 型             | 説明                          |
|:-------------------|:-----------------|:-------------------------------------|
| <span class="code-snippet">available</span>        | ブール値          | <span class="code-snippet">false</span>の場合、<span class="code-snippet">list</span>内のバリデータキーはこのパブリッシャーによりサポートされていない可能性があります。 |
| <span class="code-snippet">expiration</span>       | 文字列           | この公開済みリストの有効期限を人間が読み取れる形式で示します。 |
| <span class="code-snippet">list</span>             | 配列            | 公開済みバリデータキーからなる配列。   |
| <span class="code-snippet">pubkey_publisher</span> | 文字列           | リストパブリッシャーのEd25519またはECDSA公開鍵（16進数）。 |
| <span class="code-snippet">seq</span>              | 符号なし整数 | 公開済みリストのシーケンス番号。 |
| <span class="code-snippet">version</span>          | 符号なし整数 | リストフォーマットのバージョン。      |

### 考えられるエラー

* [汎用エラータイプ][]のすべて。

<!--{# common link defs #}-->
{% include '_snippets/rippled-api-links.md' %}
{% include '_snippets/tx-type-links.md' %}
{% include '_snippets/rippled_versions.md' %}
