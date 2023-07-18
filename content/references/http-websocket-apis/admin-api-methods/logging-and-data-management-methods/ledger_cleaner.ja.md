---
html: ledger_cleaner.html
parent: logging-and-data-management-methods.html
blurb: レジャークリーナーを制御し、レジャーデータベースの破損を検出して修復できる非同期メンテナンスをする。
labels:
  - データ保持
---
# ledger_cleaner
[[ソース]](https://github.com/ripple/rippled/blob/df54b47cd0957a31837493cd69e4d9aade0b5055/src/ripple/rpc/handlers/LedgerCleaner.cpp "Source")

<span class="code-snippet">ledger_cleaner</span>コマンドは[レジャークリーナー](https://github.com/ripple/rippled/blob/f313caaa73b0ac89e793195dcc2a5001786f916f/src/ripple/app/ledger/README.md#the-ledger-cleaner)を制御します。レジャークリーナーは、<span class="code-snippet">rippled</span>のレジャーデータベースの破損を検出して修復できる非同期メンテナンスプロセスです。

_<span class="code-snippet">ledger_cleaner</span>メソッドは、権限のないユーザーは実行できない[管理メソッド](admin-api-methods.html)です。_

### 要求フォーマット
要求フォーマットの例:

<!-- MULTICODE_BLOCK_START -->

*WebSocket*

```json
{
    "command": "ledger_cleaner",
    "max_ledger": 13818756,
    "min_ledger": 13818000,
    "stop": false
}
```

<!-- MULTICODE_BLOCK_END -->

要求には以下のパラメーターが含まれます。

| `Field`       | 型                        | 説明                             |
|:--------------|:--------------------------|:---------------------------------|
| <span class="code-snippet">ledger</span> | 数値 - [レジャーインデックス][] | _（省略可）_ 指定されている場合は、指定されたレジャーのみをチェックして訂正します。 |
| <span class="code-snippet">max_ledger</span> | 数値 - [レジャーインデックス][] | _（省略可）_ レジャーインデックスがこの番号以下のレジャーをチェックするようにレジャークリーナーを設定します。 |
| <span class="code-snippet">min_ledger</span> | 数値 - [レジャーインデックス][] | _（省略可）_ レジャーインデックスがこの番号以上のレジャーをチェックするようにレジャークリーナーを設定します。 |
| <span class="code-snippet">full</span> | ブール値 | _（省略可）_ trueの場合は、指定されたレジャーのレジャー状態オブジェクトとトランザクションを修正します。デフォルトではfalseです。<span class="code-snippet">ledger</span>が指定されている場合は、自動的に<span class="code-snippet">true</span>に設定されます。 |
| <span class="code-snippet">fix_txns</span> | ブール値 | _（省略可）_ trueの場合は、指定されたレジャーのトランザクションを修正します。指定されている場合は<span class="code-snippet">full</span>をオーバーライドします。 |
| <span class="code-snippet">check_nodes</span> | ブール値 | _（省略可）_ trueの場合は、指定されているレジャーのレジャー状態オブジェクトを修正します。指定されている場合は<span class="code-snippet">full</span>をオーバーライドします。 |
| <span class="code-snippet">stop</span> | ブール値 | _（省略可）_ trueの場合は、レジャークリーナーを無効にします。 |

### 応答フォーマット

処理が成功した応答の例:

<!-- MULTICODE_BLOCK_START -->

*JSON-RPC*

```json
200 OK

{
   "result" : {
      "message" : "Cleaner configured",
      "status" : "success"
   }
}

```

<!-- MULTICODE_BLOCK_END -->

この応答は[標準フォーマット][]に従っており、正常に完了した場合は結果に次のフィールドが含まれます。

| `Field`   | 型     | 説明                             |
|:----------|:-------|:---------------------------------|
| <span class="code-snippet">message</span> | 文字列 | <span class="code-snippet">Cleaner configured</span>: 正常終了の場合。 |

### 考えられるエラー

* いずれかの[汎用エラータイプ][]。
* <span class="code-snippet">internal</span>: いずれかのパラメーターが正しく指定されていない場合。（これはバグです。本来のエラーコードは<span class="code-snippet">invalidParams</span>です。）

<!--{# common link defs #}-->
{% include '_snippets/rippled-api-links.md' %}
{% include '_snippets/tx-type-links.md' %}
{% include '_snippets/rippled_versions.md' %}
