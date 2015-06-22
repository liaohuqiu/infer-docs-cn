---
layout: support
title: Infer | 帮助和支持
id: support
category: support
---

## 帮助和支持

欢迎通过以下渠道反馈你遇到的问题。当然对于PR，我们更加欢迎。

### GitHub issues

[GitHub issues](https://github.com/facebook/Infer/issues) 是一个提问，查找答案和提交新 issue 的好地方，欢迎你来。

### Twitter

关注 [@fbinfer](https://twitter.com/fbinfer)，获取关于 Infer 的最新的新闻。

### IRC

Freenode.net 上的 IRC 频道 [#infer](irc://chat.freenode.net/infer)。

## 常见问题集锦

### 运行 "infer -- \<build command\>" 失败

请确认：

- \<build command\> 编译命令本身运行没问题
- `infer` 能在 `$PATH` 变量中找到。(试试 `which infer`，看 `infer` 所在的路径)

### "ImportError: No module named xml.etree.ElementTree"

确认 Python 的 `xml` 包已经安装。比如在 OpenSuse 13.1, 在
[`python-xmldiff`](http://software.opensuse.org/download.html?project=XML&package=python-xmldiff)
这个包中。

### 其他问题

其他不在以上列表中的问题，请通过以上提到的方式联系我们。

## FAQ

常见问题解答。随时更新。

### Windows 上能用吗

抱歉，暂不支持，用 Windows 的话，可以试试建一个 Linux 虚拟机试试。
