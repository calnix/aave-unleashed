# AToken

* [https://solidity-by-example.org/inheritance/](https://solidity-by-example.org/inheritance/)
* [https://medium.com/coinmonks/how-aave-hacked-the-erc-20-token-in-the-most-beautiful-way-3e04fb4410fb](https://medium.com/coinmonks/how-aave-hacked-the-erc-20-token-in-the-most-beautiful-way-3e04fb4410fb)
* [https://mirror.xyz/freesuton.eth/EagGvVQZtLhsnnsTTwhHA6VwekMbImMmvniSWAm6SNg](https://mirror.xyz/freesuton.eth/EagGvVQZtLhsnnsTTwhHA6VwekMbImMmvniSWAm6SNg)



Firstly, it is important to note the inheritance chain of the AToken contract:

```solidity
contract AToken is VersionedInitializable, ScaledBalanceTokenBase, EIP712Base, IAToken
```

