# Why use Indexes



In a traditional banking setting, interest accruals and corresponding balances can be calculated continuously over any desired period: monthly, daily, hourly. Calculating continuous compounding up to even a per second resolution is trivial as computing power is not expensive.

The same approach cannot be applied on-chain, because computation has to be paid for in gas; and the more complex the computation, the greater the gas fees. Consequently, on-chain lending protocols, such as Aave, use an index to track interest in a gas-efficient manner.&#x20;

1. Gas Efficiency: Gas is a measure of computational effort required to execute transactions or smart contracts on the blockchain. On-chain lending protocols need to handle multiple calculations and updates for each user's lending position. By using an index, the protocol can reduce gas costs by performing calculations based on the index value rather than recalculating interest from scratch for each transaction.
2. Computational Cost: Traditional financial institutions have dedicated systems and infrastructure to handle interest calculations, maintain historical records, and handle complex financial instruments. On-chain lending protocols, operating within the constraints of decentralized networks, need to optimize computational costs. Using an index allows for efficient storage and retrieval of historical interest rates, reducing the computational burden compared to localized storing of interest rates for each transaction.

{% hint style="info" %}
Traditional financial lending, operating within centralized systems, can rely on centralized databases and infrastructure to handle interest calculations without the same constraints.
{% endhint %}
