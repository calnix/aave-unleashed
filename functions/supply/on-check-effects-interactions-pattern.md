# On check-effects-interactions pattern

You might have noticed that supply breaks the check-effects-interactions pattern, by making an external `safeTransferFrom` call before the aToken collateral is issued to the user.

The motivation behind adhering to check-effects-interactions pattern is&#x20;

* it cannot be avoided to hand over control flow to an external entity.
* you want to guard your functions against re-entrancy attacks.

Adherence to this pattern would require that all the state-changing internal accounting occur first, then the external `safeTransferFrom` to occur at the end of supply.

**However, breaking it is safer**

* You don’t want to be giving collateral to user before the underlying capital is taken in
* On re-entry, instead of the protocol receiving capital, the user would have free collateral to make malicious moves.&#x20;
