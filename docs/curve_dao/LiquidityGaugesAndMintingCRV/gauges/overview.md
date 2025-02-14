Inflation is directed to users who provide liquidity within the protocol. This usage is measured via “Liquidity Gauge” contracts. Each pool has an individual liquidity gauge. The Gauge Controller maintains a list of gauges and their types, with the weights of each gauge and type. For implementation details see [here](../overview.md#liquidity-gauges).

!!!deploy "Contract Source & Deployment"
    There are several versions of liquidity gauge contracts in use. Source code for these contracts is available on [Github](https://github.com/curvefi/curve-dao-contracts/tree/master/contracts/gauges).

Easiest way to obtain the gauge address of a liquidity pool is by querying `get_gauge` on the [MetaRegistry](/docs/registry/MetaRegistryAPI.md).


## **Gauge Types**

Each liquidity gauge is assigned a type within the gauge controller. Grouping gauges by type allows the DAO to adjust the emissions according to type, making it possible to e.g. end all emissions for a single type.

!!!warning
    Types 3 and 6 have been deprecated.

| Gauge Type   | Description | 
| -------- | -------|
| `0`      |  `Ethereum (stableswap pools)`   | 
| `1`      |  `Fantom`| 
| `2`      |  `Polygon (Matic)` | 
| `4`      |  `xDai`|
| `5`      |  `Ethereum (crypto pools)`|
| `7`      |  `Arbitrum`|
| `8`      |  `Avalance`|
| `9`      |  `Harmony`|
| `10`      |  `Fundraising`|
| `11`      |  `Optimism`|