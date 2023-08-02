# Proposal validation strategies

**Proposal validation strategy is used to determine whether a particular `author` is allowed to create a proposal.**&#x20;

Each Space has to set a proposal validation strategy which consists of an address and a set of `params` that are stored in the space. These strategies should have the following interface:&#x20;

```solidity
function validate(address author, bytes calldata params, bytes calldata userParams) external returns (bool);
```

* `author`: The author of the proposal.
* `params`: Parameter array set in the space contract that is the same for every user of the Strategy.
* `userParams`: Parameter array submitted by the `author` when they create a proposal, and can therefore be different for each user.&#x20;

There is no requirement to use either of these parameter arrays in your strategy, in which case you can just pass an empty array.

DAOs are free to write their own custom strategies that suit their own needs however we provide the following approaches:

### Proposition power

A strategy that validates an `author` to create a proposal if their `propositionPower` calculated from a set of Voting strategies exceeds a `proposalThreshold` value. It means that if the proposal threshold is set to `5`, the author needs to have at least `5` Proposition power to create a proposal. This strategy uses the same logic as [Voting Strategies](https://app.gitbook.com/o/-LFgTZvhAg63US8GVxGf/s/Z1apxjsgt60dN7Nlmu01/\~/changes/20/protocol-sx-evm/voting-strategies), so refer to that section for more information.

When calling this strategy, `params` should be:

```solidity
uint256 proposalThreshold = ...
Strategy[] allowedStrategies ...
params = abi.encode(proposalThreshold, allowedStrategies);
```

`userParams` should contain the indexed strategies that the `author` has power with:

```solidity
IndexedStrategy[] userStrategies = ...
userParams = abi.encode(userStrategies);
```

### Active proposal limiter&#x20;

A strategy that validates an `author` to create a proposal if the `author` has not exceeded a limit of `maxActiveProposals`.

Each time an `author` creates a proposal, a counter is incremented up to a limit of `maxActiveProposals`. After this point, no more proposals can be created until a `cooldown` period has elapsed since the most recent proposal they created.

Using this strategy can help to prevent proposal creation spam in your space.&#x20;

### And more!

These strategies can be combined and extended to provide flexible Proposal validation mechanisms. The interface for Proposal validation strategies can be found [here](https://github.com/snapshot-labs/sx-evm/blob/main/src/interfaces/IProposalValidationStrategy.sol).&#x20;
