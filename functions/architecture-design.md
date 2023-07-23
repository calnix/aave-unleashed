# Architecture Design

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

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
