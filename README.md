# NFT Subgraph Workshop


- [Contract to index](https://etherscan.io/address/0x5180db8f5c931aae63c74266b211f580155ecac8#readContract)
- [Google Slides for NFT Subgraph Development Workshop](https://docs.google.com/presentation/d/174VIStFqNiUTB5kV571ylJNXZLEPqSx8m7J0aydomTE/edit?usp=sharing)
- Questions: **[twitter.com/incrypto32](https://twitter.com/incrypto32)**

## Prerequisites

- Install graph-cli: `yarn global add @graphprotocol/graph-cli`

## First Steps

- [Find the contract on Etherscan](https://etherscan.io/address/0x5180db8f5c931aae63c74266b211f580155ecac8#readContract)
- [Find the contract creation transaction for startBlock](https://etherscan.io/tx/0x97b84c76fa9e178763956701ef2b05116dffd0193b10c2d181c4385e76ab7e53)
- Download the ABI


- Run this command to initialise the subgraph with events:

```bash
graph init \
    --studio \
    --protocol ethereum \
    --from-contract 0x5180db8f5c931aae63c74266b211f580155ecac8 \
    --index-events \
    --contract-name Cryptocoven \
    --network mainnet \
    cryptocoven crytocoven-subgraph
```

- Inspect the source
- Change startblock:
```
source:
    address: "0x5180db8f5c931aae63c74266b211f580155ecac8"
    abi: Cryptocoven
    startBlock: 11743743
```
– Create new subgraph on [Subgraph Studio](https://thegraph.com/studio/) named "Cryptocoven"
- `graph auth --studio ...`
- `yarn deploy`

## Remove unused Entities and Events

```graphql
# schema.graphql
type Transfer @entity {
  id: ID!
  from: Bytes! # address
  to: Bytes! # address
  tokenId: BigInt! # uint256
}
```

### Extend events

Make them immutable for performance improvements

```graphql
type Transfer @entity(immutable: true) {
  id: ID!
  from: Bytes! # address
  to: Bytes! # address
  tokenId: BigInt! # uint256
  timestamp: BigInt!
  blockNumber: BigInt!
}
```

Use `event.transaction.hash.toHex() + "-" + event.logIndex.toString()` as the id for events
```typescript
export function handleTransfer(event: TransferEvent): void {
  let entity = new Transfer(
    event.transaction.hash.toHex() + "-" + event.logIndex.toString()
  )
  entity.blockNumber = event.block.number;
  entity.timestamp = event.block.timestamp;
  entity.from = event.params.from
  entity.to = event.params.to
  entity.tokenId = event.params.tokenId
  entity.save()
}
```

### Store logical entities

- Identify the important entities: Token, Owner, Contract
- Link them

```graphql
type Transfer @entity(immutable: true) {
  id: ID!
  from: Owner!
  to: Owner!
  token: Token!
  timestamp: BigInt!
  blockNumber: BigInt!
}

type Token @entity {
  id: ID!
  owner: Owner
  uri: String
  transfers: [Transfer!]! @derivedFrom(field: "token")
  contract: Contract
}

type Owner @entity {
  id: Bytes! # Use bytes as ID
  ownedTokens: [Token!]! @derivedFrom(field: "owner")
  balance: BigInt
}

type Contract @entity {
  id: ID!
  name: String
  symbol: String
  totalSupply: BigInt
  mintedTokens: [Token!]! @derivedFrom(field: "contract")
}
```




## Other resources

- https://soulbound.xyz
- https://github.com/Developer-DAO/resources
- https://dev.to/dabit3/the-complete-guide-to-full-stack-ethereum-development-3j13
- https://github.com/itsjerryokolo/CryptoPunks
- https://github.com/dabit3/building-a-subgraph-workshop
- https://thegraph.com/docs/developer/quick-start
- https://thegraph.com/discord
