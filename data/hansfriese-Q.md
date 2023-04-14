
### [L-01] No validation about `fundingSplitBps` in `ETHCrowdfundBase._initialize`

- https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/crowdfund/ETHCrowdfundBase.sol#L141

```solidity
    fundingSplitBps = opts.fundingSplitBps;
```
In `ETHCrowdfundBase._initialize`, there is no validation about `fundingSplitBps`. So if `fundingSplitBps` > 1e4, it will cause underflow during `_processContribution`.

```solidity
            uint96 feeAmount = (amount * fundingSplitBps_) / 1e4;
            amount -= feeAmount;
```

So we should validate `fundingSplitBps <= 1e4` in `ETHCrowdfundBase._initialize`.


### [L-02] No validation about `exchangeRateBps` in `ETHCrowdfundBase._initialize`
When `exchangeRateBps == 0`, voting power will be zero [here](https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/crowdfund/ETHCrowdfundBase.sol#L233).

- https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/crowdfund/ETHCrowdfundBase.sol#L139

```solidity
    exchangeRateBps = opts.exchangeRateBps;
```

We should check if `exchangeRateBps > 0`.

### [NC-01] Missing comment

- https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/crowdfund/InitialETHCrowdfund.sol#L50


```solidity
    struct BatchContributeArgs {
        // IDs of cards to credit the contributions to. When set to 0, it means
        uint256[] tokenIds;
```

The comment in `BatchContributeArgs` is not complete. We can find full comment about it in other struct `BatchContributeForArgs`.

```solidity
    struct BatchContributeForArgs {
        // IDs of cards to credit the contributions to. When set to 0, it means
        // a new one should be minted.
        uint256[] tokenIds;
```

We should add the missing comment for better understandings.


### [NC-02] Missing Natspec

- https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/party/PartyGovernance.sol#L464-L475

Natspec is missing. For instance, in PartyGovernance.distribute, there is no explanation about amount parameter.

```solidity
    /// @dev Allow this to be called by the party itself for `FractionalizeProposal`.
    /// @param tokenType The type of token to distribute.
    /// @param token The address of the token to distribute.
    /// @param tokenId The ID of the token to distribute. Currently unused but
    ///                may be used in the future to support other distribution types.
    /// @return distInfo The information about the created distribution.
    function distribute(
        uint256 amount,
        ITokenDistributor.TokenType tokenType,
        address token,
        uint256 tokenId
    )
```