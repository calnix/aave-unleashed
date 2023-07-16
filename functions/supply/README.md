# supply

## Overview

<img src="../../.gitbook/assets/file.excalidraw (14) (1).svg" alt="" class="gitbook-drawing">

* `executeSupply` contains the core logics of the supply function

## executeSupply

<figure><img src="../../.gitbook/assets/image (68).png" alt=""><figcaption></figcaption></figure>

### Execution flow

* [ ] <mark style="color:orange;">cache</mark>
* [ ] <mark style="color:orange;">updateState</mark>
* [ ] validateSupply
* [ ] <mark style="color:orange;">updateInterestRates</mark>
* [ ] transfer & mint
* [ ] If isFirstSupply (setup `userConfig`, if first supply & asset can be used as collateral)

We will breakdown and examine the unique sections of logics within **supply**. Code delineated in orange are common functions and can be explored in that [section](../common-functions/).

## Visual Aid

{% embed url="https://link.excalidraw.com/readonly/4hKXoUS5ExJDQxut21rn?darkMode=true" %}
[https://link.excalidraw.com/readonly/4hKXoUS5ExJDQxut21rn?darkMode=true](https://link.excalidraw.com/readonly/4hKXoUS5ExJDQxut21rn?darkMode=true)
{% endembed %}
