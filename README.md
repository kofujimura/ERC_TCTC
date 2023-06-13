---
title: Token-Controlled Token Circulation 
description: Access control scheme based on token ownership.
author: Ko Fujimura (@kofujimura)
discussions-to: <URL>
status: Draft
type: Standards Track or Informational
category: ERC
created: 2023-06-11
requires: <EIP number(s)> # Only required when you reference an EIP in the `Specification` section. Otherwise, remove this field.
---
## Abstract

This EIP presents a new access control scheme called Token-Control Token Circulation (TCTC). By representing roles or privileges as tokens, the need for developing off-chain tools that grant or revoke role is eliminated. This approach can potentially enhance system security and decrease development costs.
  
## Motivation

There are numerous methods to implement access control for privileged actions. A commonly utilized pattern is "role-based" access control as specified in ERC-5982. This method, however, necessitates the use of an off-chain management tool to grant or revoke required roles through its interface. Additionally, as many wallets lack a user interface that displays the privileges granted by a role, users are often unable to comprehend the status of their privileges through the wallet.
  
Use case ...

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

1. Smart contracts implementing the ERC-XXXX standard MUST implement privileges required by the role as token specified in ERC721 interface.
2. To assign a role to the account, ERC721 `mint` method MUST be used.
3. To revoke a role from the account, ERC721 `burn` method MUST be used.
4. To check if the account has the required role, check that the balance of the control token MUST greater than 0 using ERC721 `balanceOf` method. 
5. A role in a compliant smart contract is represented in the format of `bytes32`. It's RECOMMENDED the value of such role is computed as a
`keccak256` hash of a string of the role name, in this format: `bytes32 role = keccak256("<role_name>")`. such as `bytes32 role = keccak256("MINTER")`.
  
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
// Author: Ko Fujimura <ko@fujimura.com>
// Open source repo: https://github.com/kofujimura/TCTC

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
  
  
## Security Considerations

<!--
  All EIPs must contain a section that discusses the security implications/considerations relevant to the proposed change. Include information that might be important for security discussions, surfaces risks and can be used throughout the life cycle of the proposal. For example, include security-relevant design decisions, concerns, important discussions, implementation-specific guidance and pitfalls, an outline of threats and risks and how they are being addressed. EIP submissions missing the "Security Considerations" section will be rejected. An EIP cannot proceed to status "Final" without a Security Considerations discussion deemed sufficient by the reviewers.

  The current placeholder is acceptable for a draft.

  TODO: Remove this comment before submitting
-->

Needs discussion.

## Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).
