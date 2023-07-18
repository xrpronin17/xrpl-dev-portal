---
html: crawl_shards.html
parent: logging-and-data-management-methods.html
blurb: Request information about which history shards peers have.
labels:
  - Data Retention
---
# crawl_shards
[[Source]](https://github.com/ripple/rippled/blob/master/src/ripple/rpc/handlers/CrawlShards.cpp "Source")

Requests information from peer servers about which [shards of historical ledger data](history-sharding.html) they have available. [New in: rippled 1.2.0][]

_The <span class="code-snippet">crawl_shards</span> method is an [admin method](admin-api-methods.html) that cannot be run by unprivileged users._

### Request Format

An example of the request format:

<!-- MULTICODE_BLOCK_START -->

*WebSocket*

```json
{
  "command": "crawl_shards",
  "public_key": true,
  "limit": 0
}
```

*JSON-RPC*

```json
{
  "method": "crawl_shards",
  "params": [
    {
      "public_key": true,
      "limit": 0
    }
  ]
}
```

<!-- MULTICODE_BLOCK_END -->

**Note:** There is no command-line syntax for this method. Use the [json method][] to access this from the command line.

The request includes the following fields:

| `Field`  | Type    | Description                                             |
|:---------|:--------|:--------------------------------------------------------|
| <span class="code-snippet">public_key</span> | Boolean | _(Optional)_ If <span class="code-snippet">true</span>, the response includes the node public keys (for peer-to-peer communications) of servers that were crawled. The default is <span class="code-snippet">false</span>. |
| <span class="code-snippet">limit</span>  | Number  | _(Optional)_ How many hops deep to search. The default is 0, which searches direct peers only. With a limit of <span class="code-snippet">1</span>, searches peers' peers also. The maximum value is <span class="code-snippet">3</span>. |

**Caution:** The number of peers potentially searched grows exponentially as <span class="code-snippet">limit</span> increases. With a limit of 2 or 3, it can take several seconds for the server to respond to the API request.


### Response Format

An example of a successful response:

<!-- MULTICODE_BLOCK_START -->

*WebSocket*

```json
{
  "result": {
    "complete_shards": "1-2,5,8-9,584,1973,2358",
    "peers": [
      {
        "complete_shards": "1-2,8,47,371,464,554,653,857,1076,1402,1555,1708,1813,1867",
        "public_key": "n9LxFZiySnfDSvfh23N94UxsFkCjWyrchTeKHcYE6tJJQL5iejb2"
      },
      {
        "complete_shards": "8-9,584",
        "ip": "192.168.1.132",
        "public_key": "n9MN5xwYqbrj64rtfZAXQy7Y3sNxXZJeLt7Lj61a9DYEZ4SE2tQQ"
      }
    ]
  },
  "status": "success",
  "type": "response"
}
```


*JSON-RPC*

```json
200 OK

{
  "result": {
    "complete_shards": "1-2,5,8-9,584,1973,2358",
    "peers": [
      {
        "complete_shards": "1-2,8,47,371,464,554,653,857,1076,1402,1555,1708,1813,1867",
        "public_key": "n9LxFZiySnfDSvfh23N94UxsFkCjWyrchTeKHcYE6tJJQL5iejb2"
      },
      {
        "complete_shards": "8-9,584",
        "ip": "192.168.1.132",
        "public_key": "n9MN5xwYqbrj64rtfZAXQy7Y3sNxXZJeLt7Lj61a9DYEZ4SE2tQQ"
      }
    ],
    "status": "success"
  }
}
```


<!-- MULTICODE_BLOCK_END -->

The response follows the [standard format][], with a successful result containing the following fields:

| `Field`           | Type   | Description                                     |
|:------------------|:-------|:------------------------------------------------|
| <span class="code-snippet">complete_shards</span> | String | _(May be omitted)_ The range of [history shards](history-sharding.html) that are available on the local server. This may be an empty string, or a disjointed range. For example, <span class="code-snippet">1-2,5,7-9</span> indicates that shards 1, 2, 5, 7, 8, and 9 are available. Omitted if this server does not have history sharding enabled. |
| <span class="code-snippet">peers</span>           | Array  | _(May be omitted)_ List of **Peer Shard Objects** (see below) describing which history shards each peer has available. The response omits this field if no peers within the number of hops specified by <span class="code-snippet">limit</span> have any shards. |

#### Peer Shard Objects

Each member of the <span class="code-snippet">peers</span> array of the response is an object that describes one server in the peer-to-peer network. The list only includes peers that have at least one complete [history shard](history-sharding.html) available. Each object in the array has the following fields:

| `Field`   | Type   | Description                                             |
|:----------|:-------|:--------------------------------------------------------|
| <span class="code-snippet">complete_shards</span> | String | The range of complete history shards this peer has available. This may be disjointed. For example, <span class="code-snippet">1-2,5,7-9</span> indicates that shards 1, 2, 5, 7, 8, and 9 are available. |
| <span class="code-snippet">incomplete_shards</span> | String | _(May be omitted)_ A comma-separated list of history shards this peer has partially downloaded, and percent completion for each. For example, <span class="code-snippet">1:50,2:25</span> indicates that shard 1 is 50% downloaded and shard 2 is 25% downloaded. [New in: rippled 1.8.1][] |
| <span class="code-snippet">public_key</span> | String | _(Omitted unless the request specified <span class="code-snippet">"public_key": true</span>)_ The public key this peer uses for peer-to-peer communications, in the XRP Ledger's [base58 format](base58-encodings.html). |

The <span class="code-snippet">ip</span> field is no longer provided. [Removed in: rippled 1.8.1][]


### Possible Errors

- Any of the [universal error types][].
- <span class="code-snippet">invalidParams</span> - One or more required fields were omitted from the request, or a provided field was specified as the wrong data type.
- <span class="code-snippet">reportingUnsupported</span> - ([Reporting Mode][] servers only) This method is not available in Reporting Mode.


<!--{# common link defs #}-->
{% include '_snippets/rippled-api-links.md' %}
{% include '_snippets/tx-type-links.md' %}
{% include '_snippets/rippled_versions.md' %}
