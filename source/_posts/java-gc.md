---
title: Java 分代垃圾回收
tags:
- Java
- 学习笔记
categories:
- Java
- 学习笔记
# toc: false
date: 2022-06-22 22:05
---

什么是 Java 分代垃圾回收？

分代是指：Java 根据对象的生命周期时长，将对象分为**年轻代、年老代和持久代**，根据不同代采用不同策略垃圾回收。

在 JVM 中，堆内存划分为 Eden、Survivor 和 Tenured/Old 区，Eden、Survivor 区存储年轻代对象，Tenured/Old 区存储年老代的对象。

垃圾回收主要回收的是年轻代和年老代的对象空间。


### 年轻代

所有新创建的对象都会进入 Eden 区，当 Eden 区存满了（达到一定比例）的时候，就会启动垃圾回收，回收无用对象。经过垃圾回收后依然有用的对象，会进入 Survivor 区。

### 年老代

在年轻代中经历了 N（默认 15）次垃圾回收后依旧存在的对象，就会被放入到 Tenured/Old 区，成为年老代。年老代中的对象是生命周期较长的对象。

### 持久代

持久代一般用于存放静态文件，如：Java 类、方法。持久代对垃圾回收没什么影响。

![图片.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/da484db56ad34c339ae9d59622e2f701~tplv-k3u1fbpfcp-watermark.image?)
![java-gc](../images/java-gc.png)

### Minor GC

清理年轻代区域内存，Eden 满了就会触发 Minor GC。

### Major GC

清理年老代区域内存。

### Full GC 

清理年轻代和年老代区域内存，成本比较高，可能会对系统性能产生影响。


### 可能导致 Full GC 的情况

- 年老代内存打满
- 持久代内存打满
- System.gc() 被显式调用（注意：是建议 Java 启动 GC，但是否启动依据 Java 内部决定）
- 上一次 GC 之后堆内存各区域分配策略动态变化
