# Lamina1-Creating-L1-Token-Contract

## 1. Install Dependencies

In a new project, initialize npm and install required dependencies like OpenZeppelin contracts, ethers.js, and Hardhat for deployment:

```
npm init -y
npm install @openzeppelin/contracts ethers hardhat
```
Initialize a new Hardhat project:

```
npx hardhat
```
Select "Create an empty hardhat.config.js".

## 2. Develop the ERC-20 Token Smart Contract

Create a new Solidity file in contracts/L1Token.sol:

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract L1Token is ERC20, Ownable {
    constructor(uint256 initialSupply) ERC20("Lamina1Token", "L1T") {
        _mint(msg.sender, initialSupply);
    }

    function mint(address to, uint256 amount) public onlyOwner {
        _mint(to, amount);
    }

    function burn(uint256 amount) public {
        _burn(msg.sender, amount);
    }
}
```
This contract includes:

- mint: Allows the owner to mint new tokens.
- burn: Allows token holders to burn their tokens.
  
## 3. Configure Hardhat for Deployment

In hardhat.config.js, configure the network to connect to Lamina1 or any other compatible blockchain network:

```
require('@nomiclabs/hardhat-ethers');

module.exports = {
  solidity: "0.8.18",
  networks: {
    lamina1: {
      url: "https://rpc.lamina1.com", // Replace with Lamina1 RPC URL
      accounts: [process.env.PRIVATE_KEY] // Your private key
    }
  }
};
```
Make sure to load environment variables using dotenv if necessary:

```
npm install dotenv
```
Then, in your hardhat.config.js, import dotenv and load the environment variables:

```
require('dotenv').config();
```
## 4. Write Deployment Script

Create a new script scripts/deploy.js to deploy your contract:

```
async function main() {
    const [deployer] = await ethers.getSigners();
    console.log("Deploying contracts with the account:", deployer.address);

    const Token = await ethers.getContractFactory("L1Token");
    const token = await Token.deploy(1000000); // Mint 1 million tokens

    console.log("Token deployed to:", token.address);
}

main()
    .then(() => process.exit(0))
    .catch((error) => {
        console.error(error);
        process.exit(1);
    });
```
## 5. Deploy the Contract

To deploy the contract, run the following command:

```
npx hardhat run scripts/deploy.js --network lamina1
```
Make sure the private key and network details are correctly set in your environment.

## 6. Interact with the Token

Once the contract is deployed, you can interact with it using ethers.js. Here’s an example of interacting with your token:

### a) Check Balance
```
const token = await ethers.getContractAt("L1Token", tokenAddress);
const balance = await token.balanceOf(deployer.address);
console.log("Token balance:", balance.toString());
```
### b) Transfer Tokens
```
await token.transfer(recipientAddress, ethers.utils.parseUnits("100", 18)); // Send 100 tokens
```
### c) Mint New Tokens
```
await token.mint(recipientAddress, ethers.utils.parseUnits("500", 18)); // Mint 500 tokens
```
### d) Burn Tokens
```
await token.burn(ethers.utils.parseUnits("50", 18)); // Burn 50 tokens
```
## 7. Testing the Contract

In your test/L1Token.test.js, write tests to ensure your contract behaves correctly:

```
const { expect } = require("chai");

describe("L1Token contract", function () {
    let Token;
    let token;
    let owner;
    let addr1;
    let addr2;

    beforeEach(async function () {
        Token = await ethers.getContractFactory("L1Token");
        [owner, addr1, addr2, _] = await ethers.getSigners();
        token = await Token.deploy(1000000); // Mint 1 million tokens
    });

    it("Should assign the total supply of tokens to the owner", async function () {
        const ownerBalance = await token.balanceOf(owner.address);
        expect(await token.totalSupply()).to.equal(ownerBalance);
    });

    it("Should transfer tokens between accounts", async function () {
        await token.transfer(addr1.address, 50);
        const addr1Balance = await token.balanceOf(addr1.address);
        expect(addr1Balance).to.equal(50);

        await token.connect(addr1).transfer(addr2.address, 50);
        const addr2Balance = await token.balanceOf(addr2.address);
        expect(addr2Balance).to.equal(50);
    });

    it("Should mint new tokens", async function () {
        await token.mint(addr1.address, 100);
        const addr1Balance = await token.balanceOf(addr1.address);
        expect(addr1Balance).to.equal(100);
    });

    it("Should burn tokens", async function () {
        await token.burn(100);
        const ownerBalance = await token.balanceOf(owner.address);
        expect(ownerBalance).to.equal(999900); // 100 burned from 1 million
    });
});
```
Run tests with:

```
npx hardhat test
```
## 8. Conclusion
This guide shows how to create, deploy, and interact with an ERC-20 token on Lamina1. We've also included minting and burning functionality, along with full unit tests to ensure the contract behaves as expected.
