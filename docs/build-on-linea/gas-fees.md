---
title: Gas fees on Linea
sidebar_position: 6
---

 If you're familiar with gas fees on Ethereum, then you know that they heavily fluctuate depending on how busy the network is (for a refersher on gas click [here](https://support.metamask.io/hc/en-us/articles/4404600179227-User-Guide-Gas#:~:text=A%20normal%20transaction%20sending%20ETH,transactions%20also%20cost%2021%2C000%20gas.)). Linea's gas fees are dependent on Ethereum, but the underlying calculations are somewhat complex. This article will explain how the L2 fees are calculated, but the **TLDR is that Linea's gas fees should be around 1/15th of Ethereum's, and we hope to reduce them even further in the future.**

## How do I check the gas price on Linea?

 If you want to check the gas price on Linea, run the code below.

``` bash
curl https://linea-mainnet.infura.io/v3/your-infura-api-key \
  -X POST \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_gasPrice","params": [],"id":1}'
```
:::note
The output of the request returns a hexadecimal equivalent of an integer representing the current gas price in wei. Convert the hexadecimal value into decimals to get the wei value. You can use any hexadecimal to decimal converter such as [RapidTables](https://www.rapidtables.com/convert/number/hex-to-decimal.html).

:::

For more information on this method, take a look at [eth_gasPrice JSON_RPC](https://docs.infura.io/networks/ethereum/json-rpc-methods/eth_gasprice).

You can also use the eth_gasprice method on Ethereum and then compare the two values to see if L2 gas price = ~L1/15 gas price.

``` bash
curl https://mainnet.infura.io/v3/your-infura-api-key \
  -X POST \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_gasPrice","params": [],"id":1}'
```

## The Formula

**Total L2 Fee Equation**

$$
\text{total\_L2\_fee} = \text{L2\_base\_fee} + \text{L2\_miner\_tip}
$$

The L2 fee equation is comprised of two parts, the base fee and the miner tip. The base fee is essentially 0 and can be ignored for the portion of the calculation. 

However, the miner tip equation will be explained in more detail below.


**L2 Miner Tip Equation**

$$
L2 \text{ miner tip} = \text{base\_fee\_coefficient} \times \text{last-L1-base-fee} + \text{priority\_fee\_coefficient} \times \text{WA}(GasUsedRatio \text{(L1-eth\_feeHistory(latest, batchSubmissionPercentile, numBlocks))})
$$

In this equation, the following variables are constants that may be subject to change in the future as we refine our gas fee calculation: ```base_fee_coefficient = 0.066```, ```prority_fee_coefficient = 0.066```, ```batchSubmissionPercentile = 15```, and ```numBlocks = 200```.

The first portion of the equation is simply the base_fee_coefficient multiplied by the last L1 base fee.

The second part of the equation multiplies the priority_fee_coefficient by the Weighted Average (WA) of the gas used ratio that the eth_feeHistory method returns.


We use the [eth_feeHistory JSON-RPC method](https://docs.infura.io/networks/ethereum/json-rpc-methods/eth_feehistory) with the constant parameters given above to figure out what the 'GasUsedRatio' is on Ethereum and then find the weighted average of the values.

The request would look like the code below.

```bash 
curl https://mainnet.infura.io/v3/YOUR-API-KEY \
  -X POST \
  -H "Content-Type: application/json" \
  -d '{"id": 1, "jsonrpc": "2.0", "method": "eth_feeHistory", "params": ["200", "latest", [15]] }'

```
:::note
Since we are including 200 blocks, calculating the weighted average would be quite tedious.

:::