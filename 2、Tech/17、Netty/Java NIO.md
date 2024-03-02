Channel
数据的双向传输通道

Buffer
缓冲区

Selector
选择器，需要将 channel 注册到 selector 上，selector 维护了所有注册的 channel，并进行轮询，监听 channel
上发生的事件。
