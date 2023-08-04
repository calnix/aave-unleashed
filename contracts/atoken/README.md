# AToken

## Inheritance

Firstly, it is important to note the inheritance chain of the AToken contract:

```solidity
contract AToken is VersionedInitializable, ScaledBalanceTokenBase, EIP712Base, IAToken
```

AToken inherits VersionedInitializable, ScaledBalanceTokenBase, EIP712Base and IAToken.

### **VersionedInitializable**

* Abstract helper contract to implement initializer functions.
* Inspired by the OpenZeppelin Initializable contract.

<details>

<summary><strong>VersionedInitializable.sol</strong></summary>

{% code overflow="wrap" fullWidth="true" %}
```solidity
abstract contract VersionedInitializable {
  /**
   * @dev Indicates that the contract has been initialized.
   */
  uint256 private lastInitializedRevision = 0;

  /**
   * @dev Indicates that the contract is in the process of being initialized.
   */
  bool private initializing;

  /**
   * @dev Modifier to use in the initializer function of a contract.
   */
  modifier initializer() {
    uint256 revision = getRevision();
    require(
      initializing || isConstructor() || revision > lastInitializedRevision,
      'Contract instance has already been initialized'
    );

    bool isTopLevelCall = !initializing;
    if (isTopLevelCall) {
      initializing = true;
      lastInitializedRevision = revision;
    }

    _;

    if (isTopLevelCall) {
      initializing = false;
    }
  }

  /**
   * @notice Returns the revision number of the contract
   * @dev Needs to be defined in the inherited class as a constant.
   * @return The revision number
   */
  function getRevision() internal pure virtual returns (uint256);

  /**
   * @notice Returns true if and only if the function is running in the constructor
   * @return True if the function is running in the constructor
   */
  function isConstructor() private view returns (bool) {
    // extcodesize checks the size of the code stored in an address, and
    // address returns the current address. Since the code is still not
    // deployed when running a constructor, any checks on its code size will
    // yield zero, making it an effective way to detect if a contract is
    // under construction or not.
    uint256 cs;
    //solium-disable-next-line
    assembly {
      cs := extcodesize(address())
    }
    return cs == 0;
  }

  // Reserved storage space to allow for layout changes in the future.
  uint256[50] private ______gap;
}

```
{% endcode %}

</details>

This contract is used to assist with initialization. It is modified from the Initializable contract of OpenZeppelin, and the revision version number is introduced. When the version number becomes larger, it can be initialized again.

<figure><img src="../../.gitbook/assets/image (4) (1).png" alt=""><figcaption><p><strong>VersionedInitializable.sol</strong></p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption><p>AToken.sol</p></figcaption></figure>

The revision number is defined on AToken.sol, which is obtained via `getRevision`. However, when the modifier `initializer` executes, it calls `getRevision` as defined in AToken.sol, due to override.

{% hint style="info" %}
Both revision values and `getRevision` are defined on contracts inheriting VersionedInitializable; this keeps VersionedInitializable modular as it does not need to be altered for each inheritance.&#x20;
{% endhint %}

### IncentivizedERC20

ScaledBalanceTokenBase inherits MintableIncentivizedERC20, which in turn inherits IncentivizedERC20.&#x20;

IncentivizedERC20 is similar to the standard ERC-20, with two differences.&#x20;

1. **balanceOf is modified with struct `UserState`:**

{% code title="IncentivizedERC20" %}
```solidity
/**
   * @dev UserState - additionalData is a flexible field.
   * ATokens and VariableDebtTokens use this field store the index of the
   * user's last supply/withdrawal/borrow/repayment. StableDebtTokens use
   * this field to store the user's stable rate.
   */
  struct UserState {
    uint128 balance;
    uint128 additionalData;
  }
  
// Map of users address and their state data (userAddress => userStateData)
mapping(address => UserState) internal _userState;

function balanceOf(address account) public view virtual override returns (uint256) {
  return _userState[account].balance;
}
```
{% endcode %}

balance saved in UserState is the actual balance, and additionalData is the liquidity index.

2. **\_transfer will apply incentives via IAaveIncentivesController**

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

It is also interesting to note that transferFrom incorporates both approve and transfer.

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

### Mintable Incentivized ERC20 <a href="#2.2.2-mintableincentivizederc20" id="2.2.2-mintableincentivizederc20"></a>

Abstract contract MintableIncentivizedERC20 inherits IncentivizedERC20, and implements only two functions:

* \_mint&#x20;
* \_burn &#x20;

These two functions will update the balance in `_totalSupply` and `UserState`, and then call IAaveIncentivesController.

### ScaledBalanceTokenBase <a href="#2.2.3-scaledbalancetokenbase" id="2.2.3-scaledbalancetokenbase"></a>

Abstract contract ScaledBalanceTokenBase inherits MintableIncentivizedERC20 and implements three functions:

* \_mintScaled
* \_burnScaled
* \_transfer

### Visual Aid

<img src="../../.gitbook/assets/file.excalidraw.svg" alt="" class="gitbook-drawing">

### EIP712Base <a href="#2.3-eip712base" id="2.3-eip712base"></a>

* WIP

## balanceOf

Aave implements a modified version of the ERC-20 standard for its aTokens, such that the balance of aTokens increments without any transactions.&#x20;

Let's examine the implementation for `balanceOf`:

<figure><img src="../../.gitbook/assets/image (125).png" alt=""><figcaption><p><a href="https://github.com/aave/aave-v3-core/blob/29ff9b9f89af7cd8255231bc5faf26c3ce0fb7ce/contracts/protocol/tokenization/AToken.sol#L128">AToken::balanceOf</a></p></figcaption></figure>

`override(IncentivizedERC20, IERC20)` serves to override `balanceOf` functions declared within its inheritance tree, so that the parent functions do not get called.&#x20;

<details>

<summary>On multiple inheritance</summary>

![](<../../.gitbook/assets/image (2) (1) (1).png>)

In short, we need to specify the contracts if we are overriding from more than one contract. Otherwise, you just need to use the override keyword.

Read more: [https://solidity-by-example.org/inheritance/](https://solidity-by-example.org/inheritance/)

</details>

A user's aToken balance is the multiplication of two components:

1. `super.balanceOf`&#x20;
2. `POOL.getReserveNormalizedIncome()`

In a more digestible form, this basically translates to:

```solidity
super.balanceOf(user) * POOL.getReserveNormalizedIncome(_underlyingAsset)
        scaledBalance * currentLiquidityIndex
```

**Execution flow:**&#x20;

<figure><img src="../../.gitbook/assets/image (151).png" alt=""><figcaption><p>super.balanceOf(user)</p></figcaption></figure>

### `super.BalanceOf`&#x20;

This calls balanceOf as declared in IncentivizedERC20.sol.

* Each user has a struct, `UserState` associated with their address via the mapping `_userState`
* The `balance` element within the `UserState` stores user's scaled balance&#x20;
* `super.balanceOf` returns `_userState[account].balance`

{% hint style="info" %}
`super.BalanceOf` returns the scaled balance of a user.
{% endhint %}

### `getReserveNormalizedIncome`

<figure><img src="../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

`getReserveNormalizedIncome` returns the latest liquidity index, as `currentLiquidityIndex`.&#x20;

### Putting them together

```solidity
       super.balanceOf(user) * POOL.getReserveNormalizedIncome(_underlyingAsset)
 _userState[account].balance * currentLiquidityIndex
           userState.balance * currentLiquidityIndex
           "scaledBalance"   * "latest liquidity Index"
```

**Example:**

* user deposits 1000 DAI, when Index = 1.1
* scaledBalance = 909
* `_userState[account].balance` = 909

### Visual Aid

<img src="../../.gitbook/assets/file.excalidraw (1).svg" alt="" class="gitbook-drawing">

## mint

**Execution flow**

AToken::`mint` -> ScaledBalanceTokenBase::`_mintScaled` ->  MintableIncentivizedERC20::`_mint`

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption><p>AToken inherits ScaledBalanceTokenBase</p></figcaption></figure>

* `amountScaled`, the amount to mint is scaled against the liquidity index in `_mintScaled`.&#x20;
* `_mint` increments the user's balance by `amountScaled`.
* user's balance held in `_userState[account].balance`.

<figure><img src="../../.gitbook/assets/image (99).png" alt=""><figcaption></figcaption></figure>

### Execution flow: Visual Aid

Here is a more complete execution flow chart spanning the relevant portions across different contracts.

<img src="../../.gitbook/assets/file.excalidraw (20).svg" alt="" class="gitbook-drawing">
