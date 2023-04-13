# Party DAO Gas Findings
## [G-01] Use `maxTotalContributations_` memory variable instead of `maxTotalContributions`
`maxTotalContributions` has already been saved to the `maxTotalContributions_` memory variable.  
It's cheaper to read from memory than from storage.  

Gas saved: 124 Gas per tx

```diff
diff --git a/contracts/crowdfund/ETHCrowdfundBase.sol b/contracts/crowdfund/ETHCrowdfundBase.sol
index 4392655..886c09c 100644
--- a/contracts/crowdfund/ETHCrowdfundBase.sol
+++ b/contracts/crowdfund/ETHCrowdfundBase.sol
@@ -212,7 +212,7 @@ contract ETHCrowdfundBase is Implementation {
             _finalize(maxTotalContributions_);
 
             // Refund excess contribution.
-            uint96 refundAmount = newTotalContributions - maxTotalContributions;
+            uint96 refundAmount = newTotalContributions - maxTotalContributions_;
             if (refundAmount > 0) {
                 amount -= refundAmount;
                 payable(msg.sender).transferEth(refundAmount);
```

## [G-02] No need to cache `fundingSplitRecipient` in memory variable
`fundingSplitRecipient` is only used once. So it is not beneficial to cache it in a memory variable.  

Gas saved: Negligible but code is simplified

```diff
diff --git a/contracts/crowdfund/ETHCrowdfundBase.sol b/contracts/crowdfund/ETHCrowdfundBase.sol
index 4392655..17af949 100644
--- a/contracts/crowdfund/ETHCrowdfundBase.sol
+++ b/contracts/crowdfund/ETHCrowdfundBase.sol
@@ -222,9 +222,8 @@ contract ETHCrowdfundBase is Implementation {
         }
 
         // Subtract fee from contribution amount if applicable.
-        address payable fundingSplitRecipient_ = fundingSplitRecipient;
         uint16 fundingSplitBps_ = fundingSplitBps;
-        if (fundingSplitRecipient_ != address(0) && fundingSplitBps_ > 0) {
+        if (fundingSplitRecipient != address(0) && fundingSplitBps_ > 0) {
             uint96 feeAmount = (amount * fundingSplitBps_) / 1e4;
             amount -= feeAmount;
         }
@@ -237,9 +236,8 @@ contract ETHCrowdfundBase is Implementation {
         amount = (votingPower * 1e4) / exchangeRateBps;
 
         // Add back fee to contribution amount if applicable.
-        address payable fundingSplitRecipient_ = fundingSplitRecipient;
         uint16 fundingSplitBps_ = fundingSplitBps;
-        if (fundingSplitRecipient_ != address(0) && fundingSplitBps_ > 0) {
+        if (fundingSplitRecipient != address(0) && fundingSplitBps_ > 0) {
             amount = (amount * 1e4) / (1e4 - fundingSplitBps_);
         }
     }
```

## [G-03] Cache `party` in memory variable
`party` is used multiple times so it can be cached in a memory variable.

Gas saved: 149 per tx

```diff
diff --git a/contracts/crowdfund/InitialETHCrowdfund.sol b/contracts/crowdfund/InitialETHCrowdfund.sol
index 8ab3b5c..159e30a 100644
--- a/contracts/crowdfund/InitialETHCrowdfund.sol
+++ b/contracts/crowdfund/InitialETHCrowdfund.sol
@@ -325,15 +325,16 @@ contract InitialETHCrowdfund is ETHCrowdfundBase {
         }
 
         // Get amount to refund.
-        uint96 votingPower = party.votingPowerByTokenId(tokenId).safeCastUint256ToUint96();
+        Party party_ = party;
+        uint96 votingPower = party_.votingPowerByTokenId(tokenId).safeCastUint256ToUint96();
         amount = _calculateRefundAmount(votingPower);
 
         if (amount > 0) {
             // Get contributor to refund.
-            address payable contributor = payable(party.ownerOf(tokenId));
+            address payable contributor = payable(party_.ownerOf(tokenId));
 
             // Burn contributor's party card.
-            party.burn(tokenId);
+            party_.burn(tokenId);
 
             // Refund contributor.
             contributor.transferEth(amount);
```

## [G-04] Save `votingPowerByCard[i]` in memory variable
Gas saved: 1365 per tx

```diff
diff --git a/contracts/crowdfund/ReraiseETHCrowdfund.sol b/contracts/crowdfund/ReraiseETHCrowdfund.sol
index 580623d..68ccf4e 100644
--- a/contracts/crowdfund/ReraiseETHCrowdfund.sol
+++ b/contracts/crowdfund/ReraiseETHCrowdfund.sol
@@ -352,14 +352,15 @@ contract ReraiseETHCrowdfund is ETHCrowdfundBase, CrowdfundNFT {
         uint96 minContribution_ = minContribution;
         uint96 maxContribution_ = maxContribution;
         for (uint256 i; i < votingPowerByCard.length; ++i) {
-            if (votingPowerByCard[i] == 0) continue;
+            uint96 vp = votingPowerByCard[i];
+            if (vp == 0) continue;
 
             // Check that the contribution equivalent of voting power is within
             // contribution range. This is done so parties may use the minimum
             // and maximum contribution values to limit the voting power of each
             // card (e.g. a party desiring a "1 card = 1 vote"-like governance
             // system where each card has equal voting power).
-            uint96 contribution = (votingPowerByCard[i] * 1e4) / exchangeRateBps;
+            uint96 contribution = (vp * 1e4) / exchangeRateBps;
             if (contribution < minContribution_) {
                 revert BelowMinimumContributionsError(contribution, minContribution_);
             }
@@ -371,9 +372,9 @@ contract ReraiseETHCrowdfund is ETHCrowdfundBase, CrowdfundNFT {
             votingPower -= votingPowerByCard[i];
 
             // Mint contributor a new party card.
-            uint256 tokenId = party.mint(contributor, votingPowerByCard[i], delegate);
+            uint256 tokenId = party.mint(contributor, vp, delegate);
 
-            emit Claimed(contributor, tokenId, votingPowerByCard[i]);
+            emit Claimed(contributor, tokenId, vp);
         }
 
         // Requires that all voting power is claimed because the contributor is
```

## [G-05] Use `mintedVotingPower_` instead of `mintedVotingPower`
Gas saved: 114 per tx per instance

```diff
diff --git a/contracts/party/PartyGovernanceNFT.sol b/contracts/party/PartyGovernanceNFT.sol
index 9ccfa1f..c6c3d14 100644
--- a/contracts/party/PartyGovernanceNFT.sol
+++ b/contracts/party/PartyGovernanceNFT.sol
@@ -149,7 +149,7 @@ contract PartyGovernanceNFT is PartyGovernance, ERC721, IERC2981 {
 
         // Update state.
         tokenId = tokenCount = tokenCount_ + 1;
-        mintedVotingPower += votingPower_;
+        mintedVotingPower = mintedVotingPower_ + votingPower_;
         votingPowerByTokenId[tokenId] = votingPower_;
 
         // Use delegate from party over the one set during crowdfund.
@@ -181,7 +181,7 @@ contract PartyGovernanceNFT is PartyGovernance, ERC721, IERC2981 {
         }
 
         // Update state.
-        mintedVotingPower += votingPower_;
+        mintedVotingPower = mintedVotingPower_ + votingPower_;
         votingPowerByTokenId[tokenId] += votingPower_;
 
         _adjustVotingPower(ownerOf(tokenId), votingPower_.safeCastUint96ToInt192(), address(0));
```

