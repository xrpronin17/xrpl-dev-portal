---
html: validator_list_sites.html
parent: status-and-debugging-methods.html
blurb: Get information about sites that publish validator lists.
labels:
  - Core Server
  - Blockchain
---
# validator_list_sites
[[Source]](https://github.com/ripple/rippled/blob/master/src/ripple/rpc/handlers/ValidatorListSites.cpp "Source")

The <span class="code-snippet">validator_list_sites</span> command returns status information of sites serving validator lists. [New in: rippled 0.80.1][]

*The <span class="code-snippet">validator_list_sites</span> method is an [admin method](admin-api-methods.html) that cannot be run by unprivileged users!*

### Request Format
An example of the request format:

<!-- MULTICODE_BLOCK_START -->

*WebSocket*

```json
{
    "id": 1,
    "command": "validator_list_sites"
}
```

*JSON-RPC*

```json
{
    "method": "validator_list_sites",
    "params": [
        {}
    ]
}
```

*Commandline*

```sh
#Syntax: validator_list_sites
rippled validator_list_sites
```

<!-- MULTICODE_BLOCK_END -->

The request includes no parameters.

### Response Format

An example of a successful response:

<!-- MULTICODE_BLOCK_START -->

*WebSocket*

```json
{
    "id":5,
    "status":"success",
    "type":"response",
    "result": {
        "validator_sites": [
            {
                "last_refresh_status": "accepted",
                "last_refresh_time": "2017-Oct-13 21:26:37",
                "refresh_interval_min": 5,
                "uri": "http://127.0.0.1:51447/validators"
            }
        ]
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
        "validator_sites": [
            {
                "last_refresh_status": "accepted",
                "last_refresh_time": "2017-Oct-13 21:26:37",
                "refresh_interval_min": 5,
                "uri": "http://127.0.0.1:51447/validators"
            }
        ]
    }
}
```

*Commandline*

```json
Loading: "/etc/rippled.cfg"
Connecting to 127.0.0.1:5005

{
    "result": {
        "status": "success",
        "validator_sites": [
            {
                "last_refresh_status": "accepted",
                "last_refresh_time": "2017-Oct-13 21:26:37",
                "refresh_interval_min": 5,
                "uri": "http://127.0.0.1:51447/validators"
            }
        ]
    }
}
```

<!-- MULTICODE_BLOCK_END -->

The response follows the [standard format][], with a successful result containing the following field:

| `Field`           | Type  | Description                      |
|:------------------|:------|----------------------------------|
| <span class="code-snippet">validator_sites</span> | Array | Array of validator site objects. |

Each member of the <span class="code-snippet">validator_sites</span> field array is an object with the following fields:

| `Field`                | Type             | Description                     |
|:-----------------------|:-----------------|:--------------------------------|
| <span class="code-snippet">last_refresh_status</span>  | String           | If present, shows the status of the most recent refresh of the site. If missing, the site has not yet been successfully queried. See **Site Status Values** below for possible states and their meanings. |
| <span class="code-snippet">last_refresh_time</span>    | String           | Human readable time when the site was last queried. If missing, the site has not yet been successfully queried. |
| <span class="code-snippet">refresh_interval_min</span> | Unsigned Integer | The number of minutes between refresh attempts. |
| <span class="code-snippet">uri</span>                  | String           | The URI of the site. |

#### Site Status Values

The <span class="code-snippet">last_refresh_status</span> field can have the following values:

| Value                 | Meaning                                              |
|:----------------------|:-----------------------------------------------------|
| <span class="code-snippet">accepted</span>            | The site provided a valid list, which your server is now using. |
| <span class="code-snippet">same_sequence</span>       | The site provided a list with the same sequence number as your existing list, so your server continued using its existing list. |
| <span class="code-snippet">unsupported_version</span> | The site provided a list, but your server does not support the list format version number in the list. You might need to [update <span class="code-snippet">rippled</span>](install-rippled.html) to a newer software version. |
| <span class="code-snippet">untrusted</span>           | The site provided a list from the site that is signed by a cryptographic key pair your server is not configured to trust. You may want to check for typos in your <span class="code-snippet">validators.txt</span> file and check to see if the list publisher changed their cryptographic keys. |
| <span class="code-snippet">stale</span>               | The site provided a list with a lower sequence number than the list your server is already using. |
| <span class="code-snippet">invalid</span>             | The site provided a list or signature that was not validly formed. |

### Possible Errors

- Any of the [universal error types][].
- <span class="code-snippet">reportingUnsupported</span> - ([Reporting Mode][] servers only) This method is not available in Reporting Mode.

<!--{# common link defs #}-->
{% include '_snippets/rippled-api-links.md' %}
{% include '_snippets/tx-type-links.md' %}
{% include '_snippets/rippled_versions.md' %}
