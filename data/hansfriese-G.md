## [GAS-1] `abi.encode()` is less efficient than `abi.encodepacked()`
[See for more information](https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison)

- https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/party/PartyGovernance.sol#L291

```solidity
    _initProposalImpl(
        IProposalExecutionEngine(_GLOBALS.getAddress(LibGlobals.GLOBAL_PROPOSAL_ENGINE_IMPL)),
        abi.encode(proposalEngineOpts)
    );
```

## [GAS-2] Setting the `constructor` to `payable`
Saves ~13 gas per instance
- https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/crowdfund/ReraiseETHCrowdfund.sol#L62
- https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/party/PartyGovernance.sol#L270
- https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/party/PartyGovernanceNFT.sol#L56
- https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/proposals/ArbitraryCallsProposal.sol#L49
- https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/proposals/ProposalExecutionEngine.sol#L99


## [GAS-3] Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier or require such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

- https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/party/PartyGovernance.sol#L444

```solidity
    function abdicateHost(address newPartyHost) external onlyHost onlyDelegateCall
```

- https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/party/PartyGovernance.sol#L617

```solidity
    function veto(uint256 proposalId) external onlyHost onlyDelegateCall {
```

- https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/party/PartyGovernance.sol#L786

```solidity
    function emergencyExecute(
        address targetAddress,
        bytes calldata targetCallData,
        uint256 amountEth
    ) external payable onlyPartyDao onlyWhenEmergencyExecuteAllowed onlyDelegateCall {
```

- https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/party/PartyGovernance.sol#L800

```solidity
    function disableEmergencyExecute() external onlyPartyDaoOrHost onlyDelegateCall
```

## [GAS-4] Use hardcoded address instead `address(this)`
Instead of using `address(this)`, it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry's `script.sol` and solmate's `LibRlp.sol` contracts can help achieve this.

References: https://book.getfoundry.sh/reference/forge-std/compute-create-address

- https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/crowdfund/InitialETHCrowdfund.sol#L245
- https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/crowdfund/InitialETHCrowdfund.sol#L358
- https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/crowdfund/InitialETHCrowdfund.sol#L374
- https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/operators/CollectionBatchBuyOperator.sol#L146
- https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/proposals/ArbitraryCallsProposal.sol#L169

## [GAS-5] Division by 2 should use bit shifting
- https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/party/PartyGovernance.sol#L420

```solidity
    uint256 mid = (low + high) / 2;
```

## [GAS-6] `<x> += <y>` Costs More Gas Than `<x> = <x> + <y>` For State Variables
- https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/crowdfund/ReraiseETHCrowdfund.sol#L235
- https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/operators/CollectionBatchBuyOperator.sol#L128
- https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/operators/CollectionBatchBuyOperator.sol#L147
- https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/operators/CollectionBatchBuyOperator.sol#L161
- https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/party/PartyGovernance.sol#L593
- https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/party/PartyGovernanceNFT.sol#L152
- https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/party/PartyGovernanceNFT.sol#L184
- https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/party/PartyGovernanceNFT.sol#L185
- https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/party/PartyGovernanceNFT.sol#L196

## [GAS-7] Using `unchecked` blocks to save gas
Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow is impossible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block.

- https://github.com/code-423n4/2023-04-party/blob/440aafacb0f15d037594cebc85fd471729bcb6d9/contracts/party/PartyGovernanceNFT.sol#L218

```solidity
    mintedVotingPower -= votingPower;
```