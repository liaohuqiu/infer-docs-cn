---
title: 检测一些开源 Android 应用
layout: post
author: dulmarod
category: blog
---

我们用 Infer 检测一些开源的 Android 应用，试着找出并修复一些问题。到现在为止，确实修复了一些问题。

其中一个应用就是搜索引擎 [DuckDuckGo](https://github.com/duckduckgo/android)。 我们发现有很多数据库的 cursor 没有及时关闭。我们报告了这个问题之后，一个开发者很快就[修复了这个问题](https://github.com/duckduckgo/android/commit/2c2d79f990dde0e44cdbecb1925b73c63bf9141d).

我们也分析了很流行的邮件客户端 [k-9](https://github.com/k9mail/k-9)。我们发现有一个文件没关闭导致泄露，我们也提交了一个 issue。有意思的是，一个开发者[修复了这个问题](https://github.com/k9mail/k-9/commit/d538278be62687758c956af62ee47c53637d67d8)，修复方式是不往这个文件写任何日志信息。看吧，Infer 帮助他们简化了他们的代码。

[Conversations](https://github.com/siacs/Conversations) is an open source XMPP/Jabber client for Android smart phones. We analyzed it as well and found a file not closed leak, which was also [fixed](https://github.com/Flowdalic/MemorizingTrustManager/commit/190c57a9a8385f4726c817924b123438af6adc2f).

[Conversations](https://github.com/siacs/Conversations) 是一个开源的 XMPP/Jabber Android 应用。我们分析这个项目时，也发现了一个文件泄露。[当然，现在修复了](https://github.com/Flowdalic/MemorizingTrustManager/commit/190c57a9a8385f4726c817924b123438af6adc2f)。
