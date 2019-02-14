# 牧羊人模块

[HerderSCPDriver](HerderSCPDriver.h)是[SCP
protocol](../scp)的具体实现,  以"transaction sets"和"ledger
numbers"方式运作构成OSCH词汇，它是[SCPDriver class](../scp/SCPDriver.h)的一个子类, 因此阅读这个类，并理解每一个子类作用和所在位置，将理解SCP协议的具体化实现。 

# 关键实现细节  

牧羊人模块指定一个账本号（ledger number）为一个插槽（slot）在SCP协议中，交易集哈希（以及关闭时间和基本费用）是它试图就每个'slot'达成的值。

Header充当SCP和LedgerManager之间粘合剂

## 与LedgerManager交互
Herder尽可能地序列化“slot externalize”事件，以便LedgerManager只能看到严格单调的分类帐关闭事件（并使用追赶来处理任何潜在的间隙）。

## 与SCP交互
Herder有两个主要的操作状态

### "Tracking"状态
Herder知道哪个一个插槽是最后序列化的，只处理下一个插槽的SCP消息。
当接受到未来的SCP消息存储起来供以后使用：接收未来消息不一定表示存在问题。
当其他对等设备继续运行时，网络可能已延迟当前插槽的消息。

#### 超时（Timeout）
Herder设置超时以在预期的下一个插槽上取得进展，如果达到此超时，则将其状态更改为“Not Tracking”。

#### 挑选一个初始位置
当一个ledger被关闭，LedgerManager是处在同步过程，herder负责挑选一个起始位置发送PREPARING消息。

### "Not Tracking"状态
Herder不知道哪个slot是最后外部化的，这是header目标是返回tracking状态。为了做到这一点，它开始处理从最小的插槽号开始的所有SCP消息，最大化其中一个插槽实际外部化的机会。

请注意，转移到此状态并不一定意味着LedgerManager将不同步：它可能只是需要一个异常时间（某种网络中断，分区等）以使节点达成共识。
