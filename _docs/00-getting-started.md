---
id: getting-started
title: 开始使用
layout: docs
permalink: /docs/getting-started.html
section: Quick Start
section_order: 00
order: 01
---
## 依赖

Python >= 2.7.

## 下载

我们提供了一个预编译的二进制包，目前有 Mac 和 Linux 64 位版本。

- Mac OS X: [https://github.com/facebook/infer/releases/download/v0.1.0/infer-osx-v0.1.0.tar.xz](https://github.com/facebook/infer/releases/download/v0.1.0/infer-osx-v0.1.0.tar.xz)
- Linux: [https://github.com/facebook/infer/releases/download/v0.1.0/infer-linux64-v0.1.0.tar.xz](https://github.com/facebook/infer/releases/download/v0.1.0/infer-linux64-v0.1.0.tar.xz)


## 安装

在命令行，到下载目录，解压：

- Mac OS X

    ```bash
    tar xf infer-osx-v0.1.0.tar.xz
    ```

- Linux

    ```bash
    tar xf infer-linux64-v0.1.0.tar.xz
    ```

解压后会有一个 ```infer-osx-v0.1.0/``` 目录 (或者
```infer-linux64-v0.1.0/``` 目录)， Infer 的主执行目录是 ```infer-osx-v0.1.0/infer/infer/bin/```.

### 设置 PATH 变量

我们建议把 Infer 的执行目录加入到环境变量中，这样使用起来会简便一些。当然，你也可以用绝对路径。本文档后续默认执行路径已加入到环境变量中。

如果你使用 Bask， 你可以使用以下命令设置环境变量。

```bash
cd infer-*v0\.\1\.0 &&
echo "export PATH=\"\$PATH:`pwd`/infer/infer/bin\"" \ >> ~/.bash_profile &&
source ~/.bash_profile
```

你可用通过在命令行执行 ```echo $SHELL``` 确定所使用的 shell。根据具体的情况调整上面的命令。

如果你是在 linux 中，如果 `~/.bash_profile` 不存在的话，你也许需要把 `~/.bash_profile` 替换成 `~/.bashrc`。
