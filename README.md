# 🛡️ Deny users from accessing a smart contract

A Denial of Service (DOS) attack is a type of attack that is designed to disable, shut down, or disrupt a network, website, or service. Essentially it means that the attacker somehow can prevent regular users from accessing the network, website, or service therefore denying them service. This is a very common attack which we all know about in web2 as well but today we will try to imitate a Denial of Service attack on a smart contract

# 🤔 Overview

There will be two smart contracts - Good.sol and Attack.sol. Good.sol will be used to run a sample auction where it will have a function in which the current user can become the current winner of the auction by sending Good.sol higher amount of ETH than was sent by the previous winner. After the winner is replaced, the old winner is sent back the money which he initially sent to the contract.

Attack.sol will attack in such a manner that after becoming the current winner of the auction, it will not allow anyone else to replace it even if the address trying to win is willing to put in more ETH. Thus Attack.sol will bring Good.sol under a DOS attack because after it becomes the winner, it will deny the ability for any other address to becomes the winner.

```shell
gitpod /workspace/LW3-denial-of-service (main) $ npx hardhat test
Compiled 3 Solidity files successfully


  Denial of Service
Good Contract's Address: 0x5FbDB2315678afecb367f032d93F642f64180aa3
Attack Contract's Address 0xe7f1725E7734CE288F8367e1Bb143E90bb3F0512
    ✔ After being declared the winner, Attack.sol should not allow anyone else to become the winner (3916ms)

  Lock
    Deployment
      ✔ Should set the right unlockTime (181ms)
      ✔ Should set the right owner
      ✔ Should receive and store the funds to lock
      ✔ Should fail if the unlockTime is not in the future
    Withdrawals
      Validations
        ✔ Should revert with the right error if called too soon (64ms)
        ✔ Should revert with the right error if called from another account
        ✔ Shouldn't fail if the unlockTime has arrived and the owner calls it (69ms)
      Events
        ✔ Should emit an event on withdrawals
      Transfers
        ✔ Should transfer the funds to the owner


  10 passing (4s)
```
# 👮 Prevention

You can create a separate withdraw function for the previous winners.

## Example:
```shell
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.4;

contract Good {
    address public currentWinner;
    uint public currentAuctionPrice;
    mapping(address => uint) public balances;

    constructor() {
        currentWinner = msg.sender;
    }

    function setCurrentAuctionPrice() public payable {
        require(msg.value > currentAuctionPrice, "Need to pay more than the currentAuctionPrice");
        balances[currentWinner] += currentAuctionPrice;
        currentAuctionPrice = msg.value;
        currentWinner = msg.sender;
    }

    function withdraw() public {
        require(msg.sender != currentWinner, "Current winner cannot withdraw");

        uint amount = balances[msg.sender];
        balances[msg.sender] = 0;

        (bool sent, ) = msg.sender.call{value: amount}("");
        require(sent, "Failed to send Ether");
    }
}
```