Supplying tokens to the JustLend DAO allows users to earn interest on their digital assets while simultaneously using them as collateral for borrowing. When users supply tokens, these assets are deposited into the JustLend DAO liquidity pool, a system of smart contracts designed to facilitate secure, over-collateralized borrowing.

In return, users receive jTokens, TRC-20 tokens representing their supplied assets. jTokens can be redeemed at any time for the underlying assets. As interest accrues, the exchange rate of jTokens relative to the underlying assets increases over time, reflecting the earned interest. This ensures that users benefit from a seamless and dynamic interest compounding mechanism while maintaining liquidity through redeemable jTokens.

### **How It Works**

**Interest Accrual:** Supplied tokens automatically earn interest based on the current market supply rate. Interest is dynamically accrued as the balance of supplied tokens grows, ensuring that users benefit from up-to-date rates.

**Determining Interest Rates:** Interest rates for suppliers are primarily influenced by the borrow utilization rate, which reflects the proportion of assets borrowed relative to the total pool supply. Governance parameters, such as the collateral factor and interest rates, can also be adjusted through community decisions.

**Dynamic Rate Updates:** As tokens are supplied, borrowed, repaid, or withdrawn from the liquidity pool, interest rates are updated in real time. These adjustments are guided by on-chain data, including token balances, oracle-determined prices, and the borrow utilization ratio.

### **How Do I Supply Assets**
Supplying can be done with a user interface [SBM V1](https://app.justlend.org/homeNew?lang=en-US) or SBM V2. Before we walk through the steps of a supplying sequence, let’s cover some key parameters:

SBM:
* `Supply APY:` the annual rewards from the jTokens users receive by supplying assets, which influenced by the borrow utilization rate and fluctuated by the time;
* `Total Supply:` the total supply in the market. As the total supply changes, the Supply APY will also change accordingly;
* `Suppliers:` the amount of users participating in the supply market.

SBM V2:
* `Supply APY:` the annualized return earned from supplying assets to the Vault. It is influenced by the market’s borrow utilization rate and may fluctuate over time;
* `Total Supply:` the total amount of assets supplied in the market. As the total supply changes, the Supply APY will adjust accordingly;
* `Collateral Support:` refers to the types of assets supported across the underlying Markets connected to this Vault. While you only deposit a single asset into the Vault, your liquidity may be lent out to borrowers who pledge these supported collateral tokens.


#### Supply Assets
1. Connect your Web3 wallet on TronLink or other supported wallet app to the JustLend DAO ([https://justlend.org](https://justlend.org)).
2. To Supply Asset on SBM V1:
* Navigate to **“SBM V1”**, choose the asset you wish to supply, then click **「Supply」** in the corresponding  market.
* Enter the amount you want to supply and click **「Supply」**.

3. To supply Asset on SBM V2:
* Navigate to **“SBM V2”**, select the asset under All Supply Vaults, then click **「Details」**.
* Choose **Supply**, enter the supply amount, then click **「Supply」**.

After the contract is successfully approved, the assets will be transferred from your wallet to the JustLend DAO protocol. Once they are borrowed by other users, they will begin generating interest. This interest will be automatically added to your Supply Balance.
