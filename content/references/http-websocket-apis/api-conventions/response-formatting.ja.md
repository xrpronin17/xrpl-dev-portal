---
html: response-formatting.html
parent: api-conventions.html
blurb: WebSocket、JSON-RPC、コマンドラインインターフェイスの応答のフォーマットとその応答に含まれるフィールド。
---
# 応答フォーマット

<span class="code-snippet">rippled</span> APIからの応答のフォーマットは、メソッドが呼び出されたインターフェイス（WebSocket、JSON-RPC、コマンドライン）に応じて多少異なります。コマンドラインインターフェイスがJSON-RPCを呼び出すため、コマンドラインインターフェイスとJSON-RPCインターフェイスは同じフォーマットを使用します。

成功した場合の応答に含まれるフィールドは、以下の通りです。

| `Field`         | 型     | 説明                                     |
|:----------------|:---------|:------------------------------------------------|
| <span class="code-snippet">id</span>            | （場合により異なる） | （WebSocketのみ）この応答の要求元となった要求で指定されているID。 |
| <span class="code-snippet">status</span>        | 文字列   | （WebSocketのみ）値が<span class="code-snippet">success</span>である場合、要求がサーバーによって正常に受信され、理解されたことを示します。 |
| <span class="code-snippet">result.status</span> | 文字列   | （JSON-RPCおよびコマンドライン）値が<span class="code-snippet">success</span>である場合、要求がサーバーによって正常に受信され、理解されたことを示します。 |
| <span class="code-snippet">type</span>          | 文字列   | （WebSocketのみ）値が<span class="code-snippet">response</span>の場合、コマンドに対する正常な応答であることを示します。[非同期の通知](subscribe.html)では、<span class="code-snippet">ledgerClosed</span>や<span class="code-snippet">transaction</span>など異なる値が使用されます。 |
| <span class="code-snippet">result</span>        | オブジェクト   | クエリーの結果。内容はコマンドによって異なります。 |


## 成功した場合の応答の例

<!-- MULTICODE_BLOCK_START -->

*WebSocket*

```
{
 "id": 2,
 "status": "success",
 "type": "response",
 "result": {
   "account_data": {
     "Account": "r9cZA1mLK5R5Am25ArfXFmqgNwjZgnfk59",
     "Balance": "27389517749",
     "Flags": 0,
     "LedgerEntryType": "AccountRoot",
     "OwnerCount": 18,
     "PreviousTxnID": "B6B410172C0B65575D89E464AF5B99937CC568822929ABF87DA75CBD11911932",
     "PreviousTxnLgrSeq": 6592159,
     "Sequence": 1400,
     "index": "4F83A2CF7E70F77F79A307E6A472BFC2585B806A70833CCD1C26105BAE0D6E05"
   },
   "ledger_index": 6760970
 }
}
```

*JSON-RPC*

```
HTTP Status: 200 OK
{
   "result": {
       "account_data": {
           "Account": "r9cZA1mLK5R5Am25ArfXFmqgNwjZgnfk59",
           "Balance": "27389517749",
           "Flags": 0,
           "LedgerEntryType": "AccountRoot",
           "OwnerCount": 18,
           "PreviousTxnID": "B6B410172C0B65575D89E464AF5B99937CC568822929ABF87DA75CBD11911932",
           "PreviousTxnLgrSeq": 6592159,
           "Sequence": 1400,
           "index": "4F83A2CF7E70F77F79A307E6A472BFC2585B806A70833CCD1C26105BAE0D6E05"
       },
       "ledger_index": 6761012,
       "status": "success"
   }
}
```
*コマンドライン*

```
{
   "result": {
       "account_data": {
           "Account": "r9cZA1mLK5R5Am25ArfXFmqgNwjZgnfk59",
           "Balance": "27389517749",
           "Flags": 0,
           "LedgerEntryType": "AccountRoot",
           "OwnerCount": 18,
           "PreviousTxnID": "B6B410172C0B65575D89E464AF5B99937CC568822929ABF87DA75CBD11911932",
           "PreviousTxnLgrSeq": 6592159,
           "Sequence": 1400,
           "index": "4F83A2CF7E70F77F79A307E6A472BFC2585B806A70833CCD1C26105BAE0D6E05"
       },
       "ledger_index": 6761012,
       "status": "success"
   }
}
```

<!-- MULTICODE_BLOCK_END -->
