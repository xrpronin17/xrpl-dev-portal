---
html: xrpljs2-migration-guide.html
blurb: JavaScriptコードをより新しいクライアントライブラリに移行する方法を学びましょう。
parent: https://js.xrpl.org/
---
# ripple-lib 1.xからxrpl.js 2.xへの移行ガイド

**ripple-lib** (1.x)ライブラリからJavaScript / TypeScriptコードを移行し、代わりにXRP Ledger用**xrpl.js** (2.x)ライブラリーを使用するためには、以下の手順に従ってください。

**ヒント:** 必要な場合は、依然[legacy 1.x "RippleAPI"用ドキュメンテーション](https://github.com/XRPLF/xrpl.js/blob/1.x/docs/index.md)にアクセスできます。

## 差異の概略

xrpl.js v2.0では、多くのフィールドと機能に"新しい"名前があります。より正確には、xrpl.jsは現在、[HTTP / WebSocket APIs](http-websocket-apis.html)と同じ名前を使用しています。XRP Ledgerで実行可能な"OfferCancel"のような[トランザクションタイプ](transaction-types.html)をライブラリが使用する場所では、"orderCancellation"オブジェクトのようなripple-libに特有の構造はなくなりました。ripple-lib 1.xでこれらの構造をリターンする多くのAPIメソッドはなくなりました。2.0では、WebSocket APIと同じフォーマットでリクエスト、レスポンスを行います。

ripple-lib 1.xからの包括的な<span class="code-snippet">RippleAPI</span>クラスもなくなりました。xrpl.js 2.xでは、ネットワーク運用のための<span class="code-snippet">Client</span>クラスがあり、その他全ての運用は厳格にオフラインです。アドレスとキーのための新しい<span class="code-snippet">Wallet</span>クラス、また、トップレベルの<span class="code-snippet">xrpl</span>オブジェクトの下にその他のクラスとプロパティがあります。
## 定型文での比較

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


## バリデーション結果

デフォルトでは、ripple-lib 1.xにおけるほとんどのメソッドは、[コンセンサスプロセス](consensus.html)によって検証された最終結果をリターンするのみでした。xrpl.jsと同等の多くのメソッドは、WebSocket APIをコールするために[<span class="code-snippet">Client.request()</span>メソッド](https://js.xrpl.org/classes/Client.html#request)を使用します。WebSocket APIにおいて、XRP Ledgerサーバーのデフォルト設定では、検証済みデータだけはなく未検証のデータを含むことがあります。

[分散型取引所](decentralized-exchange.html)の状態を調べる時のように、完了見込みの多数のトランザクション結果が保留中であるため、現時点のオープンレジャーを使用したい場合があります。また、完了したトランザクション結果を取り込んだ検証済みのレジャーを使用したい場合もあります。

xrpl.js 2.0が<span class="code-snippet">Client.request()</span>を使用してAPIリクエストをする際、明確に[使用するレジャー番号を指定する](basic-data-types.html#specifying-ledgers)必要があります。例えば、最新の_検証済みレジャー_を使用してトラストラインを調べるためには:

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


## トランザクションの送信

xrpl.jsには、トランザクションの署名および送信のための、また、XRP Ledgerブロックチェーンのトランザクション最終結果の確認を待機するための特有の補助機能があります:

- トランザクション送信および[最終結果](finality-of-results.html)の待機のために<span class="code-snippet">submitAndWait()</span>を使用します。トランザクションが検証された場合、これは[txメソッド][]レスポンスにリゾルブし、そうでない場合、例外処理(exception)となります。例外処理(exception)は、トランザクションが検証されなかったことを保証しません。例えば、サーバーに[より大きなギャップ](reliable-transaction-submission.html#ledger-gaps)がある場合、トランザクションは、そのギャップの中で検証される可能性があります。
- 即時の送信およびリターンのために<span class="code-snippet">submit()</span>を使用します。これは[submitメソッド][]レスポンスにリゾルブし、仮の(最終ではない)結果を表示します。もしXRP Ledgerサーバーへのトランザクション送信に問題があった場合、このメソッドは例外処理(exception)のみとなります。

どちらのメソッドに関しても、準備済みトランザクション説明と[<span class="code-snippet">Wallet</span>インスタンス](#キーおよびウォレット)をパスすることによって、署名済みトランザクションをメソッドに直接パス、もしくは、送信直前にトランザクションに署名することができます。

```js
const tx_json = await client.autofill({
  "TransactionType": "AccountSet",
  "Account": wallet.address, // "wallet"はWalletクラスのインスタンス
  "SetFlag": xrpl.AccountSetAsfFlags.asfRequireDest
})
try {
  const submit_result = await client.submitAndWait(tx_json, wallet)
  // submitAndWait() はトランザクションの結果が確定するまでreturnしません。
  // トランザクションがネットワークに確認されなかった場合、XrplErrorが発生します。
  // ディザスタリカバリには対応しません。
  console.log("Transaction result:", submit_result)
} catch(err) {
  console.log("Error submitting transaction:", err)
}
```

もしくは、トランザクション署名のためにwalletの<span class="code-snippet">sign</span>メソッドを、送信のために<span class="code-snippet">submitAndWait(tx_blob)</span>を使用することができます。
停電やその他災害から復旧させる[信頼できるトランザクションの送信](reliable-transaction-submission.html)のビルドに便利です。(ライブラリは単独でディザスタリカバリに対処しません。)

### LastLedgerSequenceのコントロール

ripple-lib 1.xでは、トランザクションを準備し、準備済みトランザクションの<span class="code-snippet">LastLedgerSequence</span>パラメータを、その時点で最新の検証済みレジャー番号 _以降_ のレジャー番号を指定する際、<span class="code-snippet">instructions.maxLedgerVersionOffset</span>を利用できました。2.0では、最新の検証済みレジャー番号を調べ、トランザクションのオートフィル前に<span class="code-snippet">LastLedgerSequence</span>を明確に指定することが可能です。

**xrpl.js 2.0:**

```js
const vli = await client.getLedgerIndex()

const prepared = await client.autofill({
  "TransactionType": "Payment",
  "Account": sender,
  "Amount": xrpl.xrpToDrops("50.2"),
  "Destination": "rPT1Sjq2YGrBMTttX4GZHjKu9dyfzbpAYe",
  "LastLedgerSequence": vli+75 // デフォルトの1分以内ではなく5分以内
})
```

旧メソッド同様、デフォルトでは、<span class="code-snippet">Client.autofill()</span>は合理的な<span class="code-snippet">LastLedgerSequence</span>値を提示します。<span class="code-snippet">LastLedgerSequence</span>フィールドが _ない_ トランザクションを準備するには、<span class="code-snippet">null</span>値の<span class="code-snippet">LastLedgerSequence</span>を提示します:

```js
const prepared = await client.autofill({
  "TransactionType": "Payment",
  "Account": sender,
  "Amount": xrpl.xrpToDrops("50.2"),
  "Destination": "rPT1Sjq2YGrBMTttX4GZHjKu9dyfzbpAYe",
  "LastLedgerSequence": null // トランザクションは有効期限切れになりません
})
```


## キーおよびウォレット

xrpl.js 2.0は、[暗号鍵](cryptographic-keys.html)の管理およびトランザクションの署名のために、新しい[<span class="code-snippet">Wallet</span>クラス](https://js.xrpl.org/classes/Wallet.html)を採用します。
これは、ripple-lib 1.xにおいてシードや秘密鍵を取得していた機能に代わるもので、多様なアドレス符号化やタスク生成も処理します。

### キーの生成

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

### シードおよび署名からの取得

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


## イベントおよびサブスクリプション

1.xでは、<span class="code-snippet">RippleAPI</span>クラスの<span class="code-snippet">.on()</span>メソッドを使用してレジャーイベントとAPIエラーにサブスクリプションできました。もしくは、<span class="code-snippet">.connection.on()</span>を使用して特定のWebSocketメッセージタイプにサブスクリプションできました。これらは、[<span class="code-snippet">Client.on()</span>メソッド](https://js.xrpl.org/classes/Client.html#on)に統合されました。さらに、XRP Ledgerサーバーに接続する際、クライアントライブラリは、自動的にレジャークローズイベントにサブスクリプションしなくなったため、ハンドラを追加するだけでなく、レジャークローズイベントを取得するために **明確に台帳ストリームにサブスクリプションする必要があります** 。

レジャークローズイベントにサブスクリプションするには、<span class="code-snippet">Client.(method)</span>を使用し、<span class="code-snippet">"streams": ["ledger"]</span>で[subscribeメソッド][]をコールします。イベントハンドラを追加するには、<span class="code-snippet">Client.on(event_type, callback)</span>を使用します。これらのコールは任意の順で実行可能です。

1.xからのRippleAPI特有の<span class="code-snippet">ledger</span>イベントタイプは削除され、代わりに、<span class="code-snippet">ledgerClosed</span>イベントを使用します。これらのイベントメッセージは同じデータを含んでいますが、フォーマットはWebSocket APIの[レジャーストリーム](subscribe.html#レジャーストリーム)メッセージに対応しています。

例:

**ripple-lib 1.x:**

```js
api.on("ledger", (ledger) => {
  console.log(`Ledger #${ledger.ledgerVersion} closed!
    It contains ${ledger.transactionCount} transaction(s) and has
    the ledger_hash ${ledger.ledgerHash}.`
  )
})
// "ledger"イベントはAPI接続が確立後自動的に開始します。
```

**xrpl.js 2.0:**

```js
client.on("ledgerClosed", (ledger) => {
  console.log(`Ledger #${ledger.ledger_index} closed!
    It contains ${ledger.txn_count} transaction(s) and has
    the ledger_hash ${ledger.ledger_hash}.`
  )
})
// ”ledgerClosed "イベントを取得するには、"ledger "ストリームを明示的にサブスクライブする必要があります。
client.request({
  "command": "subscribe",
  "streams": ["ledger"]
})
```


## 比較対照

ripple-lib 1.xでは、全てのメソッドとプロパティは、<span class="code-snippet">RippleAPI</span>クラスのインスタンスでした。xrpl.js 2.xでは、ライブラリの静的メソッドと特定のクラスに属するメソッドがあります。以下のテーブルにおいて、<span class="code-snippet">Client.method()</span>という表記方法は、<span class="code-snippet">method()</span>が<span class="code-snippet">Client</span>クラスのインスタンスに属することを意味します。

**注記: 以下のテーブルには、3カラムあります。縦にスクロールすると、全ての情報を確認できます。**

| RippleAPIインスタンスメソッド/プロパティ | xrpl.jsメソッド/プロパティ | 注記 |
|-------------------|----------------|---|
| <span class="code-snippet">new ripple.RippleAPI({server: url})</span> | [<span class="code-snippet">new xrpl.Client(url)</span>](https://js.xrpl.org/classes/Client.html#constructor) | 複数のサーバに接続するには<span class="code-snippet">xrpl.BroadcastClient([url1, url2, ..])</span> を使用してください。 |
| <span class="code-snippet">request(command, options)</span> | [<span class="code-snippet">Client.request(options)</span>](https://js.xrpl.org/classes/Client.html#request) | <ul><li>WebSocket API との一貫性を保つために <span class="code-snippet">command</span> フィールドを <span class="code-snippet">options</span> オブジェクトに移動しました。</li><li>1.x では、このメソッドの戻り値 (Promise がリゾルブしたとき) は <span class="code-snippet">result</span> オブジェクトのみでした。現在は、[WebSocket 応答フォーマット](response-formatting.html) 全体が返されます。同様の値を得るには、戻り値の <span class="code-snippet">result</span> フィールドを読み取ってください。 |
| <span class="code-snippet">hasNextPage()</span> | [<span class="code-snippet">xrpl.hasNextPage(response)</span>](https://js.xrpl.org/modules.html#hasNextPage) | こちらもご覧ください。 [<span class="code-snippet">Client.requestNextPage()</span>](https://js.xrpl.org/classes/Client.html#requestNextPage) および [<span class="code-snippet">Client.requestAll()</span>](https://js.xrpl.org/classes/Client.html#requestAll) |
| <span class="code-snippet">requestNextPage()</span> | [<span class="code-snippet">Client.requestNextPage()</span>](https://js.xrpl.org/classes/Client.html#requestNextPage) | |
| <span class="code-snippet">computeBinaryTransactionHash()</span> | [<span class="code-snippet">xrpl.hashes.hashTx()</span>](https://js.xrpl.org/modules.html#hashes) | |
| <span class="code-snippet">classicAddressToXAddress()</span> | [<span class="code-snippet">xrpl.classicAddressToXAddress()</span>](https://js.xrpl.org/modules.html#classicAddressToXAddress) | 現在は、モジュールの静的メソッドです。 |
| <span class="code-snippet">xAddressToClassicAddress()</span> | [<span class="code-snippet">xrpl.xAddressToClassicAddress()</span>](https://js.xrpl.org/modules.html#xAddressToClassicAddress) | 現在は、モジュールの静的メソッドです。 |
| <span class="code-snippet">renameCounterpartyToIssuer(object)</span> | (削除済み - 注記カラムを参照) | xrpl.jsは常に<span class="code-snippet">issuer</span>を既に使用しているので、今後は必要ありません。 |
| <span class="code-snippet">formatBidsAndAsks()</span> | (削除済み - 注記カラムを参照) | No longer needed after changes to <span class="code-snippet">getOrderbook()</span>. |
| <span class="code-snippet">connect()</span> | [<span class="code-snippet">Client.connect()</span>](https://js.xrpl.org/classes/Client.html#connect) | |
| <span class="code-snippet">disconnect()</span> | [<span class="code-snippet">Client.disconnect()</span>](https://js.xrpl.org/classes/Client.html#disconnect) | |
| <span class="code-snippet">isConnected()</span> | [<span class="code-snippet">Client.isConnected()</span>](https://js.xrpl.org/classes/Client.html#isConnected) | |
| <span class="code-snippet">getServerInfo()</span> | (削除済み - 注記カラムを参照) | 代わりに [<span class="code-snippet">Client.request()</span>](https://js.xrpl.org/classes/Client.html#request) を使って [server_info メソッド][] を呼び出してください。 |
| <span class="code-snippet">getFee()</span> | (削除済み - 注記カラムを参照) | [トランザクションコスト][]を自動的に提供するには [<span class="code-snippet">Client.autofill()</span>](https://js.xrpl.org/classes/Client.html#autofill) を使ってください。または <span class="code-snippet">Client.request({"command": "fee"})</span> を使って、現在のトランザクションコスト ( _XRPのdrops_ ) についての情報を調べることができます。 |
| <span class="code-snippet">getLedgerVersion()</span> | [<span class="code-snippet">Client.getLedgerIndex()</span>](https://js.xrpl.org/classes/Client.html#getLedgerIndex) | |
| <span class="code-snippet">getTransaction()</span> | [<span class="code-snippet">Client.request()</span>](https://js.xrpl.org/classes/Client.html#request) | 代わりに [<span class="code-snippet">Client.request()</span>](https://js.xrpl.org/classes/Client.html#request) を使って [tx メソッド][] を呼び出してください。**警告:** <span class="code-snippet">getTransaction()</span> とは異なり、<span class="code-snippet">tx</span> メソッドは [検証されていない最終結果](#バリデーション結果) を返すことがあります。トランザクションに対してアクションを起こす前に、レスポンスオブジェクトの中に <span class="code-snippet">"validated": true</span> があるかどうかを必ず確認するようにしてください。 |
| <span class="code-snippet">getTransactions()</span> | (削除済み - 注記カラムを参照) | 代わりに [<span class="code-snippet">Client.request()</span>](https://js.xrpl.org/classes/Client.html#request) を使って [account_txメソッド][] を呼び出してください。 |
| <span class="code-snippet">getTrustlines()</span> |  (削除済み - 注記カラムを参照) | 代わりに [<span class="code-snippet">Client.request()</span>](https://js.xrpl.org/classes/Client.html#request) を使って [account_lines メソッド][] を呼び出してください。**警告:** <span class="code-snippet">getTrustlines()</span> とは異なり、<span class="code-snippet">account_lines</span> は [検証されていない最終結果](#バリデーション結果) を返すことがあります。 |
| <span class="code-snippet">getBalances()</span> | [<span class="code-snippet">Client.getBalances()</span>](https://js.xrpl.org/classes/Client.html#getBalances) | |
| <span class="code-snippet">getBalanceSheet()</span> | (削除済み - 注記カラムを参照) | 代わりに [<span class="code-snippet">Client.getBalances()</span>](https://js.xrpl.org/classes/Client.html#getBalances) を使うか、 [<span class="code-snippet">Client.request()</span>](https://js.xrpl.org/classes/Client.html#request) を使って [gateway_balancesメソッド][] を呼び出してください。 |
| <span class="code-snippet">getPaths()</span> | (削除済み - 注記カラムを参照) | 代わりに [<span class="code-snippet">Client.request()</span>](https://js.xrpl.org/classes/Client.html#request) を使って [ripple_path_findメソッド][] を呼び出してください。 |
| <span class="code-snippet">getOrders()</span> | (削除済み - 注記カラムを参照) | 代わりに [<span class="code-snippet">Client.request()</span>](https://js.xrpl.org/classes/Client.html#request) を使って [account_offers メソッド][] を呼び出してください。 |
| <span class="code-snippet">getOrderbook()</span> | [<span class="code-snippet">Client.getOrderbook()</span>](https://js.xrpl.org/classes/Client.html#getOrderbook) | |
| <span class="code-snippet">getSettings()</span> | (削除済み - 注記カラムを参照) | 代わりに [<span class="code-snippet">Client.request()</span>](https://js.xrpl.org/classes/Client.html#request) を使って [account_info メソッド][] を呼び出してください。個々のフラグ設定のブール値を取得するには、 <span class="code-snippet">Flags</span> フィールドで <span class="code-snippet">xrpl.parseAccountRootFlags()</span> を使用します。**警告:** <span class="code-snippet">getSettings()</span>とは異なり、<span class="code-snippet">account_info</span> は [検証されていない最終結果](#バリデーション結果)を返すことがあります。 |
| <span class="code-snippet">getAccountInfo(address, options)</span> | (削除済み - 注記カラムを参照) | 代わりに [<span class="code-snippet">Client.request()</span>](https://js.xrpl.org/classes/Client.html#request) を使って [account_info メソッド][] を呼び出してください。**警告:** <span class="code-snippet">getAccountInfo()</span> とは異なり、<span class="code-snippet">account_info</span> は [検証されていない最終結果](#バリデーション結果) を返すことがあります。 |
| <span class="code-snippet">getAccountObjects(address, options)</span> | (削除済み - 注記カラムを参照) | 代わりに [<span class="code-snippet">Client.request()</span>](https://js.xrpl.org/classes/Client.html#request) を使って [account_objects メソッド][] を呼び出してください。**警告:** <span class="code-snippet">getAccountObjects()</span> とは異なり、<span class="code-snippet">account_objects</span> は [検証されていない最終結果](#バリデーション結果) を返すことがあります。 |
| <span class="code-snippet">getPaymentChannel()</span> | (削除済み - 注記カラムを参照) | 代わりに [<span class="code-snippet">Client.request()</span>](https://js.xrpl.org/classes/Client.html#request) を使って [ledger_entry method](ledger_entry.html#get-paychannel-object) を呼び出してください。**警告:** <span class="code-snippet">getPaymentChannel()</span>とは異なり、<span class="code-snippet">ledger_entry</span> は [検証されていない最終結果](#バリデーション結果)を返す可能性があります。|
| <span class="code-snippet">getLedger()</span> | (削除済み - 注記カラムを参照) | <span class="code-snippet">Client.request()</span>](https://js.xrpl.org/classes/Client.html#request) を使って、正確に [ledger メソッド][] を呼び出してください。**渓谷:** <span class="code-snippet">getLedger()</span>とは異なり、<span class="code-snippet">ledger</span> は [検証されていない最終的なレジャー](#バリデーション結果)を返すことがあります。 |
| <span class="code-snippet">parseAccountFlags()</span> | [<span class="code-snippet">xrpl.parseAccountRootFlags()</span>](https://js.xrpl.org/modules.html#parseAccountRootFlags) | 現在は、モジュールの静的メソッドです。 |
| <span class="code-snippet">prepareTransaction()</span> | [<span class="code-snippet">Client.autofill()</span>](https://js.xrpl.org/classes/Client.html#autofill) | 詳しくは、[トランザクション送信](#トランザクションの送信)を参照してください。 |
| <span class="code-snippet">preparePayment()</span> | (削除済み - 注記カラムを参照) | [Paymentトランザクション][] を構築し、代わりに [<span class="code-snippet">Client.autofill()</span>](https://js.xrpl.org/classes/Client.html#autofill) を使用します。 |
| <span class="code-snippet">prepareTrustline()</span> | (削除済み - 注記カラムを参照) | [TrustSetトランザクション][]を構築し、代わりに [<span class="code-snippet">Client.autofill()</span>](https://js.xrpl.org/classes/Client.html#autofill) を使用します。 |
| <span class="code-snippet">prepareOrder()</span> | (削除済み - 注記カラムを参照) | [OfferCreateトランザクション][] を構築し、代わりに [<span class="code-snippet">Client.autofill()</span>](https://js.xrpl.org/classes/Client.html#autofill) を使用します。 |
| <span class="code-snippet">prepareOrderCancellation()</span> | (削除済み - 注記カラムを参照) | [OfferCancelトランザクション][]を構築し、[<span class="code-snippet">Client.autofill()</span>](https://js.xrpl.org/classes/Client.html#autofill)を代わりに使用することができます。 |
| <span class="code-snippet">prepareSettings()</span> | (削除済み - 注記カラムを参照) | ほとんどの設定には、代わりに [AccountSetトランザクション][]を構築します。通常キーをローテート変更するには、[SetRegularKeyトランザクション][]を作成します。マルチシグの設定を追加または更新するには、代わりに[SignerListSetトランザクション][]を構築してください。これら3つの場合とも、トランザクションを準備するために [<span class="code-snippet">Client.autofill()</span>](https://js.xrpl.org/classes/Client.html#autofill) を使用します。 |
| <span class="code-snippet">prepareEscrowCreation()</span> | (削除済み - 注記カラムを参照) | [EscrowCreateトランザクション][]を構築し、代わりに [<span class="code-snippet">Client.autofill()</span>](https://js.xrpl.org/classes/Client.html#autofill) を使用します。 |
| <span class="code-snippet">prepareEscrowCancellation()</span> | (削除済み - 注記カラムを参照) | [EscrowCancelトランザクション][]を構築し、代わりに [<span class="code-snippet">Client.autofill()</span>](https://js.xrpl.org/classes/Client.html#autofill) を使用します。 |
| <span class="code-snippet">prepareEscrowExecution()</span> | (削除済み - 注記カラムを参照) | [EscrowFinishトランザクション][] を構築し、代わりに [<span class="code-snippet">Client.autofill()</span>](https://js.xrpl.org/classes/Client.html#autofill) を使用します。 |
| <span class="code-snippet">preparePaymentChannelCreate()</span> | (削除済み - 注記カラムを参照) | [PaymentChannelCreateトランザクション][] を構築し、代わりに [<span class="code-snippet">Client.autofill()</span>](https://js.xrpl.org/classes/Client.html#autofill) を使用します。 |
| <span class="code-snippet">preparePaymentChannelClaim()</span> | (削除済み - 注記カラムを参照) | [PaymentChannelClaimトランザクション][] を構築し、代わりに [<span class="code-snippet">Client.autofill()</span>](https://js.xrpl.org/classes/Client.html#autofill) を使用します。 |
| <span class="code-snippet">preparePaymentChannelFund()</span> | (削除済み - 注記カラムを参照) | [PaymentChannelFundトランザクション][] を構築し、代わりに [<span class="code-snippet">Client.autofill()</span>](https://js.xrpl.org/classes/Client.html#autofill) を使用します。 |
| <span class="code-snippet">prepareCheckCreate()</span> | (削除済み - 注記カラムを参照) | [CheckCreateトランザクション][] を構築し、代わりに [<span class="code-snippet">Client.autofill()</span>](https://js.xrpl.org/classes/Client.html#autofill) を使用します。 |
| <span class="code-snippet">prepareCheckCancel()</span> | (削除済み - 注記カラムを参照) | [CheckCancelトランザクション][] を構築し、代わりに [<span class="code-snippet">Client.autofill()</span>](https://js.xrpl.org/classes/Client.html#autofill) を使用します。 |
| <span class="code-snippet">prepareCheckCash()</span> | (削除済み - 注記カラムを参照) | [CheckCashトランザクション][] を構築し、代わりに [<span class="code-snippet">Client.autofill()</span>](https://js.xrpl.org/classes/Client.html#autofill) を使用します。 |
| <span class="code-snippet">prepareTicketCreate()</span> | (削除済み - 注記カラムを参照) | [TicketCreateトランザクション][] を構築し、代わりに [<span class="code-snippet">Client.autofill()</span>](https://js.xrpl.org/classes/Client.html#autofill) を使用します。 |
| <span class="code-snippet">sign()</span> | [<span class="code-snippet">Wallet.sign()</span>](https://js.xrpl.org/classes/Wallet.html#sign) | 詳しくは、[キーおよびウォレット](#キーおよびウォレット)を参照してください。 |
| <span class="code-snippet">combine()</span> | [<span class="code-snippet">xrpl.multisign()</span>](https://js.xrpl.org/modules.html#multisign) | |
| <span class="code-snippet">submit()</span> | [<span class="code-snippet">Client.submit()</span>](https://js.xrpl.org/classes/Client.html#submit) | また、信頼性の高いトランザクション送信も可能になりました。詳細は、[トランザクション送信](#トランザクションの送信)を参照してください。 |
| <span class="code-snippet">generateXAddress()</span> | [<span class="code-snippet">xrpl.Wallet.generate()</span>](https://js.xrpl.org/classes/Wallet.html#generate) | <span class="code-snippet">xrpl.Wallet.generate()</span> で [<span class="code-snippet">Wallet</span> インスタンス](https://js.xrpl.org/classes/Wallet.html) を作成し、ウォレットのインスタンスで <span class="code-snippet">.getXAddress()</span> を呼び出して X-address を取得します。 詳しくは、[キーおよびウォレット](#キーおよびウォレット)を参照してください。 |
| <span class="code-snippet">generateAddress()</span> | [<span class="code-snippet">xrpl.Wallet.generate()</span>](https://js.xrpl.org/classes/Wallet.html#generate) | [<span class="code-snippet">Wallet</span>インスタンス](https://js.xrpl.org/classes/Wallet.html)を作成します。詳しくは、[キーおよびウォレット](#キーおよびウォレット)を参照してください。 |
| <span class="code-snippet">isValidAddress()</span> | [<span class="code-snippet">xrpl.isValidAddress()</span>](https://js.xrpl.org/modules.html#isValidAddress) | 現在は、モジュールの静的メソッドです。 |
| <span class="code-snippet">isValidSecret()</span> | [<span class="code-snippet">xrpl.isValidSecret()</span>](https://js.xrpl.org/modules.html#isValidSecret) | 現在は、モジュールの静的メソッドです。 |
| <span class="code-snippet">deriveKeypair()</span> | [<span class="code-snippet">xrpl.deriveKeypair()</span>](https://js.xrpl.org/modules.html#deriveKeypair) | 現在は、モジュールの静的メソッドです。 |
| <span class="code-snippet">deriveAddress()</span> | (削除済み - 注記カラムを参照) | 公開鍵からX Addressを取得するために <span class="code-snippet">xrpl.decodeXAddress()</span> を使用し、必要であれば <span class="code-snippet">xAddressToClassicAddress()</span> を使用してクラシックアドレスを取得します。 |
| <span class="code-snippet">generateFaucetWallet()</span> | [<span class="code-snippet">Client.fundWallet()</span>](https://js.xrpl.org/classes/Client.html#fundWallet) | <span class="code-snippet">on_testnet</span>ブール変数は削除されました。ライブラリは、接続しているネットワークに適したDevnetまたはTestnetのfaucetを自動的に選択します。オプションで [<span class="code-snippet">Wallet</span> インスタンス](https://js.xrpl.org/classes/Wallet.html) を提供すると、faucetは関連するアドレスに資金を供給/補充します。そうでなければ、メソッドは新しいWalletインスタンスを作成します。そうでなければ、このメソッドは新しいウォレットインスタンスを作成します。戻り値は現在、`{wallet: <object: Wallet instance>, balance: <str: XRP of drops>}という形のオブジェクトになります。 |
| <span class="code-snippet">signPaymentChannelClaim()</span> | [<span class="code-snippet">xrpl.signPaymentChannelClaim()</span>](https://js.xrpl.org/modules.html#signPaymentChannelClaim) | 現在は、モジュールの静的メソッドです。 |
| <span class="code-snippet">verifyPaymentChannelClaim()</span> | [<span class="code-snippet">xrpl.verifyPaymentChannelClaim()</span>](https://js.xrpl.org/modules.html#verifyPaymentChannelClaim) | 現在は、モジュールの静的メソッドです。 |
| <span class="code-snippet">computeLedgerHash()</span> | [<span class="code-snippet">xrpl.hashes.hashLedger()</span>](https://js.xrpl.org/modules.html#hashes) | |
| <span class="code-snippet">xrpToDrops()</span> | [<span class="code-snippet">xrpl.xrpToDrops()</span>](https://js.xrpl.org/modules.html#xrpToDrops) | 現在は、モジュールの静的メソッドです。 |
| <span class="code-snippet">dropsToXrp()</span> | [<span class="code-snippet">xrpl.dropsToXrp()</span>](https://js.xrpl.org/modules.html#dropsToXrp) | 現在は、モジュールの静的メソッドです。 |
| <span class="code-snippet">iso8601ToRippleTime()</span> | [<span class="code-snippet">xrpl.isoTimeToRippleTime()</span>](https://js.xrpl.org/modules.html#isoTimeToRippleTime) | 現在は、モジュールの静的メソッドです。 |
| <span class="code-snippet">rippleTimeToISO8601()</span> | [<span class="code-snippet">xrpl.rippleTimeToISOTime()</span>](https://js.xrpl.org/modules.html#rippleTimeToISOTime) | 現在は、モジュールの静的メソッドです。 また、新しいメソッド [<span class="code-snippet">rippleTimeToUnixTime()</span>](https://js.xrpl.org/modules.html#rippleTimeToUnixTime) を使うと、UNIXエポック 1970-01-01 00:00:00 UTC からのミリ秒単位のUNIXスタイルのタイムスタンプを取得することができます。 |
| <span class="code-snippet">txFlags.Universal.FullyCanonicalSig</span> | (削除済み - 注記カラムを参照) | [RequireFullyCanonicalSig amendment][]に伴い、不要となりました。 |
| <span class="code-snippet">txFlags.Payment.NoRippleDirect</span> | <span class="code-snippet">xrpl.PaymentFlags.tfNoDirectRipple</span> | |
| <span class="code-snippet">txFlags.Payment.PartialPayment</span> | <span class="code-snippet">xrpl.PaymentFlags.tfPartialPayment</span> | |
| <span class="code-snippet">txFlags.Payment.LimitQuality</span> | <span class="code-snippet">xrpl.PaymentFlags.tfLimitQuality</span> | |
| <span class="code-snippet">txFlags.OfferCreate.Passive</span> | <span class="code-snippet">xrpl.OfferCreateFlags.tfPassive</span> | |
| <span class="code-snippet">txFlags.OfferCreate.ImmediateOrCancel</span> | <span class="code-snippet">xrpl.OfferCreateFlags.tfImmediateOrCancel</span> | |
| <span class="code-snippet">txFlags.OfferCreate.FillOrKill</span> | <span class="code-snippet">xrpl.OfferCreateFlags.tfFillOrKill</span> | |
| <span class="code-snippet">txFlags.OfferCreate.Sell</span> | <span class="code-snippet">xrpl.OfferCreateFlags.tfSell</span> | |
| <span class="code-snippet">accountSetFlags</span> | <span class="code-snippet">xrpl.AccountSetAsfFlags</span> | モジュールレベルでEnumになりました。 |
| <span class="code-snippet">schemaValidator</span> | (削除済み - 注記カラムを参照) | TypeScriptを使用して、ほとんどの型を検証することができます。 |
| <span class="code-snippet">schemaValidate()</span> | (削除済み - 注記カラムを参照) | TypeScriptを使用して、ほとんどの型を検証することができます。 トランザクションオブジェクトの検証を行うために <span class="code-snippet">xrpl.validate(transaction)</span> を呼び出すこともできます。 |
| <span class="code-snippet">.on("ledger", callback)</span> | [<span class="code-snippet">Client.on("ledgerClosed", callback)</span>](https://js.xrpl.org/classes/Client.html#on) | **注意:** ledger streamもサブスクライブする必要があります。例と詳細については、[イベントとサブスクリプション](#イベントおよびサブスクリプション)を参照してください。 |
| <span class="code-snippet">.on("error", callback)</span> | [<span class="code-snippet">Client.on("error", callback)</span>](https://js.xrpl.org/classes/Client.html#on) | |
| <span class="code-snippet">.on("connected", callback)</span> | [<span class="code-snippet">Client.on("connected", callback)</span>](https://js.xrpl.org/classes/Client.html#on) | |
| <span class="code-snippet">.on("disconnected", callback)</span> | [<span class="code-snippet">Client.on("connected", callback)</span>](https://js.xrpl.org/classes/Client.html#on) | |


<!--{# common link defs #}-->
{% include '_snippets/rippled-api-links.md' %}			
{% include '_snippets/tx-type-links.md' %}			
{% include '_snippets/rippled_versions.md' %}
