---
title: Token-Controlled Token Circulation 
description: Access control scheme based on token ownership.
author: Ko Fujimura (@kofujimura)
discussions-to: <URL>
status: Draft
type: Standards Track or Informational
category: ERC
created: 2023-06-11
requires: EIP-721
---
## Abstract

This EIP presents a new access control scheme called Token-Control Token Circulation (TCTC). By representing roles or privileges as EIP-721 tokens, the need for developing off-chain tools that grant or revoke role is eliminated. This approach can potentially enhance system security and decrease development costs.
  
## Motivation

There are numerous methods to implement access control for privileged actions. A commonly utilized pattern is "role-based" access control as specified in EIP-5982. This method, however, necessitates the use of an off-chain management tool to grant or revoke required roles through its interface. Additionally, as many wallets lack a user interface that displays the privileges granted by a role, users are often unable to comprehend the status of their privileges through the wallet.

Use case <to be added>

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

1. Smart contracts implementing the EIP-XXXX(this EIP) standard MUST represent the privilege required by the role as an EIP-721 token. The tokens that represent privileges are called `control tokens` in this EIP.
2. To grant a role to an account, a `control token` representing the privilege SHOULD be minted to the account using `safeMint` method defined in ERC-5679.
3. To revoke a role from an account, the `control token` representing the privilege SHOULD be burned using the burn method defined in EIP-5679.
4. To check if an account has the required role, a compliant smart contract SHOULD verify that the balance of the control token is greater than 0 using the `balanceOf` method defined in EIP-721.  
5. A role in a compliant smart contract is represented in the format of `bytes32`. It's RECOMMENDED the value of such role is computed as a `keccak256` hash of a string of the role name, in this format: `bytes32 role = keccak256("<role_name>")`. such as `bytes32 role = keccak256("MINTER")`.
  
## Rationale

<!--
  The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages.

  The current placeholder is acceptable for a draft.

  TODO: Remove this comment before submitting
-->

TBD

## Backwards Compatibility

This EIP is designed to be compatible for EIP-721, EIP-1155, and EIP-5679 respectively.

## Reference Implementation

The following presents a straightforward example of a contract that implements a modifier. This modifier checks if an account possesses the necessary role, and the contract includes a function that grants a specific role to a designated account.
  
```
// SPDX-License-Identifier: Apache-2.0

pragma solidity ^0.8.9;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

contract TokenController {
    mapping (bytes32 => address []) private _controlTokens;

    modifier onlyHasToken(bytes32 r, address u) {
        require(_checkHasToken(r,u), "TokenController: not has a required token");
        _;
    }

    // Specifies the contract ID of the token that must be owned by a user with the 
    // specified role. Multiple calls are allowed, in this case the user must own at 
    // least one of the specified token.
    function _grantRoleByToken (bytes32 r, address c) internal {
        _controlTokens[r].push(c);
    }

    function _checkHasToken (bytes32 r, address u) internal view returns (bool) {
        for (uint i = 0; i < _controlTokens[r].length; i++) {
            if (IERC721(_controlTokens[r][i]).balanceOf(u) > 0) return true;
        }
        return false;
    }
}
```

The following is a simple example of utilizing `TokenController` within an ERC721 token to define "minter" and "burner" roles. Accounts possessing these roles are allowed to create new tokens and destroy existing tokens, facilitated by specifying the control token:  
  
```
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.9;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import "./TokenController.sol";

contract MyToken is ERC721, TokenController {
    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
    bytes32 public constant BURNER_ROLE = keccak256("HOLDER_ROLE");

    constructor() ERC721("MyToken", "MTK") {
        // Specifies the deployed contract ID of the control token.
        // This sample contract is deployed on Goerli.
        _grantRoleByToken(MINTER_ROLE, 0xF1e33c646a12F68bC8015b4AED29BB316fA2D593);
        _grantRoleByToken(BURNER_ROLE, 0xcDc6fD5F29E2641f25c90235eDA984f99aA3a1DD);
    }

    function safeMint(address to, uint256 tokenId)
        public onlyHasToken(MINTER_ROLE, msg.sender)
    {
        _safeMint(to, tokenId);
    }

    function burn(uint256 tokenId)
        public onlyHasToken(BURNER_ROLE, msg.sender)
    {
        _burn(tokenId);
    }
}  
```
  
## Security Considerations

Needs discussion.

## Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).
