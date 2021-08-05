# 0x00 Java反序列化基础

Java反序列化触发的方法是`getObject`方法，因为Java开发者（包括内置库的开发者）经常会在这里写自己的逻辑，导致存在构造gadget chains的风险。

**第一节** 0x01 反序列化方法对比，将举例阐述相关概念及其与PHP、Python反序列化的相同与不同。

在Java安全-反序列化漏洞的学习过程中，很多文章都是从CommonsCollections这条利用链开始学习，但是CommonsCollections链存在很多比较难理解的概念。

因此，第一次学习推荐大家去看ysoserial的URLDNS链，并深入源码，动态调试，跟进学习。

ysoserial调试方法：https://wx.zsxq.com/dweb2/index/topic_detail/244415545824541

# 0x01 Java安全

