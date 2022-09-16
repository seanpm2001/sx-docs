# Home

Governance mechanisms obey a trilemma between **decentralisation**, **cost**, and **flexibility**. On the one hand, you have systems like [Snapshot](https://snapshot.org/#/) that provide a very cheap experience \[1] with lots of flexibility. However it relies on one trusting the Snapshot off-chain protocol to deliver the verdict of a particular governance proposal and to not censor votes. Flexibility comes from the wide range of voting strategies than one can employ to calculate the voting power for each user.

On the other hand, you have governance systems that run fully on-chain on Ethereum mainnet. [Compound Governor](https://github.com/compound-finance/compound-protocol/blob/master/contracts/Governance/GovernorBravoDelegate.sol) is one example of such a system. All the voting logic is computed on-chain which provides an equivalent degree of decentralisation to Ethereum itself. The compromise of such a system is a high cost of participation due to the high gas costs incurred when transacting on the blockchain. The flexibility of the system is also limited by cost as sophisticated voting strategies require increased on-chain logic and therefore it would cost even more and limit participation further to utilise them.

Snapshot X aims to bridge this divide by providing a fully on-chain governance system that is 50 to 100x cheaper than current solutions that run on Ethereum mainnet. We hope that this will unlock massive increases in on-chain governance participation and flexibility of the governance mechanism without having to make any compromises on decentralisation. This is achieved by running the voting logic on StarkNet, which provides cheap computation whilst inheriting all of the security guarantees of Ethereum itself. Once voting on a proposal has ended, an StarkNet -> Ethereum message bridge can be utilised to allow transactions inside the proposal to be permissionless executed on Ethereum mainnet. 

 <p align="center">
   <img src="/images/trilemma.png" width=100%>
</p>
