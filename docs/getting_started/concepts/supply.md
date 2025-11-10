Supplying tokens to the JustLend DAO allows users to earn interest on their digital assets while simultaneously using them as collateral for borrowing. When users supply tokens, these assets are deposited into the JustLend DAO liquidity pool, a system of smart contracts designed to facilitate secure, over-collateralized borrowing.

In return, users receive jTokens, TRC-20 tokens representing their supplied assets. jTokens can be redeemed at any time for the underlying assets. As interest accrues, the exchange rate of jTokens relative to the underlying assets increases over time, reflecting the earned interest. This ensures that users benefit from a seamless and dynamic interest compounding mechanism while maintaining liquidity through redeemable jTokens.

### **How It Works**

**Interest Accrual:** Supplied tokens automatically earn interest based on the current market supply rate. Interest is dynamically accrued as the balance of supplied tokens grows, ensuring that users benefit from up-to-date rates.

**Determining Interest Rates:** Interest rates for suppliers are primarily influenced by the borrow utilization rate, which reflects the proportion of assets borrowed relative to the total pool supply. Governance parameters, such as the collateral factor and interest rates, can also be adjusted through community decisions.

**Dynamic Rate Updates:** As tokens are supplied, borrowed, repaid, or withdrawn from the liquidity pool, interest rates are updated in real time. These adjustments are guided by on-chain data, including token balances, oracle-determined prices, and the borrow utilization ratio.

### **How Do I Supply Assets**
Supplying can be done with a user interface [JustLend SBM](https://app.justlend.org/homeNew?lang=en-US) or SBM V2. Before we walk through the steps of a supplying sequence, let’s cover some key parameters:

SBM:
* `Supply APY:` the annual rewards from the jTokens users receive by supplying assets, which influenced by the borrow utilization rate and fluctuated by the time;
* `Total Supply:` the total supply in the market. As the total supply changes, the Supply APY will also change accordingly;
* `Suppliers:` the amount of users participating in the supply market.

SBM V2:
* `Supply APY:` the annualized return earned from supplying assets to the Vault. It is influenced by the market’s borrow utilization rate and may fluctuate over time;
* `Total Supply:` the total amount of assets supplied in the market. As the total supply changes, the Supply APY will adjust accordingly;
* `Collateral Support:` indicates whether the supplied asset can be used as collateral to borrow other assets on the JustLend DAO platform.


#### Supply Assets
1. Connect your Web3 wallet on TronLink or other supported wallet app to the JustLend DAO ([https://justlend.org](https://justlend.org)).
2. Navigate to “SBM” or “SBM V2”, and choose the asset you wish to supply.
* To supply TRX on SBM, click 「Supply」 in the TRX market.
* To supply TRX on SBM V2, select the TRX market under Supply Vaults, then click 「Details」.
3. Enter the amount you want to supply and click 「Supply」.
* The assets will be transferred directly from your wallet to the JustLend DAO protocol, where they will immediately start earning interest. This interest will be automatically added to your Supply Balance.
