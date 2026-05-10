+++
date = '2026-05-10T00:00:00+08:00'
draft = false
title = 'OpenSpec 的使用教程'
description = '以 Hugo 博客 demo 为例，介绍 OpenSpec 的安装、初始化、工作流，以及在 Codex 和 Claude Code 中的不同使用方式。'
author = 'SmileSion'
tags = ["OpenSpec", "Codex", "Claude", "AI", "教程", "Hugo"]
+++

# OpenSpec 的使用教程

OpenSpec 是一个面向 AI 编程助手的规格驱动开发工具。它的核心思路不是一上来就让 AI 改代码，而是先把“要做什么、为什么做、怎么验收、分几步做”沉淀成文档，再让 AI 根据这些文档实施。

如果你经常让 AI 帮你改项目，应该遇到过这类问题：

- 一开始说的是 A，AI 改着改着变成了 B。
- 需求没写清楚，代码实现出来才发现方向不对。
- 一个会话里聊明白了，换一个新会话后上下文又丢了。
- 功能做完了，但没有把设计和决策沉淀下来。

OpenSpec 解决的是这个过程问题。它会把每次变更放到 `openspec/changes/` 下面，用 `proposal.md`、`design.md`、`specs/`、`tasks.md` 这类文件记录上下文。这样你、AI、后续的新会话都能围绕同一套文档继续工作。

这篇文章用一个 Hugo 博客的小 demo 来讲 OpenSpec 的基本用法：假设我们想给博客新增一个“文章底部推荐阅读”功能。

## 1. 安装 OpenSpec

OpenSpec 当前推荐使用 Node.js 环境安装。先确认本机 Node.js 版本：

```shell
node --version
```

官方文档要求 Node.js 20.19.0 或更高版本。如果版本太低，建议先通过 nvm、fnm、Homebrew 或系统包管理器升级 Node.js。

使用 npm 全局安装：

```shell
npm install -g @fission-ai/openspec@latest
```

也可以使用 pnpm、yarn 或 bun：

```shell
pnpm install -g @fission-ai/openspec@latest
yarn global add @fission-ai/openspec@latest
bun install -g @fission-ai/openspec@latest
```

安装完成后检查版本：

```shell
openspec --version
```

能正常输出版本号，就说明 CLI 已经可用了。

参考：

- OpenSpec Getting Started：<https://openspec.pro/getting-started/>
- OpenSpec CLI Reference：<https://github.com/Fission-AI/OpenSpec/blob/main/docs/cli.md>

## 2. 在项目中初始化

进入你的项目目录，例如一个 Hugo 博客项目：

```shell
cd ~/code/my-hugo-blog
openspec init
```

初始化时 OpenSpec 会让你选择要配置的 AI 工具，例如 Claude Code、Codex、Cursor 等。你可以根据自己实际使用的工具选择。

初始化后，项目中通常会出现类似结构：

```text
openspec/
├── config.yaml
├── specs/
└── changes/
```

不同 AI 工具还会生成对应的指令文件或 skill 文件。比如 Codex 会更偏向 skill 调用，Claude Code 则可以直接使用 slash command。

初始化完成后，可以运行：

```shell
openspec list --json
```

如果输出类似下面这样，说明当前没有正在进行的变更：

```json
{"changes":[]}
```

## 3. OpenSpec 的基本工作流

可以把 OpenSpec 理解成一条从“想法”到“归档”的流水线：

```text
Explore -> Propose -> Apply -> Verify -> Archive
```

对应含义如下：

| 阶段 | 作用 | 产物 |
| --- | --- | --- |
| Explore | 只读探索项目，理解现状和风险 | 项目理解、方案讨论 |
| Propose | 创建变更提案，明确做什么和为什么 | `proposal.md` |
| Design | 说明实现思路和关键决策 | `design.md` |
| Specs | 写可验收的需求规格 | `specs/**/spec.md` |
| Tasks | 拆分实施步骤 | `tasks.md` |
| Apply | 按任务实施代码或内容改动 | 实际项目变更 |
| Verify | 运行测试、构建、人工 review | 验证结果 |
| Archive | 归档完成的 change，沉淀规格 | `openspec/changes/archive/` |

OpenSpec 的价值不只是“多写几个文档”，而是让 AI 在动手前先对齐边界：

- 这个功能为什么要做？
- 哪些内容属于目标，哪些不做？
- 验收标准是什么？
- 分几步实施？
- 后续新会话如何继续？

## 4. Demo：给 Hugo 博客添加推荐阅读

假设我们有一个 Hugo 博客，想新增一个功能：

> 在文章底部显示 3 篇推荐阅读，优先展示同标签文章，如果同标签不足，再展示最近文章。

这个需求很适合用 OpenSpec 管理。因为它看起来简单，但实际会牵涉：

- Hugo 模板放在哪里改。
- 推荐文章的筛选规则是什么。
- 当前文章是否要排除。
- 没有同标签文章时怎么降级。
- 移动端样式是否需要适配。

如果直接对 AI 说“帮我加推荐阅读”，AI 很可能马上开始改模板。但用 OpenSpec 时，可以先探索。

## 5. Codex 中的使用方式

在 Codex 里，本项目使用的是 OpenSpec skills。也就是说，不直接输入 Claude 那种 `/opsx:*` 命令，而是调用对应 skill。

### 5.1 Explore：先只读探索

可以这样让 Codex 分析 Hugo 博客结构：

```text
$openspec-explore
请探索当前 Hugo 博客项目，重点分析：

1. Hugo 配置文件在哪里
2. 当前主题和 layout 覆盖关系
3. 单篇文章模板在哪里
4. 文章底部已有的导航、分享、评论区域在哪里
5. 如果要添加“推荐阅读”，建议修改哪些文件

要求：
- 只读分析，不修改代码
- 最后输出一份中文项目理解报告
```

Explore 阶段的重点是“不改代码”。它适合用来弄清楚现状，避免还没理解项目结构就开始动手。

### 5.2 Propose：创建变更提案

探索清楚后，可以让 Codex 创建 change：

```text
$openspec-propose
创建一个 OpenSpec change：add-related-posts。

目标：
给 Hugo 博客文章页添加“推荐阅读”区域。

要求：
1. 推荐同标签文章优先
2. 排除当前文章
3. 最多展示 3 篇
4. 同标签不足时用最近文章补齐
5. 保持 PaperMod 当前视觉风格
6. 先只生成 proposal、design、specs、tasks，不实现代码
```

生成后，通常会出现：

```text
openspec/changes/add-related-posts/
├── proposal.md
├── design.md
├── specs/
└── tasks.md
```

这一步完成后，建议先 review 文档，而不是马上 apply。因为这里决定了后面的实现边界。

### 5.3 Apply：按任务实施

确认提案和任务没问题后，再让 Codex 实施：

```text
$openspec-apply-change
根据 add-related-posts 实施这个 change。
```

Codex 会读取 `proposal.md`、`design.md`、`specs/`、`tasks.md`，按 `tasks.md` 里的 checkbox 一项项完成，并在完成后把任务标记为 `[x]`。

### 5.4 Archive：完成后归档

构建、预览、review 都通过后，可以归档：

```text
$openspec-archive-change
归档 add-related-posts。
```

归档的意义是把这个变更从“进行中”变成“已完成”，并把规格沉淀下来。以后新会话再看项目时，不需要靠聊天记录回忆这个功能是怎么来的。

## 6. Claude Code 中的使用方式

Claude Code 的用法和 Codex 不太一样。Claude Code 可以直接使用 OpenSpec 的 slash command，所以写法更像这样：

```text
/opsx:explore
```

或者：

```text
/opsx:propose add-related-posts
```

同样以 Hugo 博客“推荐阅读”为例。

### 6.1 Explore

```text
/opsx:explore
请探索当前 Hugo 博客项目，重点分析：

1. 文章页模板在哪里
2. PaperMod 的 post nav、share、comments 分别在哪里
3. 推荐阅读适合放在哪个 partial 或 template 中
4. 有没有现成的 tags、related pages 或 recent pages 可以复用

只分析，不实现。
```

### 6.2 Propose

```text
/opsx:propose add-related-posts

给 Hugo 博客文章页添加“推荐阅读”区域：
- 最多展示 3 篇
- 优先同标签文章
- 排除当前文章
- 不足时用最近文章补齐
- 样式保持 PaperMod 风格
```

### 6.3 Apply

```text
/opsx:apply add-related-posts
```

Claude Code 会根据 change 下的文档执行任务。执行过程中如果发现设计不清楚，可以让它先更新 `design.md` 或 `tasks.md`，再继续实现。

### 6.4 Archive

```text
/opsx:archive add-related-posts
```

归档后，这个 change 就从 active changes 中移出，后续只保留为项目历史和规格沉淀。

## 7. Codex 和 Claude 的区别

两者都可以配合 OpenSpec 使用，区别主要在入口命令。

| 阶段 | Codex skill 写法 | Claude Code slash command |
| --- | --- | --- |
| 探索 | `$openspec-explore` | `/opsx:explore` |
| 提案 | `$openspec-propose` | `/opsx:propose <change-name>` |
| 实施 | `$openspec-apply-change` | `/opsx:apply <change-name>` |
| 归档 | `$openspec-archive-change` | `/opsx:archive <change-name>` |

我的理解是：

- 在 Codex 中，更适合把 OpenSpec 当作一组技能来调用。
- 在 Claude Code 中，更适合把 OpenSpec 当作一组项目命令来调用。
- 不管入口是什么，最终核心都是同一套 `openspec/changes/<change-name>/` 文档。

## 8. 常用检查命令

查看当前有哪些 active changes：

```shell
openspec list --json
```

查看某个 change 的状态：

```shell
openspec status --change "add-related-posts"
```

查看 JSON 状态，方便 AI 或脚本读取：

```shell
openspec status --change "add-related-posts" --json
```

查看项目支持的 OpenSpec 指令：

```shell
openspec --help
openspec init --help
```

## 9. 使用建议

OpenSpec 不适合把所有小修改都流程化。比如改一个错别字、换一个链接，直接改就好。

它更适合这类场景：

- 需求会影响多个文件。
- 你想让 AI 先分析再动手。
- 功能有明确验收标准。
- 你希望换一个新 session 后还能继续上下文。
- 你想保留一次功能变更的设计记录。

我一般会按这个节奏使用：

```text
小问题：直接让 AI 改
中等功能：OpenSpec propose + apply
复杂功能：先 explore，再 propose，再 review，最后 apply
```

对于个人博客来说，OpenSpec 的价值不在于“流程感”，而在于它能让每次改动都有一个清楚的来龙去脉。尤其是当博客里混合了 Hugo 模板、shortcode、CSS、JavaScript 和内容文件时，先把修改边界写清楚，会比直接让 AI 猜要稳得多。
