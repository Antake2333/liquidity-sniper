# AX-50 Liquidity Sniper

This bot require you to run the GETH client + use ethers framework. All addresses and private keys contained have been changed for the sake of this public repo.

This is heavily based on https://github.com/Supercycled/cake_sniper, so major thanks to him.

Use at you own risk.

## Global Workflow

AX-50 is a frontrunning bot primarily aimed at liquidity sniping on AMM like PancakeSwap. Liquidity sniping is the most profitable way I found to use it. But you can add pretty much any features involving frontrunning (liquidation, sandwich attacks etc..).

Being able to frontrun implies building on top of GETH client and having access to the mempool. My BSC node was running on AWS, hence the ax-50/config folder that I needed to send back and forth to the server with sniping config.

The bot is divided in 2 sections:
1. Configurations: An initial phase (previous to the snipe) where we configure the environment:
  * A trigger contract which is setted up with the configuration of the token you want to snipe (token address, route of swap, amount, wallet address that receives, etc). 
  * A swarm of accounts/wallets that will clogg the mempool once the liquidity is added, executing the snipe. This swarm of accounts is useful because we will be racing against other bots trying to frontrun the liquidity addition. So the more accounts trying the better the odds. Ideally one of all the accounts will succesfully snipe while the others will fail/revert (without doing nothing, except wasting gas).
2. Sniping: An observation phase where we listen to txs in the mempool waiting for the liquidity addition to appear. Once we spot it we clogg the mempool with our own txs executing the trigger that will perform the snipe (one tx per account in the swarm). All txs have the same gas as the liquidity addition tx, so the mempool sets them (ideally) at the same priority as the liq. addition, hence frontrunning others.

## Setup

1. Make sure to first deploy all contracts using the truffle migrations (create a `.env` file with `BINANCE_MAINNET_WALLET_PRIVATE_KEY` or `BINANCE_MAINNET_WALLET_MNEMONIC`). Running them should configure:
- The trigger custom router address with your CustomPCSRouter
- The trigger admin with the deployer wallet (this is important)

2. Create a `config/local.json` file following the template provided inside the directory (`config/template.local.json). This will be used by our scripts in the configuration phase.

3. If you don't have a swarm yet, create one running `npm run create-swarm`. This should create a `config/bee_book.json` similar to the template one (`config/template.bee_book.json`) to be used by the sniper on each snipe

4. \[Optional\] Preview the order you will create and snipe with `npm run order-preview`, to avoid undesired results.

5. Configure the trigger contract with the provided order running `npm run configure-trigger`

6. \[Optional\] If you want to recover the spread bnb in the swarm or rollback the trigger configuration (recovering the bnb supplied), run `npm run swarm-refund` / `npm run withdraw-trigger`. Else leave it there for future snipes.

7. Configure the `config/configurations.json` file, which will be used at the 'sniping' phase. It's important to have a read and write node where you will stream de mempool txs (read) and write your own ones once the addLiquidity one is found. So bear in mind you will need to either set up your own node or get one from somewhere.

In future snipes, you can avoid most of the steps and just run step 2 & 5 & 7, simply configuring the trigger for a new snipe.

## Usage

If you have already configured the trigger contract, simply leave the client running with `go run ./...`. Once the liquidity is added it should snipe it transparently.

And that's it! the bot should be working without hassles! The bot is currently defined to work with BSC and PancakeSwap. But you can adapt is to whatever EVM blockchain with its equivalent copy of Uniswap V2. To do this, just change the variables in the config directory + contracts.

## Donations

If you found the bot useful and you want to share some of those juicy profits with me, I accept donations through BEP20 (BSC) at `0x8f5d3374373aDA8b2c201C5cAc4c384FD42d2390` in any type of token (hopefully one with liquidity hehe)

Cheers and happy sniping!