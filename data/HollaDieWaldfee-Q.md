# Party DAO Low Risk and Non-Critical Issues
## Summary
| Risk | Title                                                                                      | File                       | Instances |
| ---- | ------------------------------------------------------------------------------------------ | -------------------------- | --------- |
| L-01 | Check that none of the `authorities` is zero address                                       | PartyFactory.sol           | 1         |
| L-02 | ETH is not refunded when `allowArbCallsToSpendPatyETH=true`                                | ArbitraryCallsProposal.sol | 1         |
| L-03 | Comments state that pre-existing ETH can be used but it can't                              | -                          | 2         |
| L-04 | Issue due to rounding from previous C4 contest is still present in new crowdfund contracts | -                          | 2         |
| L-05 | Use `delegationsByContributor[contributor]` instead of `delegate` when minting party card  | InitialETHCrowdfund.sol    | 1         |
| N-01 | Introduce separate `vetoThresholdBps` for vetoing a proposal                               | VetoProposal.sol           | 1         |
| N-02 | `OperationExecuted` event is defined but never emitted                                     | OperatorProposal.sol       | 1         |
| N-03 | Use `transferEth` instead of `transfer` for transferring ETH                               | -                          | 4         |

## [L-01] Check that none of the `authorities` is zero address
The [`PartyFactory.createParty`](https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/party/PartyFactory.sol#L18-L45) function should check that the `authorities` array contains addresses that are not the zero address.  

I mention this because in the previous version of the `PartyFactory` contract, there was only one `authority` and it was checked to not be the zero address such as to ensure that governance NFTs can be minted:  

[Link](https://github.com/PartyDAO/party-protocol/blob/3313c24c85d7429346af939897c19deeef7952f5/contracts/party/PartyFactory.sol#L33-L35)  
```solidity
if (authority == address(0)) {
    revert InvalidAuthorityError(authority);
}
```

However now it is only checked that the `authorities` array is not empty:  

[Link](https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/party/PartyFactory.sol#L26-L28)  
```solidity
if (authorities.length == 0) {
    revert NoAuthorityError();
}
```

I think you should check that there is no zero address in this array. The current check is not sufficient when compared to what has been checked previously. The current check is weaker.  


## [L-02] ETH is not refunded when `allowArbCallsToSpendPatyETH=true`
The `ArbitraryCallsProposal` contract does not refund `ETH` to the `msg.sender` if `allowArbCallsToSpendPartyEth=true`.  

It only refunds when `allowArbCallsToSpendPartyEth=false`:  

[Link](https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/proposals/ArbitraryCallsProposal.sol#L108-L110)  
```solidity
if (!allowArbCallsToSpendPartyEth && ethAvailable > 0) {
    payable(msg.sender).transferEth(ethAvailable);
}
```

The reason this is the case is that it is not expected that the `msg.sender` will provide `ETH` if it is allowed to spend `ETH` from the Party's balance.  

I don't think this is a good assumption. There should be a refund mechanism.  

Please also refer to my report #5 which discusses a more severe similar issue. Some of the reasoning there also applies here. Specifically that there are two broad options to implement refunds when `allowArbCallsToSpendPartyEth=true`.  

1. Use `msg.value` first
2. Use Party balance first

The sponsor needs to decide which policy (if any) to use.  

## [L-03] Comments state that pre-existing ETH can be used but it can't
It is possible to provide an initial contribution to the `ReraiseETHCrowdfund` and `InitialETHCrowdfund` contracts.  

The initial contribution is processed when the `initialize` function is called.  

In both contracts it is stated that pre-existing ETH is used for the initial contribution (i.e. ETH that is owned by the contract but not sent along with the call to the `initialize` function):  

[https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/crowdfund/InitialETHCrowdfund.sol#L118-L131](https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/crowdfund/InitialETHCrowdfund.sol#L118-L131)  

[https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/crowdfund/ReraiseETHCrowdfund.sol#L80-L87](https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/crowdfund/ReraiseETHCrowdfund.sol#L80-L87)  

Hoever the initial contribution is only measured by looking at `msg.value`:  

[Link](https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/crowdfund/InitialETHCrowdfund.sol#L120)  
```solidity
uint96 initialContribution = msg.value.safeCastUint256ToUint96();
```

This means that pre-existing ETH is not actually processed and just sits in the contract without being used for anything. It can't even be rescued.  

It was assessed with the sponsor that they don't actually want to allow pre-existing ETH to be used for the initial contribution.  

Therefore the comments should be removed so users don't make a mistake and lose their ETH.  

Fix:  
```diff
diff --git a/contracts/crowdfund/InitialETHCrowdfund.sol b/contracts/crowdfund/InitialETHCrowdfund.sol
index 8ab3b5c..bcc65e2 100644
--- a/contracts/crowdfund/InitialETHCrowdfund.sol
+++ b/contracts/crowdfund/InitialETHCrowdfund.sol
@@ -115,12 +115,9 @@ contract InitialETHCrowdfund is ETHCrowdfundBase {
             })
         );
 
-        // If the deployer passed in some ETH during deployment, credit them
-        // for the initial contribution.
         uint96 initialContribution = msg.value.safeCastUint256ToUint96();
         if (initialContribution > 0) {
-            // If this contract has ETH, either passed in during deployment or
-            // pre-existing, credit it to the `initialContributor`.
+            // credit msg.value to the `initialContributor`.
             _contribute(
                 crowdfundOpts.initialContributor,
                 crowdfundOpts.initialDelegate,
```

```diff
diff --git a/contracts/crowdfund/ReraiseETHCrowdfund.sol b/contracts/crowdfund/ReraiseETHCrowdfund.sol
index 580623d..6ad81fc 100644
--- a/contracts/crowdfund/ReraiseETHCrowdfund.sol
+++ b/contracts/crowdfund/ReraiseETHCrowdfund.sol
@@ -77,12 +77,9 @@ contract ReraiseETHCrowdfund is ETHCrowdfundBase, CrowdfundNFT {
             0 // Ignored. Will use customization preset from party.
         );
 
-        // If the deployer passed in some ETH during deployment, credit them
-        // for the initial contribution.
         uint96 initialContribution = msg.value.safeCastUint256ToUint96();
         if (initialContribution > 0) {
-            // If this contract has ETH, either passed in during deployment or
-            // pre-existing, credit it to the `initialContributor`.
+            // credit msg.value to the `initialContributor`.
             _contribute(opts.initialContributor, opts.initialDelegate, initialContribution, "");
         }
```

## [L-04] Issue due to rounding from previous C4 contest is still present in new crowdfund contracts
In the previous C4 contest an issue has been found that due to rounding it might not be possible to achieve an unanimous vote:  

[https://code4rena.com/reports/2022-09-party/#m-10-possible-that-unanimous-votes-is-unachievable](https://code4rena.com/reports/2022-09-party/#m-10-possible-that-unanimous-votes-is-unachievable)

The issue also exists in the new crowdfund contracts (`InitialETHCrowdfund`, `ReraiseETHCrowdfund`).  

I agree that it would be best to introduce the fix suggested by the warden in the old report.  

However the sponsor told me that the `minContribution` amount will be set large enough such that the rounding issue cannot occur.  

Therefore I don't think this issue is worth reporting as Medium in this contest again. But I'd like to bring it up because it's still present in the new contracts.  

## [L-05] Use `delegationsByContributor[contributor]` instead of `delegate` when minting party card
The `ÌnitialETHCrowdfund._contribute` function currently uses `delegate` as the delegate when minting a party card [Link](https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/crowdfund/InitialETHCrowdfund.sol#L302).  

This is wrong. Instead `delegationsByContributor[contributor]` should be used. This is because if User A contributes for another User B and User B has already set a delegate, i.e. `delegationsByContributor[contributor]!=address(0)` then this delegate should be used and not the `delegate` parameter supplied by User A.  

However I don't see any impact in this behavior because if `delegationsByContributor[contributor]!=address(0)` then the party also has a delegate set for the `contributor`. And so this delegate is used above the delegate from the crowdfund anyway. So this finding is Low severity at most.  

Fix:  
```diff
diff --git a/contracts/crowdfund/InitialETHCrowdfund.sol b/contracts/crowdfund/InitialETHCrowdfund.sol
index 8ab3b5c..6c76e88 100644
--- a/contracts/crowdfund/InitialETHCrowdfund.sol
+++ b/contracts/crowdfund/InitialETHCrowdfund.sol
@@ -299,7 +299,7 @@ contract InitialETHCrowdfund is ETHCrowdfundBase {
 
         if (tokenId == 0) {
             // Mint contributor a new party card.
-            party.mint(contributor, votingPower, delegate);
+            party.mint(contributor, votingPower, delegationsByContributor[contributor]);
         } else if (disableContributingForExistingCard) {
             revert ContributingForExistingCardDisabledError();
         } else if (party.ownerOf(tokenId) == contributor) {
```

## [N-01] Introduce separate `vetoThresholdBps` for vetoing a proposal
Currently the same [`passThresholdBps`](https://github.com/PartyDAO/party-protocol/blob/3313c24c85d7429346af939897c19deeef7952f5/contracts/party/PartyGovernance.sol#L81) variable is used for accepting proposals as well as vetoing proposals.  

`passTresholdBps` is a percentage of the `totalVotingPower` that is required.  

I recommend to introduce a separate `vetoThresholdBps` governance parameter that is used to determine the percentage of votes necessary to veto a proposal.  

Using separate thresholds allows for greater flexibility.  

E.g. a Party might want a high consensus of 60% to accept a proposal but might want to require only 10% of votes to veto a proposal. Such a setup is not possible currently which unnecessarily restricts the flexibility of the protocol.  


## [N-02] `OperationExecuted` event is defined but never emitted
In the `OperatorProposal` contract the [`OperationExecuted`](https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/proposals/OperatorProposal.sol#L20) event is defined but it is never emitted.  

Therefore I recommend to emit this event when the operation is executed successfully.  

Fix:  
```diff
diff --git a/contracts/proposals/OperatorProposal.sol b/contracts/proposals/OperatorProposal.sol
index 23e2897..5899bec 100644
--- a/contracts/proposals/OperatorProposal.sol
+++ b/contracts/proposals/OperatorProposal.sol
@@ -44,6 +44,8 @@ contract OperatorProposal {
         // Execute the operation.
         data.operator.execute{ value: data.operatorValue }(data.operatorData, executionData);
 
+        emit OperationExecuted(msg.sender);
+        
         // Nothing left to do.
         return "";
     }
```

## [N-03] Use `transferEth` instead of `transfer` for transferring ETH
The [automated findings](https://gist.github.com/HollaDieWaldfee100/673ab665890ab92d5f3f82c8736ffea5#L-2) have flagged the below instances as unsafe ERC20 operations.  

This is wrong. They are not ERC20 operations. Instead they are just the `transfer` function that is a built-in Solidity function for sending ETH.  

However the usage of this function is still [discouraged](https://consensys.net/diligence/blog/2019/09/stop-using-soliditys-transfer-now/) because it limits the Gas that the callee can consume to `2300`.  

Instead use the `transferEth` function that is used elsewhere in the codebase to transfer ETH.  

There are 4 instances:  
[https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/crowdfund/InitialETHCrowdfund.sol#L204](https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/crowdfund/InitialETHCrowdfund.sol#L204)  

[https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/crowdfund/InitialETHCrowdfund.sol#L267](https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/crowdfund/InitialETHCrowdfund.sol#L267)  

[https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/crowdfund/ReraiseETHCrowdfund.sol#L152](https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/crowdfund/ReraiseETHCrowdfund.sol#L152)  

[https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/crowdfund/ReraiseETHCrowdfund.sol#L201](https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/crowdfund/ReraiseETHCrowdfund.sol#L201)  



