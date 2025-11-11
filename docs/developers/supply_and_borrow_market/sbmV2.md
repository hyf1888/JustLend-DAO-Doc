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
* `supplyShares:` the number of shares representing the assets a user has supplied to the market.
* `borrowShares:` the number of shares representing the assets a user has borrowed from the market.
* `collateral:` the amount of assets a user has deposited as collateral in the market.


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
* `totalSupplyAssets:` the total amount of assets supplied to the market and available for lending.
* `totalSupplyShares:` the total number of supply shares in the market.
* `totalBorrowAssets:` the total amount of assets borrowed from the market.
* `totalBorrowShares:` the total number of borrowed shares in the market.
* `lastUpdate:` the timestamp of the last market state update.
* `fee:` the fee rate charged by the market.

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
* `loanToken:` the address of the token used for borrowing.
* `collateralToken:` the address of the token used as collateral.
* `oracle:` the address of the price oracle contract.
* `irm:` the address of the interest rate model contract.
* `lltv:` the loan-to-value ratio (LTV) that determines the maximum borrowing limit based on the collateral value.


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
* `cap:` the maximum amount of assets that the vault can supply to this market.
* `enabled:` indicates whether the market is active; serves as a switch to control whether investments can be made to the market.
* `marketType:` the type of the market. This is used for compatibility with other lending protocols. Currently, only the Moolah market type is supported.
* `removableAt:` a timestamp indicating when the market can be immediately removed from the withdrawal queue.


### **5. MarketAllocation**
Used when reallocating asset investments across different markets.
``` solidity
struct MarketAllocation {
  MarketParams marketParams;
  uint256 assets;
}
```
* `marketParams:` specifies the target market to which the allocation is applied.
* `assets:` the amount of assets allocated to the specified market.


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
* `authorizer:` the address that is granting authorization or revocation.
* `authorized:` the address that is being granted or revoked the authorization.
* `isAuthorized:` a boolean indicating whether the authorization is being granted (true) or revoked (false).
* `nonce:` a unique number to prevent replay attacks. This value must be incremented each time a new authorization is signed.
* `deadline:` a timestamp indicating when the authorization will expire. If the current block timestamp exceeds this value, the authorization is no longer valid.


### **7. Signature**
The Signature struct represents the ECDSA (Elliptic Curve Digital Signature Algorithm) signature used to verify that a particular authorization was signed by the authorizer.
``` solidity
struct Signature {
    uint8 v;    
    bytes32 r;    
    bytes32 s;   
}
```
* `v:` the recovery id (0 or 1) used to recover the public key from the signature. It indicates which of the two possible curve points was used for signing.
* `r:` the "r" component of the ECDSA signature, which is derived from the elliptic curve.
* `s:` the "s" component of the ECDSA signature, which is another part of the signature that helps uniquely identify it.

<br>

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
    * `id:` each market is identified by a unique Id, represented as a bytes32 value.
    * `marketParams:` the configuration parameters of the target market (includes loan token, collateral token, oracle, interest rate model, and LTV ratio).


#### **2. Supply**
The supply function allows users to provide liquidity to the market for lending.
``` solidity
function supply(MarketParams memory marketParams, uint256 assets, uint256 shares, address onBehalf, bytes calldata data)
```
* **Parameter description:**
    * `marketParams:` the configuration parameters of the target market (includes loan token, collateral token, oracle, interest rate model, and LTV ratio).
    * `assets:` the amount of tokens the user wants to supply to the market.
    * `shares:` the number of supply shares to mint corresponding to the deposited assets.
    * `onBehalf:` the address that will receive the supply shares. This allows users to supply assets on behalf of another account.
    * `data:` additional encoded data for interaction logic or integrations.
* **Returns:** None, reverts on error.

**Event**
``` solidity
Supply(Id indexed id, address indexed caller, address indexed onBehalf, uint256 assets, uint256 shares)
```
* This event is emitted when assets are supplied to a market.
    * `id:` the unique market identifier (bytes32), representing the market where the supply occurred.
    * `caller:` the address that submitted the transaction.
    * `onBehalf:` the address on whose behalf the assets were supplied, the supply shares are credited to this address.
    * `assets:` the amount of tokens supplied to the market.
    * `shares:` the number of supply shares minted corresponding to the supplied assets.


#### **3. Withdraw**
The withdraw  function allows users to withdraw the previously supplied liquidity from the market.
``` solidity
function withdraw(MarketParams memory marketParams, uint256 assets, uint256 shares, address onBehalf, address receiver)
```
* **Parameter description:**
    * `marketParams:` the configuration parameters of the target market (includes loan token, collateral token, oracle, interest rate model, and LTV ratio).
    * `assets:` the amount of tokens to withdraw from the market.
    * `shares:` the number of supply shares to redeem for the corresponding assets.
    * `onBehalf:` the address whose position will be reduced. This enables withdrawals to be made on behalf of another account.
    * `receiver:` the address that will receive the withdrawn assets.
* **Returns:** None, reverts on error.

**Event**
``` solidity
Withdraw(Id indexed id, address caller, address indexed onBehalf, address indexed receiver, uint256 assets, uint256 shares)
```
* This event is emitted when assets are withdrawn from a market.
    * `id:` the unique market identifier (bytes32), representing the market from which the withdrawal occurred.
    * `caller:` the address that submitted the transaction.
    * `onBehalf:` the address whose supply shares are being withdrawn.
    * `receiver:` the address that receives the withdrawn assets.
    * `assets:` the amount of tokens withdrawn from the market.
    * `shares:` the number of supply shares redeemed corresponding to the withdrawn assets.


#### **4. Borrow**
The borrow function allows users to borrow funds from the JustLend DAO V2 market.
``` solidity
function borrow(MarketParams memory marketParams, uint256 assets, uint256 shares, address onBehalf, address receiver)
```
* **Parameter description:**
    * `marketParams:` the configuration parameters of the target market (includes loan token, collateral token, oracle, interest rate model, and LTV ratio).
    * `assets:` the amount of tokens the user intends to borrow from the market.
    * `shares:` the number of borrowing shares to mint corresponding to the borrowed assets.
    * `onBehalf:` the address whose position will be increased. This allows borrowing on behalf of another account.
    * `receiver:` the address that will receive the borrowed assets.
* **Returns:** None, reverts on error.

**Event**
``` solidity
Borrow(Id indexed id, address caller, address indexed onBehalf, address indexed receiver, uint256 assets, uint256 shares)
```
* This event is emitted when assets are borrowed from a market.
    * `id:` the unique market identifier (bytes32), representing the market from which the borrowing occurred.
    * `caller:` the address that submitted the transaction.
    * `onBehalf:` the address whose borrowed shares are recorded.
    * `receiver:` the address that receives the borrowed assets.
    * `assets:` the amount of tokens borrowed from the market.
    * `shares:` the number of borrowed shares corresponding to the borrowed assets.
 

#### **5. Repay**
The repay function allows users to repay the funds borrowed from the JustLend DAO V2 market.
``` solidity
function repay(MarketParams memory marketParams, uint256 assets, uint256 shares, address onBehalf, bytes calldata data)
```
* **Parameter description:**
    * `marketParams:` the configuration parameters of the target market (includes loan token, collateral token, oracle, interest rate model, and LTV ratio).
    * `assets:` the amount of tokens to repay to the market.
    * `shares:` the number of borrowing shares to burn corresponding to the repaid assets.
    * `onBehalf:` the address whose borrowing position will be reduced. This enables repayments to be made on behalf of another account.
    * `data:` additional encoded data for interaction logic or integrations.
* **Returns:** None, reverts on error.

**Event**
``` solidity
Repay(Id indexed id, address indexed caller, address indexed onBehalf, uint256 assets, uint256 shares)
```
* This event is emitted when borrowed assets are repaid to the market.
    * `id:` the unique market identifier (bytes32), representing the market where the repayment occurred.
    * `caller:` the address that submitted the transaction.
    * `onBehalf:` the address whose borrowed assets are being repaid.
    * `assets:` the amount of tokens repaid to the market.
    * `shares:` the number of borrowed shares corresponding to the repaid assets.


#### **6. Supply Collateral**
The supply collateral function allows users to supply collateral assets to the JustLend DAO V2 market.
``` solidity
function supplyCollateral(MarketParams memory marketParams, uint256 assets, address onBehalf, bytes calldata data)
```
* **Parameter description:**
    * `marketParams:` the configuration parameters of the target market (includes loan token, collateral token, oracle, interest rate model, and LTV ratio).
    * `assets:` the amount of collateral tokens to deposit into the market.
    * `onBehalf:` the address whose collateral position will be increased. This allows users to add collateral on behalf of another account.
    * `data:` additional encoded data for interaction logic or integrations.
* **Returns:** None, reverts on error.

**Event**
``` solidity
SupplyCollateral(Id indexed id, address indexed caller, address indexed onBehalf, uint256 assets)
```
* This event is emitted when collateral is supplied to the market.
    * `id:` the unique market identifier (bytes32), representing the market where the collateral is supplied.
    * `caller:` the address that submitted the transaction.
    * `onBehalf:` the address whose collateral balance is credited.
    * `assets:` the amount of collateral tokens supplied to the market.


#### **7. Withdraw Collateral**
The withdraw collateral function allows users to withdraw their previously supplied collateral from the JustLend DAO V2 market.
``` solidity
function withdrawCollateral(MarketParams memory marketParams, uint256 assets, address onBehalf, address receiver)
```
* **Parameter description:**
    * `marketParams:` the configuration parameters of the target market (includes loan token, collateral token, oracle, interest rate model, and LTV ratio).
    * `assets:` the amount of collateral tokens to withdraw from the market.
    * `onBehalf:` the address whose collateral position will be reduced. This allows collateral to be withdrawn on behalf of another account.
    * `receiver:` the address that will receive the withdrawn collateral assets.
* **Returns:** None, reverts on error.

**Event**
``` solidity
WithdrawCollateral(Id indexed id, address caller, address indexed onBehalf, address indexed receiver, uint256 assets)
```
* This event is emitted when collateral is withdrawn from the market.
    * `id:` the unique market identifier (bytes32), representing the market where the collateral is withdrawn.
    * `caller:` the address that submitted the transaction.
    * `onBehalf:` the address whose collateral balance is reduced.
    * `receiver:` the address that receives the withdrawn collateral assets.
    * `assets:` the amount of collateral tokens withdrawn from the market.


#### **8. Liquidate**
The liquidate function allows liquidators to repay a portion of a borrower's debt in exchange for seizing the borrower's collateral when their position becomes undercollateralized.
``` solidity
function liquidate(MarketParams memory marketParams, address borrower, uint256 seizedAssets, uint256 repaidShares, bytes calldata data) 
```
* **Parameter description:**
    * `marketParams:` the configuration parameters of the target market (includes loan token, collateral token, oracle, interest rate model, and LTV ratio).
    * `borrower:` the address of the user whose position is being liquidated.
    * `seizedAssets:` the amount of collateral assets to be seized from the borrower.
    * `repaidShares:` the number of borrowing shares repaid on behalf of the borrower.
    * `data:` additional encoded data for custom liquidation logic or integrations.
* **Returns:** None, reverts on error.

**Event**
``` solidity
Liquidate(Id indexed id, address indexed caller, address indexed borrower, uint256 repaidAssets, uint256 repaidShares, uint256 seizedAssets, uint256 badDebtAssets, uint256 badDebtShares)
```
* This event is emitted when a liquidation occurs in the market.
    * `id:` the unique market identifier (bytes32), representing the market where the liquidation takes place.
    * `caller:` the address that submitted the transaction (the liquidator).
    * `borrower:` the address of the borrower being liquidated.
    * `repaidAssets:` the amount of borrowed assets repaid during the liquidation.
    * `repaidShares:` the number of borrowed shares repaid.
    * `seizedAssets:` the amount of collateral assets seized by the liquidator.
    * `badDebtAssets:` the amount of assets classified as bad debt after liquidation.
    * `badDebtShares:` the number of borrowed shares corresponding to the bad debt.


#### **9. Set Authorization**
Sets or updates the authorization status for a specific address.
``` solidity
function setAuthorization(address authorized, bool newIsAuthorized)
```
* **Parameter description:**
    * `authorized:` the address to grant or revoke authorization.
    * `newIsAuthorized:` a boolean indicating the authorization status, true to authorize, false to revoke.
* **Returns:** None, reverts on error.

**Event**
``` solidity
SetAuthorization(address indexed caller, address indexed authorizer, address indexed authorized, bool newIsAuthorized)
```
* This event is emitted when authorization status for an address is updated.
    * `caller:` the address that initiated the transaction.
    * `authorizer:` the address granting the authorization.
    * `authorized:` the address receiving the authorization.
    * `newIsAuthorized:` the new authorization status, true for granted, false for revoked.


#### **10. Set Authorization With Signature**
Sets authorization for an address using an off-chain signature, while recording a nonce to prevent replay attacks.
``` solidity
function setAuthorizationWithSig(Authorization memory authorization, Signature calldata signature)
```
* **Parameter description:**
    * `authorization:` the authorization data structure containing the authorization details.
    * `signature:` the ECDSA signature proving that the authorization was signed by the authorizer.
* **Returns:** None, reverts on error.


#### **11. Accrue Interest**
The accrueInterest function updates the interest accrued in the JustLend DAO V2 market based on the latest block timestamp. It synchronizes the market’s supply and borrow states to reflect the most recent interest calculations.
``` solidity
function accrueInterest(MarketParams memory marketParams)
```
* **Parameter description:**
    * `marketParams:` the configuration parameters of the target market (includes loan token, collateral token, oracle, interest rate model, and LTV ratio).
* **Returns:** None, reverts on error.

**Event**
``` solidity
AccrueInterest(Id indexed id, uint256 prevBorrowRate, uint256 interest, uint256 feeShares)
```
* This event is emitted during the execution of functions such as borrow, repay, and supply, which update the market’s borrowing state.
    * `id:` the unique market identifier (bytes32), representing the market where the interest is accrued.
    * `prevBorrowRate:` the previous borrowing rate before the update.
    * `interest:` the number of interest accrued in this update.
    * `feeShares:` the portion of accrued interest distributed to the fee recipient, represented in shares.
 

#### **12. Is Healthy**
Checks whether a borrower’s account in a specific market is healthy. 
``` solidity
function isHealthy(MarketParams memory marketParams, Id id, address borrower) external view returns (bool)
```
* **Parameter description:**
    * `marketParams:` the configuration parameters of the target market (includes loan token, collateral token, oracle, interest rate model, and LTV ratio).
    * `id:` the market’s unique identifier.
    * `borrower:` the address of the borrower whose account health is being checked.
* **Returns:** None, reverts on error.


#### **13. Get Price**
Retrieves the relative price between the collateral token and the loan token in a specific market.
``` solidity
function getPrice(MarketParams memory marketParams) public view returns (uint256)
```
* **Parameter description:**
    * `marketParams:` the configuration parameters of the target market (includes loan token, collateral token, oracle, interest rate model, and LTV ratio).
* **Returns:** the value of one collateral token expressed in loan token units.


#### **14. Get WhiteList**
Retrieves the whitelist of a specified market.
``` solidity
function getWhiteList(Id id) external view returns (address[])
```
* **Parameter description:**
    * `id:` the market’s unique identifier.
* **Returns:** the list of addresses included in the whitelist for the specified market.


#### **15. Is WhiteList**
Checks whether a specific account is included in the whitelist of the market identified by id.
``` solidity
function isWhiteList(Id id, address account) public view returns (bool)
```
* **Parameter description:**
    * `id:` the market’s unique identifier.
    * `account:` the address to check for whitelist eligibility.
* **Returns:** true if the whitelist is not set or if the account is included in it; otherwise, false.


#### **16. Get Liquidation Whitelist**
Retrieves the list of addresses in the liquidation whitelist for the specified market identified by id.
``` solidity
function getLiquidationWhitelist(Id id) external view returns (address[])
```
* **Parameter description:**
    * `id:` the unique identifier of the market.
* **Returns:** a list of addresses authorized to perform liquidation operations in the specified market.


#### **17. Is Liquidation Whitelist**
Checks whether a given account is included in the liquidation whitelist for the specified market.
``` solidity
function isLiquidationWhitelist(Id id, address account) external view returns (bool)
```
* **Parameter description:**
    * `id:` the unique identifier of the market.
    * `account:` the address to be checked in the liquidation whitelist.
* **Returns:** returns true if the account is in the whitelist; otherwise, returns false.


#### **18. Minimum Loan**
Retrieves the minimum loan amount allowed in the specified market.
``` solidity
function minLoan(MarketParams memory marketParams) public view returns (uint256)
```
* **Parameter description:**
    * `marketParams:` the configuration parameters of the target market (includes loan token, collateral token, oracle, interest rate model, and LTV ratio).
* **Returns:** the minimum amount of loan assets that can be borrowed from the market.


#### **19. Get Id**
Returns the unique identifier (ID) corresponding to the specified market.
``` solidity
function getId(MarketParams memory marketParams) external pure returns (bytes32 id)
```
* **Parameter description:**
    * `marketParams:` the configuration parameters of the target market (includes loan token, collateral token, oracle, interest rate model, and LTV ratio).
* **Returns:**
    * `id:` the unique market ID, calculated as keccak256(marketParams).
 

#### **20. Paused**
Checks whether the market is globally paused.
``` solidity
function paused() public view virtual returns (bool)
```
* **Parameter description:** N/A.
* **Returns:** true if the protocol is globally paused and all markets are disabled; false otherwise.


#### **21. Borrow Rate Full View**
Retrieves both the average borrow rate of a given market since the last update and the current borrow rate at the target utilization level.
``` solidity
function borrowRateFullView(Id id) public view returns (uint256 avgBorrowRate, int256 targetBorrowRate)
```
* **Parameter description:**
    * `id:` the unique market ID.
* **Returns:**
    * `avgBorrowRate:` the average borrow rate since the last interest update.
    * `targetBorrowRate:` the current borrow rate at the target utilization level.
 


