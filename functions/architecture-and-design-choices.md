# Architecture & Design choices

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

While this screenshot is of v2, (lendingPool was renamed Pool in v3), it is useful to illustrate the architecture of Aave on the whole.

The Pool contract contains all the user-facing functions, and serves as the single access point into Aave. Regardless of which asset the user might want to interact with, or what they wish to do, it will always be through the Pool contract.

### Design choices

**Advantages**

1. Single Access Point

* Makes integration easy as integrators simply need to point to the Pool contract.
* Single consistent entry-point makes for better UX and security.

2. Upgrading/Implementing new features

Easy to add new functionalities and have them on all the assets at once. Simply upgrade the Pool contract, and they are available on all the assets.&#x20;

V3 added functionalities:

* Permit for supply, repay &#x20;
* EIP712 signatures for credit delegation

{% hint style="info" %}
However, on compound you must upgrade all the different cToken contracts, to introduce a new feature/function across the board.
{% endhint %}

**Disadvantages**

If a custom behaviour is needed for a particular asset, implementing can be tricky. For example, a specific action that occurs on supply, withdraw, etc.

* This has not become an issue to date.&#x20;
* However, if it becomes an issue, a specific function can be added to the pool contract just for that asset.

### Pool contract

There is no execution code on the Pool contract.&#x20;

In v2, the lendingPool contract was rather big, almost exceeding the limit of 24 kb. Solution was to split the code into their logical groups; code was modularized via libraries.

* E.g. SupplyLogic.sol library holds all the relevant logic pertaining to the supply action
* supply function on Pool.sol calls executeSupply defined on SupplyLogic.sol library
* Pool retains the necessary storage/state data, while "importing" execution logic from various libraries.

Libraries were deployed independently as linked libraries - containing external functions. This helped to downsize contract code.

{% hint style="info" %}
One simple way to reduce contract size is using a [library](https://solidity.readthedocs.io/en/v0.6.10/contracts.html#libraries). Don't declare the library functions as internal as those will be [added to the contract](https://ethereum.stackexchange.com/questions/12975/are-internal-functions-in-libraries-not-covered-by-linking) directly during compilation.&#x20;

But if you use public functions, then those will be in fact in a separate library contract. Consider [using for](https://solidity.readthedocs.io/en/v0.6.10/contracts.html#using-for) to make the use of libraries more convenient.
{% endhint %}

{% hint style="success" %}
While these libraries are independently deployed and their external functions can be accessed by anyone, they have to be called by the Pool contract to enact a change with Aave protocol.
{% endhint %}
