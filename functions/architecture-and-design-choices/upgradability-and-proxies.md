# 🚧 Upgradability and Proxies

**Aave has different markets on same chain:**

* aave v3
* aave AMM&#x20;

For example, the Pool address for the main market is different from the Pool address for the AMM market.

**PoolAddressesProviderRegistry**

* to get PoolAddressesProvider for a specific market

**AddressProvider(PoolAddressesProvider)**

* getPool
* gives us the address for Pool
* Pool address can change on upgrade
