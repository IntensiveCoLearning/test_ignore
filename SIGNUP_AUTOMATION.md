# Signup 自动化功能说明

本项目实现了自动化的用户注册流程，当用户提交 signup issue 并被管理员审核通过后，系统会自动完成以下操作：

## 功能特性

1. **自动权限分配**：为通过审核的用户自动分配仓库的 write 权限
2. **自动文件创建**：根据用户信息自动创建个人学习文件
3. **智能信息提取**：从 issue 内容中提取用户信息并格式化
4. **自动通知**：在 issue 中自动回复处理结果

## 使用流程

### 1. 用户提交 signup issue

用户需要：

- 使用提供的 issue 模板创建新的 issue
- 标题必须以 `Signup - ` 开头，例如：`Signup - 张三`
- 正确填写所有必要信息，特别是 **GitHub ID**

### 2. 管理员审核

管理员需要：

- 审核用户提交的信息
- 确认用户符合参与条件
- 将 issue **Close as completed**（重要：必须选择 "completed" 而不是普通的 close）

### 3. 系统自动处理

当 issue 被 close as completed 后，系统会：

- 验证 GitHub ID 的有效性
- 为用户分配仓库的 write 权限
- 创建以 GitHub ID 命名的 `.md` 文件
- 在 issue 中回复处理结果

## Issue 模板格式

用户在创建 signup issue 时需要包含以下信息：

```markdown
## 报名信息

**GitHub ID:** your-github-username

## 个人信息

**姓名/昵称:** 你的姓名或昵称
**时区:** UTC+8
**联系方式:** 你的联系方式（推荐 Telegram）

## 自我介绍

请简单介绍一下自己...

## 学习计划

请简单描述你的学习计划和目标...
```

## 生成的用户文件格式

系统会自动创建如下格式的用户文件：

```markdown
---
timezone: UTC+8
---

> 请在上边的 timezone 添加你的当地时区(UTC)，这会有助于你的打卡状态的自动化更新

# 用户名

**GitHub ID:** username
**联系方式:** contact-info

## 自我介绍

[从 issue 中提取的自我介绍内容]

## 学习计划

[从 issue 中提取的学习计划内容]

## Notes

<!-- Content_START -->

### 2024.01.01

开始学习！

<!-- Content_END -->
```

## 技术实现

### GitHub Actions Workflow

文件位置：`.github/workflows/signup-automation.yml`

触发条件：

- 事件类型：`issues` with `closed` action
- 条件筛选：
  - `github.event.issue.state_reason == 'completed'`
  - `startsWith(github.event.issue.title, 'Signup - ')`

### 权限要求

workflow 需要以下权限：

- `contents: write` - 用于创建和提交文件
- `issues: write` - 用于在 issue 中回复
- `administration: write` - 用于管理仓库协作者

需要配置 `PAT_WITH_INVITE_PERMISSIONS` secret，该 token 需要有邀请协作者的权限。

## 常见问题

### Q: 为什么我的 signup issue 没有被自动处理？

A: 请检查：

1. issue 标题是否以 `Signup - ` 开头
2. issue 是否被 "Close as completed"（不是普通的 close）
3. GitHub ID 是否正确填写
4. 是否已经存在同名的用户文件

### Q: 如何修改用户信息？

A: 自动创建的用户文件可以后续手动编辑，或者联系管理员删除后重新提交 signup issue。

### Q: 系统提取信息不准确怎么办？

A: 请严格按照 issue 模板格式填写信息，确保使用正确的标题格式（如 `**GitHub ID:**`）。

## 维护说明

### 修改 issue 模板

模板文件位置：`.github/ISSUE_TEMPLATE/signup.md`

### 修改用户文件模板

在 workflow 文件中的 "Create user markdown file" 步骤中修改文件内容模板。

### 调试和日志

可以在 GitHub Actions 的运行日志中查看详细的处理过程和错误信息。
