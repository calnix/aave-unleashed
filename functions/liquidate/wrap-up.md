# Wrap-up

<figure><img src="../../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

* Transfer the debt asset from liquidator to protocol&#x20;
* handleRepayment



### TO-DO

Think about:

* liquidator takes collateral asset frm user
  * debt as collateral + liq. bonus as collateral
* Protocol takes debt asset from liquidator
  * to make supplier whole
* two legs of transfers - how can this be made 1?
  * take collateral frm user
  * transfer liq.bonus to liquidator + swap baseCollateral to debtAsset
  * including a swap is problematic -> e.g. going to Uni to swap&#x20;
  * might incur more fees and gas
* alternative to swap&#x20;
  * internal pool?
  * something w/ flashloan
