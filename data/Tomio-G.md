Title: Consider delete empty block or emit something

impact:
The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.

Proof of Concept:
[Party.sol#L47](https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/party/Party.sol#L47)
________________________________________________________________________

Title: The `abi.encodeWithSelector` function is more gas efficient than `abi.encodeCall`

Proof of Concept:
[PartyFactory.sol#L49](https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/party/PartyFactory.sol#L49)

Recommended Mitigation Steps:
```
abi.encodeWithSelector(Party.initialize.selector, (initData))
```
________________________________________________________________________

Title: Expression for `constant` values such as a call to `keccak256()`, should use `immutable` rather than `constant`

Proof of Concept:
[ProposalStorage.sol#L19](https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/proposals/ProposalStorage.sol#L19)

Recommended Mitigation Steps:
Change from `constant` to `immutable`
reference: [here](https://github.com/ethereum/solidity/issues/9232)
________________________________________________________________________

Title: Use `uint64` instead of `uint96`

Proof of Concept:
[BuyCrowdfund.sol#L33](https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/crowdfund/BuyCrowdfund.sol#L33)
this variable can be optimized by changing it to a uint64 if the maximum value it needs to hold is smaller than 2^64. This will save gas because smaller integer sizes are cheaper to store and manipulate.
________________________________________________________________________

Title: Remove unused functions

Proof of Concept:
[CrowdfundNFT.sol#L49-L80](https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/crowdfund/CrowdfundNFT.sol#L49-L80)
The functions `transferFrom`, `safeTransferFrom`, `safeTransferFrom`, `approve`, and `setApprovalForAll` are never used in the contract, and they can be removed to reduce the bytecode size and gas cost.
________________________________________________________________________

Title: Use `view` and `pure`

Proof of Concept:
[CrowdfundNFT.sol#L84-L94](https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/crowdfund/CrowdfundNFT.sol#L84-L94)
The functions `getApproved` and `isApprovedForAll` don't modify the state and can be marked with the view modifier to indicate that they only read from the contract. Similarly, the function getApproved doesn't access the state and can be marked with the pure modifier.
________________________________________________________________________

Title: Use `require` instead of `if` and `revert`

Proof of Concept: 
[CrowdfundNFT.sol#L125](https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/crowdfund/CrowdfundNFT.sol#L125) , [CrowdfundNFT.sol#L150](https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/crowdfund/CrowdfundNFT.sol#L150)
In the `ownerOf` and `_burn` functions, instead of using an if statement followed by a revert statement, it's better to use a single require statement to check the condition and revert if it fails. This can save some gas.

Recommended Mitigation Steps:
```
require(owner != address(0), InvalidTokenError(tokenId));
```
________________________________________________________________________

Title: Use `view` Function Modifier:

Proof of Concept: 
[CrowdfundFactory.sol#L107](https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/crowdfund/CrowdfundFactory.sol#L107)
The `_prepareGate` function does not modify any state variables. you can use the `view` function modifier to indicate that the function does not change the state. This helps to improve code readability and optimize gas consumption.
________________________________________________________________________

Title: Use `immutable` instead of `constant`

Proof of Concept: 
[ListOnZoraProposal.sol#L58-L61](https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/proposals/ListOnZoraProposal.sol#L58-L61)
This can save some gas costs since it avoids the need to recompute the hash value every time they are accessed.
________________________________________________________________________

Title: Avoid returning empty strings

Proof of Concept:
[PartyGovernanceNFT.sol#L88-L108](https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/party/PartyGovernanceNFT.sol#L88-L108)
Returning an empty string from a function call is a waste of gas, as it requires a storage read. Consider returning a non-empty string or removing the return statement altogether.
________________________________________________________________________

Title: Inline the modifier function to reduce gas costs.

Proof of Concept:
[PartyGovernanceNFT.sol#L35-L40](https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/party/PartyGovernanceNFT.sol#L35-L40)

Recommended Mitigation Steps:
```
 modifier onlyMinter() {
        require(msg.sender == mintAuthority, "..");
        _;
    }
```
________________________________________________________________________

Title: Consider using uint128 instead of uint256 for the getDistributionShareOf 

Proof of Concept:
[PartyGovernanceNFT.sol#L111](https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/party/PartyGovernanceNFT.sol#L111)
since it is unlikely that the voting power of any token will exceed 2^128.
________________________________________________________________________

Title: Avoid unnecessary function calls

Proof of Concept:
[PartyGovernanceNFT.sol#L72](https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/party/PartyGovernanceNFT.sol#L72)

Recommended Mitigation Steps:
`ERC721.ownerOf(tokenId)` can be replaced with `ownerOf(tokenId)` as the contract inherits from ERC721. This can reduce the gas cost of the transaction.
________________________________________________________________________

Title: `>=` is cheaper than `>`

Impact:
Strict inequalities (`>`) are more expensive than non-strict ones (`>=`). This is due to some supplementary checks (ISZERO, 3 gas)

Proof of Concept:
[TokenDistributor.sol#L167](https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/distribution/TokenDistributor.sol#L167)

Recommended Mitigation Steps:
Consider using `>=` instead of `>` to avoid some opcodes
________________________________________________________________________

Title: Unchecking arithmetics operations that can't underflow/overflow

Proof of Concept:
[TokenDistributor.sol#L170](https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/distribution/TokenDistributor.sol#L170) Should be unchecked due to L#167

Recommended Mitigation Steps:
Use `unchecked` can save gas
________________________________________________________________________

Title: Caching `length` for loop can save gas

Proof of Concept:
[TokenDistributor.sol#L239](https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/distribution/TokenDistributor.sol#L239)
[ListOnOpenseaProposal.sol#L291](https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/proposals/ListOnOpenseaProposal.sol#L291)
[Crowdfund.sol#L180](https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/crowdfund/Crowdfund.sol#L180)
[Crowdfund.sol#L300](https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/crowdfund/Crowdfund.sol#L300)

Recommended Mitigation Steps:
Change to:

```
    uint256 Length = infos.length;
    for (uint256 i = 0; i < Length; ++i) {
```
________________________________________________________________________

Title: Using unchecked and prefix increment is more effective for gas saving:

Proof of Concept:
[TokenDistributor.sol#L230](https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/distribution/TokenDistributor.sol#L230)
[TokenDistributor.sol#L239](https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/distribution/TokenDistributor.sol#L239)
[ListOnOpenseaProposal.sol#L291](https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/proposals/ListOnOpenseaProposal.sol#L291)
[Crowdfund.sol#L180](https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/crowdfund/Crowdfund.sol#L180)
[Crowdfund.sol#L242](https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/crowdfund/Crowdfund.sol#L242)
[Crowdfund.sol#L300](https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/crowdfund/Crowdfund.sol#L300)
[Crowdfund.sol#L348](https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/crowdfund/Crowdfund.sol#L348)

Recommended Mitigation Steps:
Change to:

```
for (uint256 i = 0; i < infos.length;) {
// ...
unchecked { ++i; }
}
```
________________________________________________________________________

Title: Default value initialization

Impact:
If a variable is not set/initialized, it is assumed to have the default value (0, false, 0x0 etc depending on the data type). Explicitly initializing it with its default value is an anti-pattern and wastes gas.

Proof of Concept:
[Crowdfund.sol#L180](https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/crowdfund/Crowdfund.sol#L180)
[Crowdfund.sol#L242](https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/crowdfund/Crowdfund.sol#L242)
[Crowdfund.sol#L300](https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/crowdfund/Crowdfund.sol#L300)
[Crowdfund.sol#L348](https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/crowdfund/Crowdfund.sol#L348)

Recommended Mitigation Steps:
Remove explicit initialization for default values.
________________________________________________________________________