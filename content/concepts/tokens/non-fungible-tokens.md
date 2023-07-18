---
html: non-fungible-tokens.html
parent: tokens.html
blurb: Introduction to XRPL NFTs.
labels:
 - Non-fungible Tokens, NFTs
---

# Non-Fungible Tokens

The XRP Ledger natively supports non-fungible tokens (NFTs, or “nifties” in the vernacular).  Non-fungible tokens serve to encode ownership of unique physical, non-physical, or purely digital goods, such as works of art or in-game items.

_(Added by the [NonFungibleTokensV1_1 amendment][].)_


## Background

The XRP Ledger offers support for [tokens](tokens.html). Such assets are, primarily, fungible.

> Fun·gi·ble /´fənjəbəl/ (adj) <!-- SPELLING_IGNORE: fənjəbəl -->
>
> 1. able to replace or be replaced by another identical item; mutually interchangeable. <!-- STYLE_OVERRIDE: identical -->

Fungible tokens can be traded between users for XRP or other issued assets on the XRP Ledger's decentralized exchange. This makes them ideal for payments.


A good example of a fungible item might be a postage stamp. If you are standing around in 1919 and need to send a letter by airmail, you would buy a 24-cent stamp and affix it to your envelope. If you lost that stamp, you could use a different 24-cent stamp or use 2 10-cent stamps and 2 2-cent stamps. Very fungible.

![Jenny Stamps](img/nft-concepts1.png "Jenny Stamps")

But since you are standing around in 1919, you might be offered 24-cent airmail stamps where the aeroplane on the stamp is accidentally printed upside down. These are the world famous “Inverted Jenny” stamps. Only 100 were circulated on a single sheet of stamps, making them extremely rare and sought after. The current value of each mint condition stamp is appraised at over $1.5 million dollars.

![Jenny Stamps](img/nft-concepts2.png "Jenny Stamps")

Those stamps cannot be replaced by an ordinary 24-cent stamp. They have become _non-fungible_.

To represent digital assets similar to these, use the XRP Ledger's Non-Fungible Tokens feature (sometimes referred to by its standards draft number, [XLS-20](https://github.com/XRPLF/XRPL-Standards/discussions/46)).


## NFTs on the XRP Ledger

On the XRP Ledger, a non-fungible token is represented as a [NFToken][] object. A <span class="code-snippet">NFToken</span> is a unique, indivisible unit that is not used for payments. Users can mint (create), hold, buy, sell, and burn (destroy) such tokens.

The ledger stores up to 32 <span class="code-snippet">NFToken</span> objects owned by the same account in a single [NFTokenPage object][] to save space. As a result, the owner's [reserve requirement](reserves.html) for <span class="code-snippet">NFToken</span> objects only increases when the ledger needs to make a new page to store additional tokens.

Accounts can also name a broker, or "Authorized Minter", who can mint and sell <span class="code-snippet">NFToken</span> objects on their behalf.

<span class="code-snippet">NFToken</span> objects have several settings that are defined when the token is minted and cannot be changed later. These include:

- Various identifying data that uniquely defines the token.
- Whether the issuer can burn the token regardless of who currently holds it.
- Whether the holder of the token can transfer it to others. (The <span class="code-snippet">NFToken</span> can always be sent to or from the issuer directly.)
    - If transfers are allowed, the issuer can charge a transfer fee as a percentage of the sale price.
- Whether the holder can sell the <span class="code-snippet">NFToken</span> for [fungible token](tokens.html) amounts, or only for XRP.


## <span class="code-snippet">NFToken</span> Lifecycle

Anyone can create a new <span class="code-snippet">NFToken</span> using the [NFTokenMint transaction][] type. The <span class="code-snippet">NFToken</span> lives on the [NFTokenPage object][] of the issuing account. Either the owner or an interested party can send a [NFTokenCreateOffer transaction][] to propose buying or selling the <span class="code-snippet">NFToken</span>; the ledger tracks the proposed transfer as a [NFTokenOffer object][], and deletes the <span class="code-snippet">NFTokenOffer</span> when either side accepts or cancels the offer. If the <span class="code-snippet">NFToken</span> is transferable, it can be traded multiple times between accounts.

You can destroy a <span class="code-snippet">NFToken</span> you own using the [NFTokenBurn transaction][]. If the issuer minted the token with <span class="code-snippet">tfBurnable</span> flag enabled, the issuer can also burn the token regardless of the current owner. (This could be useful, for example, for a token that represents a ticket to an event which is used up at some point.)

![The NFT Lifecycle](img/nft-lifecycle.png "NFT Lifecycle Image")

For more info about transferring <span class="code-snippet">NFToken</span> objects, see [Trading NFTokens on the XRP Ledger](non-fungible-token-transfers.html).

<!--{# common link defs #}-->
{% include '_snippets/rippled-api-links.md' %}			
{% include '_snippets/tx-type-links.md' %}			
{% include '_snippets/rippled_versions.md' %}
