### Low 1: reentrancy in CollectionBatchBuyOperator.sol
CollectionBatchBuyOperator tries to prevent reentrancy here but it doesn't do anything. An attacker can simply set the callTarget to a contract they control which can then invoke CollectionBatchBuyOperator's execute to achieve reentrancy. I don't see this causing any problem beyond the issue that I've described in one of my medium reports (operator can profit from executing).
```
        if (callTarget == address(this)) {
            revert CallProhibitedError(callTarget, callData);
        }
```
https://github.com/code-423n4/2023-04-party/blob/main/contracts/operators/CollectionBatchBuyOperator.sol#L208-L210


### Low 2: CollectionBatchBuyOperator doesn't refund operator value if operator spent their own fund
no time left

### Low 3: integer overflow can cause ETH crowdfunds to finish immediately
Crowdfund creator may want to set a long duration so that they can achieve the maximumTotalContribution, but this causes an integer overflow here:
https://github.com/code-423n4/2023-04-party/blob/main/contracts/crowdfund/ETHCrowdfundBase.sol#L137
As a result, the crowdfund may finish immediately.
https://github.com/code-423n4/2023-04-party/blob/main/contracts/crowdfund/ETHCrowdfundBase.sol#L158
 
### Low 4: it's possible for a user to contribute and get no voting power
https://github.com/code-423n4/2023-04-party/blob/main/contracts/party/PartyGovernanceNFT.sol#L146-L148
When a malicious authority sees a call to mint or addVotingPower, they can front run it with a call to mint to increase mintedVotingPower_ to totalVotingPower, thus leaving user with no voting power even though they contributed. This can also be caused by a legitimate authority that forgot to call increase totalVotingPower
