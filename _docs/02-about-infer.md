---
id: about-Infer
title: 关于 Infer
layout: docs
permalink: /docs/about-Infer.html
section: Foundations
section_order: 02
order: 01
---

Infer 是一个静态程序分析工具,可以对 Java、C 和 Objective-C 程序进行分析，此工具是用 [OCaml](https://ocaml.org/) 写成的。   
Infer 最早部署在 Facebook 内部，用于发布移动应用之前对每一行代码进行分析，目前 Facebook 使用此工具分析所开发的 Android、iOS 应用,包括 Facebook Messenger、Instagram 和其他一些应用。
Infer 不仅仅用于移动应用程序的分析，还可以分析  C、Java 等不是 Android 系统的代码。
目前 Infer 着重于发现一些诸如空指针的访问、资源和内存的泄露等导致手机程序崩溃或性能严重下降的问题。


Infer 源自 2013 年收购的一家初创企业 Monoidics。Monoidics 本身就是根据最近的学术研究成果，特别是在 separation logic 和 bi-abduction。在 Facebook 内部，Infer 进行持续的迭代开发，并根据从开发者收集的反馈进行相应的调整。开源之后我们将继续开发 Infer，已造福其他使用者，并且我们可以与社区的其他开发者已开发出更好的程序检验技术为目标而为之奋斗。
