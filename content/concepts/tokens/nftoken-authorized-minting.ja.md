---
name: 代理Mint
html: nftoken-authorized-minting.html
parent: non-fungible-tokens.html
blurb: NFTのMintを他のアカウントに代行してもらうことができます。
labels:
 - 非代替性トークン, NFT
---
# 他のアカウントでNFTを処理することを許可する

各アカウントは、自分に代わってNFTをMintすることができる認可されたMinterを最大一人設定することができまます。認可Minter機能を利用することで、クリエイターは別のアカウントにNFTをMintさせることができ、より多くのNFTを作ることに集中することができます。

## 認可Minterの割り当て

認可Minterを<span class="code-snippet">AccountSet</span>トランザクションで設定します。

``` json
tx_json = {
  "TransactionType": "AccountSet",
  "Account": "rrE5EgHN4DfjXhR9USecudHm7UyhTYq6m",
  "NFTokenMinter": "r3riWB2TDWRmwmT7FRKdRHjqm6efYu4s9C",
  "SetFlag": xrpl.AccountSetAsfFlags.asfAuthorizedNFTokenMinter
}
```

<span class="code-snippet">NFTokenMinter</span> 同じXRP Ledgerインスタンス上のアカウントのIDです。<span class="code-snippet">asfAuthorizedNFTokenMinter</span>フラグは<span class="code-snippet">NFTokenMinter</span>に指定するアカウントが<span class="code-snippet">Account</span>の代理としてNFTをMintすることを許可します。

*注記*: <span class="code-snippet">asfAuthorizedNFTokenMinter</span>フラグは<span class="code-snippet">AccountSet</span>トランザクションでのみ使用されます。これは、トランザクションがAccountRoot上のNFTokenMinterフィールドの存在または値に影響を与えるかどうかを示します。実際、AccountRootには対応するフラグはありません。

## 認可Minterの割り当て解除

認可Minterを削除するには、<span class="code-snippet">AccountSet</span>トランザクションを使用して、<span class="code-snippet">asfAuthorizedNFTokenMinter</span>フラグをクリアしてください。

``` json
tx_json = {
  "TransactionType": "AccountSet",
  "Account": "rrE5EgHN4DfjXhR9USecudHm7UyhTYq6m",
  "ClearFlag": xrpl.AccountSetAsfFlags.asfAuthorizedNFTokenMinter
}
```

## 他のアカウントのNFTをMintする

標準的な <span class="code-snippet">NFTokenMint</span> トランザクションを使用して、別のアカウントのNFTをMintすることができます。違いは、<span class="code-snippet">Issuer</span>フィールド、つまりNFTをMintするアカウントのIDを含める必要があることです。

```json
const transactionBlob = {
  "TransactionType": "NFTokenMint",
  "Account": "r3riWB2TDWRmwmT7FRKdRHjqm6efYu4s9C",
  "URI": xrpl.convertStringToHex("ipfs://bafybeigdyrzt5sfp7udm7hu76uh7y26nf4dfuylqabf3oclgtqy55fbzdi"),
  "Flags": 8,
  "TransferFee": 5000,
  "NFTokenTaxon": 0,
  "Issuer": "rrE5EgHN4DfjXhR9USecudHm7UyhTYq6m", // 別アカウントでMintする際に必要
}
```

NFTの所有者がNFTを売却した場合、取引手数料（売却額に対する割合）が<span class="code-snippet">Issuer</span>に指定したアカウントに送金されまれます。

