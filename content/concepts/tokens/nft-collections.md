---
html: nft-collections.html
parent: non-fungible-tokens.html
blurb: You can mint NFTs as collections using the NFT Taxon field.
labels:
 - Non-fungible Tokens, NFTs
---
# Minting NFTs into Collections

You can use the <span class="code-snippet">NFTokenTaxon</span> field to group NFTs into collections. As the minter, you can choose any numeric value from <span class="code-snippet">0x0</span> to <span class="code-snippet">0xFFFFFFFF</span>and assign it to NFTs as you create them. The significance of the taxon is entirely up to you.

For example, for your first collection, you might set the <span class="code-snippet">NFTokenTaxon</span> to <span class="code-snippet">1</span>. You might have a collection of NFTs with taxon values of <span class="code-snippet">316</span>, <span class="code-snippet">420</span>, or <span class="code-snippet">911</span>. You might use taxons that start with a digit to indicate the type of NFT (for example, all Real Estate NFTs have a taxon that starts with <span class="code-snippet">2</span>).

While the <span class="code-snippet">NFTokenTaxon</span> field is required, you can set the value to <span class="code-snippet">0</span> if you don't intend to create a collection.

See [NFTokenTaxon](nftoken.html#nftokentaxon).

