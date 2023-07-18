---
html: partial-payments.html
parent: payment-types.html
blurb: Partial payments subtract fees from the amount sent, delivering a flexible amount. Partial payments are useful for returning unwanted payments without incurring additional costs.
labels:
  - Payments
  - Security
---
# Partial Payments

In the default case, the <span class="code-snippet">Amount</span> field of a [Payment transaction][] in the XRP Ledger specifies the exact amount to deliver, after charging for exchange rates and [transfer fees](transfer-fees.html). The "Partial Payment" flag ([<span class="code-snippet">tfPartialPayment</span>](payment.html#payment-flags)) allows a payment to succeed by reducing the amount received instead of increasing the amount sent. Partial payments are useful for [returning payments](become-an-xrp-ledger-gateway.html#bouncing-payments) without incurring additional costs to oneself.

The amount of XRP used for the [transaction cost](transaction-cost.html) is always deducted from the sender’s account, regardless of the type of transaction.

Partial payments can be used to exploit naive integrations with the XRP Ledger to steal money from exchanges and gateways. The [Partial Payments Exploit](#partial-payments-exploit) section of this document describes how this exploit works and how you can avoid it.

## Semantics

### Without Partial Payments

When sending a Payment that does not use the Partial Payment flag, the <span class="code-snippet">Amount</span> field of the transaction specifies the exact amount to deliver, and the <span class="code-snippet">SendMax</span> field specifies the maximum amount and currency to send. If a payment cannot deliver the full <span class="code-snippet">Amount</span> without exceeding the <span class="code-snippet">SendMax</span> parameter, or the full amount cannot be delivered for any other reason, the transaction fails. If the <span class="code-snippet">SendMax</span> field is omitted from the transaction instructions, it is considered to be equal to the <span class="code-snippet">Amount</span>. In this case, the payment can only succeed if the total amount of fees is 0.

In other words:

    Amount + (fees) = (sent amount) ≤ SendMax

In this formula, "fees" refers to [transfer fees](transfer-fees.html) and currency exchange rates. The "sent amount" and the delivered amount (<span class="code-snippet">Amount</span>) may be denominated in different currencies and converted by consuming Offers in the XRP Ledger's decentralized exchange.

**Note:** The <span class="code-snippet">Fee</span> field of the transaction refers to the XRP [transaction cost](transaction-cost.html), which is destroyed to relay the transaction to the network. The exact transaction cost specified is always debited from the sender and is completely separate from the fee calculations for any type of payment.

### With Partial Payments

When sending a Payment that has the Partial Payment flag enabled, the <span class="code-snippet">Amount</span> field of the transaction specifies a maximum amount to deliver. Partial payments can succeed at sending _some_ of the intended value despite limitations including fees, not enough liquidity, not enough space in the receiving account's trust lines, or other reasons.

The optional <span class="code-snippet">DeliverMin</span> field specifies a minimum amount to deliver. The <span class="code-snippet">SendMax</span> field functions the same as with non-partial payments. The partial payment transaction is successful if it delivers any amount equal or greater than the <span class="code-snippet">DeliverMin</span> field without exceeding the <span class="code-snippet">SendMax</span> amount. If the <span class="code-snippet">DeliverMin</span> field is not specified, a partial payment can succeed by delivering any positive amount.

In other words:

    Amount ≥ (Delivered Amount) = SendMax - (Fees) ≥ DeliverMin > 0

### Partial Payment Limitations

Partial Payments have the following limitations:

- A partial payment cannot provide the XRP to fund an address; this case returns the [result code][] <span class="code-snippet">telNO_DST_PARTIAL</span>.
- Direct XRP-to-XRP payments cannot be partial payments; this case returns the [result code][] <span class="code-snippet">temBAD_SEND_XRP_PARTIAL</span>.
    - However, cross-currency payments that involve XRP as one of the currencies _can_ be partial payments.

[result code]: transaction-results.html

### The <span class="code-snippet">delivered_amount</span> Field

To help understand how much a partial payment actually delivered, the metadata of a successful Payment transaction includes a <span class="code-snippet">delivered_amount</span> field. This field describes the amount actually delivered, in the [same format](basic-data-types.html#specifying-currency-amounts) as the <span class="code-snippet">Amount</span> field.

For non-partial payments, the <span class="code-snippet">delivered_amount</span> field of the transaction metadata is equal to the <span class="code-snippet">Amount</span> field of the transaction. When a payment delivers [tokens](tokens.html), the <span class="code-snippet">delivered_amount</span> may be slightly different than the <span class="code-snippet">Amount</span> field due to rounding.

The delivered amount is **not available** for transactions that meet **both** of the following criteria:

- Is a partial payment
- Is included in a validated ledger before 2014-01-20

If both conditions are true, then <span class="code-snippet">delivered_amount</span> contains the string value <span class="code-snippet">unavailable</span> instead of an actual amount. If this happens, you can only determine the actual delivered amount by reading the <span class="code-snippet">AffectedNodes</span> in the transaction's metadata. If the transaction delivered tokens and the <span class="code-snippet">issuer</span> of the <span class="code-snippet">Amount</span> is the same account as the <span class="code-snippet">Destination</span> address, the delivered amount may be divided among multiple <span class="code-snippet">AffectedNodes</span> members representing trust lines to different counterparties.

You can find the <span class="code-snippet">delivered_amount</span> field in the following places:

| API | Method | Field |
|-----|--------|-------|
| [JSON-RPC / WebSocket][] | [account_tx method][] | <span class="code-snippet">result.transactions</span> array members' <span class="code-snippet">meta.delivered_amount</span> |
| [JSON-RPC / WebSocket][] | [tx method][] | <span class="code-snippet">result.meta.delivered_amount</span> |
| [JSON-RPC / WebSocket][] | [transaction_entry method][] | <span class="code-snippet">result.metadata.delivered_amount</span> |
| [JSON-RPC / WebSocket][] | [ledger method][] (with transactions expanded) | <span class="code-snippet">result.ledger.transactions</span> array members' <span class="code-snippet">metaData.delivered_amount</span> [New in: rippled 1.2.1][] |
| [WebSocket][] | [Transaction subscriptions](subscribe.html#transaction-streams) | Subscription messages' <span class="code-snippet">meta.delivered_amount</span> [New in: rippled 1.2.1][] |
| ripple-lib v1.x | <span class="code-snippet">getTransaction</span> method | <span class="code-snippet">outcome.deliveredAmount</span> |
| ripple-lib v1.x | <span class="code-snippet">getTransactions</span> method | array members' <span class="code-snippet">outcome.deliveredAmount</span> |

[WebSocket]: http-websocket-apis.html
[JSON-RPC / WebSocket]: http-websocket-apis.html

## Partial Payments Exploit

If a financial institution's integration with the XRP Ledger assumes that the <span class="code-snippet">Amount</span> field of a Payment is always the full amount delivered, malicious actors may be able to exploit that assumption to steal money from the institution. This exploit can be used against gateways, exchanges, or merchants as long as those institutions' software does not process partial payments correctly.

**The correct way to process incoming Payment transactions is to use [the <span class="code-snippet">delivered_amount</span> metadata field](#the-delivered_amount-field),** not the <span class="code-snippet">Amount</span> field. This way, an institution is never mistaken about how much it _actually_ received.


### Exploit Scenario Steps

To exploit a vulnerable financial institution, a malicious actor does something like this:

1. The malicious actor sends a Payment transaction to the institution. This transaction has a large <span class="code-snippet">Amount</span> field and has the **<span class="code-snippet">tfPartialPayment</span>** flag enabled.
2. The partial payment succeeds (result code <span class="code-snippet">tesSUCCESS</span>) but actually delivers a very small amount of the currency specified.
3. The vulnerable institution reads the transaction's <span class="code-snippet">Amount</span> field without looking at the <span class="code-snippet">Flags</span> field or <span class="code-snippet">delivered_amount</span> metadata field.
4. The vulnerable institution credits the malicious actor in an external system, such as the institution's own ledger, for the full <span class="code-snippet">Amount</span>, despite only receiving a much smaller <span class="code-snippet">delivered_amount</span> in the XRP Ledger.
5. The malicious actor withdraws as much of the balance as possible to another system before the vulnerable institution notices the discrepancy.
    - Malicious actors usually prefer to convert the balance to another crypto-currency such as Bitcoin, because blockchain transactions are usually irreversible. With a withdrawal to a fiat currency system, the financial institution may be able to reverse or cancel the transaction several days after it initially executes.
    - In the case of an exchange, the malicious actor can also withdraw an XRP balance directly back into the XRP Ledger.

In the case of a merchant, the order of operations is slightly different, but the concept is the same:

1. The malicious actor requests to buy a large amount of goods or services.
2. The vulnerable merchant invoices the malicious actor for the price of those goods and services.
3. The malicious actor sends a Payment transaction to the merchant. This transaction has a large <span class="code-snippet">Amount</span> field and has the **<span class="code-snippet">tfPartialPayment</span>** flag enabled.
4. The partial payment succeeds (result code <span class="code-snippet">tesSUCCESS</span>) but delivers only a very small amount of the currency specified.
5. The vulnerable merchant reads the transaction's <span class="code-snippet">Amount</span> field without looking at the <span class="code-snippet">Flags</span> field or <span class="code-snippet">delivered_amount</span> metadata field.
6. The vulnerable merchant treats the invoice as paid and provides the goods or services to the malicious actor, despite only receiving a much smaller <span class="code-snippet">delivered_amount</span> in the XRP Ledger.
7. The malicious actor uses, resells, or absconds with the goods and services before the merchant notices the discrepancy.

### Further Mitigations

Using [the <span class="code-snippet">delivered_amount</span> field](#the-delivered_amount-field) when processing incoming transactions is enough to avoid this exploit. Still, additional proactive business practices can also avoid or mitigate the likelihood of this and similar exploits. For example:

- Add additional sanity checks to your business logic for processing withdrawals. Never process a withdrawal if the total balance you hold in the XRP Ledger does not match your expected assets and obligations.
- Follow "Know Your Customer" guidelines and strictly verify your customers' identities. You may be able to recognize and block malicious users in advance, or pursue legal action against a malicious actor who exploits your system.


## See Also

- **Tools:**
    - [Transaction Sender](tx-sender.html)
- **Concepts:**
    - [Transaction Basics](transaction-basics.html)
- **Tutorials:**
    - [Look Up Transaction Results](look-up-transaction-results.html)
    - [Monitor Incoming Payments with WebSocket](monitor-incoming-payments-with-websocket.html)
    - [Use Specialized Payment Types](use-specialized-payment-types.html)
    - [List XRP as an Exchange](list-xrp-as-an-exchange.html)
- **References:**
    - [Payment transaction][]
    - [Transaction Metadata](transaction-metadata.html)
    - [account_tx method][]
    - [tx method][]

<!--{# common link defs #}-->
{% include '_snippets/rippled-api-links.md' %}			
{% include '_snippets/tx-type-links.md' %}			
{% include '_snippets/rippled_versions.md' %}
