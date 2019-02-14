# 牧羊人模块

[HerderSCPDriver](HerderSCPDriver.h)是[SCP
protocol](../scp)的具体实现,  以"transaction sets"和"ledger
numbers"方式运作构成OSCH词汇，它是[SCPDriver class](../scp/SCPDriver.h)的一个子类, 因此阅读这个类，并理解每一个子类作用和所在位置，将理解SCP协议的具体化实现。 

# 关键实现细节  

牧羊人模块指定一个账本号（ledger number）为一个插槽（slot）在SCP协议中，交易集哈希（以及关闭时间和基本费用）是它试图就每个'slot'达成的值。

Header充当SCP和LedgerManager之间粘合剂

## 与LedgerManager交互

Herder serializes "slot externalize" events as much as possible so that
LedgerManager only sees strictly monotonic ledger closing events (and deal with
 any potential gaps using catchup).

## 与SCP交互
Herder有两个主要的操作状态

### "Tracking"状态
Herder知道哪个一个插槽是最后序列化的，只处理下一个插槽的SCP消息。
当接受到未来的SCP消息存储起来供以后使用：接收未来消息不一定表示存在问题。
当其他对等设备继续运行时，网络可能已延迟当前插槽的消息。

#### Timeout
Herder places a timeout to make progress on the expected next slot, if it
 reaches this timeout, it changes its state to "Not tracking".

#### Picking the initial position
When a ledger is closed and LedgerManager is in sync, herder is responsible
 for picking a starting position to send a PREPARING message.

### "Not Tracking"状态
Herder does not know which slot got externalized last, its goal is to go back
 to the tracking state.
In order to do this, it starts processing all SCP messages starting with the
 smallest slot number, maximizing the chances that one of the slots actually
 externalizes.

Note that moving to this state does not necessarily mean that the
 LedgerManager would move out of sync: it could just be that it takes an
 abnormal time (network outage of some sort, partitioning, etc) for nodes to
 reach consensus.
