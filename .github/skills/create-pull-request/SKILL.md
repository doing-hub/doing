---
name: create-pull-request
description: Create branches, run pre-submit checks, and submit Pull Requests via gh CLI. Auto-detects target branch based on change type. Use when the user wants to create a PR, submit changes, push code, or asks for help with the PR workflow.
---

# Create Pull Request Workflow

帮助创建分支、运行预检查、并通过 gh 命令创建 Pull Request 的完整工作流。

## Quick Start

当用户请求创建 PR 时，按以下步骤执行：

```
Task Progress:
- [ ] Step 1: 分析变更类型和范围
- [ ] Step 2: 确保分支命名正确
- [ ] Step 3: 运行预检查
- [ ] Step 4: 推送代码
- [ ] Step 5: 创建 Pull Request
```

---

## Step 1: 分析变更类型

首先并行执行以下命令了解当前状态：

```bash
git status                    # 查看未提交的文件
git diff --stat              # 查看改动范围
git log -1 --format='%s'     # 查看最近的 commit message
git branch --show-current    # 查看当前分支名
```

### 变更类型判断

| 类型 | 关键词/特征 | 分支前缀 | 目标分支 |
|------|------------|---------|---------|
| 新特性 | 新增功能、新组件、新 API | `feat/` | `feature` |
| Bug 修复 | 修复、fix、bug | `fix/` | `master` |
| 文档 | README、docs、注释 | `docs/` | `master` |
| 重构 | 重构、优化结构、不改变行为 | `refactor/` | `master` |
| 样式 | 样式、CSS、视觉调整 | `style/` | `master` |
| 性能 | 性能优化、加速 | `perf/` | `master` |
| 测试 | 测试用例、coverage | `test/` | `master` |
| 构建 | webpack、build、打包 | `build/` | `master` |
| CI | 工作流、GitHub Actions | `ci/` | `master` |
| 依赖 | 升级依赖、package.json | `deps/` | `master` |

---

## Step 2: 分支命名

### 命名规范

```
<type>/<description>
```

- **type**: 见上表的分支前缀
- **description**: 小写字母，用连字符分隔单词，简短但有描述性

### 示例

```bash
feat/add-dark-mode-support
fix/button-style-in-safari
docs/update-api-reference
refactor/simplify-state-management
style/improve-table-border
perf/optimize-render-performance
```

### 创建分支（如需要）

```bash
# 如果当前在 master/feature/main 分支，需要创建新分支
git checkout -b <type>/<description>
```

---

## Step 3: 预检查

### 检查清单

在提交 PR 前，按顺序运行以下检查：

```bash
# 1. TypeScript 类型检查
npm run tsc 2>&1 || npx tsc --noEmit 2>&1

# 2. ESLint 检查
npm run lint 2>&1 || npx eslint . 2>&1

# 3. 运行测试（可选，根据项目配置）
npm test -- --passWithNoTests 2>&1
```

### 常见检查命令（按项目）

| 项目类型 | 类型检查 | Lint |
|---------|---------|------|
| 全局检查 | `pnpm run tsc` | `pnpm run lint` |
| 通用 React | `npx tsc --noEmit` | `npm run lint` |
| Next.js | `npm run build` | `npm run lint` |

### 处理检查失败

如果检查失败：

1. 显示错误信息给用户
2. 询问是否需要帮助修复
3. 修复后重新运行检查

---

## Step 4: 推送代码

### 确保代码已提交

```bash
# 检查是否有未提交的更改
git status --porcelain
```

如果有未提交的更改，先询问用户是否需要帮助创建 commit。

### 推送到远程

```bash
# 推送并设置上游分支
git push -u origin HEAD
```

---

## Step 5: 创建 Pull Request

### 确定目标分支

根据 Step 1 的分析自动选择：

- **新特性 (feat/)** → `feature` 分支（若存在）
- **其他所有类型** → `main` 分支（本仓库默认分支）

如果 `feature` 不存在，回退到 `main`（必要时再回退到 `master`）。

### 确定 PR 标题（必须通过校验）

本仓库的 PR 校验要求标题以 **emoji + 类型** 开头，且必须匹配以下列表之一：

```
✨ feat
🐛 fix
📝 docs
💄 style
♻️ refactor
⚡️ perf
✅ test
👷 build
🤖 ci
🐳 chore
🔙 revert
📚 types
```

示例：

```
✨ feat: add dark mode support
🐛 fix: resolve button style issues in Safari
📝 docs: update API documentation for Table component
♻️ refactor: simplify form validation logic
```

### 生成 PR 内容（必须包含英文/中文 Changelog）

本仓库会校验 PR body 的 `🇺🇸 English` / `🇨🇳 Chinese` 字段是否非空。
请直接沿用 `.github/PULL_REQUEST_TEMPLATE.md` 的结构：

```bash
gh pr create --base <target-branch> --title "<emoji> <type>: <description>" --body "$(cat <<'EOF'
#### 💻 Change Type

<!-- For change type, change [ ] to [x]. -->

- [ ] ✨ feat
- [ ] 🐛 fix
- [ ] 📝 docs
- [ ] 💄 style
- [ ] ♻️ refactor
- [ ] ⚡️ perf
- [ ] ✅ test
- [ ] 👷 build
- [ ] 🤖 ci
- [ ] 🐳 chore
- [ ] 🔙 revert
- [ ] 📚 types

#### 🔗 Related Issue

<!-- Link to the issue that is fixed by this PR -->

<!-- Example: Fixes #123, Closes #456, Related to #789 -->

#### 🔀 Description of Change

<!-- Thank you for your Pull Request. Please provide a description above. -->

| Language   | Change |
| ---------- | ------ |
| 🇺🇸 English | <english-changelog> |
| 🇨🇳 Chinese | <chinese-changelog> |

#### 🧪 How to Test

<!-- Please describe how you tested your changes -->

<!-- For AI features, please include test prompts or scenarios -->

- [ ] Tested locally
- [ ] Added/updated tests
- [ ] No tests needed

#### 📸 Screenshots / Videos

<!-- If this PR includes UI changes, please provide screenshots or videos -->

| Before | After |
| ------ | ----- |
| ...    | ...   |

#### 📝 Additional Information

<!-- Add any other context about the Pull Request here. -->

<!-- Breaking changes? Migration guide? Performance impact? -->
EOF
)"
```

## Troubleshooting

### gh 命令未安装

```bash
# macOS
brew install gh

# 登录
gh auth login
```

### 远程分支不存在

```bash
# 检查远程分支列表
git fetch --all
git branch -r | grep -E '(master|main|feature)'
```

### PR 创建失败

常见原因：

1. 没有 push 代码 → 先 `git push -u origin HEAD`
2. 没有登录 gh → 运行 `gh auth login`
3. 分支已有 PR → 使用 `gh pr view` 查看现有 PR
