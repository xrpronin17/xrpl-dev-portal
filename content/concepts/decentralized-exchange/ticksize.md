---
html: ticksize.html
parent: decentralized-exchange.html
blurb: Issuers can set custom tick sizes for currencies to reduce churn in order books over miniscule differences in exchange rates.
labels:
  - Decentralized Exchange
  - Tokens
---
# Tick Size

_(Added by the [TickSize amendment][].)_

When an Offer is placed into an order book, its exchange rate is truncated based on the <span class="code-snippet">TickSize</span> values set by the issuers of the currencies involved in the Offer. When trading XRP and a token, the <span class="code-snippet">TickSize</span> from the token's issuer applies. When trading two tokens, the Offer uses the smaller <span class="code-snippet">TickSize</span> value (that is, the one with fewer significant digits). If neither token has a <span class="code-snippet">TickSize</span> set, the default behavior applies.

The <span class="code-snippet">TickSize</span> value truncates the number of _significant digits_ in the exchange rate of an offer when it gets placed in an order book. Issuers can set <span class="code-snippet">TickSize</span> to an integer from <span class="code-snippet">3</span> to <span class="code-snippet">15</span> using an [AccountSet transaction][]. The exchange rate is represented as significant digits and an exponent; the <span class="code-snippet">TickSize</span> does not affect the exponent. This allows the XRP Ledger to represent exchange rates between assets that vary greatly in value (for example, a highly inflated currency compared to a rare commodity). The lower the <span class="code-snippet">TickSize</span> an issuer sets, the larger the increment traders must offer to be considered a higher exchange rate than the existing Offers.

The <span class="code-snippet">TickSize</span> does not affect the part of an Offer that can be executed immediately. (For that reason, OfferCreate transactions with <span class="code-snippet">tfImmediateOrCancel</span> are unaffected by <span class="code-snippet">TickSize</span> values.) If the Offer cannot be fully executed, the transaction processing engine calculates the exchange rate and truncates it based on <span class="code-snippet">TickSize</span>. Then, the engine rounds the remaining amount of the Offer from the "less important" side to match the truncated exchange rate. For a default OfferCreate transaction (a "buy" Offer), the <span class="code-snippet">TakerPays</span> amount (the amount being bought) gets rounded. If the <span class="code-snippet">tfSell</span> flag is enabled (a "sell" Offer) the <span class="code-snippet">TakerGets</span> amount (the amount being sold) gets rounded.

When an issuer enables, disables, or changes the <span class="code-snippet">TickSize</span>, Offers that were placed under the previous setting are unaffected.

## See Also

- [Dev Blog: Introducing the TickSize Amendment](https://xrpl.org/blog/2017/ticksize-voting.html#ticksize-amendment-overview)
- **References:**
    - [AccountSet transaction][]
    - [book_offers method][]
    - [OfferCreate transaction][]



<!--{# common link defs #}-->
{% include '_snippets/rippled-api-links.md' %}
{% include '_snippets/tx-type-links.md' %}
{% include '_snippets/rippled_versions.md' %}
