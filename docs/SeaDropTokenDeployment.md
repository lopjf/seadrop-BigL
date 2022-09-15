# Token Deployment

An example script to deploy a token contract is located at [DeployAndConfigureExampleToken.s.sol](../script/DeployAndConfigureExampleToken.s.sol). It can be run with `forge script script/DeployAndConfigureExampleToken.s.sol --rpc-url [any_network_alchemy_rpc_url] --broadcast -vvvv --private-key [priv_key] --etherscan-api-key [api_key] --verify --retries 10`

### ERC721SeaDrop

`ERC721SeaDrop` contains only an Owner role (assigned to the deployer of the contract) that has authorization for all methods.

To split responsibilities with an Administrator role who can set fee parameters, use [`ERC721PartnerSeaDrop`](#erc721partnerseadrop).

1. Deploy `src/ERC721SeaDrop.sol` with constructor args `string name, string symbol, address[] allowedSeaDrop`
   1. e.g. `forge create --rpc-url [rpc_url] src/ERC721SeaDrop.sol:ERC721SeaDrop --constructor-args "TokenTest1" "TEST1" \[seadrop_address\] --private-key [priv_key] --etherscan-api-key [api_key] --verify`
1. Set the token max supply with `token.setMaxSupply()`
1. Set the creator payout address with `token.updateCreatorPayoutAddress()`
1. Set the contract URI with `token.setContractURI()`
1. Set the base URI with `token.setBaseURI()`
1. Set the drop URI with `token.setDropURI()`
   1. See [Format of Drop URI](#format-of-drop-uri)
1. Optionally:
   1. Set the provenance hash for random metadata reveals with `token.setProvenanceHash()`
      1. Must be set before first token is minted
   1. Set an allow list drop stage with `token.updateAllowList()`
   1. Set a token gated drop stage with `token.updateTokenGatedDrop()`
   1. Add server-side signers with `token.updateSignedMintValidationParams()`
1. Set a public drop stage with `token.updatePublicDrop()`

### ERC721PartnerSeaDrop

`ERC721PartnerSeaDrop` is a token contract designed to split responsibilities between an Owner and Administrator.

1. Deploy `src/ERC721PartnerSeaDrop.sol` with constructor args `string name, string symbol, address administrator, address[] allowedSeaDrop`
   1. e.g. `forge create --rpc-url [rpc_url] src/ERC721PartnerSeaDrop.sol:ERC721PartnerSeaDrop --constructor-args "TokenTest1" "TEST1" [administrator_address] \[seadrop_address\] --private-key [priv_key] --etherscan-api-key [api_key] --verify`
1. Required to be sent by token Owner:
   1. Set the token max supply with `token.setMaxSupply()`
   1. Set the creator payout address with `token.updateCreatorPayoutAddress()`
   1. Set the contract URI with `token.setContractURI()`
   1. Set the base URI with `token.setBaseURI()`
1. Can be sent by token Owner or Administrator:
   1. Set the drop URI with `token.setDropURI()`
      1. See [Format of Drop URI](#format-of-drop-uri)
1. Optionally:
   1. Set the provenance hash for random metadata reveals with `token.setProvenanceHash()`
      1. Must be set before first token is minted
   1. Set an allow list drop stage with `token.updateAllowList()`
      1. See [Format of Allow List URI](#format-of-drop-uri)
   1. Set a token gated drop stage with `token.updateTokenGatedDrop()`
      1. Administrator must first initialize with feeBps.
1. Set a public drop stage with `token.updatePublicDrop()`
   1. Administrator must first initialize with feeBps.with `token.updatePublicDrop()`
   1. If `restrictFeeRecipients` is set to true, Administrator must set allowed fee recipients with `token.updateAllowedFeeRecipient()`

## Specifications

### Format of Drop URI

Follows the pattern of `tokenURI` — could be either on-chain data blob or external URI (IPFS) — that contains the metadata related to the drop and corresponding drop stages.

#### Example

```json
[
  {
    "name": "My Public Stage",
    "description": "My public stage description.",
    "promoImage": "https://google.com",
    "isPublic": true,
    "mintPrice": 1000000000000000000,
    "maxTotalMintableByWallet": 50,
    "maxTokenSupplyForStage": 5000,
    "startTime": 1659045594,
    "endTime": 1659045594,
    "feeBps": 10000
  },
  {
    "name": "My Token Gated Stage",
    "description": "My token gated stage description",
    "promoImage": "https://google.com",
    "isPublic": false,
    "allowedTokenAddresses": ["0x8a90cab2b38dba80c64b7734e58ee1db38b8992e"],
    "mintPrice": 1000000000000000,
    "maxTotalMintableByWallet": 5,
    "maxTokenSupplyForStage": 1000,
    "startTime": 1659043594,
    "endTime": 1659044594,
    "feeBps": 10000
  }
]
```

#### JSON Schema

```json
{
  "$schema": "http://json-schema.org/draft-04/schema#",
  "type": "object",
  "properties": {
    "name": {
      "type": "string"
    },
    "description": {
      "type": "string"
    },
    "promoImage": {
      "type": "string"
    },
    "isPublic": {
      "type": "boolean"
    },
    "allowListURI": {
      "type": "string"
    },
    "allowedTokenAddresses": {
      "type": "array",
      "items": {
        "type": "string"
      }
    },
    "mintPrice": {
      "type": "integer"
    },
    "maxTotalMintableByWallet": {
      "type": "integer"
    },
    "maxTokenSupplyForStage": {
      "type": "integer"
    },
    "startTime": {
      "type": "integer"
    },
    "endTime": {
      "type": "integer"
    },
    "feeBps": {
      "type": "integer"
    }
  }
}
```

### Format of Allow List URI

#### Example

```json
[
  {
    "address": "0xf0E16c071E2cd421974dCb76d9af4DeDB578E059",
    "mintPrice": 1000000000000000000,
    "maxTotalMintableByWallet": 10,
    "startTime": 1659045594,
    "endTime": 1659045594,
    "dropStageIndex": 1,
    "maxTokenSupplyForStage": 1000,
    "feeBps": 1000,
    "restrictFeeRecipients": true
  },
  {
    "address": "0x829bd824b016326a401d083b33d092293333a830",
    "mintPrice": 1000000000000000000,
    "maxTotalMintableByWallet": 5,
    "startTime": 1659045594,
    "endTime": 1659045594,
    "dropStageIndex": 2,
    "maxTokenSupplyForStage": 500,
    "feeBps": 1250,
    "restrictFeeRecipients": false
  }
]
```

#### JSON Schema

```json
{
  "$schema": "http://json-schema.org/draft-04/schema#",
  "type": "object",
  "properties": {
    "address": {
      "type": "string"
    },
    "mintPrice": {
      "type": "integer"
    },
    "maxTotalMintableByWallet": {
      "type": "integer"
    },
    "startTime": {
      "type": "integer"
    },
    "endTime": {
      "type": "integer"
    },
    "dropStageIndex": {
      "type": "integer"
    },
    "maxTokenSupplyForStage": {
      "type": "integer"
    },
    "feeBps": {
      "type": "integer"
    },
    "restrictFeeRecipients": {
      "type": "boolean"
    }
  }
}
```