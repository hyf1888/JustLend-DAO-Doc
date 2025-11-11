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

<br>

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
* **collateral:** the amount of assets a user has deposited as collateral in the market.


### **2. Market**
``` solidity
struct Market {
  uint128 totalSupplyAssets;
  uint128 totalSupplyShares;
  uint128 totalBorrowAssets;
  uint128 totalBorrowShares;
  uint128 lastUpdate;
  uint128 fee;
}
```
* **totalSupplyAssets:** the total amount of assets supplied to the market and available for lending.
* **totalSupplyShares:** the total number of supply shares in the market.
* **totalBorrowAssets:** the total amount of assets borrowed from the market.
* **totalBorrowShares:** the total number of borrowed shares in the market.
* **lastUpdate:** the timestamp of the last market state update.
* **fee:** the fee rate charged by the market.

**Relationship between assets and shares:** At the beginning, one token corresponds to one share. Over time, as the market generates interest income, the total amount of tokens increases while the total number of shares remains constant. Consequently, each share becomes redeemable for more tokens, allowing users to earn yield when they withdraw their funds.


### **3. MarketParams**
``` solidity
struct MarketParams {
  address loanToken;
  address collateralToken;
  address oracle;
  address irm;
  uint256 lltv;
}
```
* **loanToken:** the address of the token used for borrowing.
* **collateralToken:** the address of the token used as collateral.
* **oracle:** the address of the price oracle contract.
* **irm:** the address of the interest rate model contract.
* **lltv:** the loan-to-value ratio (LTV) that determines the maximum borrowing limit based on the collateral value.


### **4. MarketConfig**
Represents the configuration of each market within the vault.
``` solidity
struct MarketConfig {
  uint176 cap;
  bool enabled;
  MarketType marketType;
  uint64 removableAt;
}
```
* **cap:** the maximum amount of assets that the vault can supply to this market.
* **enabled:** indicates whether the market is active; serves as a switch to control whether investments can be made to the market.
* **marketType:** the type of the market. This is used for compatibility with other lending protocols. Currently, only the Moolah market type is supported.
* **removableAt:** a timestamp indicating when the market can be immediately removed from the withdrawal queue.


### **5. MarketAllocation**
Used when reallocating asset investments across different markets.
``` solidity
struct MarketAllocation {
  MarketParams marketParams;
  uint256 assets;
}
```
* **marketParams:** specifies the target market to which the allocation is applied.
* **assets:** the amount of assets allocated to the specified market.


### **6. Authorization**
The Authorization struct represents the data required to grant or revoke authorization for a specific address.
``` solidity
struct Authorization {
    address authorizer;   
    address authorized;   
    bool isAuthorized;   
    uint256 nonce;        
    uint256 deadline;    
}
```
* **authorizer:** the address that is granting authorization or revocation.
* **authorized:** the address that is being granted or revoked the authorization.
* **isAuthorized:** a boolean indicating whether the authorization is being granted (true) or revoked (false).
* **nonce:** a unique number to prevent replay attacks. This value must be incremented each time a new authorization is signed.
* **deadline:** a timestamp indicating when the authorization will expire. If the current block timestamp exceeds this value, the authorization is no longer valid.


### **7. Signature**
The Signature struct represents the ECDSA (Elliptic Curve Digital Signature Algorithm) signature used to verify that a particular authorization was signed by the authorizer.
``` solidity
struct Signature {
    uint8 v;    
    bytes32 r;    
    bytes32 s;   
}
```
* **v:** The recovery id (0 or 1) used to recover the public key from the signature. It indicates which of the two possible curve points was used for signing.
* **r:** The "r" component of the ECDSA signature, which is derived from the elliptic curve.
* **s:** The "s" component of the ECDSA signature, which is another part of the signature that helps uniquely identify it.


## **Contracts ABI**

### **Moolah Market**
The JustLend DAO V2 market adopts a one-to-one model, where each market consists of a single collateral asset paired with a single borrowable asset. All markets are managed within the Moolah contract. Each market is uniquely identified by a Market ID, which is composed of the borrowable asset, collateral asset, oracle, interest rate model, and liquidation factor.

#### **1. Create Market**
The Create Market function creates a new market in JustLend DAO V2  by initializing a set of parameters within the Moolah contract.
``` solidity
function createMarket(MarketParams memory marketParams) external
```
* **Parameter description:**
    * `marketParams:` the configuration parameters of the target market (includes loan token, collateral token, oracle, interest rate model, and LTV ratio).
* **Returns:** None, reverts on error.

**Event**
``` solidity
CreateMarket(Id indexed id, MarketParams marketParams)
```
* This event is emitted when successfully creating the market.
    * `Id:` each market is identified by a unique Id, represented as a bytes32 value.
    * `marketParams:` the configuration parameters of the target market (includes loan token, collateral token, oracle, interest rate model, and LTV ratio).


