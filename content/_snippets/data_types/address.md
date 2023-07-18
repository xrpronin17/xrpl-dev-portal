Accounts in the XRP Ledger are identified by an address in the XRP Ledger's [base58][] format. The address is derived from the account's master [public key](https://en.wikipedia.org/wiki/Public-key_cryptography), which is in turn derived from a secret key. An address is represented as a string in JSON and has the following characteristics:

* Between 25 and 35 characters in length
* Starts with the character <span class="code-snippet">r</span>
* Uses alphanumeric characters, excluding the number "<span class="code-snippet">0</span>" capital letter "<span class="code-snippet">O</span>", capital letter "<span class="code-snippet">I</span>", and lowercase letter "<span class="code-snippet">l</span>"
* Case-sensitive
* Includes a 4-byte checksum so that the probability of generating a valid address from random characters is approximately 1 in 2<sup>32</sup>

> **Note:** The XRP community has [proposed](https://github.com/XRPLF/XRPL-Standards/issues/6) an **X**-address format that "packs" a [destination tag](source-and-destination-tags.html) into the address. These addresses start with an <span class="code-snippet">X</span> (for the mainnet) or a <span class="code-snippet">T</span> (for the [testnet](parallel-networks.html)). Exchanges and wallets can use X-addresses to represent all the data a customer needs to know in one value. For more information, see the [X-address format site](https://xrpaddress.info/) and [codec](https://github.com/xrp-community/xrpl-tagged-address-codec).
>
> The XRP Ledger protocol only supports "classic" addresses natively, but many [client libraries](client-libraries.html) support X-addresses too.

{% if currentpage.md != "concept-accounts.md" %}
For more information, see [Accounts](accounts.html) and [base58 Encodings](base58-encodings.html).
{% endif %}
