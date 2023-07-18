---
html: can_delete.html
parent: logging-and-data-management-methods.html
blurb: Allow online deletion of ledgers up to a specific ledger.
labels:
  - Data Retention
---
# can_delete
[[Source]](https://github.com/ripple/rippled/blob/master/src/ripple/rpc/handlers/CanDelete.cpp "Source")

The <span class="code-snippet">can_delete</span> method informs the <span class="code-snippet">rippled</span> server of the latest ledger version which may be deleted when using [online deletion with advisory deletion enabled](online-deletion.html#advisory-deletion). If advisory deletion is not enabled, this method does nothing.

_The <span class="code-snippet">can_delete</span> method is an [admin method](admin-api-methods.html) that cannot be run by unprivileged users._

### Request Format

An example of the request format:

<!-- MULTICODE_BLOCK_START -->

*WebSocket*

```json
{
  "id": 2,
  "command": "can_delete",
  "can_delete": 11320417
}
```

*JSON-RPC*

```json
{
    "method": "can_delete",
    "params": [
        {
            "can_delete": 11320417
        }
    ]
}
```

*Commandline*

```sh
#Syntax: can_delete [<ledger_index>|<ledger_hash>|now|always|never]
rippled can_delete 11320417
```

<!-- MULTICODE_BLOCK_END -->

The request accepts the following parameter:

| `Field`      | Type    | Description                                         |
|:-------------|:--------|:----------------------------------------------------|
| <span class="code-snippet">can_delete</span> | String or Integer | _(Optional)_ The [Ledger Index][] of the maximum ledger version to allow to be deleted. The special case <span class="code-snippet">never</span> disables <span class="code-snippet">online deletion</span>. The special case <span class="code-snippet">always</span> enables automatic <span class="code-snippet">online deletion</span> as if advisory deletion was disabled. The special case now allows <span class="code-snippet">online deletion</span> one time at the next validated ledger that meets or exceeds the configured <span class="code-snippet">online_delete</span> value. If omitted, the server makes no changes (but still replies with the current <span class="code-snippet">can_delete</span> value).


### Response Format

The response follows the [standard format][], with a successful result containing the following fields:

| `Field`      | Type    | Description                                         |
|:-------------|:--------|:----------------------------------------------------|
| <span class="code-snippet">can_delete</span> | Integer | The maximum ledger index that may be removed by the online deletion routine. |

Use this command with no parameter to query the existing <span class="code-snippet">can_delete</span> setting.

### Possible Errors

- Any of the [universal error types][].
- <span class="code-snippet">invalidParams</span> - One or more fields are specified incorrectly, or one or more required fields are missing.
- <span class="code-snippet">lgrNotFound</span> - The ledger specified by the <span class="code-snippet">can_delete</span> field of the request does not exist, or it does exist but the server does not have it.
- <span class="code-snippet">notEnabled</span> - If either online deletion or advisory deletion are not enabled in the server's configuration.
- <span class="code-snippet">notReady</span> - The server is not ready to run online deletion at the moment. This usually means the server has recently started up and has not yet acquired a validated ledger.
- <span class="code-snippet">reportingUnsupported</span> - ([Reporting Mode][] servers only) This method is not available in Reporting Mode.

## See Also

- [Online Deletion](online-deletion.html)
- [Configure Advisory Deletion](configure-advisory-deletion.html)

<!--{# common link defs #}-->
{% include '_snippets/rippled-api-links.md' %}
{% include '_snippets/tx-type-links.md' %}
{% include '_snippets/rippled_versions.md' %}
