# Embedded vs Linked Libraries

### Embedded library

An embedded library is a library that contains only **internal** functions.

* If a smart contract is consuming a library which have only internal functions, then the EVM simply embeds the library into the contract (hardcoded into contract at compile time).
* Instead of using `delegatecall` to call a function, it simply uses `JUMP` statement (normal method call). There is no need to separately deploy library in this scenario.

### Linked Library

A linked library is a library that contains public or external functions.

* The library needs to be deployed as a separate contract, and a unique address will be generated for it upon deployment.
* This address needs to be linked with calling contract.
