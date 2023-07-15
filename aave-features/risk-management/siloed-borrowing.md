---
description: https://docs.aave.com/developers/whats-new/siloed-borrowing
---

# Siloed Borrowing

Siloed borrowing allows assets with potentially manipulatable oracles (for example illiquid Uni V3 pairs) to be listed on Aave as single borrow asset.

This means that if an asset is configured as _siloed_, it can't be borrowed in a position at the same time as with other assets.

This helps mitigating the risk associated with such assets from impacting the overall solvency of the protocol. Please see the&#x20;
