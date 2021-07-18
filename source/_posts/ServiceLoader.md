---
title: ServiceLoader
tags: [java]
date: 2021-07-16 12:02:31
categories: java
---

ServiceLoader：使用到编译时注解时，定义注解并在目标元素上标注上注解后，都还需要定义一个具体的注解处理器。注解处理器的作用在于对注解的发现与处理，如实现自定义的注解处理逻辑，生成新的Java文件等。那注释处理器时如何在编译阶段被javac编译器发现并调用的呢，这其中的过程实际上用到了ServiceLoader机制。

