The [HTTP / WebSocket APIs](http-websocket-apis.html) support two formats of currency code:

- **[Standard Currency Codes](currency-formats.html#standard-currency-codes):** As a 3-character string such as <span class="code-snippet">"EUR"</span> or <span class="code-snippet">"USD"</span>.
- **[Nonstandard Currency Codes](currency-formats.html#nonstandard-currency-codes):** As a 160-bit hexadecimal string, such as <span class="code-snippet">"0158415500000000C1F76FF6ECB0BAC600000000"</span>. This is uncommon.

Tokens with the same code can [ripple](rippling.html) across connected trust lines. Currency codes have no other behavior built into the XRP Ledger.
