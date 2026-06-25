---
layout: default
title: CLAUDE.md
parent: 🤖 AI 实战笔记
nav_order: 6
---

# CLAUDE.md

## 沟通与规划

- 用中文与我对话，不要过多解释，保持简洁、清晰、准确
- 如果不理解需求，需要和我确认
- 编码或修改前，先明确关键假设；如果存在多种合理理解，先列出取舍并询问，不要擅自猜测
- 如果需求很复杂，创建一个 todo.md 存储在本地，然后和我交互，是否按todo执行
- 如果使用skills发生功能冲突，要让我决定到底用哪个

## 编码原则

- 优先用最简单、最少的代码解决问题，不做投机性抽象、额外配置或未要求的功能
- 精准修改，只改和需求直接相关的内容；发现无关问题可以指出，但不要顺手重构或清理
- 代码需要添加中文注释，代码开头添加程序说明
- 提供代码时，为关键点和难以理解的部分添加注释
- 所有API必须有异常处理
- 数据库操作必须用事务
- SQLite 数据库使用 PRAGMA journal_mode=WAL 和 PRAGMA synchronous=NORMAL，以 WAL 模式替代回滚日志模式，大幅减少 IO 开销
- 简单任务尽量用单python工具实现，尽量少引用第三方库，先查看并调用当前项目目录下的aipython目录中的python程序完成个性化功能，如果需要创建新功能，新建python文件放在此目录中
- 使用ffmpeg工具处理音视频文件

## 文件与格式

- 目录、文件名全部需要用""引起来，因为可能会有中文或空格
- Shell中输出代码不要带行号
- 生成的CSV、MD等文本文件，使用UTF-8编码，并增加BOM头
- 所有文本/Markdown文档统一使用 Windows/PC 换行符（CRLF）
- 每次对话的chat history，都追加到当前目录的chat_history.md文件后面，用markdown格式化并增加时间戳；禁止覆盖旧内容

## 安全与确认

- 任何危险操作例如rm、del等，都需要和我确认，即使是在YOLO下
- 禁止自动执行git commit、push、merge、pull等操作，和我确认后才能执行
- 涉及外部网络、下载、安装依赖、调用外部API、部署、数据库迁移等操作，必须先说明目的并获得确认
- 禁止执行部署、数据库迁移、curl 外部服务等危险命令，如果我是我要求的，要反复和我确认
- 修改重要配置或全局规则前，先确认可恢复方式；必要时先备份
- 生产环境一律跑在 sandbox + 最小权限 token 下

## 测试与验证

- 执行任务时要定义可验证的完成标准；新功能或修复应尽量用测试复现和验证
- 新功能必须先写测试
- 白名单只允许跑幂等校验：lint / test / type-check / build 这类幂等校验；执行前先说明命令目的，非白名单命令必须确认
- PR 模板/CI 必须要强制执行， 要把 agents.md 的 checks 拉齐，保证 checks 全绿

## 文档与同步

- 所有 agents.md 等所有的 .md文件，一律走 Code Review，把它们也当做代码，不准默默合；需要说明变更点、冲突处理和验证结果
- 在当前项目目录中创建Docs目录，放置项目文档，包括但不限于：requirements, design, tasks, api, readme, deployment，optimization-plan，changelog等等
- - Git Config:  User: patdelphi / patdelphi@outlook.com. Credentials via `gh auth`.
- 如果我说同步Obsidian，需要在C:\Nut\ppobs\Claude 目录下，创建项目同名目录，然后放置相关md文档；commit & push 必须再次确认后执行

## 并发与收尾

- 使用多线程任务，比如subagents同时在执行任务，要避免线程冲突，避免抢LLM API，要做好调度，按顺序执行，一定要计时，发现超时情况，要中断任务通知我
- 最终回复需要简要说明改了哪些文件、验证了什么、哪些事项未执行
