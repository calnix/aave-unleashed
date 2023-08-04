# 🚧 more on flags

**validateSupply**

* ACTIVE&#x20;
* NOT FROZEN & NOT PAUSED

**validateWithdraw**

* ACTIVE
* NOT PAUSED
* no check for frozen

**validateBorrow**

* ACTIVE&#x20;
* NOT PAUSED
* NOT FROZEN
* Borrowing enabled

**validateRepay**

* ACTIVE&#x20;
* NOT PAUSED

**liquidationCall**

* ACTIVE
* NOT PAUSED

{% hint style="info" %}
Why must bitmap invert then AND. why not just create the bitmap with 0000, instead of F.
{% endhint %}
