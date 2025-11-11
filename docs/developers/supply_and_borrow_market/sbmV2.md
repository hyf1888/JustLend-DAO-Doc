# SBM V2

JustLend DAO V2 is a fully upgraded decentralized lending protocol built on the TRON network. It introduces a dual-layer isolated margin mechanism composed of Vaults and Markets, along with an Adaptive Curve Interest Rate Model (IRM) for dynamic rate adjustment. The V2 system primarily consists of several core smart contracts — **Moolah Market**, **Moolah Vault**, **TRX Provider**, **Resilient Oracle**, and **Interest Rate Model** — each serving a distinct purpose within the ecosystem:

* **Moolah Market** handles core lending and borrowing operations, including supply, withdrawal, collateral management, and liquidation. **MoolahMarket.sol** allows users to:
    * Market Creation
    * Supply
    * Withdraw
    * Borrow
    * Repay
    * Supply Collateral
    * Withdraw Collateral
    * Liquidation
* **Moolah Vault** manages pooled user assets, allocates funds to lending markets, and issues transferable shares following the ERC-4626 standard. **MoolahVault.sol** allows users to:
    * Vault Creation
    * Deposit
    * Mint
    * Withdraw
* **TRX Provider** enables seamless interaction between native TRX and wrapped TRX (WTRX), allowing users to supply or borrow using TRX directly. **TRXProvider.sol** allows users to:
    * Deposit
    * Mint
    * Withdraw
    * Redeem
    * Borrow
    * Repay
    * Supply Collateral
    * Withdraw Collateral
* **Resilient Oracle** aggregates multiple price feeds (up to three per token) to ensure reliable and accurate price data for the protocol. 
* **Interest Rate Model (IRM)** dynamically adjusts borrowing and supply rates in real time to maintain market utilization near the optimal level. 

The source code is available on [Github](https://github.com/justlend/justlend-protocol/blob/main/contracts/CToken.sol).


## **Configuration**

### **1. Position**
In each market, every user has a corresponding Position, which records the user’s supplied, borrowed, and collateralized assets within that market.
``` solidity
struct Position {
  uint256 supplyShares;
  uint128 borrowShares;
  uint128 collateral;
}
```
* **supplyShares:** the number of shares representing the assets a user has supplied to the market.
* **borrowShares:** the number of shares representing the assets a user has borrowed from the market.
* **collateral::** the amount of assets a user has deposited as collateral in the market.


### **Get Cash**
Calling this method gets the total amount of underlying balance currently available to this market.
``` solidity
function getCash() public view returns (uint)
```

* **Parameter description:** N/A
* **Returns:** The quantity of underlying assets owned by this contract.


### **Total Borrows**
Calling this method gets the sum of the currently loaned-outs and the accrued interests.
``` solidity
function totalBorrowsCurrent() external nonReentrant returns (uint)
```

* **Parameter description:** N/A
* **Returns:** The total borrows with interest.


### **Borrow Balance**
Calling this method accrues interest to the updated borrowIndex and then calculates the account's borrow balance using the updated borrowIndex.
``` solidity
function borrowBalanceCurrent(address account) external nonReentrant returns (uint)
```

* **Parameter description:**
    * `account:` the address whose balance should be calculated after updating borrowIndex.
* **Returns:** The total borrows with interest.


### **Borrow Rate**
Calling this method gets the current per-block borrow interest rate for this jToken.
``` solidity
function borrowRatePerBlock() external view returns (uint)
```

* **Parameter description:** N/A
* **Returns:** The borrow interest rate per block, scaled by 1e18.


### **Total Supply**
Calling this method gets the total number of tokens in circulation.
``` solidity
function totalSupply() external view returns (uint256)
```

* **Parameter description:** N/A
* **Returns:** The supply of tokens.


### **Underlying Balance**
Calling this method gets the underlying balance of the owner.
``` solidity
function balanceOfUnderlying(address owner) external returns (uint)
```

* **Parameter description:**
    * `owner:` the address of the account.
* **Returns:** The amount of underlying owned by owner.


### **Supply Rate**
Calling this method gets the current per-block supply interest rate for this jToken.
``` solidity
function supplyRatePerBlock() external view returns (uint)
```

* **Parameter description:** N/A
* **Returns:** The supply interest rate per block, scaled by 1e18.


### **Total Reserves**
Calling this method gets the reserves. Reserve represents a portion of historical interest set aside as cash which can be withdrawn or transferred through the protocol's governance.
``` solidity
function totalReserves() returns (uint)
```

* **Parameter description:** N/A
* **Returns:** The total amount of reserves.


### **Reserve Factor**
Calling this method gets the current reserve factor.
``` solidity
function reserveFactorMantissa() returns (uint)
```

* **Parameter description:** N/A
* **Returns:** The current reserve factor.


### **Liquidation Incentive**
By calling the liquidationIncentiveMantissa function of the Unitroller contract, liquidation incentives can be inquired. Liquidators will be given a proportion of the borrower's collateral as an incentive, which is defined as liquidation incentive. This is to encourage liquidators to perform liquidation of underwater accounts.
``` solidity
function liquidationIncentiveMantissa() view returns (uint)
```

* **Parameter description:** N/A
* **Returns:** The liquidationIncentive, scaled by 1e18, is multiplied by the closed borrow amount from the liquidator to determine how much collateral can be seized.


### **Get Account Liquidity**
By calling the getAccountLiquidity function of the Unitroller contract, account information can be accessed through an account's address to determine whether the account should be liquidated or not.
``` solidity
getAccountLiquidity(address account) view returns (uint, uint, uint)
```

* **Parameter description:**
    * `account:` user address.
* **Returns:** The amount of underlying owned by owner.
    * `error:` error code, 0 means success.
    * `liquidity:` liquidity.
    * `shortfall:` When the value is bigger than 0, the current account does not meet the market requirement for collateralization and needs to be liquidated.

Note: There should be at most one non-zero value between liquidity and shortfall.



## **Contracts ABI**

### **Borrow**
Calling this method borrows assets from JustLend DAO protocol to the sender's owner address.
``` solidity
function borrow(uint borrowAmount) external returns (uint)
```

* **Parameter description:**
    * `borrowAmount:` the amount of the underlying asset to borrow.
* **Returns:** None, reverts on error.

#### **Event**
``` solidity
Borrow(address borrower, uint borrowAmount, uint accountBorrows, uint totalBorrows, uint borrowIndex)
```

* Emits when user successfully borrow.
    * `borrower:` address of borrow assets account;
    * `borrowAmount:` the amount of borrowed assets;
    * `accountBorrows:` the account borrow the assets;
    * `totalBorrows:` total borrow assets form the account;
    * `borrowIndex:` the index of this borrow order.


### **repayBorrow**
Calling this method repays their own borrow.
``` solidity
function repayBorrow(uint amount) external payable
```

* **Parameter description:**
    * `amount:` the amount of the asset to repay.
* **Returns:** None, reverts on error.

#### **Event**
``` solidity
RepayBorrow(address payer, address borrower, uint repayAmount, uint accountBorrows, uint totalBorrows, uint borrowIndex)
```

* Emits when user successfully repay borrow.
    * `payer:` operate repay borrow;
    * `borrower:` address of borrow assets account;
    * `repayAmount:` the amount of repaid assets;
    * `accountBorrows:` the account borrow the assets;
    * `totalBorrows:` total borrow assets form the account;
    * `borrowIndex:` the index of this borrow order.
