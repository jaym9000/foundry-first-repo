# Crowd-Sourcing FundMe Contract

## Overview
The **Crowd-Sourcing FundMe Contract** is a decentralized smart contract built on Ethereum using Solidity. It allows multiple users to contribute ETH towards a common fund. Contributions are validated based on a minimum value in USD, ensuring that the amount users send is equivalent to at least **5 USD worth of ETH**, regardless of ETH price fluctuations. This is made possible by integrating **Chainlink's price feed**.

Once contributions are collected, the contract's owner (the account that deployed it) can withdraw the accumulated funds.

---

## Features
- **Fund Contributions**: Users can contribute ETH if their contribution meets the minimum value of 5 USD (converted using Chainlink's price feed).
- **Minimum Contribution in USD**: Ensures all contributions are at least worth 5 USD in ETH, dynamically adjusting for market fluctuations.
- **Owner Withdrawal**: Only the owner (deployer) can withdraw the accumulated funds.
- **Fallback Mechanism**: Accepts direct ETH transfers via the `receive()` and `fallback()` functions.
- **Efficient Withdrawals**: Resets funders’ balances and clears the list of contributors upon withdrawal.

---

## Contract Structure

### 1. **FundMe Contract**
The **FundMe** contract manages fund contributions, withdrawals, and interacts with Chainlink's price feed for ETH to USD conversion.

**Key Variables**:
- `i_owner`: The owner of the contract (only they can withdraw funds).
- `MINIMUM_USD`: The minimum contribution in USD (5 USD). This is checked using the Chainlink price feed.
- `s_addressToAmountFunded`: A mapping that tracks how much ETH each address has contributed.
- `s_funders`: An array of addresses that have contributed ETH.
- `s_priceFeed`: The Chainlink price feed contract for ETH/USD price data.

**Key Functions**:
- **`fund()`**: Allows users to send ETH to the contract, as long as the contribution is greater than or equal to 5 USD.
    ```solidity
    function fund() public payable {
        require(
            msg.value.getConversionRate(s_priceFeed) >= MINIMUM_USD,
            "You need to spend more ETH!"
        );
        s_addressToAmountFunded[msg.sender] += msg.value;
        s_funders.push(msg.sender);
    }
    ```

- **`cheaperWithdraw()`**: Allows the owner to withdraw all funds and resets the list of funders.
    ```solidity
    function cheaperWithdraw() public onlyOwner {
        uint256 fundersLength = s_funders.length;
        for (uint256 funderIndex = 0; funderIndex < fundersLength; funderIndex++) {
            address funder = s_funders[funderIndex];
            s_addressToAmountFunded[funder] = 0;
        }
        s_funders = new address      (bool callSuccess, ) = payable(msg.sender).call{value: address(this).balance}("");
        require(callSuccess, "Call failed");
    }
    ```

- **`getVersion()`**: Returns the version of the Chainlink price feed.
- **`fallback()` & `receive()`**: Handle direct ETH transfers to the contract, automatically triggering the `fund()` function.

### 2. **PriceConverter Library**
The **PriceConverter** library contains functions to interact with Chainlink's price feed, converting ETH to USD.

### **Functions:**
- **`getPrice()`**: Retrieves the current ETH price in USD from the Chainlink price feed.
    ```solidity
    function getPrice(AggregatorV3Interface priceFeed) internal view returns (uint256) {
        (, int256 answer, , , ) = priceFeed.latestRoundData();
        return uint256(answer * 1e10); // Adjusts decimals
    }
    ```

- **`getConversionRate()`**: Converts a given amount of ETH to USD.
    ```solidity
    function getConversionRate(uint256 ethAmount, AggregatorV3Interface priceFeed) internal view returns (uint256) {
        uint256 ethPrice = getPrice(priceFeed);
        uint256 ethAmountInUsd = (ethPrice * ethAmount) / 1e18;
        return ethAmountInUsd;
    }
    ```

---

## How It Works

### 1. **Contributing to the Fund**
Users call the `fund()` function and send ETH to the contract. The contract ensures the contribution is worth at least 5 USD by converting ETH to USD using the Chainlink price feed.

```solidity
function fund() public payable {
    require(
        msg.value.getConversionRate(s_priceFeed) >= MINIMUM_USD,
        "You need to spend more ETH!"
    );
    s_addressToAmountFunded[msg.sender] += msg.value;
    s_funders.push(msg.sender);
}
```

If a user sends ETH directly to the contract, the receive() or fallback() functions are triggered, which call fund() and process the contribution.

### 2. **Checking Contributions**
Anyone can check the amount contributed by a specific address using the getAddressToAmountFunded() getter function:

```solidity
function getAddressToAmountFunded(address fundingAddress) external view returns (uint256);
```
### 3. **Withdrawing Funds**
Only the owner can withdraw the funds. This is done using the cheaperWithdraw() function, which also resets the contributions and clears the list of funders.

Example of the withdrawal logic:

```solidity
function cheaperWithdraw() public onlyOwner {
    uint256 fundersLenght = s_funders.length;
    for (uint256 funderIndex = 0; funderIndex < fundersLenght; funderIndex++) {
        address funder = s_funders[funderIndex];
        s_addressToAmountFunded[funder] = 0;
    }
    s_funders = new address ;
 ool callSuccess, ) = payable(msg.sender).call{value: address(this).balance}("");
    require(callSuccess, "Call failed");
}
```
## Deployment
### 1. **Deploying to Ethereum or Testnet**
To deploy the contract, you need to provide the address of the Chainlink ETH/USD price feed contract for the network you're deploying to. For example:

Sepolia Testnet: 0x0567F2323251f0A5f8fB9F4eFB9DBcc83C9B1B89 (ETH/USD Price Feed)
The contract constructor takes this price feed address as an argument:

```solidity
constructor(address pricefeed) {
    i_owner = msg.sender;
    s_priceFeed = AggregatorV3Interface(pricefeed);
}
```
### 2. **Interacting with the Contract**
After deployment, users can contribute funds using the fund() function. The owner can withdraw all accumulated funds using the cheaperWithdraw() function.

## **Getting Started**
### **Prerequisites**
- **`Solidity`**`: Basic understanding of Solidity and smart contract development.
- **`Node.js & npm`**`: Install Node.js (for JavaScript tooling).
- **`Ethereum Wallet`**`: Set up a wallet like MetaMask for interacting with Ethereum networks.
- **`Chainlink Price Feeds`**`: Familiarity with Chainlink's price feeds for ETH/USD conversion.

### **Installation**
Clone the repository:

```bash
git clone https://github.com/your-username/fund-me-contract.git
cd fund-me-contract
```
Install dependencies (if using Hardhat or Truffle):

```bash
npm install
```
### **Deploying the Contract**
Deploy the contract to your chosen Ethereum network using a tool like Hardhat or Truffle. You'll need the address of the Chainlink price feed for the network (e.g., Sepolia Testnet or Mainnet).

After deployment, interact with the contract using Web3.js or Ethers.js for calling functions like fund() and cheaperWithdraw().

### **License**
This project is licensed under the MIT License - see the LICENSE file for details.

### **Conclusion**
The Crowd-Sourcing FundMe Contract provides a decentralized, transparent way to collect and manage ETH contributions. It ensures that each contribution meets a fair value in USD by leveraging Chainlink’s real-time price feeds. The contract is simple to deploy and use, making it an ideal solution for Ethereum-based crowd funding applications.


# Thank you!

If you appreciated this, feel free to follow me or donate! *Below is JM's address*

ETH/Arbitrum/Optimism/Polygon/etc Address: 0x0f879bFF6c5cb229AFBaFCfFFE6C0FC29f95c796

[![Patrick Collins Twitter](https://img.shields.io/badge/Twitter-1DA1F2?style=for-the-badge&logo=twitter&logoColor=white)](https://x.com/JM_Mahoro)
[![Patrick Collins Linkedin](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/jeanmarcc/)


<!-- Testing krunchdata https://kdta.io/b6T40  -->