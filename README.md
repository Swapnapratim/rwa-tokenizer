# The Methodology

We can tokenize real world assets by combining any of the following traits:
- Asset location: 
  - On or Off Chain Asset Represented 
  - Nomenclature: [`AOn`, `AOff`] 
    - Note: By using an on-chain asset, it could be considered no longer "real world".
- Collateral location: 
  - On or Off-Chain Collateral 
  - Nomenclature: [`COn`, `COff`] 
- Backing type:
  - Direct backing or Indirect (synthetic)
  - Nomenclature: [`DB`, `SB`]

So since we have 3 categories each with 2 options, we have 8 different types of RWAs.

<details>
<summary>Examples of the 8 assets</summary>

- Onchain asset, onchain collateral, direct backing 
  - `AOnCOnDB`
  - Examples: WETH
  - Not demo'd in this repo
- Onchain asset, onchain collateral, indirect backing (synthetic)
  - `AOnCOnSB`
  - Examples: WBTC 
  - Not demo'd in this repo
- Onchain asset, offchain collateral, direct backing 
  - `AOnCOffDB`
  - Examples: N/A
  - Maybe like a wrapped BTC ETF?
  - Not demo'd in this repo
- Onchain asset, offchain collateral, indirect backing (synthetic)
  - `AOnCOffSB`
  - Examples: N/A
  - Maybe like a wrapped BTC ETF that represents an ETH ETF?
  - Not demo'd in this repo
- Offchain asset, onchain collateral, direct backing 
  - `AOffCOnDB`
  - Examples: N/A 
  - Like a stablecoin backed by other stablecoins (sort of DAI lmao)
  - Not demo'd in this repo
- Offchain asset, onchain collateral, indirect backing (synthetic)
  - `AOffCOnSB`
  - Examples: DAI
  - In this repo: sTSLA w/ chainlink price feeds
- Offchain asset, offchain collateral, direct backing 
  - `AOffCOffDB`
  - Examples: USDC
  - In this repo: dTSLA w/ chainlink functions
- Offchain asset, offchain collateral, indirect backing (synthetic)
  - `AOffCOffSB`
  - Examples: USDT
  - In this repo: sTSLA w/ chainlink functions
  
### Examples don't make sense
- Directly Backed On-Chain Asset Representation with Off-Chain Collateral
  - We would represent ETH on Chain by collateralizing it with some off-chain version of ETH? Like an ETH ETF? Weird...
- Synthetic On-Chain Asset Representation with Off-Chain Collateral
  - The same issue as above, but even weirder since we'd back our on-chain asset with something like MSFT shares
- Directly Backed Off-Chain Asset Representation with On-Chain Collateral 
  - How could you directly back an off-chain asset with on-chain collateral? By doing that you essentially make it "synthetic" automatically

### Examples that would make sense 
- Synthetic Off-Chain Asset Representation with Off-Chain Collateral
  - This would be like a synthetic index fund share, but backed by shares of different stocks.
</details>


## dTSLA.sol

### V1
1. Only the owner can mint `dTSLA`
2. Anyone can redeem `dTSLA` for `USDC` or "the stablecoin" of choice.
  - Chainlink functions will kick off a `TSLA` sell for USDC, and then send it to the contract
3. The user will have to then call `finishRedeem` to get their `USDC`.

### V2 
1. Users can send USDC -> `dTSLA.sol` via `sendMintRequest` via Chainlink Functions. This will kick off the following:
  - USDC will be sent to Alpaca
  - USDC will be sold for USD 
  - USD will be used to buy TSLA shares
  - The Chainlink Functions will then callback to `_mintFulFillRequest`, to enable `dTSLA` tokens to the user.
2. The user can then call `finishMint` to withdraw their minted `dTSLA` tokens. 

## sTSLA.sol

This is essentially a synthetic TSLA token, it's follows a similar architecture to a stablecoin, where we use a Chainlink price feed to govern the price of the token.

You can learn more about how to build these by following the [Cyfrin Updraft](https://updraft.cyfrin.io/) curriculum. 

## BridgedWETH.sol

So, token transfers are baked into the CCIP protocol, but you have to work with the DONs to get these setup. We will show you how to create your own token pools with CCIP and not bother working with the DONs, since CCIP allows you to send arbitrary data to the other chain.

1. WETH contract on "home" chain 
2. BridgedWETH contract on all other chains 
3. Chainlink CCIP Sender & Receiver Contract 
  - Lock the WETH 
  - Emit the message to unlock on the other chain 
  - Mint the WETH 
  - Will need to burn the WETH and send it back 

## Installation

1. Clone the repo, navigate to the directory, and install dependencies with `make`
```
git clone https://github.com/Swapnapratim/rwa-tokenizer.git
cd rwa-tokenizer
make
```

# Details on Examples 
## Fully-reserved On-Chain Asset Representation with On-Chain Collateral: frCross-Chain WETH 
   1. Collateral: On-Chain ETH 
      1. Since the collateral is the same, this is a fully-reserved/backed asset! 
   2. Stability Mechanism: Algorithmic 
   3. Using: Chainlink CCIP 
      1. Would be improved by a Chainlink Proof of Reserve (PoR) 

Technically, even the "normal" [WETH token](https://etherscan.io/token/0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2) could be considered a fully-reserved on-chain collateralized asset! 

## Synthetic On-Chain Asset Representation with On-Chain Collateral: sCross-Chain WETH 
   1. Collateral: Random ERC20s 
      1. Since the collateral is different, this is a synthetic asset! 
   2. Stability Mechanism: Algorithmic 
   3. Using: Chainlink Price Feeds, Chainlink CCIP 
      1. Would be improved by a Chainlink Proof of Reserve (PoR) 
   
## Synthetic Off-Chain Asset Representation with On-Chain Collateral: sMSFT 
   1. Collateral: On-Chain ETH 
      1. Since the collateral is different, this is a synthetic asset! 
   2. Stability Mechanism: Algorithmic 
      1. We can have an algorithmic stability mechanism because we are using on-chain collateral 
   3. Using: Chainlink Price Feeds 

## Fully-reserved Off-Chain Asset Representation with Off-Chain Collateral: frMSFT 
   1. Collateral: Off-Chain MSFT shares 
      1. Makes it fully-reserved! 
   2. Stability Mechanism: Governed / Centralized 
      1. It's borderline impossible to make this algorithmic at the moment 
      2. There is too much trust needed in custodian holding the MSFT shares! 
   3. Using: Chainlink Functions 
      1. Would be improved by a Chainlink Proof of Reserve (PoR) 

  

# Currently Live Examples of Tokenized RWAs
Some good examples of these are:
1. Fully-reserved On-Chain Asset Representation with On-Chain Collateral: [Wrapped BTC](https://www.bitgo.com/newsroom/press-releases/bitgo-adopts-chainlink-enable-on-chain-auditing-for-wbtc/)
   1. This uses a Chainlink PoR (proof of reserve) network to track the amount of BTC in a wallet on the BTC network. 
   2. Only if that wallet has enough BTC (checked by CL) can they then mint WBTC on the ETH network
2. Synthetic On-Chain Asset Representation with On-Chain Collateral: [RAI](https://reflexer.finance/)
   1. RAI token is a "synthetic dollar" that is backed by ETH.
   2. Most stablecoins fall into either being backed or synthetic dollars. 
3. Fully-reserved Off-Chain Asset Representation with Off-Chain Collateral: [USDC](https://www.circle.com/en/usdc)
   1. Each USDC is backed 1:1 by a dollar equivalent in a bank account.
   2. It could be improved by a Chainlink PoR, pushing it away from the centralization that comes with it being governed.
4. Synthetic Off-Chain Asset Representation with Off-Chain Collateral: [USDT](https://tether.to/)
   1. Each USDT is backed by a basket of assets that equal the value of a dollar.
   2. This would be improved by a Chainlink PoR
