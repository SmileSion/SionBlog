+++
date = '2026-01-07T10:11:05+08:00'
draft = false
title = 'Mac mini M4 搭建 Minecraft 高性能服务器教程'
description = '关于如何使用Mac mini搭建我的世界整合包服务器教程'
author = 'SmileSion'
tags = ["Java","MacOs","Minecraft"]


+++

## 0. 前言

Mac mini M4 拥有顶级的单核性能和极小的体积，尤其是经过补贴之后**16+256GB**配置只需要3000左右的价格就能买到手，我宣布这就是最适合用来开MC的服务器主机。

本人尝试拿它开启了homestead服务器，是一个包含400+mod的整合包，实测前中期**10人**的情况下稳定运行压力不大（视距6，模拟距离8），奈何被家里的烂网制裁，只能转去第三方租赁面板服。

B站博主也有测试过，在MC服务器领域，完全不输i9-14900K，视频参考： [最适合开MC服务器的电脑？是不到4000块的M4 Mac Mini？！](https://www.bilibili.com/video/BV1NLf5YKEvp/?share_source=copy_web&vd_source=48e18bfa82c8056dcaa225baad68293c)

**注意：** 服务器性能没问题不代表你家网没问题，你在拥有Mac mini之前应该考虑是否有稳定的网络和足够的上行带宽！！
{{< img src="image-20260107101904173.png" alt="fufu macmini m4" >}}

## 1. Mac 系统防休眠设置 

Macmini不连显示器的情况下断开SSH连接之后，它就美滋滋的睡过去了，只监听22端口有人连接才能正常运行，所以如果是当作MC服务器，请不要让它休眠。

### 开启远程访问

激活Mac mini后，在「系统设置」->「通用」->「共享」中，打开 **远程登录 (SSH)**。这样你就可以在其他电脑甚至手机上通过 `ssh` 随时管理你的服务器。

### 开启持久模式 (`caffeinate`)

在配置好远程连接之后，请在终端使用 `caffeinate` 命令。

- `-i`: 防止系统计算睡眠。
- `-s`: 防止系统在接通电源时睡眠。
- `-m`: 防止磁盘睡眠。
- `-d`:防止显示器休眠。

```shell
caffeinate -ism
```

## 2. 环境搭建

### 安装适配 M4 的 JDK

一定要用 **ARM64** 原生版本，否则性能损失巨大。推荐使用 Homebrew 安装：

```shell
# 安装 Homebrew (如果还没装)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 安装 Java 21 (适配 1.20.5+)
brew install openjdk@21
```

## 3. 创建与启动服务器

1. **准备文件夹**：`mkdir ~/MCServer && cd ~/MCServer`

2. **下载服务端**：在你的整合包页面下载对应的服务端，一般是serverpack之类的压缩包。

3. **修改启动脚本**：一般服务端文件夹内自带start.sh或者run.sh，你可以修改他们写好的启动脚本来更改java版本或者性能调优，一般使用vim编辑修改即可。

   **关于JVM性能优化：**在你的启动脚本中，将 Java 启动行修改为：

   ```shell
   caffeinate -ism java -Xms5G -Xmx12G \
   -XX:+UseG1GC \
   -XX:+ParallelRefProcEnabled \
   -XX:MaxGCPauseMillis=200 \
   -XX:+UnlockExperimentalVMOptions \
   -XX:+DisableExplicitGC \
   -XX:+AlwaysPreTouch \
   -XX:G1NewSizePercent=30 \
   -XX:G1MaxNewSizePercent=40 \
   -XX:G1HeapRegionSize=8M \
   -XX:G1ReservePercent=20 \
   -XX:G1HeapWastePercent=5 \
   -XX:G1MixedGCCountTarget=4 \
   -XX:InitiatingHeapOccupancyPercent=15 \
   -XX:G1MixedGCLiveThresholdPercent=90 \
   -XX:G1RSetUpdatingPauseTimePercent=5 \
   -XX:SurvivorRatio=32 \
   -XX:+PerfDisableSharedMem \
   -XX:MaxTenuringThreshold=1 \
   -jar server.jar nogui
   ```
   

​	**为什么这么改？**

>    - **`-XX:+UseG1GC` (垃圾回收器之王)**：
>      - **解释**：这是目前最适合大内存 MC 服务器的回收器。它将内存切成很多小块（Region），哪里脏了扫哪里，有效避免了老旧服务器常见的“全服瞬间卡顿（Stop-the-world）”。
>    - **`-XX:+AlwaysPreTouch` (内存预热)**：
>      - **解释**：由于 Mac M4 使用的是**统一内存**，这个参数会在服务器启动时强制系统立即分配全部 12G 物理内存。如果不加这个，系统会“挤牙膏”式分配，导致游戏过程中因为申请内存产生瞬间掉帧。
>    - **`-XX:MaxGCPauseMillis=200` (硬核限时)**：
>      - **解释**：告诉 Java：“每次回收垃圾的时间不要超过 200 毫秒”。在 M4 的高主频下，这个参数能让回收动作快如闪电，玩家几乎感知不到。
>    - **`-XX:+ParallelRefProcEnabled` (多核并行)**：
>      - **解释**：M4 拥有多个性能核心。开启此项可以让垃圾回收的引用处理过程并行化，充分榨干 M4 多核的并行处理能力。
>    - **`-XX:G1HeapRegionSize=8M` (区块大小调整)**：
>      - **解释**：针对 Homestead 这种有大量复杂模组（Mod）的服务器，调大区块大小可以减少内存碎片的产生，提升内存利用率。
>    - **`-XX:+DisableExplicitGC` (拒绝乱指挥)**：
>      - **解释**：防止某些写得不好的模组（Mod）在代码里乱叫“现在给我回收垃圾”，统一交给 JVM 自动管理，保证平稳。

4.**运行脚本开启服务器**


   *赋予权限*：`chmod +x start.sh`

   *启动服务器*：`./start.sh`

   **注意：**如果你想保证数据和系统的安全，请创建一个单独的账户用来开设服务器和内网穿透。

## 4. 内网穿透

如果你没有公网 IP，`frp` 是最稳定、最专业的方案。

frp下载地址：https://github.com/fatedier/frp/releases

frp使用指南：https://gofrp.org/zh-cn/docs/examples/

### 准备工作

你需要一台带公网 IP 的轻量服务器（如腾讯云/阿里云）作为“中转站”。

### 服务端 (frps - 云服务器上配置)

编辑 `frps.toml`:

```toml
bindPort = 7000
```

### 客户端 (frpc - Mac mini 上配置)

1. 下载适合 macOS 的 frp 客户端。

2. 编辑 `frpc.toml`:

   ```toml
   serverAddr = "你的云服务器IP"
   serverPort = 7000
   
   [[proxies]]
   name = "mc-server"
   type = "tcp"
   localIP = "127.0.0.1"
   localPort = 25565
   remotePort = 25565
   ```
   
3. 在 Mac 终端运行：`./frpc -c frpc.toml`


## 5. 性能监控与自动维护

- **监控**：在终端输入 `top -o cpu` 观察 M4 核心的负载（你会发现它轻松得过分）。

- **后台运行**：建议使用 tmux 开启会话，这样即便你关闭了终端窗口，服务器也会继续运行：

  ```shell
  tmux new -s mcserver
  ./start.sh
  # 按下 Ctrl+B，再按 D 即可脱离会话
  ```

- **服务器卡顿排查**：一般服务端都自带spark模组，请查看wiki使用spark相关命令排查具体是哪个mod影响服务器运行。
