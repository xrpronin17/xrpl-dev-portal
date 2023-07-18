---
html: error-formatting.html
parent: api-conventions.html
blurb: Error formats and common error codes for WebSocket, JSON-RPC, and Commandline interfaces.
labels:
  - Development
---
# Error Formatting

It is impossible to list all the possible ways an error can occur. Some may occur in the transport layer (for example, loss of network connectivity), in which case the results vary depending on what client and transport you are using. However, if the <span class="code-snippet">rippled</span> server successfully receives your request, it tries to respond in a standardized error format.

**Caution:** When your request results in an error, the entire request is copied back as part of the response, so that you can try to debug the error. However, this also includes any secrets that were passed as part of the request. When sharing error messages, be very careful not to accidentally expose important account secrets to others.


Some example errors:

<!-- MULTICODE_BLOCK_START -->

*WebSocket*

```json
{
  "id": 3,
  "status": "error",
  "type": "response",
  "error": "ledgerIndexMalformed",
  "request": {
    "account": "r9cZA1mLK5R5Am25ArfXFmqgNwjZgnfk59",
    "command": "account_info",
    "id": 3,
    "ledger_index": "-",
    "strict": true
  }
}
```

*JSON-RPC*

```json
HTTP Status: 200 OK

{
    "result": {
        "error": "ledgerIndexMalformed",
        "request": {
            "account": "r9cZA1mLK5R5Am25ArfXFmqgNwjZgnfk59",
            "command": "account_info",
            "ledger_index": "-",
            "strict": true
        },
        "status": "error"
    }
}
```

*Commandline*

```json
{
    "result": {
        "error": "ledgerIndexMalformed",
        "request": {
            "account": "r9cZA1mLK5R5Am25ArfXFmqgNwjZgnfk59",
            "command": "account_info",
            "ledger_index": "-",
            "strict": true
        },
        "status": "error"
    }
}
```

<!-- MULTICODE_BLOCK_END -->


## WebSocket Format

| `Field`   | Type     | Description                                           |
|:----------|:---------|:------------------------------------------------------|
| <span class="code-snippet">id</span>      | (Varies) | ID provided in the Web Socket request that prompted this response |
| <span class="code-snippet">status</span>  | String   | <span class="code-snippet">"error"</span> if the request caused an error              |
| <span class="code-snippet">type</span>    | String   | Typically <span class="code-snippet">"response"</span>, which indicates a successful response to a command. |
| <span class="code-snippet">error</span>   | String   | A unique code for the type of error that occurred     |
| <span class="code-snippet">request</span> | Object   | A copy of the request that prompted this error, in JSON format. **Caution:** If the request contained any secrets, they are copied here! |
| <span class="code-snippet">api_version</span> | Number | _(May be omitted)_ The <span class="code-snippet">api_version</span> specified in the request, if any. |


## JSON-RPC Format

Some JSON-RPC request respond with an error code on the HTTP layer. In these cases, the response is a plain-text explanation in the response body. For example, if you forgot to specify the command in the <span class="code-snippet">method</span> parameter, the response is like this:

```
HTTP Status: 400 Bad Request
Null method
```

For other errors that returned with HTTP status code 200 OK, the responses are formatted in JSON, with the following fields:

| `Field`          | Type   | Description                                      |
|:-----------------|:-------|:-------------------------------------------------|
| <span class="code-snippet">result</span>         | Object | Object containing the response to the query      |
| <span class="code-snippet">result.error</span>   | String | A unique code for the type of error that occurred |
| <span class="code-snippet">result.status</span>  | String | <span class="code-snippet">"error"</span> if the request caused an error         |
| <span class="code-snippet">result.request</span> | Object | A copy of the request that prompted this error, in JSON format. **Caution:** If the request contained any account secrets, they are copied here! **Note:** The request is re-formatted in WebSocket format, regardless of the request made. |


## Universal Errors

All methods can potentially return any of the following values for the <span class="code-snippet">error</span> code:

- <span class="code-snippet">amendmentBlocked</span> - The server is [amendment blocked](amendments.html#amendment-blocked-servers) and needs to be updated to the latest version to stay synced with the XRP Ledger network.
- <span class="code-snippet">failedToForward</span> - ([Reporting Mode][] servers only) The server tried to forward this request to a P2P Mode server, but the connection failed.
- <span class="code-snippet">invalid_API_version</span> - The server does not support the [API version number](request-formatting.html#api-versioning) from the request.
- <span class="code-snippet">jsonInvalid</span> - (WebSocket only) The request is not a proper JSON object.
    - JSON-RPC returns a 400 Bad Request HTTP error in this case instead.
- <span class="code-snippet">missingCommand</span> - (WebSocket only) The request did not specify a <span class="code-snippet">command</span> field.
    - JSON-RPC returns a 400 Bad Request HTTP error in this case instead.
- <span class="code-snippet">noClosed</span> - The server does not have a closed ledger, typically because it has not finished starting up.
- <span class="code-snippet">noCurrent</span> - The server does not know what the current ledger is, due to high load, network problems, validator failures, incorrect configuration, or some other problem.
- <span class="code-snippet">noNetwork</span> - The server is having trouble connecting to the rest of the XRP Ledger peer-to-peer network (and is not running in stand-alone mode).
- <span class="code-snippet">tooBusy</span> - The server is under too much load to do this command right now. Generally not returned if you are connected as an admin.
- <span class="code-snippet">unknownCmd</span> - The request does not contain a [command](http-websocket-apis.html) that the <span class="code-snippet">rippled</span> server recognizes.
- <span class="code-snippet">wsTextRequired</span> - (WebSocket only) The request's [opcode](https://tools.ietf.org/html/rfc6455#section-5.2) is not text. <!-- SPELLING_IGNORE: opcode -->


<!--{# common link defs #}-->
{% include '_snippets/rippled-api-links.md' %}			
{% include '_snippets/tx-type-links.md' %}			
{% include '_snippets/rippled_versions.md' %}
