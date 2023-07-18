---
html: xrpljs2-migration-guide.html
blurb: Learn how to migrate JavaScript code to the newer client library format.
parent: https://js.xrpl.org/
---
# Migration Guide for ripple-lib 1.x to xrpl.js 2.x

Follow these instructions to migrate JavaScript / TypeScript code using the **ripple-lib** (1.x) library to use the **xrpl.js** (2.x) library for the XRP Ledger instead.

**Tip:** You can still access [documentation for the legacy 1.x "RippleAPI"](https://github.com/XRPLF/xrpl.js/blob/1.x/docs/index.md) if necessary.

## High-Level Differences

Many fields and functions have "new" names in xrpl.js v2.0; or more accurately, xrpl.js now uses the same names as the [HTTP / WebSocket APIs](http-websocket-apis.html). Structures that were unique to ripple-lib such as an <span class="code-snippet">orderCancellation</span> object are gone; in their place the library uses the XRP Ledger's native [transaction types](transaction-types.html) like "OfferCancel". Many API methods that return these structures in ripple-lib 1.x are gone; with 2.0, you make requests and get responses in the same format as in the WebSocket API.

The catch-all <span class="code-snippet">RippleAPI</span> class from ripple-lib 1.x is also gone. With xrpl.js 2.x, there's a <span class="code-snippet">Client</span> class to handle network operations, and all other operations are strictly offline. There's a new <span class="code-snippet">Wallet</span> class for addresses & keys, and other classes and properties under the top-level <span class="code-snippet">xrpl</span> object.

## Boilerplate Comparison

**ripple-lib 1.10.0:**

```js
const ripple = require('ripple-lib');

(async function() {
  const api = new ripple.RippleAPI({
    server: 'wss://xrplcluster.com'
  });

  await api.connect();

  // Your code here

  api.disconnect();
})();
```

**xrpl.js 2.0.0:**

```js
const xrpl = require("xrpl");

(async function() {
  const client = new xrpl.Client('wss://xrplcluster.com');

  await client.connect();

  // Your code here

  client.disconnect();
})();
```


## Validated Results

By default, most methods in ripple-lib 1.x only returned results that were validated by the [consensus process](consensus.html) and therefore final. <!-- STYLE_OVERRIDE: therefore --> The xrpl.js equivalents of many methods use the [<span class="code-snippet">Client.request()</span> method](https://js.xrpl.org/classes/Client.html#request) to call the WebSocket API, where the XRP Ledger server's default settings often use the current (pending) ledger to serve data which is not final.

Sometimes you want to use the current open ledger because it has the pending results of many transactions that are likely to succeed, such as when looking up the state of the [decentralized exchange](decentralized-exchange.html). In other cases, you want to use a validated ledger, which only incorporates the results of transactions that are finalized.

When making API requests with xrpl.js 2.0 using <span class="code-snippet">Client.request()</span>, you should explicitly [specify what ledger to use](basic-data-types.html#specifying-ledgers). For example, to look up trust lines using the latest _validated ledger_:

**ripple-lib 1.x:**

```js
const trustlines = await api.getTrustlines("rf1BiGeXwwQoi8Z2ueFYTEXSwuJYfV2Jpn")
console.log(trustlines)
```

**xrpl.js 2.0:**

```js
const trustlines = await client.request({
  "command": "account_lines",
  "account": "rf1BiGeXwwQoi8Z2ueFYTEXSwuJYfV2Jpn",
  "ledger_index": "validated"
})
console.log(trustlines.result)
```


## Transaction Submission

In xrpl.js, there are specific helper functions for signing and submitting transactions and waiting for the XRP Ledger blockchain to confirm those transactions' final outcomes:

- Use <span class="code-snippet">submitAndWait()</span> to submit a transaction and wait for its [final outcome](finality-of-results.html). If the transaction becomes validated, this resolves to a [tx method][] response; otherwise, it raises an exception. An exception does not guarantee that the transaction was not validated. For example, if the server has a [ledger gap](reliable-transaction-submission.html#ledger-gaps), then the transaction could have been validated in that gap.
- Use <span class="code-snippet">submit()</span> to submit and return immediately. This resolves to a [submit method][] response, which shows the preliminary (non-final) result. This method only raises an exception if there was a problem sending the transaction to the XRP Ledger server.

For both methods, you can pass a signed transaction to the method directly, or you can sign the transaction right before submitting, by passing prepared transaction instructions and a [<span class="code-snippet">Wallet</span> instance](#keys-and-wallets).

```js
const tx_json = await client.autofill({
  "TransactionType": "AccountSet",
  "Account": wallet.address, // "wallet" is an instance of the Wallet class
  "SetFlag": xrpl.AccountSetAsfFlags.asfRequireDest
})
try {
  const submit_result = await client.submitAndWait(tx_json, wallet)
  // submitAndWait() doesn't return until the transaction has a final result.
  // Raises XrplError if the transaction doesn't get confirmed by the network.
  // Does not handle disaster recovery.
  console.log("Transaction result:", submit_result)
} catch(err) {
  console.log("Error submitting transaction:", err)
}
```

Alternatively, you can use the <span class="code-snippet">sign</span> method of a wallet to sign a transaction and then use <span class="code-snippet">submitAndWait(tx_blob)</span> to submit it. This can be useful for building [reliable transaction submission](reliable-transaction-submission.html) that can recover from power outages and other disasters. (The library does not handle disaster recovery on its own.)

### Controlling LastLedgerSequence
<!-- SPELLING_IGNORE: lastledgersequence -->

In ripple-lib 1.x, you could specify a <span class="code-snippet">instructions.maxLedgerVersionOffset</span> when preparing a transaction to define the <span class="code-snippet">LastLedgerSequence</span> parameter of the prepared transaction as being some number of ledgers _after_ the latest validated one at the time. In 2.0, you can do this by looking up the latest validated ledger index, then specifying the <span class="code-snippet">LastLedgerSequence</span> explicitly before auto-filling the transaction.

**xrpl.js 2.0:**

```js
const vli = await client.getLedgerIndex()

const prepared = await client.autofill({
  "TransactionType": "Payment",
  "Account": sender,
  "Amount": xrpl.xrpToDrops("50.2"),
  "Destination": "rPT1Sjq2YGrBMTttX4GZHjKu9dyfzbpAYe",
  "LastLedgerSequence": vli+75 // gives ~5min, rather than the default ~1min
})
```

Like the old prepare methods, <span class="code-snippet">Client.autofill()</span> provides a reasonable <span class="code-snippet">LastLedgerSequence</span> value by default. To prepare a transaction _without_ a <span class="code-snippet">LastLedgerSequence</span> field, provide a <span class="code-snippet">LastLedgerSequence</span> with the value <span class="code-snippet">null</span>:

```js
const prepared = await client.autofill({
  "TransactionType": "Payment",
  "Account": sender,
  "Amount": xrpl.xrpToDrops("50.2"),
  "Destination": "rPT1Sjq2YGrBMTttX4GZHjKu9dyfzbpAYe",
  "LastLedgerSequence": null // Transaction never expires
})
```


## Keys and Wallets
<!-- STYLE_OVERRIDE: wallet -->

xrpl.js 2.0 introduces a new [<span class="code-snippet">Wallet</span> class](https://js.xrpl.org/classes/Wallet.html) for managing [cryptographic keys](cryptographic-keys.html) and signing transactions. This replaces functions that took seed or secret values in ripple-lib 1.x, and handles various address encoding and generation tasks as well.

### Generating Keys

**ripple-lib 1.x:**

```js
const api = new RippleAPI()
const {address, secret} = api.generateAddress({algorithm: "ed25519"})
console.log(address, secret)
// rJvMQ3cwtyrNpVJDTW4pZzLnGeovHcdE6E s████████████████████████████
```

**xrpl.js 2.0:**

```js
const wallet = xrpl.Wallet.generate("ed25519")
console.log(wallet)
// Wallet {
//   publicKey: 'ED872A4099B61B0C187C6A27258F49B421AC384FBAD23F31330E666A5F50E0ED7E',
//   privateKey: 'ED224D2BDCF6382030C7612654D2118C5CEE16344C81CB36EC7A01EC7D95C5F737',
//   classicAddress: 'rMV3CPSXAdRpW96bvvnSu4zHTZ6ETBkQkd',
//   seed: 's████████████████████████████'
// }
```

### Deriving from Seed and Signing

**ripple-lib 1.x:**

```js
const api = new RippleAPI()
const seed = 's████████████████████████████';
const keypair = api.deriveKeypair(seed)
const address = api.deriveAddress(keypair.publicKey)
const tx_json = {
  "Account": address,
  "TransactionType":"Payment",
  "Destination":"rPT1Sjq2YGrBMTttX4GZHjKu9dyfzbpAYe",
  "Amount":"13000000",
  "Flags":2147483648,
  "LastLedgerSequence":7835923,
  "Fee":"13",
  "Sequence":2
}
const signed = api.sign(JSON.stringify(tx_json), seed)
```

**xrpl.js 2.0:**

```js
const wallet = xrpl.Wallet.fromSeed('s████████████████████████████')
const tx_json = {
  "Account": wallet.address,
  "TransactionType":"Payment",
  "Destination":"rPT1Sjq2YGrBMTttX4GZHjKu9dyfzbpAYe",
  "Amount":"13000000",
  "Flags":2147483648,
  "LastLedgerSequence":7835923,
  "Fee":"13",
  "Sequence":2
}
const signed = wallet.sign(tx_json)
```


## Events and Subscriptions

In 1.x, you could subscribe to ledger events and API errors using the <span class="code-snippet">.on()</span> method of the <span class="code-snippet">RippleAPI</span> class; or you could subscribe to specific WebSocket message types using <span class="code-snippet">.connection.on()</span>. These have been merged into the [<span class="code-snippet">Client.on()</span> method](https://js.xrpl.org/classes/Client.html#on). Additionally, the client library no longer automatically subscribes to ledger close events when connecting to an XRP Ledger server. To get ledger close events, you still add a handler, but **you must also explicitly subscribe to the ledger stream**.

To subscribe to ledger close events, use <span class="code-snippet">Client.request(method)</span> to call the [subscribe method][] with <span class="code-snippet">"streams": ["ledger"]</span>. To attach event handlers, use <span class="code-snippet">Client.on(event_type, callback)</span>. You can make these calls in either order.

The RippleAPI-specific <span class="code-snippet">ledger</span> event type from 1.x has been removed; instead, use <span class="code-snippet">ledgerClosed</span> events. These event messages contain the same data, but the format matches the [Ledger Stream](subscribe.html#ledger-stream) messages in the WebSocket API.

Example:

**ripple-lib 1.x:**

```js
api.on("ledger", (ledger) => {
  console.log(`Ledger #${ledger.ledgerVersion} closed!
    It contains ${ledger.transactionCount} transaction(s) and has
    the ledger_hash ${ledger.ledgerHash}.`
  )
})
// "ledger" events happen automatically while API is connected.
```

**xrpl.js 2.0:**

```js
client.on("ledgerClosed", (ledger) => {
  console.log(`Ledger #${ledger.ledger_index} closed!
    It contains ${ledger.txn_count} transaction(s) and has
    the ledger_hash ${ledger.ledger_hash}.`
  )
})
// Must explicitly subscribe to the "ledger" stream to get "ledgerClosed" events
client.request({
  "command": "subscribe",
  "streams": ["ledger"]
})
```


## Reference of Equivalents

In ripple-lib 1.x all methods and properties were on instances of the <span class="code-snippet">RippleAPI</span> class. In xrpl.js 2.x, some methods are static methods of the library and some methods belong to specific classes. In the following table, the notation <span class="code-snippet">Client.method()</span> means that <span class="code-snippet">method()</span> belongs to instances of the <span class="code-snippet">Client</span> class.

**Note: The following table has 3 columns. You may need to scroll horizontally to see all the information.**

| RippleAPI instance method / property | xrpl.js method / property | Notes |
|-------------------|----------------|---|
| <span class="code-snippet">new ripple.RippleAPI({server: url})</span> | [<span class="code-snippet">new xrpl.Client(url)</span>](https://js.xrpl.org/classes/Client.html#constructor) | Use <span class="code-snippet">xrpl.BroadcastClient([url1, url2, ..])</span> to connect to multiple servers. |
| <span class="code-snippet">request(command, options)</span> | [<span class="code-snippet">Client.request(options)</span>](https://js.xrpl.org/classes/Client.html#request) | <ul><li>The <span class="code-snippet">command</span> field moved into the <span class="code-snippet">options</span> object for consistency with the WebSocket API.</li><li>In 1.x the return value of this method (when the Promise resolves) was only the <span class="code-snippet">result</span> object. Now it returns the whole [WebSocket response format](response-formatting.html); to get the equivalent value, read the <span class="code-snippet">result</span> field of the return value. |
| <span class="code-snippet">hasNextPage()</span> | [<span class="code-snippet">xrpl.hasNextPage(response)</span>](https://js.xrpl.org/modules.html#hasNextPage) | See also: [<span class="code-snippet">Client.requestNextPage()</span>](https://js.xrpl.org/classes/Client.html#requestNextPage) and [<span class="code-snippet">Client.requestAll()</span>](https://js.xrpl.org/classes/Client.html#requestAll) |
| <span class="code-snippet">requestNextPage()</span> | [<span class="code-snippet">Client.requestNextPage()</span>](https://js.xrpl.org/classes/Client.html#requestNextPage) | |
| <span class="code-snippet">computeBinaryTransactionHash()</span> | [<span class="code-snippet">xrpl.hashes.hashTx()</span>](https://js.xrpl.org/modules.html#hashes) | |
| <span class="code-snippet">classicAddressToXAddress()</span> | [<span class="code-snippet">xrpl.classicAddressToXAddress()</span>](https://js.xrpl.org/modules.html#classicAddressToXAddress) | Now a static method on the module. |
| <span class="code-snippet">xAddressToClassicAddress()</span> | [<span class="code-snippet">xrpl.xAddressToClassicAddress()</span>](https://js.xrpl.org/modules.html#xAddressToClassicAddress) | Now a static method on the module. |
| <span class="code-snippet">renameCounterpartyToIssuer(object)</span> | (Removed - see Notes column) | No longer needed because xrpl.js always uses <span class="code-snippet">issuer</span> already. |
| <span class="code-snippet">formatBidsAndAsks()</span> | (Removed - see Notes column) | No longer needed after changes to <span class="code-snippet">getOrderbook()</span>. |
| <span class="code-snippet">connect()</span> | [<span class="code-snippet">Client.connect()</span>](https://js.xrpl.org/classes/Client.html#connect) | |
| <span class="code-snippet">disconnect()</span> | [<span class="code-snippet">Client.disconnect()</span>](https://js.xrpl.org/classes/Client.html#disconnect) | |
| <span class="code-snippet">isConnected()</span> | [<span class="code-snippet">Client.isConnected()</span>](https://js.xrpl.org/classes/Client.html#isConnected) | |
| <span class="code-snippet">getServerInfo()</span> | (Removed - see Notes column) | Use [<span class="code-snippet">Client.request()</span>](https://js.xrpl.org/classes/Client.html#request) to call the [server_info method][] instead. |
| <span class="code-snippet">getFee()</span> | (Removed - see Notes column) | Use [<span class="code-snippet">Client.autofill()</span>](https://js.xrpl.org/classes/Client.html#autofill) to provide a sensible [transaction cost][] automatically, or use <span class="code-snippet">Client.request({"command": "fee"})</span> to look up information about the current transaction cost (in _drops of XRP_). |
| <span class="code-snippet">getLedgerVersion()</span> | [<span class="code-snippet">Client.getLedgerIndex()</span>](https://js.xrpl.org/classes/Client.html#getLedgerIndex) | |
| <span class="code-snippet">getTransaction()</span> | [<span class="code-snippet">Client.request()</span>](https://js.xrpl.org/classes/Client.html#request) | Use [<span class="code-snippet">Client.request()</span>](https://js.xrpl.org/classes/Client.html#request) to call the [tx method][] instead. **Warning:** Unlike <span class="code-snippet">getTransaction()</span>, the <span class="code-snippet">tx</span> method can return [results that are not validated and final](#validated-results). Be sure to look for <span class="code-snippet">"validated": true</span> in the response object before taking action in response to a transaction. |
| <span class="code-snippet">getTransactions()</span> | (Removed - see Notes column) | Use [<span class="code-snippet">Client.request()</span>](https://js.xrpl.org/classes/Client.html#request) to call the [account_tx method][] instead. |
| <span class="code-snippet">getTrustlines()</span> |  (Removed - see Notes column) | Use [<span class="code-snippet">Client.request()</span>](https://js.xrpl.org/classes/Client.html#request) to call [account_lines method][] instead. **Warning:** Unlike <span class="code-snippet">getTrustlines()</span>, <span class="code-snippet">account_lines</span> can return [results that are not validated and final](#validated-results). |
| <span class="code-snippet">getBalances()</span> | [<span class="code-snippet">Client.getBalances()</span>](https://js.xrpl.org/classes/Client.html#getBalances) | |
| <span class="code-snippet">getBalanceSheet()</span> | (Removed - see Notes column) | Use [<span class="code-snippet">Client.getBalances()</span>](https://js.xrpl.org/classes/Client.html#getBalances) instead, or use [<span class="code-snippet">Client.request()</span>](https://js.xrpl.org/classes/Client.html#request) to call the [gateway_balances method][]. |
| <span class="code-snippet">getPaths()</span> | (Removed - see Notes column) | Use [<span class="code-snippet">Client.request()</span>](https://js.xrpl.org/classes/Client.html#request) to call [ripple_path_find method][] instead. |
| <span class="code-snippet">getOrders()</span> | (Removed - see Notes column) | Use [<span class="code-snippet">Client.request()</span>](https://js.xrpl.org/classes/Client.html#request) to call the [account_offers method][] instead. |
| <span class="code-snippet">getOrderbook()</span> | [<span class="code-snippet">Client.getOrderbook()</span>](https://js.xrpl.org/classes/Client.html#getOrderbook) | |
| <span class="code-snippet">getSettings()</span> | (Removed - see Notes column) | Use [<span class="code-snippet">Client.request()</span>](https://js.xrpl.org/classes/Client.html#request) to call the [account_info method][] instead. Use <span class="code-snippet">xrpl.parseAccountRootFlags()</span> on the <span class="code-snippet">Flags</span> field to get the boolean values of individual flag settings. **Warning:** Unlike <span class="code-snippet">getSettings()</span>, <span class="code-snippet">account_info</span> can return [results that are not validated and final](#validated-results). |
| <span class="code-snippet">getAccountInfo(address, options)</span> | (Removed - see Notes column) | Use [<span class="code-snippet">Client.request()</span>](https://js.xrpl.org/classes/Client.html#request) to call the [account_info method][] instead. **Warning:** Unlike <span class="code-snippet">getAccountInfo()</span>, <span class="code-snippet">account_info</span> can return [results that are not validated and final](#validated-results). |
| <span class="code-snippet">getAccountObjects(address, options)</span> | (Removed - see Notes column) | Use [<span class="code-snippet">Client.request()</span>](https://js.xrpl.org/classes/Client.html#request) to call the [account_objects method][] instead. **Warning:** Unlike <span class="code-snippet">getAccountObjects()</span>, <span class="code-snippet">account_objects</span> can return [results that are not validated and final](#validated-results). |
| <span class="code-snippet">getPaymentChannel()</span> | (Removed - see Notes column) | Use [<span class="code-snippet">Client.request()</span>](https://js.xrpl.org/classes/Client.html#request) to call the [ledger_entry method](ledger_entry.html#get-paychannel-object) instead. **Warning:** Unlike <span class="code-snippet">getPaymentChannel()</span>, <span class="code-snippet">ledger_entry</span> can return [results that are not validated and final](#validated-results). |
| <span class="code-snippet">getLedger()</span> | (Removed - see Notes column) | Use [<span class="code-snippet">Client.request()</span>](https://js.xrpl.org/classes/Client.html#request) to call the [ledger method][] exactly. **Warning:** Unlike <span class="code-snippet">getLedger()</span>, <span class="code-snippet">ledger</span> can return [ledgers that are not validated and final](#validated-results). |
| <span class="code-snippet">parseAccountFlags()</span> | [<span class="code-snippet">xrpl.parseAccountRootFlags()</span>](https://js.xrpl.org/modules.html#parseAccountRootFlags) | Now a static method on the module. |
| <span class="code-snippet">prepareTransaction()</span> | [<span class="code-snippet">Client.autofill()</span>](https://js.xrpl.org/classes/Client.html#autofill) | See [Transaction Submission](#transaction-submission) for details. |
| <span class="code-snippet">preparePayment()</span> | (Removed - see Notes column) | Construct a [Payment transaction][] and use [<span class="code-snippet">Client.autofill()</span>](https://js.xrpl.org/classes/Client.html#autofill) instead. |
| <span class="code-snippet">prepareTrustline()</span> | (Removed - see Notes column) | Construct a [TrustSet transaction][] and use [<span class="code-snippet">Client.autofill()</span>](https://js.xrpl.org/classes/Client.html#autofill) instead. |
| <span class="code-snippet">prepareOrder()</span> | (Removed - see Notes column) | Construct an [OfferCreate transaction][] and use [<span class="code-snippet">Client.autofill()</span>](https://js.xrpl.org/classes/Client.html#autofill) instead. |
| <span class="code-snippet">prepareOrderCancellation()</span> | (Removed - see Notes column) | Construct an [OfferCancel transaction][] and use [<span class="code-snippet">Client.autofill()</span>](https://js.xrpl.org/classes/Client.html#autofill) and use [<span class="code-snippet">Client.autofill()</span>](https://js.xrpl.org/classes/Client.html#autofill) instead. |
| <span class="code-snippet">prepareSettings()</span> | (Removed - see Notes column) | For most settings, construct an [AccountSet transaction][] instead. To rotate change a regular key, construct a [SetRegularKey transaction][]. To add or update multi-signing settings, construct a [SignerListSet transaction][] instead. In all three cases, use [<span class="code-snippet">Client.autofill()</span>](https://js.xrpl.org/classes/Client.html#autofill) to prepare the transaction. |
| <span class="code-snippet">prepareEscrowCreation()</span> | (Removed - see Notes column) | Construct an [EscrowCreate transaction][] and use [<span class="code-snippet">Client.autofill()</span>](https://js.xrpl.org/classes/Client.html#autofill) instead. |
| <span class="code-snippet">prepareEscrowCancellation()</span> | (Removed - see Notes column) | Construct an [EscrowCancel transaction][] and use [<span class="code-snippet">Client.autofill()</span>](https://js.xrpl.org/classes/Client.html#autofill) instead. |
| <span class="code-snippet">prepareEscrowExecution()</span> | (Removed - see Notes column) | Construct an [EscrowFinish transaction][] and use [<span class="code-snippet">Client.autofill()</span>](https://js.xrpl.org/classes/Client.html#autofill) instead. |
| <span class="code-snippet">preparePaymentChannelCreate()</span> | (Removed - see Notes column) | Construct a [PaymentChannelCreate transaction][] and use [<span class="code-snippet">Client.autofill()</span>](https://js.xrpl.org/classes/Client.html#autofill) instead. |
| <span class="code-snippet">preparePaymentChannelClaim()</span> | (Removed - see Notes column) | Construct a [PaymentChannelClaim transaction][] and use [<span class="code-snippet">Client.autofill()</span>](https://js.xrpl.org/classes/Client.html#autofill) instead. |
| <span class="code-snippet">preparePaymentChannelFund()</span> | (Removed - see Notes column) | Construct a [PaymentChannelFund transaction][] and use [<span class="code-snippet">Client.autofill()</span>](https://js.xrpl.org/classes/Client.html#autofill) instead. |
| <span class="code-snippet">prepareCheckCreate()</span> | (Removed - see Notes column) | Construct a [CheckCreate transaction][] and use [<span class="code-snippet">Client.autofill()</span>](https://js.xrpl.org/classes/Client.html#autofill) instead. |
| <span class="code-snippet">prepareCheckCancel()</span> | (Removed - see Notes column) | Construct a [CheckCancel transaction][] and use [<span class="code-snippet">Client.autofill()</span>](https://js.xrpl.org/classes/Client.html#autofill) instead. |
| <span class="code-snippet">prepareCheckCash()</span> | (Removed - see Notes column) | Construct a [CheckCash transaction][] and use [<span class="code-snippet">Client.autofill()</span>](https://js.xrpl.org/classes/Client.html#autofill) instead. |
| <span class="code-snippet">prepareTicketCreate()</span> | (Removed - see Notes column) | Construct a [TicketCreate transaction][] and use [<span class="code-snippet">Client.autofill()</span>](https://js.xrpl.org/classes/Client.html#autofill) instead. |
| <span class="code-snippet">sign()</span> | [<span class="code-snippet">Wallet.sign()</span>](https://js.xrpl.org/classes/Wallet.html#sign) | See [Keys and Wallets](#keys-and-wallets) for details. |
| <span class="code-snippet">combine()</span> | [<span class="code-snippet">xrpl.multisign()</span>](https://js.xrpl.org/modules.html#multisign) | |
| <span class="code-snippet">submit()</span> | [<span class="code-snippet">Client.submit()</span>](https://js.xrpl.org/classes/Client.html#submit) | Reliable transaction submission is now also available; for details, see [Transaction Submission](#transaction-submission). |
| <span class="code-snippet">generateXAddress()</span> | [<span class="code-snippet">xrpl.Wallet.generate()</span>](https://js.xrpl.org/classes/Wallet.html#generate) | Create a [<span class="code-snippet">Wallet</span> instance](https://js.xrpl.org/classes/Wallet.html) with <span class="code-snippet">xrpl.Wallet.generate()</span> then call <span class="code-snippet">.getXAddress()</span> on the wallet instance to get an X-address. See [Keys and Wallets](#keys-and-wallets) for details. |
| <span class="code-snippet">generateAddress()</span> | [<span class="code-snippet">xrpl.Wallet.generate()</span>](https://js.xrpl.org/classes/Wallet.html#generate) | Creates a [<span class="code-snippet">Wallet</span> instance](https://js.xrpl.org/classes/Wallet.html). See [Keys and Wallets](#keys-and-wallets) for details. |
| <span class="code-snippet">isValidAddress()</span> | [<span class="code-snippet">xrpl.isValidAddress()</span>](https://js.xrpl.org/modules.html#isValidAddress) | Now a static method on the module. |
| <span class="code-snippet">isValidSecret()</span> | [<span class="code-snippet">xrpl.isValidSecret()</span>](https://js.xrpl.org/modules.html#isValidSecret) | Now a static method on the module. |
| <span class="code-snippet">deriveKeypair()</span> | [<span class="code-snippet">xrpl.deriveKeypair()</span>](https://js.xrpl.org/modules.html#deriveKeypair) | Now a static method on the module. |
| <span class="code-snippet">deriveAddress()</span> | (Removed - see Notes column) | Use <span class="code-snippet">xrpl.decodeXAddress()</span> to get an X-address from a public key, then use <span class="code-snippet">xAddressToClassicAddress()</span> to get the classic address if necessary. |
| <span class="code-snippet">generateFaucetWallet()</span> | [<span class="code-snippet">Client.fundWallet()</span>](https://js.xrpl.org/classes/Client.html#fundWallet) | The <span class="code-snippet">on_testnet</span> boolean has been removed; the library automatically picks the Devnet or Testnet faucet as appropriate for the network you're connected to. You can optionally provide a [<span class="code-snippet">Wallet</span> instance](https://js.xrpl.org/classes/Wallet.html) to have the faucet fund/refill the associated address; otherwise, the method creates a new Wallet instance. The return value now resolves to an object in the form <span class="code-snippet">{wallet: <object: Wallet instance>, balance: <str: drops of XRP>}</span> |
| <span class="code-snippet">signPaymentChannelClaim()</span> | [<span class="code-snippet">xrpl.signPaymentChannelClaim()</span>](https://js.xrpl.org/modules.html#signPaymentChannelClaim) | Now a static method on the module. |
| <span class="code-snippet">verifyPaymentChannelClaim()</span> | [<span class="code-snippet">xrpl.verifyPaymentChannelClaim()</span>](https://js.xrpl.org/modules.html#verifyPaymentChannelClaim) | Now a static method on the module. |
| <span class="code-snippet">computeLedgerHash()</span> | [<span class="code-snippet">xrpl.hashes.hashLedger()</span>](https://js.xrpl.org/modules.html#hashes) | |
| <span class="code-snippet">xrpToDrops()</span> | [<span class="code-snippet">xrpl.xrpToDrops()</span>](https://js.xrpl.org/modules.html#xrpToDrops) | Now a static method on the module. |
| <span class="code-snippet">dropsToXrp()</span> | [<span class="code-snippet">xrpl.dropsToXrp()</span>](https://js.xrpl.org/modules.html#dropsToXrp) | Now a static method on the module. |
| <span class="code-snippet">iso8601ToRippleTime()</span> | [<span class="code-snippet">xrpl.isoTimeToRippleTime()</span>](https://js.xrpl.org/modules.html#isoTimeToRippleTime) | Now a static method on the module. |
| <span class="code-snippet">rippleTimeToISO8601()</span> | [<span class="code-snippet">xrpl.rippleTimeToISOTime()</span>](https://js.xrpl.org/modules.html#rippleTimeToISOTime) | Now a static method on the module. You can also use the new method [<span class="code-snippet">rippleTimeToUnixTime()</span>](https://js.xrpl.org/modules.html#rippleTimeToUnixTime) to get a UNIX-style timestamp in milliseconds since the UNIX epoch of 1970-01-01 00:00:00 UTC. |
| <span class="code-snippet">txFlags.Universal.FullyCanonicalSig</span> | (Removed - see Notes column) | No longer needed following the [RequireFullyCanonicalSig amendment][]. |
| <span class="code-snippet">txFlags.Payment.NoRippleDirect</span> | <span class="code-snippet">xrpl.PaymentFlags.tfNoDirectRipple</span> | |
| <span class="code-snippet">txFlags.Payment.PartialPayment</span> | <span class="code-snippet">xrpl.PaymentFlags.tfPartialPayment</span> | |
| <span class="code-snippet">txFlags.Payment.LimitQuality</span> | <span class="code-snippet">xrpl.PaymentFlags.tfLimitQuality</span> | |
| <span class="code-snippet">txFlags.OfferCreate.Passive</span> | <span class="code-snippet">xrpl.OfferCreateFlags.tfPassive</span> | |
| <span class="code-snippet">txFlags.OfferCreate.ImmediateOrCancel</span> | <span class="code-snippet">xrpl.OfferCreateFlags.tfImmediateOrCancel</span> | |
| <span class="code-snippet">txFlags.OfferCreate.FillOrKill</span> | <span class="code-snippet">xrpl.OfferCreateFlags.tfFillOrKill</span> | |
| <span class="code-snippet">txFlags.OfferCreate.Sell</span> | <span class="code-snippet">xrpl.OfferCreateFlags.tfSell</span> | |
| <span class="code-snippet">accountSetFlags</span> | <span class="code-snippet">xrpl.AccountSetAsfFlags</span> | Now an Enum at the module level. |
| <span class="code-snippet">schemaValidator</span> | (Removed - see Notes column) | Use TypeScript to validate most types. |
| <span class="code-snippet">schemaValidate()</span> | (Removed - see Notes column) | Use TypeScript to validate most types. You can also call <span class="code-snippet">xrpl.validate(transaction)</span> to validate transaction objects. |
| <span class="code-snippet">.on("ledger", callback)</span> | [<span class="code-snippet">Client.on("ledgerClosed", callback)</span>](https://js.xrpl.org/classes/Client.html#on) | **Caution:** Must also subscribe to the ledger stream. For examples and details, see [Events and Subscriptions](#events-and-subscriptions). |
| <span class="code-snippet">.on("error", callback)</span> | [<span class="code-snippet">Client.on("error", callback)</span>](https://js.xrpl.org/classes/Client.html#on) | |
| <span class="code-snippet">.on("connected", callback)</span> | [<span class="code-snippet">Client.on("connected", callback)</span>](https://js.xrpl.org/classes/Client.html#on) | |
| <span class="code-snippet">.on("disconnected", callback)</span> | [<span class="code-snippet">Client.on("connected", callback)</span>](https://js.xrpl.org/classes/Client.html#on) | |


<!--{# common link defs #}-->
{% include '_snippets/rippled-api-links.md' %}			
{% include '_snippets/tx-type-links.md' %}			
{% include '_snippets/rippled_versions.md' %}
