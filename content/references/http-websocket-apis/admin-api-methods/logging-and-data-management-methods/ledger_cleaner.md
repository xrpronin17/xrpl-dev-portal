---
html: ledger_cleaner.html
parent: logging-and-data-management-methods.html
blurb: Configure the ledger cleaner service to check for corrupted data.
labels:
  - Data Retention
---
# ledger_cleaner
[[Source]](https://github.com/ripple/rippled/blob/df54b47cd0957a31837493cd69e4d9aade0b5055/src/ripple/rpc/handlers/LedgerCleaner.cpp "Source")

The <span class="code-snippet">ledger_cleaner</span> command controls the [Ledger Cleaner](https://github.com/ripple/rippled/blob/f313caaa73b0ac89e793195dcc2a5001786f916f/src/ripple/app/ledger/README.md#the-ledger-cleaner), an asynchronous maintenance process that can find and repair corruption in <span class="code-snippet">rippled</span>'s database of ledgers.

_The <span class="code-snippet">ledger_cleaner</span> method is an [admin method](admin-api-methods.html) that cannot be run by unprivileged users._

### Request Format
An example of the request format:

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

The request includes the following parameters:

| `Field`       | Type                      | Description                      |
|:--------------|:--------------------------|:---------------------------------|
| <span class="code-snippet">ledger</span>      | Number - [Ledger Index][] | _(Optional)_ If provided, check and correct the specified ledger only. |
| <span class="code-snippet">max_ledger</span>  | Number - [Ledger Index][] | _(Optional)_ Configure the ledger cleaner to check ledgers with ledger indexes equal or lower than this. |
| <span class="code-snippet">min_ledger</span>  | Number - [Ledger Index][] | _(Optional)_ Configure the ledger cleaner to check ledgers with ledger indexes equal or higher than this. |
| <span class="code-snippet">full</span>        | Boolean                   | _(Optional)_ If true, fix ledger state objects and transactions in the specified ledger(s). Defaults to false. Automatically set to <span class="code-snippet">true</span> if <span class="code-snippet">ledger</span> is provided. |
| <span class="code-snippet">fix_txns</span>    | Boolean                   | _(Optional)_ If true, correct transaction in the specified ledger(s). Overrides <span class="code-snippet">full</span> if provided. |
| <span class="code-snippet">check_nodes</span> | Boolean                   | _(Optional)_ If true, correct ledger state objects in the specified ledger(s). Overrides <span class="code-snippet">full</span> if provided. |
| <span class="code-snippet">stop</span>        | Boolean                   | _(Optional)_ If true, disable the ledger cleaner. |

### Response Format

An example of a successful response:

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

The response follows the [standard format][], with a successful result containing the following fields:

| `Field`   | Type   | Description                      |
|:----------|:-------|:---------------------------------|
| <span class="code-snippet">message</span> | String | <span class="code-snippet">Cleaner configured</span> on success. |

### Possible Errors

* Any of the [universal error types][].
* <span class="code-snippet">internal</span> if one the parameters is specified incorrectly. (This is a bug; the intended error code is <span class="code-snippet">invalidParams</span>.)

<!--{# common link defs #}-->
{% include '_snippets/rippled-api-links.md' %}
{% include '_snippets/tx-type-links.md' %}
{% include '_snippets/rippled_versions.md' %}
