* ### 核心部分：

  * ### Channel: 该对象模拟了通道连接，管道可以是单向的，也可以是双向的。
  * ### Buffer: 是Java类与Channels之间的纽带。Channels可以提取存放在Buffer中的数据（写），也可以存入数据供读取（取）
  * ### Selector: 提供了确定一个或多个通道当前状态的机制。使用选择器，借助单一线程，可对数量庞大的活动IO通道实施监控和维护。

* ### Channel和Buffer: Channel可对Buffer进行读写

![](/assets/overview-channels-buffers1.png)

* ### Channel和Selector: 单线程处理多个Channel.

![](/assets/overview-selectors.png)



