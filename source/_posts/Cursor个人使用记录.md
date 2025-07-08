---
title: Cursor个人使用记录
date: 2025-07-08 17:08:34
tags:
- Cursor
- AI编程
- 工具使用
categories: 技术
---

## 自定义mode

  在Agent、Ask、Manual，现在还有个Background，在这些之外，还可以自定义mode。

  我们可以设置自定义mode的一些功能：

  * 自定义命名，说明这个mode是作用于什么，例如"代码审查员"
  * 选择可以使用的Tools，`Search`、`Edit`、`Run`(终端)、`MCP`(选择MCP工具)
  * 还可以添加自定义介绍

  **注: 使用前必须先在 设置 - Chat - Custom Modes 开启 **

## AI模型

初始的AI模型可能很少，需要再 **设置 - Models** 开启。

## MCP工具

这个其他文档有详细介绍， **设置 - Tools & Integrations**  `MCP Tools`栏 - `+ New MCP Server`

MCP就是给AI权限，让AI可以去进行一些操作，举个例子fetch赋予AI自主抓取网页分析的能力，Git让AI可以查看Git分支内容。这样就可以让AI自行去分析，而不是我们获取信息再丢给AI处理。

## Rules规则

由于AI对于项目的一些习惯和规则是不熟悉的，需要进行设置。否则AI很可能会出现发散式思考，偏离需求。另外，AI也可能偷懒，需要通过规则让AI更准确的实现我们的需求。

**设置- Rules * Memories** `Project Rules`栏 - `+ Add Rule` 

# @符号工具显性指定上下文

| @工具       | 范例                            | 使用场景                                                     | 缺点                                                   |
| ----------- | ------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------ |
| @code       | @loadAndApplyRulesTask          | 我们知道哪个函数、常量或符号与当下需要生成的输出相关         | 需要大量的代码库知识                                   |
| @files      | @check_rule.md                  | 我们知道该读取或编辑哪个文件，但不知道文件的确切位置         | 可能包括许多与手头任务无关的上下文，具体取决于文件大小 |
| @folders    | @Invoker                        | 文件夹中的所有内容或大多数文件都是相关的                     | 可能包括许多与手头任务无关的上下文                     |
| @docs       | @s3 @Amazon S3                  | 第三方官方功能、接口等文档，也允许自定义文档，但只能远程访问 | 可能包括许多与手头任务无关的上下文                     |
| @past chats | @Chat Name Generation           | 应用指定历史会话                                             | 不好回溯                                               |
| @Branch     | @Branch（Diff with Main Branch) | 当前分支和Main分支进行比较，用于review代码                   | 只能和Main分支比较                                     |



## 多窗口处理

适当开始新的聊天有助于保持专注、高效，节约成本。

- Ctrl+T 或是 Alt+[点击创建 chat]，可以开始新的聊天 tab 窗口，不会影响旧的聊天窗口。
- 如果想从当前聊天窗口中间的一个对话开个新分支问答，可以用右下角...里的 Duplicate Chat，来复制对话，开启带有上下文的聊天记录的分叉对话。



## 通过 workspaces 支持多个代码库

特地说明：`.cursor/rules are supported in all floders added`

适合将高度相关的项目作为同一个 workspace 管理，比如比较独立高度相关的前后端项目，也可以是文档项目和代码项目，也可以是两个高度相关的服务项目等。

解决跨项目检索与同步更新问题，甚至可以不同语言项目的代码翻译。

[Cursor 0.5 版本发布的 Multi-Root Workspaces，才是程序员杀手应用 - 知乎](https://zhuanlan.zhihu.com/p/1909022796757042342?share_code=vcHrH8GymcLc&utm_psn=1909321731799249573)

[绝杀！Cursor 0.50 全栈开发利器：Multi-root workspaces](https://mp.weixin.qq.com/s/VAxya6NUqHcvR3qehj-nGQ)

注：存在改变团队编码协作工作方式的可能。