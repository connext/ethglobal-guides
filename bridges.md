# Bridging Tokens and Data

## Background

### What is a Bridge?

A bridge (or interoperability network) is a system that relays assets and/or data between blockchains.

### The Bridging Landscape

One of the key challenges of blockchains & distributed systems is that they **always have tradeoffs.**

For bridges, historically this has meant that bridges must navigate compromises between:

1. **Trust-minimization:** The system does not introduce new trust/security assumptions beyond those of the underlying domains.
2. **Generalizeability:** The system supports passing around arbitrary data and not just funds.
3. **Extensibility:** The system can be deployed without lots of custom work for a variety of different types of underlying domains.
4. **Latency:** How quickly the data is delivered to the destination domain.

We can classify all bridges by their properties/tradeoffs:

#### Externally Verified Bridges

![alt text](https://github.com/connext/ethglobal-guides/blob/main/assets/external.png)

Externaly verified protocols rely on an external set of validators to relay data between chains. This is typically represented as an MPC system, oracle network, or threshold multisig.

These systems have low latency, support arbitrary data passing, and are easily portable between domains but you are now trusting the bridge verifiers to secure your system.

#### Natively Verified Bridges

![alt text](https://github.com/connext/ethglobal-guides/blob/main/assets/native.png)

Natively verified protocols are ones where all of the underlying chains’ own verifiers are fully validating data passing between chains. Typically this is done by running a light client of one chain in the VM of another chain and vice versa.

These systems are the most trust-minimized systems as they rely directly on domain validators for security, support arbitrary data, and have low latency. However, because the implementation of these systems is so deeply tied with consensus, they require custom implementations for each domain.

#### Locally Verified Bridges

![alt text](https://github.com/connext/ethglobal-guides/blob/main/assets/local.png)

Locally verified protocols are ones where only the parties involved in a given cross-domain interaction verify the interaction. Locally verified protocols turn the complex n-party verification problem into a much simpler set of 2-party interactions where each party verifies only their counterparty. This model works so long as both parties are economically adversarial — i.e. there’s no way for both parties to collude to take funds from the broader chain.

These protocols are fast, extensibile, and trust-minimized but cannot support arbitrary messages due to their adversarial nature. An NFT, for instance, would not be able to be 1:1 backed by a counterparty.

#### Optimistic Bridges

![alt text](https://github.com/connext/ethglobal-guides/blob/main/assets/optimistic.png)

Optimistic bridges, similar to optimistic rollups, use **fraud proofs** to ensure the validity of data relayed across chains. Every message that passes through an optimistic bridge remains in a “pending” state during the dispute window until it is considered valid. During this time, **watchers** can dispute the message if the data is incorrect.

While slow, these protocols are extensible, generalizable, and trust-minimmized.

## Bridge Security

There are a few different types of important security in the bridging ecosystem, each asking a specific question:

1. **Economic Security.** How much does it cost to corrupt the system?
   
2. **Implementation Security.** How complex is the system to implement? What does the security hygiene of implementers look like?
   
3. **Environmental Security.** How can the system withstand attacks on underlying domains?

### Security of Different Bridge Protocols

#### Economic Security

To maximize economic security, protocols must maximize the size and diversity of their validator to increase the cost and complexity of bribes.

| Bridge Type             | Corruption Costs           |
| ----------------------- | --------------------------------- |
| `External`     | `k-of-m of the external verifiers of the system` |
| `Native`       | `k-of-n of the underlying domain validators`                            |
| `Local`        | `2-of-2 counterparties`                    |
| `Optimistic`   | `m-of-m of the watchers in the system`     |

Externally validated bridges have the lowest economic security, while natively verified bridges have the highest security.

#### Implementation Security

To maximize implementation security, protocols should be incredibly simple and easy to implement.

| Bridge Type             | Complexity       |
| ----------------------- | --------------------------------- |
| `External`     | `Medium (requires offchain coordination)`  |
| `Native`       | `High (requires custom implementations)`   |
| `Local`        | `Medium (requires offchain coordination)`  |
| `Optimistic`   | `Low (standalone, portable components)`    |

While natively verified bridges have a high degree of economic security, their lack of extensibility increases the implementation complexity substantially.

In addition to keeping the protocol simple, implementation risk can be constrained by:

- Following secure development practices (contract audits, fuzzing, etc.)
- Native mechanisms to prevent and deter fraud (slashing, watchers, pausability, etc.)

#### Environmental Security

Bridges act as an oracle of information between chains, and must be able to preserve the integrity of state between chains with differing security thresholds (i.e. prevent 51% attacks from impacting multiple domains). While these types of attacks are difficult to detect onchain, they are trivial to detect offchain. 

To constrain environment risk, protocols should use latency to their advantage to allow system relayers to detect and respond to these types of attacks.

| Bridge Type             | Latency as a safety mechanism   |
| ----------------------- | --------------------------------- |
| `External`     | `Possible, but not required.`  |
| `Native`       | `Not possible, uses state directly.`   |
| `Local`        | `Possible, but at the expense of prolonged user lockups.`  |
| `Optimistic`   | `Exists, fraud windows are built-in`    |


## What is Connext?

Connext powers fast, trust-minimized communication between blockchains.

Our goal is to create a world where:

1. Users never need to know what chain or rollup they're on (unless they want to!)
2. Developers can build applications that utilize resources from many chains/rollups simultaneously

### Modular Interoperabilty

There is no monolithic architecture that will give bridges *all* of the desirable properties, but we can get close to the ideal outcome by modularizing the protocol stack:

| Layer                   | Protocol/Stakeholders             |
| ----------------------- | --------------------------------- |
| `Application Layer`     | `Crosschain Applications (xApps)` |
| `Liquidity Layer`       | `NXTP`                            |
| `Messaging Layer`       | `Rollup AMBs, IBC, XCMP, etc.`                       |
| `Transport Layer`       | `Connext Routers`                 |


Let's take a closer look at some of these:

1. xApps are applications built by developers like you!
2. NXTP is a *liquidity* layer and *developer interface* built by Connext on top of a messaging protocol.
3. The messaging layer uses a combination of an optimistic mechanism and rollup-native AMBs to pass data between evm domains.
4. Routers “short-circuit” delays in processing messages by immediately executing calls/providing liquidity on the destination chain. **This removes latency in all user-facing cases.**

A simple mental model is to think of Connext as the liquidity layer to an AMB messaging layer. Connext and AMBs, as well as optimistic bridges, together provide the two halves of the “ideal” solution for interoperability.

### What does the Connext flow look like?

![alt text](https://github.com/connext/ethglobal-guides/blob/main/assets/optimistic.png)

In this diagram we can observe two domains, "origin" and "destination", and two paths, "fast" and "slow".

In this case, let’s assume the intention of the user is to bridge tokens from the origin to the destination domain, and have access to fast liquidity.
Unlike bridging through an AMB alone, which often comes with several hours or days of latency for the user, Connext shortcuts this by allowing its routers to front you the capital, thereby taking on the liquidity risk for you while they wait for the tokens to move through the slow path. In cases where the messaging system delivers a bridge asset (i.e. minted asset), routers will also manage that complexity for you.

It's important to note that if the user is bridging data, or if the call needs to be authenticated by the caller, the bridging operation will always go through the slow path. This maintains data integrity by allowing the AMB verification process to complete.

### What can you build with Connext?

Some example use cases:

- Execute the outcome of **DAO votes** across chains
- Lock-and-mint or burn-and-mint **token bridging**
- **Aggregate DEX liquidity** across chains in a single seamless transaction
- Crosschain **vault zaps** and **vault strategy management**
- **Lend funds** on one chain and borrow on another
- Bringing **UniV3 TWAPs** to every chain without introducing oracles
- **NFT bridging** and chain-agnostic NFT marketplaces
- **Store data on Arweave/Filecoin** directly from within an Ethereum smart contract


### How do I interact with Connext?

Developers can call `xCall`, and the protocol will then splits actions between a Liquidity Layer (Connext) and a Messaging Layer.

You can interact with xCall in ways described in the `Send your first xCall` section of this document

### How do I interact with the Connext community?

Find our discord at https://discord.gg/pef9AyEhNz and join the `dev-hub` channel

## XCall Me Maybe

### The Entrypoint

The main entrypoint for interacting with the Connext protocol is `xcall`. This method kicks off a crosschain interaction, and all the user has to do is wait for it to complete on the destination chain. There are *no required user interactions past this transaction*!

Subgraphs are available to track the progress of all in-flight transactions.

### Usage

```solidity
function xcall(XCallArgs calldata _args)
```

---

#### XCallArgs

```solidity
struct XCallArgs {
  CallParams params;
  address transactingAsset;
  uint256 transactingAmount;
  uint256 originMinOut;
}
```
**`transactingAsset`**

Refers to the contract address of the asset that is to be bridged. This could be the adopted, local, or canonical asset (see [this](../faq#what-does-it-mean-when-referring-to-canonical-representation-and-adopted-assets) for an explanation of the different kinds of assets).

Usually a xApp will have a higher-level function wrapping `xcall` in which the asset will be passed as an argument. This allows users to specify which asset they want to work with.

If the `xcall` is calldata-only (e.g. doesn't bridge any funds), any registered asset can be used here as long as `amount: 0`.

**`transactingAmount`**

The amount of tokens to bridge specified in standard format (i.e. to send 1 USDC, a token with 10^18 decimals, you must specify the amount as `1000000000000000000`).

**`originMinOut`**

The minimum amount received after the swap from adopted -> local on the origin domain. This is the way to specify slippage due to the StableSwap Pool, if applicable. 

For example, to achieve 3% slippage tolerance this can be calculated as `(amount / 100) * 97`.

---

**`CallParams`**

```solidity
struct CallParams {
  address to;
  bytes callData;
  uint32 originDomain;
  uint32 destinationDomain;
  address agent;
  address recovery;
  bool forceSlow;
  bool receiveLocal;
  address callback;
  uint256 callbackFee;
  uint256 relayerFee;
  uint256 destinationMinOut;
}
```

**`to`**

Refers to an address on the destination chain. Whether it’s a user’s wallet or the address of another contract depends on the desired use of `xcall`. 

If the `xcall` is just used to bridge funds, then the user should be able to specify where the funds get sent to.

If the `xcall` is used to send calldata to a target contract, then this address must be the address of that contract. 

**`callData`**

In the case of bridging funds only, this should be empty (""). If calldata is sent, then the encoded calldata must be passed here. 

**`originDomain / destinationDomain`**

These refer to Domain IDs (*not* equivalent to “Chain IDs”).

**`agent`**

The address that is allowed to execute transactions on behalf of `to` on the destination domain. Usually this is a relayer's job but the user can specify an address (including themselves) to do it as well.

This cannot be `address(0)` if `receiveLocal: false`. Some address must be able to call `forceReceiveLocal` on [BridgeFacet.sol](https://github.com/connext/nxtp/blob/main/packages/deployments/contracts/contracts/core/connext/facets/BridgeFacet.sol) in case transfers are facing unfavorable slippage conditions for extended periods.

**`recovery`**

The address on destination domain that should receive funds if the execution fails. This ensures that funds are never lost with failed calls.

**`forceSlow`**

Since Solidity doesn't allow for default parameters, integrators must explicitly set this to `true` when using authenticated (slow path) calls.

If the `xcall` is unauthenticated (e.g. bridging funds), setting this to `true` allows users to force the `xcall` through the slow path and save on the 0.05% transaction fee levied by routers. This is an option for users who don’t care for speed to optimize on cost.

**`receiveLocal`**

Setting this to `true` allows users to receive the local bridge-flavored asset instead of the adopted asset on the destination domain. 

**`callback`**

The address of a contract that implements the [ICallback](https://github.com/connext/nxtp/blob/main/packages/deployments/contracts/contracts/core/promise/interfaces/ICallback.sol) interface. See the [detailed spec](https://github.com/connext/nxtp/discussions/883) for callback interaction. 

If the target contract doesn’t return anything, this field must be `address(0)`. Otherwise, the specified address must be a contract.

**`callbackFee`**

Similar to the relayerFee except this is for paying relayers on the callback execution. If `callback: address(0)`, then this must be `0`.

This fee is also bump-able from the origin domain.

**`relayerFee`**

This is a fee paid to relayers for relaying the transaction to the destination domain. The fee must be high enough to satisfy relayers’ cost conditions for relaying a transaction, which includes the gas fee plus a bit extra as incentive. This is paid in the origin domain’s native asset - it’s locked on the origin domain and eventually claimed by the relayer. 

Connext contracts will assert that the `relayerFee` matches what is sent in `msg.value` for the `xcall`. If, for any reason, the initial `relayerFee` is set too low, [BridgeFacet.sol](https://github.com/connext/nxtp/blob/main/packages/deployments/contracts/contracts/core/connext/facets/BridgeFacet.sol) has a `bumpTransfer` function that can be called on the origin domain to bump (increase) the initial fee until it’s sufficient for relayers.

**`destinationMinOut`**

The minimum amount received after the swap from local -> adopted on the destination domain. This is the way to specify slippage due to the StableSwap Pool, if applicable.

For example, to achieve 3% slippage tolerance this can be calculated as `(amount / 100) * 97`.

## Developer Resources

- Check out our [hacker kit](https://www.notion.so/connext/Connext-Hacker-Kit-febb25e70b6c4596a57a1e05b6df4b19) that we put together for hackathon participants
- Our [technical documentation](https://docs.connext.network) is a great starting point to read more into technical details
- Our [Connext Academy](https://connext.academy) is a community driven information hub to read more coverage
- Join our [Discord](https://discord.gg/pef9AyEhNz) to ask the team for help or to meet the community

### Existing Implementations / Guides
- With the [SDK Quickstart](https://docs.connext.network/developers/sdk/sdk-quickstart)
- With the [Solidity Quickstart](https://docs.connext.network/developers/contracts/contracts-quickstart)
- Using any language with a local [http-server](https://github.com/connext/nxtp/tree/main/packages/examples/sdk-server)
- A [bridge frontend implementation](https://github.com/connext/nxtp/tree/main/packages/examples/bridge-reference/) shows you how to quickly build a minimal bridge UI

