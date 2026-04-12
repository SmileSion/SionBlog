+++
date = '2026-03-31T10:53:07+08:00'
draft = false
title = '什么是Channel，怎么实现的？'
tags = ['Golang']
+++

“Channel 在 Go 运行时里是一个叫 hchan 的结构体，核心是一个环形队列 + 两个等待队列（发送方和接收方），再加上一把互斥锁保证并发安全。”

