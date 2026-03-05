---
title: 使用 AI 辅助修改开源项目的实践：以浏览器插件开发为例
date: 2026-03-04 10:00:00
tags: ['ai', 'open-source', 'browser-extension', 'markdown']
---

## 背景

在日常工作中，我经常使用 [Copy as Markdown](https://github.com/yorkxin/copy-as-markdown) 这款浏览器扩展来复制网页链接为 Markdown 格式。但有一个痛点：复制的 URL 经常包含大量追踪参数（如 `spm`、`utm_*` 等），导致生成的链接非常冗长。

例如分享一篇知乎文章：
```
原始: https://zhuanlan.zhihu.com/p/22646254?utm_campaign=official_account&utm_source=weibo&utm_medium=zhihu&utm_content=zhuanlan
期望: https://zhuanlan.zhihu.com/p/22646254
```

或者复制阿里云产品页：
```
原始: https://www.aliyun.com/product/ecs?spm=5176.19720258.J_3207526240.2.e9392c4axxxxxx
期望: https://www.aliyun.com/product/ecs
```

这是一个典型的「小需求」—— 功能明确、改动不大，但需要理解一个陌生的开源项目。正好用来实践 AI 辅助开发。

## 与 AI 协作的完整流程

### 第一步：让 AI 理解项目

**我的做法**：直接让 AI 分析项目结构和功能

```
我: 请概述下当前项目功能是干啥的
```

AI 会自动扫描项目目录、阅读 README 和关键源码，然后给出结构化的项目概述，包括：
- 项目功能定位
- 技术栈（TypeScript、测试框架等）
- 代码架构（分层设计、模块划分）

**效果**：几秒钟内就对一个陌生项目有了全局认知，省去了自己翻代码的时间。

### 第二步：描述需求，让 AI 规划

**我的做法**：用自然语言描述需求和示例

```
我: 在拷贝当前网址导出为 markdown 格式时，需要去除掉 spm=xxx 这个参数。
    例如 https://www.aliyun.com/exp?spm=5176.xxx 需要变成 https://www.aliyun.com/exp
    请修改代码实现这个能力
```

AI 会：
1. 理解需求本质（URL 参数清理）
2. 自动定位需要修改的代码位置
3. 制定任务计划（创建工具模块 → 集成到现有服务 → 编写测试）
4. 逐步执行并展示进度

**效果**：AI 不仅实现了 `spm` 的清理，还主动扩展支持了 `utm_*`、`fbclid`、`gclid` 等常见追踪参数。

### 第三步：验证结果

AI 完成代码后，自动运行测试验证：

```
✓ 120 个单元测试全部通过
✓ 其中 25 个是新增的 URL 清理测试
```

我只需要检查测试覆盖的场景是否符合预期。

### 第四步：Git 操作与打包

继续用自然语言指挥：

```
我: 新建一个 feature 分支，把这个功能提交上去
我: 请编译并打包项目，便于我在 Chrome 里安装
```

AI 自动完成分支创建、代码提交、编译打包，并告知安装方法。

### 第五步：提交 PR 到原始项目

本地验证通过后，继续让 AI 处理 PR 提交：

```
我: 请继续操作，把这个 feature 在原始项目里提交个 PR
```

AI 会自动：
1. 配置 Git 远程仓库
2. 处理 SSH 密钥认证（遇到问题时会提示解决方案）
3. 推送代码到 Fork 仓库
4. 生成 PR 标题和描述内容

过程中遇到问题时，AI 会主动分析并提供解决方案：

```
# SSH 认证失败时，AI 自动检测并提示
"SSH 密钥被 GitHub 拒绝，可能公钥未添加到你的账户。已将公钥复制到剪贴板，请添加到 GitHub Settings..."
```

当发现提交存在问题时，用自然语言说明即可：

```
我: 发现 PR 存在问题，第一 commit 压缩成 1 个；第二 排除掉 .idea 目录下与项目无关的修改
```

AI 会自动执行 git rebase、重新组织 commit、force push 更新 PR。

**最终效果**：一个干净的 PR，包含 1 个 commit、仅 5 个相关文件。

## 最终效果

| 指标 | 数据 |
|------|------|
| 总耗时 | ~30 分钟 |
| 新增代码 | ~110 行（含工具函数） |
| 修改文件 | 5 个 |
| 新增测试 | 25 个用例 |
| 测试通过率 | 100%（120/120） |
| PR 状态 | 已提交到原始项目 |

修改后的效果：

| 场景 | 修改前 | 修改后 |
|------|--------|--------|
| 知乎文章 | `[文章](https://zhuanlan.zhihu.com/p/xxx?utm_source=weibo&utm_medium=zhihu)` | `[文章](https://zhuanlan.zhihu.com/p/xxx)` |
| 阿里云产品 | `[产品](https://aliyun.com/product?spm=5176.xxx)` | `[产品](https://aliyun.com/product)` |

## 人机协作最佳实践

### 什么时候适合用 AI 辅助？

| 适合 | 不太适合 |
|------|----------|
| 需求明确、边界清晰 | 需求模糊、需要大量讨论 |
| 修改陌生代码库 | 核心业务逻辑设计 |
| 重复性工作（测试、重构） | 需要深度领域知识 |
| 快速原型验证 | 安全敏感代码 |

### 如何与 AI 高效协作？

**1. 提供清晰的上下文**
```
❌ "帮我改一下这个 bug"
✅ "复制 URL 时需要去除 spm 参数，例如 xxx 应该变成 yyy"
```

**2. 让 AI 先理解再动手**
```
先问: "请概述下这个项目的架构"
再说: "请实现 xxx 功能"
```

**3. 分步验证，及时纠偏**
- 让 AI 先说方案，确认后再实现
- 每完成一步就验证（运行测试）
- 发现问题立即反馈

**4. 善用 AI 的延展能力**
- 我只提了 `spm`，AI 主动扩展了十几种追踪参数
- 我只要求实现功能，AI 自动补充了边界测试

### 人类需要把关什么？

1. **需求理解** - AI 可能过度解读或遗漏细节
2. **方案合理性** - 检查是否符合项目架构
3. **测试覆盖** - 确保测试真正验证了业务场景
4. **安全性** - 涉及敏感操作时要人工审查

## 总结

这次实践让我体会到 AI 辅助开发的核心价值：

> **把「理解代码」和「编写代码」的时间成本大幅降低，让开发者专注于「决策」和「验证」。**

对于修改开源项目这类场景，AI 特别擅长：
- 快速理解陌生代码库的结构
- 定位需要修改的位置
- 生成符合项目风格的代码
- 自动补充测试用例
- 处理 Git 操作和 PR 提交

而人类的价值在于：
- 提出真正的需求
- 判断方案是否合理
- 验证结果是否正确
- 决定是否采纳

这种人机协作模式，让「给开源项目加个小功能」从「半天的事」变成了「半小时的事」。

---

**原始项目**: [yorkxin/copy-as-markdown](https://github.com/yorkxin/copy-as-markdown)  
**我的 Fork**: [DavyJones2010/copy-as-markdown](https://github.com/DavyJones2010/copy-as-markdown)  
**提交的 PR**: [feat: add URL tracking parameters cleaner](https://github.com/yorkxin/copy-as-markdown/pull/new/feature/url-tracking-params-cleaner)
