Title: Constant definitions

Proof of Concept:
Define the `PROPOSAL_FLAG_UNANIMOUS` constant using a more descriptive name rather than a magic number. This will make the code more readable and maintainable, and reduce the risk of errors caused by incorrect values. [ProposalStorage.sol#L18](https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/proposals/ProposalStorage.sol#L18)
________________________________________________________________________

Title: Access Control

Proof of Concept:
[CollectionBuyCrowdfund.sol#L113](https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/crowdfund/CollectionBuyCrowdfund.sol#L113)
The `onlyHost` modifier it only checks that the caller is included in the list of party hosts, which can be manipulated by anyone who has access to the contract's storage. It is recommended to implement a more robust access control, such as using role-based access control system.
________________________________________________________________________

Title: Reentrancy Attack

Proof of Concept: 
[CollectionBuyCrowdfund.sol#L113](https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/crowdfund/CollectionBuyCrowdfund.sol#L113)
The buy function uses external contracts and passes along arbitrary data with the callData parameter. This can allow an attacker to launch a reentrancy attack by recursively calling back the crowdfund contract while it's still in an intermediate state, allowing the attacker to manipulate the internal state and possibly drain the funds.
________________________________________________________________________

Title: Failed transaction

Proof of Concept: 
[CrowdfundNFT.sol#L152-L153](https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/crowdfund/CrowdfundNFT.sol#L152-L153)
the `_burn` function checks if the `_owners[tokenId] == owner` before setting `_owners[tokenId]` to `address(0)` in order to burn the token. If `_owners[tokenId]` is set to `address(0)` before this check is made, the function will think that the token has already been burned and revert, resulting in a failed transaction.
________________________________________________________________________

Title: Should use the latest Solidity version instead of ^0.8

Proof of Concept: 
All contract in scope.

Impact:
This can help to ensure that any potential security vulnerabilities in the compiler have been fixed.
Use a solidity version of at least 0.8.2 to get simple compiler automatic inlining. 
Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads. 
Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings.
Use a solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value
Reference: [here](https://blog.soliditylang.org/2021/11/09/solidity-0.8.10-release-announcement/)
________________________________________________________________________
