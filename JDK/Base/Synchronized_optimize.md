# synchronized优化手段：锁膨胀、锁消除、锁粗化和自适应自旋锁

原文：https://www.toutiao.com/i6994438061325287944/?group_id=6994438061325287944



​        synchronized 在 JDK 1.5 时性能是比较低的，然而在后续的版本中经过各种优化迭代，它的性能也得到了前所未有的提升。本文就来盘点一下 synchronized 的核心优化方案。

​        synchronized 核心优化方案主要包含以下 4 个：

