---
html: paths.html
parent: tokens.html
blurb: Payments of tokens must traverse paths of connected users and order books.
labels:
  - Payments
  - Cross-Currency
---
# Paths

In the XRP Ledger, paths define a way for [tokens](tokens.html) to flow through intermediary steps as part of a payment. Paths enable [cross-currency payments](cross-currency-payments.html) by connecting sender and receiver through orders in the XRP Ledger's [decentralized exchange](decentralized-exchange.html). Paths also enable complex settlement of offsetting debts.

A single Payment transaction in the XRP Ledger can use multiple paths, combining liquidity from different sources to deliver the desired amount. Thus, a transaction includes a _path set_, which is a collection of possible paths to take. All paths in a path set must start with the same currency, and must also end with the same currency as each other.

Since XRP can be sent directly to any address, an [XRP-to-XRP transaction](direct-xrp-payments.html) does not use any paths.

## Path Steps

A path is made of steps that connect the sender to the receiver of the payment. Every step is either:

* Rippling through another address with the same currency
* Trading tokens or XRP using an order book

[Rippling](rippling.html) is the process of exchanging equivalent tokens using the same currency code. In the typical case, rippling through an issuer involves reducing the tokens issued to one party and increasing the tokens issued to another party by an equal amount. The path step specifies which account to ripple through.

[Trading tokens and possibly XRP](decentralized-exchange.html) involves going to an order book and finding the best exchange rate between the assets involved for the amount being sent. The path step specifies which currency to change to, but does not record the state of the Offers in the order book. The canonical order of transactions is not final until a ledger is validated, so you cannot know for certain which Offers a transaction will take, until after the transaction has been validated. (You can make an educated guess, since each transaction takes the best available Offers at the time it executes in the final ledger.) <!-- STYLE_OVERRIDE: will -->

In both types of steps, each intermediate address gains and loses approximately equal value: either a balance ripples from a trust line to another trust line in the same currency, or they exchange currencies according to a previously-placed order. In some cases, the amounts gained and lost may not be exactly equivalent, due to [transfer fees](transfer-fees.html), trust line quality settings, or rounding.

{{ include_svg("img/paths-examples.svg", "Diagram of three example paths") }}



# Technical Details

## Pathfinding

The <span class="code-snippet">rippled</span> API has two methods that can be used for pathfinding. The [ripple_path_find method][] does a one-time lookup of possible path sets. The [path_find method][] (WebSocket only) expands on the search with follow-up responses whenever a ledger closes or the server finds a better path.

You can have <span class="code-snippet">rippled</span> automatically fill in paths when you sign it, by including the <span class="code-snippet">build_path</span> field in a request to the [sign method][] or [<span class="code-snippet">submit</span> command (sign-and-submit mode)](submit.html#sign-and-submit-mode). However, we recommend pathfinding separately and confirming the results before signing, to avoid surprises.

**Caution:** Although <span class="code-snippet">rippled</span> is designed to search for the cheapest paths possible, it may not always find them. Untrustworthy <span class="code-snippet">rippled</span> instances could also be modified to change this behavior for profit. The actual cost to execute a payment along a path can change between submission and transaction execution.

Finding paths is a very challenging problem that changes slightly every few seconds as new ledgers are validated, so <span class="code-snippet">rippled</span> is not designed to find the absolute best path. Still, you can find several possible paths and estimate the cost of delivering a particular amount.


## Implied Steps

By convention, several steps of a path are implied by the [fields of the Payment transaction](payment.html): specifically, the <span class="code-snippet">Account</span> (sender), <span class="code-snippet">Destination</span> (receiver), <span class="code-snippet">Amount</span> (currency and amount to be delivered) and <span class="code-snippet">SendMax</span> (currency and amount to be sent, if specified). The implied steps are as follows:

* The first step of a path is always implied to be the sender of the transaction, as defined by the transaction's <span class="code-snippet">Account</span> field.
* If the transaction includes a <span class="code-snippet">SendMax</span> field with an <span class="code-snippet">issuer</span> that is not the sender of the transaction, that issuer is implied to be the second step of the path.
    * If <span class="code-snippet">issuer</span> of the <span class="code-snippet">SendMax</span> _is_ the sending address, then the path starts at the sending address, and may use any of that address's trust lines for the given currency code. See [special values for <span class="code-snippet">SendMax</span> and <span class="code-snippet">Amount</span>](payment.html#special-issuer-values-for-sendmax-and-amount) for details.
* If the <span class="code-snippet">Amount</span> field of the transaction includes an <span class="code-snippet">issuer</span> that is not the same as the <span class="code-snippet">Destination</span> of the transaction, that issuer is implied to be the second-to-last step of the path.
* Finally, last step of a path is always implied to be the receiver of a transaction, as defined by the transaction's <span class="code-snippet">Destination</span> field.


## Default Paths

In addition to explicitly specified paths, a transaction can execute along the _default path_. The default path is the simplest possible way to connect the [implied steps](#implied-steps) of the transaction.

The default path could be any of the following:

* If the transaction uses only one token (regardless of issuer), then the default path assumes the payment should ripple through the addresses involved. This path only works if those addresses are connected by trust lines.
    * If <span class="code-snippet">SendMax</span> is omitted, or the <span class="code-snippet">issuer</span> of the <span class="code-snippet">SendMax</span> is the sender, the default path needs a trust line from the sending <span class="code-snippet">Account</span> to the <span class="code-snippet">issuer</span> of the destination <span class="code-snippet">Amount</span> to work.
    * If the <span class="code-snippet">SendMax</span> and <span class="code-snippet">Amount</span> have different <span class="code-snippet">issuer</span> values, and neither are the sender or receiver, the default path is probably not useful because it would need to ripple across a trust line between the two issuers. Ripple (the company) typically discourages issuers from trusting one another directly.
* For cross-currency transactions, the default path uses the order book between the source currency (as specified in the <span class="code-snippet">SendMax</span> field) and the destination currency (as specified in the <span class="code-snippet">Amount</span> field).

The following diagram enumerates all possible default paths:

{{ include_svg("img/default-paths.svg", "Diagram of default paths") }}

You can use the [<span class="code-snippet">tfNoDirectRipple</span> flag](payment.html#payment-flags) to disable the default path. In this case, the transaction can only execute using the paths explicitly included in the transaction. Traders can use this option to take arbitrage opportunities.


## Path Specifications

A path set is an array. Each member of the path set is another array that represents an individual _path_. Each member of a path is an object that specifies the step. A step has the following fields:

| Field      | Value                  | Description                            |
|:-----------|:-----------------------|:---------------------------------------|
| <span class="code-snippet">account</span>  | String - Address       | _(Optional)_ If present, this path step represents rippling through the specified address. MUST NOT be provided if this step specifies the <span class="code-snippet">currency</span> or <span class="code-snippet">issuer</span> fields. |
| <span class="code-snippet">currency</span> | String - Currency Code | _(Optional)_ If present, this path step represents changing currencies through an order book. The currency specified indicates the new currency. MUST NOT be provided if this step specifies the <span class="code-snippet">account</span> field. |
| <span class="code-snippet">issuer</span>   | String - Address       | _(Optional)_ If present, this path step represents changing currencies and this address defines the issuer of the new currency. If omitted in a step with a non-XRP <span class="code-snippet">currency</span>, a previous step of the path defines the issuer. If present when <span class="code-snippet">currency</span> is omitted, indicates a path step that uses an order book between same-named currencies with different issuers. MUST be omitted if the <span class="code-snippet">currency</span> is XRP. MUST NOT be provided if this step specifies the <span class="code-snippet">account</span> field. |
| <span class="code-snippet">type</span>     | Integer                | **DEPRECATED** _(Optional)_ An indicator of which other fields are present. |
| <span class="code-snippet">type_hex</span> | String                 | **DEPRECATED**: _(Optional)_ A hexadecimal representation of the <span class="code-snippet">type</span> field. |

In summary, the following combination of fields are valid, optionally with <span class="code-snippet">type</span>, <span class="code-snippet">type_hex</span>, or both:

- <span class="code-snippet">account</span> by itself
- <span class="code-snippet">currency</span> by itself
- <span class="code-snippet">currency</span> and <span class="code-snippet">issuer</span> as long as the <span class="code-snippet">currency</span> is not XRP
- <span class="code-snippet">issuer</span> by itself

Any other use of <span class="code-snippet">account</span>, <span class="code-snippet">currency</span>, and <span class="code-snippet">issuer</span> fields in a path step is invalid.

The <span class="code-snippet">type</span> field, used for the binary serialization of a path set, is actually constructed through bitwise operations on a single integer. The bits are defined as follows:

| Value (Hex) | Value (Decimal) | Description |
|-------------|-----------------|-------------|
| <span class="code-snippet">0x01</span>      | 1               | A change of address (rippling): the <span class="code-snippet">account</span> field is present. |
| <span class="code-snippet">0x10</span>      | 16              | A change of currency: the <span class="code-snippet">currency</span> field is present. |
| <span class="code-snippet">0x20</span>      | 32              | A change of issuer: the <span class="code-snippet">issuer</span> field is present. |


## See Also

- **Concepts:**
    - [Cross-Currency Payments](cross-currency-payments.html)
    - [Decentralized Exchange](decentralized-exchange.html)
    - [Partial Payments](partial-payments.html)
- **References:**
    - [Payment transaction][]
    - [path_find method][] (WebSocket only)
    - [ripple_path_find method][]


<!--{# common link defs #}-->
{% include '_snippets/rippled-api-links.md' %}			
{% include '_snippets/tx-type-links.md' %}			
{% include '_snippets/rippled_versions.md' %}
