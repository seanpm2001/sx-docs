# Starknet specifics

### What's special here?

Starknet having its own language (Cairo) and being a ZK Rollup, there are some Starknet specific details. The Snapshot X Starknet implementation allows DAOs to govern their L1 DAO trustlessly, with very little fees and no user friction. Indeed, users do not have to bridge their tokens, nor do they need to create and install a Starknet wallet.

This is all handled seamlessly by the Snapshot X protocol by taking advantage of the existing native bridge, some storage proofs, and a transaction relayer.

### Overview

This diagram represents the flow of a proposal that will get executed on L1, with voters using their regular EVM account to vote.\


<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

[https://whimsical.com/storageproofs-7kjLqMvsR3okzFgMrad9Fu](https://whimsical.com/storageproofs-7kjLqMvsR3okzFgMrad9Fu)

### Storage proofs

In a nutshell, storage proofs are cryptographic proofs that a user had this token or that NFT in their wallet at a specific moment in time. We use them to compute the voting power of any user at the moment the proposal started, without the user having to bridge their tokens. This technology developed in collaboration with [Herodotus](https://herodotus.dev/), allows for a trustless balance verification that we can use to compute the voting power of users. The exact code for verifying a storage value can be found [here](https://github.com/snapshot-labs/sx-starknet/blob/develop/starknet/src/utils/single\_slot\_proof.cairo).

1. A user creates a proposal. The space registers it and [stores the start timestamp](https://github.com/snapshot-labs/sx-starknet/blob/03041771ba701a6ce0d5a95b4ead290add115348/starknet/src/space/space.cairo#L259).
2. When the proposal starts, the space caches the block number on L1 that corresponds to the stored `start timestamp`. To do that, we use [Herodotus' timestamp remapper contract](https://docs.herodotus.dev/herodotus-docs/protocol-design/timestamp-to-block-number-mapper).
3. Later on, when a user vote, he provides a storage proof of his balance at the block cached by the space. The space then provides the `storage_proof`, the `block_number`, and the `l1_contract_address` to [Herodotus' Facts Registry](https://docs.herodotus.dev/herodotus-docs/developers/contract-addresses)  for verification.
4. If the verification succeeds, the space can safely accept the vote, the `voting_power` of the user having been correctly proven and verified.

For OpenZeppelin ERC20Votes tokens, the function `get_past_votes` is reproduced in [this voting strategy](https://github.com/snapshot-labs/sx-starknet/blob/03041771ba701a6ce0d5a95b4ead290add115348/starknet/src/voting\_strategies/oz\_votes\_storage\_proof.cairo#L29) by proving two values: we first verify the checkpoint `c` containing the number of delegated power ; and then verify that the checkpoint `c + 1` is empty (ensuring that `c` was indeed the latest checkpoint). As a last detail, the `Checkpoint` structure got changed by OpenZeppelin to a Checkpoint 208, so we [also support this version](https://github.com/snapshot-labs/sx-starknet/blob/develop/starknet/src/voting\_strategies/oz\_votes\_trace\_208\_storage\_proof.cairo).

### Transaction relayer

In order to avoid the overhead of installing and managing a Starknet account, we ask users to sign their vote with their Ethereum wallet, and then simply relay the transaction on the Starknet network using our transaction relayer [Mana](../services/mana.md). Once the transaction is relayed, and by leveraging the modularity of our authenticators system, a [special authenticator](https://github.com/snapshot-labs/sx-starknet/blob/develop/starknet/src/authenticators/eth\_sig.cairo) verifies the provided signature and lets the vote be counted in!

But who pays for the relaying fees? It is expected that DAOs who want to provide their users with this ease of vote would be the paying those fees, which should be minimal anyway!

### L1 execution

Once the proposal has ended, and if the proposal has been accepted, then the transaction can be broadcast back to our [L1 Avatar Execution Strategy](https://github.com/snapshot-labs/sx-starknet/blob/develop/ethereum/src/execution-strategies/L1AvatarExecutionStrategy.sol) contract. This module works hand in hand with Zodiac's Avatar (e.g a Safe), and allows the transactions attached to the proposal to be executed directly on the avatar!

