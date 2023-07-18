---
html: nftoken-authorized-minting.html
parent: non-fungible-tokens.html
blurb: You can assign another account to mint NFTs in your stead.
labels:
 - Non-fungible Tokens, NFTs
---
# Authorizing Another Account to Mint Your NFTs

Each account can have 0 or 1 authorized minter that can mint NFTs on its behalf. By authorizing a minter, a creator can allow a different account to mint NFTs for them, which allows them to focus on making more NFTs.

## Assigning an Authorized Minter

You set the authorized minter with an <span class="code-snippet">AccountSet</span> transaction.

``` json
tx_json = {
  "TransactionType": "AccountSet",    
  "Account": "rrE5EgHN4DfjXhR9USecudHm7UyhTYq6m",
  "NFTokenMinter": "r3riWB2TDWRmwmT7FRKdRHjqm6efYu4s9C",
  "SetFlag": xrpl.AccountSetAsfFlags.asfAuthorizedNFTokenMinter
}
```

<span class="code-snippet">NFTokenMinter</span> is an account ID of an account on the same XRP Ledger instance. The <span class="code-snippet">asfAuthorizedNFTokenMinter</span> flag authorizes the <span class="code-snippet">NFTokenMinter</span> account to mint NFTs on behalf of the <span class="code-snippet">Account</span>.

*Note*: The <span class="code-snippet">asfAuthorizedNFTokenMinter</span> flag is used only in the <span class="code-snippet">AccountSet</span> transaction. It indicates whether the transaction affects the presence or value of the NFTokenMinter field on an account root. Specifically, there is no corresponding flag on the AccountRoot.

## Unassigning an Authorized Minter

To remove an authorized minter, use the <span class="code-snippet">AccountSet</span> transaction and clear the <span class="code-snippet">asfAuthorizedNFTokenMinter</span> flag.

``` json
tx_json = {
  "TransactionType": "AccountSet",
  "Account": "rrE5EgHN4DfjXhR9USecudHm7UyhTYq6m",
  "ClearFlag": xrpl.AccountSetAsfFlags.asfAuthorizedNFTokenMinter
}
```

## Minting an NFT for Another Account

You mint tokens for another account using the standard <span class="code-snippet">NFTokenMint</span> transaction. The difference is that you must include the <span class="code-snippet">Issuer</span> field, the account ID of the account for which you are minting the NFT.

```json
const transactionBlob = {
  "TransactionType": "NFTokenMint",
  "Account": "r3riWB2TDWRmwmT7FRKdRHjqm6efYu4s9C",
  "URI": xrpl.convertStringToHex("ipfs://bafybeigdyrzt5sfp7udm7hu76uh7y26nf4dfuylqabf3oclgtqy55fbzdi"),
  "Flags": 8,
  "TransferFee": 5000,
  "NFTokenTaxon": 0,
  "Issuer": "rrE5EgHN4DfjXhR9USecudHm7UyhTYq6m", // Needed when minting for another
                                                 // account.
}
```

When you or a broker sells the NFT, the TransferFee (percentage of sale) is credited to your issuing account.

