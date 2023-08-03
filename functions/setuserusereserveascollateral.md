# 🚧 setUserUseReserveAsCollateral

### Overview

This function allows suppliers to enable/disable a specific supplied asset as collateral.

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

### Execution flow

* [ ] <mark style="color:orange;">cache</mark>
* [ ] get user's AToken balance
* [ ] validate
* [ ] if user is already using asset as collateral, `return`
* [ ] if-else

We will breakdown and examine the unique sections of logics within setUserUseReserveAsCollateral. Code delineated in orange are common functions and can be explored in that [section](common-functions/).
