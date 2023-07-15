# General Execution flow

### **User-facing functions**

Most of the core user-facing functions can be called from Pool.sol:

* supply, withdraw
* borrow, repay
* liquidate, etc

We will review these functions within this section and breakdown their logic and workflow to illustrate the inner workings of Aave.&#x20;

**Most functions share a similar structure in their execution flow**

1. cache storage variables
2. update the state, system-wide (interest accrued)
3. validate action to be taken against updated system
4. if validate, change state as per action
5. update rates, reflective of latest action taken

{% hint style="info" %}
**General execution flow**

* cache -> updateState -> validation -> changeState -> updateRates&#x20;

**Except flashloan**:&#x20;

* validation -> user payload -> cache -> updateState -> changeState -> updateRates
* to protect against reentrancy and rate manipulation within the user specified payload
{% endhint %}

### **Common functions**

cache, updateState and updateRates are common components of almost all state-changing functions within Aave. They will be covered within [Common functions](general-execution-flow.md#common-functions), along other repeated functions.

* cache, updateState, updateRates&#x20;
* getFlags, getDecimals
* getSupplyCap, getBorrowCap

