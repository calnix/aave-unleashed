# L2

### Background

Moving execution off of the main chain allows for significantly decreased costs for execution and state storage. However, rollups still must post their data to the layer-1 chain to ensure data availability.

This can be seen on the "Advanced TxInfo" tab on any transaction in the ArbiScan block explorer. The transaction fee is broken down into the calldata costs to post to L1, the computation used on L2, and the L2 storage. And in almost all transactions, the L1 calldata will be the primary driver of fees.

For example, breakup of $4.19 Uniswap Trade on Arbitrum ([explorer](https://arbiscan.io/tx/0xeab92f1bfa00f2cfacf50056cdd74df1fd7f3266ee0f7cc076121cc9a45e2341#txninfo)):

* L1 Fixed Cost: $1.77
* L1 Calldata Cost: $2.30
* L2 Computation: $0.12

You can see that, the calldata cost is over 50%. Simply put: posting data to L1 is the primary bottleneck for fees on rollups.

{% hint style="info" %}
Rollups have cheap execution but expensive data-availability costs.[](https://tokeninsight.com/en/research/analysts-pick/aave-v3-is-live.-do-you-still-believe-in-defi)
{% endhint %}

## L2Pool

Due to calldata costs, Aave deploys a calldata optimized version of the Pool contract allowing users to pass compact calldata.

* [https://github.com/aave/aave-v3-core/blob/master/contracts/protocol/pool/L2Pool.sol](https://github.com/aave/aave-v3-core/blob/master/contracts/protocol/pool/L2Pool.sol)

**For example, supply:**

<figure><img src="../../.gitbook/assets/image (329).png" alt=""><figcaption><p>L2Pool.sol</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (330).png" alt=""><figcaption><p>CalldataLogic.sol</p></figcaption></figure>

The compact calldata is decoded into the usual parameters and passed into the usual supply function.

<figure><img src="../../.gitbook/assets/image (332).png" alt=""><figcaption></figcaption></figure>

## L2 encoder

* Helper contract to encode calldata, used to optimize calldata size in L2Pool for transaction cost reduction
* Intended to help generate calldata for users/front-ends.
* Contains only view functions.

**Example: encodeSupplyParams**

<figure><img src="../../.gitbook/assets/image (331).png" alt=""><figcaption><p>L2Encoder.sol</p></figcaption></figure>

`encodeSupplyParams` takes three inputs: `asset`, `amount`, and `referralCode`. It then converts these parameters into a compact representation using a single `bytes32` value.

1. `assetId`: The function fetches the `id` of the `asset` from the `DataTypes.ReserveData` struct.
2. `shortenedAmount`: The `amount` is converted to a 128-bit unsigned integer (uint128) to save space.
3. `res`: The function combines `assetId`, `shortenedAmount`, and `referralCode` into a single 256-bit unsigned integer (uint256) named `res`. It uses bitwise shifting (shl) to position the values correctly within the uint256 variable and then uses the add operation to combine them into one.
4. Finally, the function returns `res`, which is the compact representation of the supply parameters.

<figure><img src="../../.gitbook/assets/image (333).png" alt=""><figcaption></figcaption></figure>



## Links

* [https://www.scopelift.co/blog/calldata-optimizooooors](https://www.scopelift.co/blog/calldata-optimizooooors)
* [https://ethglobal.com/showcase/l2-optimizoooors-7sio4](https://ethglobal.com/showcase/l2-optimizoooors-7sio4)
* [https://www.cryptoeq.io/articles/layer-2-fees](https://www.cryptoeq.io/articles/layer-2-fees)
* [https://l2fees.info/blog/rollup-calldata-compression](https://l2fees.info/blog/rollup-calldata-compression)
