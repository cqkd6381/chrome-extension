# chrome-extension
Chrome extension development experience

# 消息传递
由于内容脚本在网页而不是扩展程序的环境中运行，它们通常需要某种方式与扩展程序的其余部分通信。例如，RSS 阅读器扩展程序可能使用内容脚本检测页面上是否存在 RSS 供稿，然后通知后台页面，为当前页面显示页面按钮图标。

扩展程序和内容脚本间的通信使用消息传递的方式。两边均可以监听另一边发来的消息，并通过同样的通道回应。消息可以包含任何有效的 JSON 对象（null、boolean、number、string、array 或 object）。对于一次性的请求有一个简单的 API，同时也有更复杂的 API，允许您通过长时间的连接与共享的上下文交换多个消息。另外您也可以向另一个扩展程序发送消息，只要您知道它的标识符，这将在跨扩展程序消息传递部分介绍。
# 简单的一次性请求
如果您只需要向您的扩展程序的另一部分发送一个简单消息（以及可选地获得回应），您应该使用比较简单的 runtime.sendMessage 方法。这些方法允许您从内容脚本向扩展程序发送可通过 JSON 序列化的消息，可选的 callback 参数允许您在需要的时候从另一边处理回应。

如下列代码所示从内容脚本中发送请求：

