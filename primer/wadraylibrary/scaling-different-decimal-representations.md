# 🚧 WIP: Scaling different decimal representations

###

* TokenA is set up with 18 decimals (WAD)
* TokenB is set up with 6 decimals

If you want to convert between different representations you have to multiply or divide depending on the difference in decimals count:

```markdown
**To convert valueA to valueB: (18 dp -> 6 dp)
    valueB = valueA / 10**(A.decimals - B.decimals)
    valueB = valueA / (10**(18-6))
```

#### Convert from WAD to 6 dp:

B is a WAD

<figure><img src="../../.gitbook/assets/image (128).png" alt=""><figcaption></figcaption></figure>

* Normalize B by dividing by its decimal places, 10\*\*18
* Then scale up by the desired decimals, 10\*\*6
* **However, when applying in solidity, we have to reverse the order of operations to ensure division is done last.**&#x20;
