# 🚧 PriceOracleSentinel

## PriceOracleSentinel

Price Oracle sentinel mainly aimed for L2s to handle eventual downtime of the sequencers by introducing a grace period for liquidations and ability for RISK\_ADMINS to disable borrowing for a reserve under specific circumstances.

* [https://docs.aave.com/developers/core-contracts/priceoraclesentinel](https://docs.aave.com/developers/core-contracts/priceoraclesentinel)

L2 networks use block producers, called sequencers, alongside distributed validation to increase throughput and support two queues for pending transactions—one on-chain requiring L1 transactions and another off-chain operated by the sequencer. The sequencers often experience downtime, which is usually short in length and has little impact on L2 users. However, if the downtime lasts longer, the network fails to produce new blocks and off-chain transactions may be rejected or dropped.

The sentinel feature was designed specifically to handle any eventual downtime centralized block producers on L2s may experience. When the sequencers experience downtime, the Aave's price feeds aren’t updated. This creates the possibility of slow flash crashes when the sequencer comes up.&#x20;

To solve the problem of untimely update of Layer 2 prices caused by Layer 1 congestion in extreme cases, Aave proposed Price Oracle Sentinel.

To solve this, V3 introduces a grace period for liquidations and disables borrowing when it detects a sequencer is down.[](https://cryptoslate.com/aave-v3-is-here-and-it-wants-to-solve-everything-you-hate-in-defi/)

It means that when the L2 price update/transaction record is delayed, the user's liquidated position will have a grace period before the next block is produced. That is to say, the user's position will not be liquidated before the oracle price is updated, but the user will not be able to borrow any more during this period.
