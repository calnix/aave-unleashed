# 🚧 Portal

Portal allows allow supplied assets to be transferred between Aave markets on different chains.

Aave v3 allows _**approved bridges**_ to burn _aTokens_ on the source network while instantly minting them on the destination network. The underlying assets can then be supplied to Aave on the destination network, in a delayed manner, by passing it to the pool after it has been moved through a bridge.

1. Minting an "unbacked" aToken
2. "Unbacked" aToken is restored to normal aToken
3. Provide a whitelist mechanism for contracts that want to use these features.

This has implications for calculating interest rates and utilization on the destination chain.&#x20;

