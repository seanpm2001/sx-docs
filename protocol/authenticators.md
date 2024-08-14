# Authenticators

Authenticators are the contracts in charge of **authenticating users** to create proposals and cast votes.&#x20;

**All proposal creation, proposal update, and vote transactions must be sent to the relevant DAO's space contract via an authenticator.**

DAOs are free to write their own custom authenticators that suit their own needs however we provide the following approaches:

### Ethereum signature authenticator

Will authenticate a user based on a message signed by an Ethereum private key. Users create an EIP712 signature for the transaction which is checked for validity in this contract.&#x20;

```solidity
function authenticate(
    uint8 v,
    bytes32 r,
    bytes32 s,
    uint256 salt,
    address target,
    bytes4 functionSelector,
    bytes calldata data
) external;
```

* `v`,`r`,`s`: ECDSA Signature
* `salt` : The salt used in the signature to prevention replays (only required for proposal creation and updating).
* `target`: The destination space contract.
* `functionSelector`: The function selector of the desired action.
* `data`: The ABI encoded transaction payload for the action. Refer to the [Space Actions](https://app.gitbook.com/o/-LFgTZvhAg63US8GVxGf/s/Z1apxjsgt60dN7Nlmu01/\~/changes/20/protocol-sx-evm/space) section for more information on the payload contents.&#x20;

This can work in conjunction with a meta transaction relayer to allow proposal creation or vote costs to be sponsored by the DAO, providing a free end user experience.

### Ethereum transaction authenticator

Will authenticate a user by checking if the caller address corresponds to the `author` or `voter` address.&#x20;

```solidity
function authenticate(address target, bytes4 functionSelector, bytes calldata data) external;
```

* `target`: The destination space contract.
* `functionSelector`: The function selector of the desired action.
* `data`: The ABI encoded transaction payload for the action. Refer to the [Space Actions](https://app.gitbook.com/o/-LFgTZvhAg63US8GVxGf/s/Z1apxjsgt60dN7Nlmu01/\~/changes/20/protocol-sx-evm/space) section for more information on the payload contents.&#x20;

{% hint style="success" %}
The core use case for this authenticator is to allow **smart contract accounts such as multi-sigs to use Snapshot X** as they have no way to generate a signature and therefore cannot authenticate via signature verification.
{% endhint %}

### And more!

Our modular approach here allows spaces to **authenticate users via other authentication methods without any changes to the space contract**.&#x20;

{% hint style="info" %}
Please note, that if you wanted to add sybil resistance to your governance process, this should not be handled by Authenticators, but by [Proposal Validation](proposal-validation-strategies.md) or [Voting Strategies](voting-strategies.md).
{% endhint %}

