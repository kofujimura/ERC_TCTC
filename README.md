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

There are numerous methods to implement access control for privileged actions. A commonly utilized pattern is "role-based" access control as specified in ERC-5982. This method, however, necessitates the use of an off-chain management tool to grant or revoke required roles through its interface. Additionally, as many wallets lack a user interface that displays the privileges granted by a role, users are often unable to comprehend the status of their privileges through the wallet.
  
Use case ...

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

1. Smart contracts implementing the ERC-XXXX standard MUST implement privilege required by the role as a EIP-721 token. The tokens that represent privileges are called `control token` in this EIP.
2. To grant a role to the account, a `control token` that represent privilege SHOULD be minted to the account using `mint` or `safeMint` method defined in EIP-721.
3. To revoke a role from the account, the `control token` that represent privilege SHOULD be burned using `burn` method defined in EIP-721.
4. To check if the account has the required role, a comliant smart contract SHOULD check that the balance of the control token MUST greater than 0 using `balanceOf` method defined in EIP-721. 
5. A role in a compliant smart contract is represented in the format of `bytes32`. It's RECOMMENDED the value of such role is computed as a `keccak256` hash of a string of the role name, in this format: `bytes32 role = keccak256("<role_name>")`. such as `bytes32 role = keccak256("MINTER")`.
  
## Rationale

<!--
  The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages.

  The current placeholder is acceptable for a draft.

  TODO: Remove this comment before submitting
-->

TBD

## Backwards Compatibility
  
No backward compatibility issues found.

## Test Cases

Needs discussion.

## Reference Implementation

<!--
  This section is optional.

  The Reference Implementation section should include a minimal implementation that assists in understanding or implementing this specification. It should not include project build files. The reference implementation is not a replacement for the Specification section, and the proposal should still be understandable without it.
  If the reference implementation is too large to reasonably be included inline, then consider adding it as one or more files in `../assets/eip-####/`. External links will not be allowed.

  TODO: Remove this comment before submitting
-->

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
  
Here’s a simple example of using `TokenController` in an ERC721 token to define a "minter" and "burner" role, which allows accounts that have it create new tokens and destroy existing tokens by specifying the controll token:  
 
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

<!--
  All EIPs must contain a section that discusses the security implications/considerations relevant to the proposed change. Include information that might be important for security discussions, surfaces risks and can be used throughout the life cycle of the proposal. For example, include security-relevant design decisions, concerns, important discussions, implementation-specific guidance and pitfalls, an outline of threats and risks and how they are being addressed. EIP submissions missing the "Security Considerations" section will be rejected. An EIP cannot proceed to status "Final" without a Security Considerations discussion deemed sufficient by the reviewers.

  The current placeholder is acceptable for a draft.

  TODO: Remove this comment before submitting
-->

Needs discussion.

## Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).
