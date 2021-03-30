# Air-Gaped Hardware wallet data format
## Simple summary
A standard to create an air-gaped hardware wallet URI for watch-only wallet
## Abstract
This standard defines how air-gaped hardware wallet description and request can be encoded with URI. This URI can then be shown as a QR code or stored into storage card for further usage.
## Knowledge
`single-account wallet`: A wallet described by one address.
`multi-account wallet`: A wallet described by extended public key, can derive different addresses by different HD path
`xfp`: The fingerprint of corresponding key of extended public key defined by [BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki#key-identifiers)

## Specification
#### Syntax
```
request: "ethereum-aghw" ":" target ("@" chain_id) ["/" function_name] ["?" parameters]

target: "watch-only" / wallet_id
wallet_id: STRING
chain_id: 1 * DIGIT
function_name: STRING
parameters = parameter *( "&" parameter)
parameter = key "=" value
key = STRING
value = STRING
```
#### Semantics
`target`: Should be "watch-only" when it is requested by air-gaped wallet. Should be wallet identifier when watch-only wallet want to request some data from an air-gaped wallet.
`wallet_id`: Should be `address` if air-gaped wallet is `single-account`. Should be `xfp/path` if air-gaped wallet is `multi-account`, e.g. For single-account wallet: `0x9858EfFD232B4033E47d90003D41EC34EcaEda94`; For multi-account wallet: `4cfc09de/m_44'_60'_0'_0_1`, it means which private key we should use in arriving operations.
`chain_id`: Optional, must be a decimal number and map onto a proper value at https://chainid.network/. Omitting it means that the wallet supports all chains so it is not needed to specify which chain to use.
`function_name`: The function can be called with `parameters`
`parameters`: The augments of one function, can be different with different `function_name`.

## Interacting usages
#### QR Code
Should encode the URI into UTF-8,  then encode with BC-UR.
#### File Transfer
Should write the URI directly into file.


## Steps and Usage
This section defines `target` and `functions` of different steps;
### Sync Wallet
#### `target`: "watch-only"
#### `functions`:
|function_name|parameters|description|
|:--|:--|:--|
|`introduce_multi`|`xfp: STRING`, `xpub: STRING`, `path: STRING`|`xfp`: the fingerprint of `xpub`, `xpub`: extended public key of multi-account wallet, starts with 'xpub'; `path`: the HD path of `xpub`, for `'/'` has special meaning in URI, we replace it with `'_'`, like `m_44'_60'_0'` |
|`introduce_single`|`address: STRING`|`address`: the address of single-account wallet|

#### example:
A multi-account wallet:
`aghw:watch-only@1/introduce_multi?xfp=4CFC09DE&path=m_44'_60'_0'&xpub=xpub6CEhpxfVPfb4KuviDUeyiXvPchbZQ9gypJ5Fx9UmFmh74X3s4A7j6St68K8bDfm7AXidVc1TKsniZ7KKJ2Jb8KkgP9KGt8yv5sx6VuXm6LD`
A single-account wallet:
`aghw:watch-only/introduce_single?address=0x9858EfFD232B4033E47d90003D41EC34EcaEda94`

### Request signing
#### `target`: wallet_id
#### `functions`:
|function_name|parameters|description|
|:--|:--|:--|
|sign_transaction|`rlp: STRING`|`rlp`: the rlp encoding of an ethereum transaction to be signed|
|sign_message|`message: STRING` |`message`: the message's hex to be signed|
|sign_hash|`hash: STRING` |`hash`: the hash to be signed|

### Submit signing result
#### `target`: "watch-only"
#### `functions`:
|function_name|parameters|description|
|:--|:--|:--|
|submit_signature|`signature: STRING`|`signature`: the  signature hex contains `r`,`s`,`v`|
