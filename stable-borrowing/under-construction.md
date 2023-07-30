# 🚧 Under construction

## Incentive to rebalance

The rebalanceStableBorrowRate function is a public function that can be called by anyone, targetting some specified wallet address.

<figure><img src="../.gitbook/assets/image (239).png" alt=""><figcaption></figcaption></figure>

But anylike liquidations, there does not seem to be any incentive to do so.

<figure><img src="../.gitbook/assets/image (240).png" alt=""><figcaption></figcaption></figure>

* So who bothers calling rebalance? Aave themselves presumably
* If so, why make it a public function if you do not intend to create a market around it?



**From discord:**

if your position is not so big you may never get rebalanced even if the rebalancing conditions are reached.&#x20;

{% hint style="info" %}
Rebalancing needs to be manually triggered by anyone, and there is more incentive to rebalance bigger positions rather than smaller, as that would affect rate more
{% endhint %}
