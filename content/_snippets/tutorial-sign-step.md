The most secure way to sign a transaction is to [sign locally with a client library](set-up-secure-signing.html#local-signing-example). Alternatively, if you run your own <span class="code-snippet">rippled</span> node you can sign the transaction using the [sign method](sign.html), but this must be done through a trusted and encrypted connection, or through a local (same-machine) connection.

In all cases, note the signed transaction's identifying hash for later.
