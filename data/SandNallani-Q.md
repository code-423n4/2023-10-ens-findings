Missing events for important state changing transactions related to reimbursing or transferring tokens.


The `ERC20MultiDelegate` contract allows a ENS token holder to either:

- Case#1: Transfer tokens from sources to targets to delegate votes, OR
- Case#2: Get reimbursed from sources and burn their votes OR
- Case#3: Deploy a proxy delegate contract to transfer tokens to, followed by delegating the votes to the intended target.

However, no events are explicitly emitted in cases 2&3. Even in the case when target requires deploying a delegate contract, no events are emitted. 

In Solidity, events are primarily used for logging contract activity, which provides external consumers with a way to track state changes without executing expensive contract calls. 

For example:

- Front-end applications often listen for contract events to provide users with real-time feedback about their transactions. Without events, users might be left wondering if their transaction had the intended effect, leading to confusion or reduced confidence in the platform.
- State variables only reflect the current state whereas events might be the only way to track state changes over time. Without them, historical data can be lost and no body will able to trace the history of the transactions on-chain.

Remediations to consider:
Emit events specifically for cases 2 and 3 to track the transaction history on-chain. 
