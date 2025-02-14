The `Factory` contract allows for permissionless deployment of Curve pools and gauges. It also acts as a registry for
finding deployed curve pools and gauges. It is deployed to the mainnet at the following address:

Source code for factory contracts may be viewed on [Github](https://github.com/curvefi/curve-factory).

!!!tip
    Pools can be deployed via the [Curve UI](https://curve.fi/#/ethereum/create-pool). More information at [resources](https://resources.curve.fi/factory-pools/creating-a-factory-pool/).
    In case of uncertainty regarding the pool parameters or others, please do not hesitate to contact curve members.


| Mainnet Factory | Contract Address |
| ----------- | ------- |
| crvUSD Pool Factory | [0x4F8846Ae9380B90d2E71D5e3D042dff3E7ebb40d](https://etherscan.io/address/0x4F8846Ae9380B90d2E71D5e3D042dff3E7ebb40d) | 
| MetaPool Factory (meta pools) | [0xB9fC157394Af804a3578134A6585C0dc9cc990d4](https://etherscan.io/address/0xB9fC157394Af804a3578134A6585C0dc9cc990d4) |  
| CryptoSwap Factory (two-coin volatile asset pools) | [0xF18056Bbd320E96A48e3Fbf8bC061322531aac99](https://etherscan.io/address/0xF18056Bbd320E96A48e3Fbf8bC061322531aac99) | 
| Tricrypto Factory (three-coin volatile asset pools) | [0x0c0e5f2fF0ff18a3be9b835635039256dC4B4963](https://etherscan.io/address/0x0c0e5f2fF0ff18a3be9b835635039256dC4B4963) | 


# **Organization**

The factory has several core components:

- The **factory** is the main contract to deploy new pools and gauges. It also acts as a registry for finding the 
  deployed pools and querying information about them.
- **Pools** are deployed via a **proxy contract**. The implementation contract targetted by the proxy is determined 
  according to the base pool. This is the same technique used to create pools in Uniswap V1.
- **Deposit contracts** (“zaps”) are used for wrapping and unwrapping underlying assets when depositing into or 
  withdrawing from pools.


!!!info
    Source code for this contract may be viewed on 
    [Github](https://github.com/curvefi/curve-factory/blob/master/contracts/Factory.vy).


!!! warning "Limitations"

    Please carefully review the limitations of the factory before deploying a new pool. Deploying a pool using an 
    incompatible token could result in **permanent losses to liquidity providers and/or traders**. Factory pools cannot be 
    killed and tokens cannot be rescued from them!
    
    - The token within the new pool must expose a decimal method and use a maximum of 18 decimal places.
    - The token’s `transfer` and `transferFrom` methods must revert upon failure.
    - Successful token transfers must move the specified number of tokens between the sender and receiver. 
      Tokens that take a fee upon a successful transfer may cause the pool to break or act unexpectedly.
    - Pools deployed by the factory cannot be paused or killed.


# **Pools**

## **Base Pool**

A metapool pairs a coin against the LP token of another pool. This other pool is referred to as the “base pool”. 
By using LP tokens, metapools allow swaps against any asset within their base pool, without diluting the base pool’s 
liquidity.
Existing base pools can be obtained by querying `base_pool_list` within the [MetaRegistry API](../registry/overview.md) or the MetaPoolFactory Contract itself.

```shell
>>> MetaRegistry.base_pool_list(0):
'0xbEbc44782C7dB0a1A60Cb6fe97d0b483032FF1C7'
```

| Current Base Pools | Coins   | Contract Address |
| ----------- | -------| ----|
| `3pool` |  `USDT <> USDC <> DAI` | [0xbEbc44782C7dB0a1A60Cb6fe97d0b483032FF1C7](https://etherscan.io/address/0xbEbc44782C7dB0a1A60Cb6fe97d0b483032FF1C7) |
| `sBTC` |  `sBTC <> wBTC <> renBTC` | [0x7fC77b5c7614E1533320Ea6DDc2Eb61fa00A9714](https://etherscan.io/address/0x7fC77b5c7614E1533320Ea6DDc2Eb61fa00A9714) |
| `renBTC` |  `renBTC <> wBTC` | [0x93054188d876f558f4a66B2EF1d97d16eDf0895B](https://etherscan.io/address/0x93054188d876f558f4a66B2EF1d97d16eDf0895B) |
| `fraxBP` |  `FRAX <> USDC` |[0xDcEF968d416a41Cdac0ED8702fAC8128A64241A2](https://etherscan.io/address/0xDcEF968d416a41Cdac0ED8702fAC8128A64241A2) |
| `sBTC2` |  `sBTC <> wBTC` | [0xf253f83AcA21aAbD2A20553AE0BF7F65C755A07F](https://etherscan.io/address/0xf253f83AcA21aAbD2A20553AE0BF7F65C755A07F) |

It is possible to enable additional base pools through a DAO vote. See [`add_base_pool`](../factory/admin_controls.md#add_base_pool).


## **Meta Pool** 
A metapool pairs a coin against the LP token of a base pool. Deployment via [`deploy_metapool`](../factory/deployer_api.md#deploy_metapool).

## **Plain Pool**
A plain pool pairs a minimum of 2 and a maximum of 4 coins. These coins are not paired with another pool. However, a plain pool can only pair assets not included in any base pool. Deployment via [`deploy_plain_pool`](../factory/deployer_api.md#deploy_plain_pool).

## **Two-Coin Crypto Pool** 
A crypto pool with two volatile coins. Deplyment via [`deploy_pool`](../factory/deployer_api.md#deploy_pool).

## **Three-Coin Crypto Pool** 
A crypto pool with three volatile coins, also called "Tricrypto" pool. Deplyment via [`deploy_pool`](../factory/deployer_api.md#deploy_pool-1).



# **Recommended Parameters**

!!!warning
    Please understand that these are just recommendations based on the performance data of previously deployed pools.
    Please look at other parts of the documentation to further understand the parameters.

## **StableSwap Pools**

| Parameter | Recommendation |
| ----------------------------- | -------------- |
| `_A` for Uncollateralized algorithmic stablecoins  | 5 - 10   |
| `_A` for Non-redeemable, collateralized assets     | 100    | 
| `_A` for Redeemable assets                         | 200 - 400|
| `_fee`                                             | - |


## **Crypto Pools**

### *Two-Coin-Crypto Pools*

Recommended parameters are similar to those of the [cbETH/ETH Pool](https://etherscan.io/address/0x5fae7e604fc3e24fd43a72867cebac94c65b404a#readContract).


### *Three-Coin-Crypto-Pools (Tricrypto)*

Recommended parameters are similar to those of the [TriCRV Pool](https://etherscan.io/address/0x4ebdf703948ddcea3b11f675b4d1fba9d2414a14#readContract).

