# VariableDebtToken

Note that debt tokens cannot be transferred and therefore do not have their related functions.

## User State

<figure><img src="../.gitbook/assets/image (248).png" alt=""><figcaption><p>IncentivizedERC20<br></p></figcaption></figure>

* `balance`: user's scaled balance&#x20;
* `additionalData`: user's last index at which his state was updated

## mint

<img src="../.gitbook/assets/file.excalidraw (17).svg" alt="" class="gitbook-drawing">

* `amountScaled`: scale the borrow amount against variableBorrowIndex
* `scaledBalance`: user's scaled balance
* `balanceIncrease`: calculate the balance increase due to interest accrued from the previous time interest was calculated for the user till now.
  * `additionalData`: index at which user's scaledBalance was previously calculated

Update `additionalData` with the current variableBorrowIndex, to reflect current mint action.

#### \_mint

<figure><img src="../.gitbook/assets/image (249).png" alt=""><figcaption></figcaption></figure>

* increment `totalSupply` by `amountScaled`
* increment user's balance by `amountScaled`
