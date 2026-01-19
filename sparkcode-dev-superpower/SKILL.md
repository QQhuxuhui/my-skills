---
name: sparkcode-dev-superpower
description: 完整的软件开发工作流，集成Claude端Superpowers（设计+规划）和Codex端Superpowers（执行+审查）。当用户请求完整的开发流程或调用 /sparkcode-dev-superpower 时使用。
---

# SparkCode Dev Superpower

**版本**: v1.2
**更新日期**: 2026-01-19

## 概述

这个技能协调Claude和Codex，实现端到端的开发工作流：
- **Claude负责**: 需求理解、设计、规划、分支管理、任务协调
- **Codex负责**: 代码实现、测试、代码审查
- **前端支持**: 自动集成 ui-ux-pro-max 技能处理前端开发
- **上下文传递**: 通过文件机制在多次Codex调用间保持状态连续性

## 前置条件

使用此技能前，请确保：
1. Codex端已安装Superpowers
2. Codex端已安装 ui-ux-pro-max 技能（用于前端开发）
3. 工作在共享的git仓库中
4. 已创建 `.sparkcode/logs/` 目录

## 工作流程

本技能包含5个阶段，按顺序执行。

### 阶段1: 需求理解与设计

**目标**: 通过提问理解用户需求，探讨技术方案

**执行**:

调用 @superpowers:brainstorming 技能：

```
使用 Skill 工具调用 superpowers:brainstorming
传递用户的需求描述作为参数
```

**输出**: 设计文档和需求澄清

**下一步**: 用户确认设计后，进入阶段2

### 阶段2: 编写实现计划

**目标**: 基于设计生成详细的实现计划，并识别前端需求

**执行**:

调用 @superpowers:writing-plans 技能，并遵循以下指导：

**前端需求检测**:

在分析需求时，检查是否包含以下特征：
- **UI/UX元素**: 用户界面、页面、组件、布局
- **前端技术**: React、Vue、HTML/CSS、JavaScript框架
- **交互功能**: 表单、按钮、动画、响应式设计
- **视觉设计**: 样式、主题、品牌元素

**计划编写规则**:

1. **识别前端任务**: 将涉及UI/UX的任务单独标识
2. **明确技能调用**: 在前端任务描述中写明：
   ```
   Task X: 实现用户登录界面
   - 使用 @ui-ux-pro-max 技能实现前端界面
   - 包含：登录表单、样式设计、交互逻辑
   ```
3. **后端任务保持原样**: 不涉及前端的任务按常规方式描述

**输出**: `docs/plans/YYYY-MM-DD-<topic>-plan.md`

**验证**:
- 计划文件已创建
- 包含任务分解（3-8个任务）
- 每个任务预估时间2-5分钟
- 前端任务已明确标注使用 @ui-ux-pro-max

**下一步**: 用户确认计划后，进入阶段3

## 前端需求识别指南

在阶段2（编写计划）时，使用以下规则判断是否需要前端开发：

### 明确的前端指标

**关键词识别**:
- UI/UX相关：界面、页面、组件、布局、视图
- 交互相关：表单、按钮、菜单、导航、弹窗
- 视觉相关：样式、主题、颜色、字体、动画
- 设备相关：响应式、移动端、桌面端、适配

**技术栈识别**:
- 前端框架：React、Vue、Angular、Svelte
- 标记语言：HTML、JSX、模板
- 样式技术：CSS、Sass、Tailwind、styled-components
- 前端库：UI组件库、图表库、动画库

**功能特征**:
- 用户可见的界面元素
- 浏览器中运行的交互逻辑
- 客户端状态管理
- 前端路由和导航

### 任务标注规范

**前端任务模板**:
```
Task X: [任务名称]
**使用 @ui-ux-pro-max 技能实现前端界面**

- [具体实现内容]
- [设计要求]
- [交互逻辑]

验收标准: [前端相关的验收标准]
```

**混合任务处理**:
如果一个任务同时涉及前端和后端，拆分成两个独立任务：
- Task X: 实现前端界面（使用 @ui-ux-pro-max）
- Task X+1: 实现后端逻辑

### 边界情况

**不算前端的情况**:
- 纯API开发（无界面）
- 命令行工具（CLI）
- 后台脚本和定时任务
- 数据处理和分析脚本

### 阶段3: 代码实现（Codex执行）

**目标**: 通过文件上下文传递机制，逐个调用Codex执行计划中的任务

**核心机制**: 使用 `.sparkcode/context.json` 作为 Claude 和 Codex 之间的上下文传递媒介，每次调用 Codex 执行单个任务，通过文件保持状态连续性。

#### 步骤3.1: 初始化上下文环境

创建目录结构和上下文文件：

```bash
# 创建目录结构
mkdir -p .sparkcode/task-outputs
mkdir -p .sparkcode/logs

# 初始化 decisions.md
cat > .sparkcode/decisions.md <<'EOF'
# 设计决策记录

本文件记录开发过程中的重要设计决策，供后续任务参考。

---
EOF
```

#### 步骤3.2: 解析计划文件生成任务列表

从计划文件中提取任务，生成 `context.json`：

```bash
plan_file="docs/plans/2026-01-19-feature-x-plan.md"

# Claude 需要解析计划文件，提取所有 Task，生成如下结构：
cat > .sparkcode/context.json <<'EOF'
{
  "session_id": "uuid-here",
  "plan_file": "docs/plans/2026-01-19-feature-x-plan.md",

  "tasks": [
    {"id": 1, "name": "创建HTML结构", "status": "pending"},
    {"id": 2, "name": "实现CSS样式", "status": "pending"},
    {"id": 3, "name": "添加JS交互", "status": "pending"}
  ],

  "current_task": 1,
  "retry_count": 0,

  "files_created": [],

  "last_error": null,
  "last_updated": "2026-01-19T10:00:00Z"
}
EOF
```

#### 步骤3.3: 单任务执行循环

**循环逻辑**（Claude 端执行）：

```
while (context.json 中有 pending 任务):
    1. 读取 context.json 获取 current_task
    2. 调用 Codex 执行当前任务
    3. 解析 Codex 输出:
       - 如果包含 TASK_COMPLETED → 继续下一个任务
       - 如果包含 TASK_FAILED:
           - retry_count < 1 → 更新 retry_count，重试当前任务
           - retry_count >= 1 → 询问用户（重试/跳过/终止）
    4. 读取更新后的 context.json
    5. 向用户报告进度
```

**Codex 调用命令**：

```bash
codex exec --full-auto --skip-git-repo-check \
"你正在执行一个多步骤开发任务。

【第一步：读取上下文】
1. 读取 .sparkcode/context.json 了解当前状态和任务列表
2. 读取 .sparkcode/decisions.md 了解之前的设计决策（如存在）
3. 读取最近的任务输出 .sparkcode/task-outputs/task-{N-1}.md（如存在）
4. 根据需要读取 files_created 中的相关文件

【第二步：执行当前任务】
执行 context.json 中 current_task 对应的任务
如果任务标注使用 @ui-ux-pro-max，调用该技能实现前端

【第三步：更新状态】
1. 更新 context.json:
   - 当前任务 status 改为 completed
   - current_task 加 1（如果还有下一个任务）
   - 更新 files_created（如有新文件）
   - 清空 last_error
   - 更新 last_updated
2. 创建 .sparkcode/task-outputs/task-{当前任务ID}.md，记录：
   - 完成了什么
   - 关键设计决策
   - 创建/修改的文件
   - 后续任务需要注意的事项
3. 如有重要设计决策，追加到 .sparkcode/decisions.md

【输出要求】
完成后输出：
- TASK_COMPLETED 或 TASK_FAILED
- 简要说明完成内容或失败原因"
```

#### 步骤3.4: 错误处理与重试

**自动重试逻辑**：

```
如果 Codex 输出包含 TASK_FAILED:
    读取 context.json 的 retry_count

    如果 retry_count < 1:
        更新 context.json: retry_count = retry_count + 1
        重新调用 Codex 执行同一任务

    否则:
        使用 AskUserQuestion 询问用户:
        选项:
          a) 再次重试
          b) 跳过此任务，继续下一个
          c) 终止执行

        根据用户选择:
          a) 重置 retry_count = 0，重试
          b) 标记任务为 skipped，current_task + 1
          c) 结束阶段3，进入错误恢复流程
```

#### 步骤3.5: 进度报告

每完成一个任务后，向用户报告：

```
任务进度: [3/5] ████████████░░░░░░░░ 60%

✓ Task 1: 创建HTML结构
✓ Task 2: 实现CSS样式
▶ Task 3: 添加JS交互 (执行中)
○ Task 4: 集成API
○ Task 5: 添加测试

新增文件: index.html, styles.css
```

#### 步骤3.6: 完成检查

所有任务完成后：

```bash
# 读取最终的 context.json
# 检查是否所有任务都是 completed 状态
# 汇总执行结果

echo "阶段3完成"
echo "- 完成任务: X/Y"
echo "- 跳过任务: Z"
echo "- 创建文件: file1, file2, ..."
```

**输出**: 实现的代码 + 执行摘要

**下一步**: 所有任务完成后，进入阶段4

### 阶段4: 代码审查（Codex审查）

**目标**: 调用Codex审查未提交的代码变更

**步骤4.1: 调用Codex审查**

使用 Bash 工具执行：

```bash
codex exec --full-auto --skip-git-repo-check \
  "Run ~/.codex/superpowers/.codex/superpowers-codex \
  use-skill superpowers:requesting-code-review"
```

**步骤4.2: 解析审查报告**

从Codex输出中提取：
- **Critical**: 严重问题（安全漏洞、逻辑错误）
- **Major**: 重要问题（性能问题、代码质量）
- **Minor**: 次要问题（代码风格、命名）
- **Suggestions**: 改进建议

**步骤4.3: 向用户展示结果**

```
审查结果:
  ✗ Critical: X
  ⚠ Major: Y
  ℹ Minor: Z
  💡 Suggestions: W

选项:
  ✓ 通过，进入分支完成阶段
  🔧 修复问题后重新审查
  👀 查看详细报告
```

**决策逻辑**:
- 如果有Critical问题: 强烈建议修复后再继续
- 如果只有Major/Minor问题: 可以选择继续或修复
- 如果无问题: 自动进入下一阶段

**输出**: 审查报告

**下一步**: 用户确认后，进入阶段5

### 阶段5: 分支完成

**目标**: 完成开发工作，决定如何集成代码

**执行**:

调用 @superpowers:finishing-a-development-branch 技能：

```
使用 Skill 工具调用 superpowers:finishing-a-development-branch
```

**流程**:
1. 运行最终测试验证
2. 展示变更摘要
3. 提供选项:
   - Merge到主分支
   - 创建Pull Request
   - 清理分支

**输出**: 完成的功能分支

**完成**: 工作流结束

## 辅助函数

### 初始化上下文环境

```bash
init_sparkcode_context() {
  local plan_file="$1"

  # 创建目录结构
  mkdir -p .sparkcode/task-outputs
  mkdir -p .sparkcode/logs

  # 初始化 decisions.md
  cat > .sparkcode/decisions.md <<'EOF'
# 设计决策记录

本文件记录开发过程中的重要设计决策，供后续任务参考。

---
EOF

  echo "✓ 上下文环境初始化完成"
}
```

### 生成 context.json

Claude 解析计划文件后，生成 context.json：

```bash
# 示例：Claude 解析计划后生成
cat > .sparkcode/context.json <<'EOF'
{
  "session_id": "生成的UUID",
  "plan_file": "docs/plans/YYYY-MM-DD-feature-plan.md",
  "tasks": [
    {"id": 1, "name": "任务1名称", "status": "pending"},
    {"id": 2, "name": "任务2名称", "status": "pending"}
  ],
  "current_task": 1,
  "retry_count": 0,
  "files_created": [],
  "last_error": null,
  "last_updated": "ISO8601时间戳"
}
EOF
```

### 检查是否有待执行任务

```bash
has_pending_tasks() {
  local context_file=".sparkcode/context.json"

  if [ ! -f "$context_file" ]; then
    echo "false"
    return 1
  fi

  # 检查是否有 pending 状态的任务
  if grep -q '"status": "pending"' "$context_file"; then
    echo "true"
    return 0
  else
    echo "false"
    return 1
  fi
}
```

### 解析 Codex 输出

```bash
parse_codex_output() {
  local output="$1"

  # 检查成功标识
  if echo "$output" | grep -q "TASK_COMPLETED"; then
    echo "COMPLETED"
    return 0
  fi

  # 检查失败标识
  if echo "$output" | grep -q "TASK_FAILED"; then
    echo "FAILED"
    return 1
  fi

  echo "UNKNOWN"
  return 2
}
```

### 更新重试计数

```bash
update_retry_count() {
  local context_file=".sparkcode/context.json"
  local current_count=$(grep -o '"retry_count": [0-9]*' "$context_file" | grep -o '[0-9]*')
  local new_count=$((current_count + 1))

  # 使用 sed 更新 retry_count
  sed -i "s/\"retry_count\": $current_count/\"retry_count\": $new_count/" "$context_file"

  echo "$new_count"
}
```

### 生成进度报告

```bash
generate_progress_report() {
  local context_file=".sparkcode/context.json"

  # Claude 读取 context.json 并生成可视化进度报告
  # 示例输出格式见步骤3.5
}
```

## 最佳实践

### 1. 计划粒度控制

- 单个计划的任务数: 3-8个
- 每个任务的预估时间: 2-5分钟
- 总计划时间: 不超过30分钟
- 过大的计划应拆分成多个子计划

### 2. Codex执行监控

- 超时时间: 30分钟（可配置）
- 日志保存: 保存完整的执行日志到 `.sparkcode/logs/`
- 进度通知: 定期更新执行进度

### 3. 错误恢复策略

- 执行前创建git stash备份
- 实时更新状态文件
- 失败后提供清晰的恢复选项
- 支持回滚到执行前的状态

### 4. 性能优化

- 使用 `--full-auto` 减少交互等待
- 缓存常用的依赖和构建产物
- 支持从失败点继续执行

## 常见问题

### Q: Codex执行超时怎么办？

A:
1. 检查 `.sparkcode/logs/` 中的日志
2. 确认Codex进程状态
3. 如果任务过大，考虑拆分计划
4. 可以手动终止并从断点继续

### Q: 如何调试Codex调用失败？

A:
1. 检查Codex是否已安装Superpowers
2. 验证命令格式是否正确
3. 查看 `.sparkcode/logs/` 中的错误日志
4. 尝试手动执行Codex命令

### Q: 计划文件格式不兼容怎么办？

A:
1. 确保两端Superpowers版本一致
2. 检查计划文件是否符合标准格式
3. 参考设计文档中的计划文件格式示例

## 示例用法

### 完整工作流示例

```
用户: /sparkcode-dev-superpower 实现用户认证功能

Claude: 我将使用 sparkcode-dev-superpower 技能来完成这个任务。

[阶段1: 需求理解与设计]
Claude: 让我先了解一下需求...
(调用 superpowers:brainstorming)

[阶段2: 编写实现计划]
Claude: 基于设计，我来创建实现计划...
(调用 superpowers:writing-plans)
输出: docs/plans/2026-01-19-user-auth-plan.md

[阶段3: 代码实现]
Claude: 现在调用Codex执行计划...
(执行 codex exec --full-auto --prompt "...")
✓ Codex执行成功
  - 完成任务: 5/5
  - 通过测试: 12/12

[阶段4: 代码审查]
Claude: 让Codex审查代码...
(执行 codex exec --full-auto --prompt "...")
审查结果:
  ✗ Critical: 0
  ⚠ Major: 1
  ℹ Minor: 3

[阶段5: 分支完成]
Claude: 准备完成分支...
(调用 superpowers:finishing-a-development-branch)
```

### 前端项目示例

```
用户: /sparkcode-dev-superpower 创建一个Todo应用的前端界面

Claude: 我将使用 sparkcode-dev-superpower 技能来完成这个任务。

[阶段1: 需求理解与设计]
Claude: 让我先了解一下需求...
(调用 superpowers:brainstorming)

[阶段2: 编写实现计划]
Claude: 基于设计，我来创建实现计划...
检测到前端需求，将在计划中标注使用 @ui-ux-pro-max
(调用 superpowers:writing-plans)
输出: docs/plans/2026-01-19-todo-app-frontend-plan.md

计划内容示例：
Task 1: 实现Todo列表界面
**使用 @ui-ux-pro-max 技能实现前端界面**
- 创建Todo列表组件
- 设计卡片样式
- 实现添加/删除/完成功能

[阶段3: 代码实现]
Claude: 现在调用Codex执行计划...
(执行 codex exec --full-auto --prompt "...")
Codex识别到 @ui-ux-pro-max 标记，自动调用该技能
✓ Codex执行成功
  - 完成任务: 3/3
  - 前端界面已实现，样式精美

[阶段4: 代码审查]
Claude: 让Codex审查代码...
(执行 codex exec --full-auto --prompt "...")
审查结果:
  ✗ Critical: 0
  ⚠ Major: 0
  ℹ Minor: 2

[阶段5: 分支完成]
Claude: 准备完成分支...
(调用 superpowers:finishing-a-development-branch)
```

### 前后端混合项目示例

```
用户: /sparkcode-dev-superpower 实现用户管理系统（包含界面和API）

[阶段2生成的计划包含]：
Task 1: 实现用户列表界面
**使用 @ui-ux-pro-max 技能实现前端界面**
- 用户列表组件
- 搜索和筛选功能
- 分页控件

Task 2: 实现用户管理API
- CRUD端点
- 数据验证
- 错误处理

Task 3: 集成前后端
- API调用
- 状态管理
- 错误处理
```

## 技术规格

### 文件结构

```
.sparkcode/
├── context.json          # 状态和任务列表
├── decisions.md          # 设计决策记录
├── logs/                 # 执行日志
└── task-outputs/         # 每个任务的输出摘要
    ├── task-1.md
    ├── task-2.md
    └── ...
```

### context.json 格式

```json
{
  "session_id": "uuid",
  "plan_file": "docs/plans/YYYY-MM-DD-<topic>-plan.md",

  "tasks": [
    {"id": 1, "name": "任务名称", "status": "completed"},
    {"id": 2, "name": "任务名称", "status": "in_progress"},
    {"id": 3, "name": "任务名称", "status": "pending"},
    {"id": 4, "name": "任务名称", "status": "skipped"}
  ],

  "current_task": 2,
  "retry_count": 0,

  "files_created": [
    "index.html",
    "styles.css"
  ],

  "last_error": null,
  "last_updated": "ISO8601 timestamp"
}
```

**任务状态说明**:
- `pending`: 待执行
- `in_progress`: 执行中（Codex 正在处理）
- `completed`: 已完成
- `skipped`: 已跳过（用户选择跳过失败任务）

### task-outputs/task-X.md 格式

```markdown
# Task X: 任务名称

## 完成内容
- 具体完成了什么

## 关键设计决策
- 为什么这样实现
- 选择了什么技术方案

## 创建/修改的文件
- file1.js (新建)
- file2.css (修改)

## 后续任务注意
- 下一个任务需要知道的信息
```

### Codex 命令格式

**单任务执行**:
```bash
codex exec --full-auto --skip-git-repo-check \
"你正在执行一个多步骤开发任务。

【第一步：读取上下文】
1. 读取 .sparkcode/context.json 了解当前状态和任务列表
2. 读取 .sparkcode/decisions.md 了解之前的设计决策（如存在）
3. 读取最近的任务输出 .sparkcode/task-outputs/task-{N-1}.md（如存在）
4. 根据需要读取 files_created 中的相关文件

【第二步：执行当前任务】
执行 context.json 中 current_task 对应的任务
如果任务标注使用 @ui-ux-pro-max，调用该技能实现前端

【第三步：更新状态】
1. 更新 context.json（任务状态、files_created、last_updated）
2. 创建 .sparkcode/task-outputs/task-{当前任务ID}.md
3. 如有重要设计决策，追加到 .sparkcode/decisions.md

【输出要求】
完成后输出：TASK_COMPLETED 或 TASK_FAILED"
```

**代码审查**:
```bash
codex exec --full-auto --skip-git-repo-check \
"审查 .sparkcode/context.json 中 files_created 列出的所有文件，
检查代码质量、安全性、最佳实践。
输出审查报告，包含 Critical/Major/Minor/Suggestions 分类。"
```

### 输出解析规则

**成功模式**:
- 输出包含: `TASK_COMPLETED`

**失败模式**:
- 输出包含: `TASK_FAILED`

## 集成说明

### 与其他技能的关系

- **依赖**: superpowers:brainstorming, superpowers:writing-plans, superpowers:finishing-a-development-branch
- **调用**: Codex端的 superpowers:executing-plans, superpowers:requesting-code-review
- **配合**: superpowers:using-git-worktrees (用于创建隔离工作空间)

### 环境要求

1. **Claude端**:
   - 已安装Superpowers
   - 可以使用Bash工具
   - 可以使用Skill工具

2. **Codex端**:
   - 已安装Superpowers
   - 可以通过 `codex exec` 调用
   - 与Claude共享git工作目录

3. **共享环境**:
   - git仓库
   - `.sparkcode/logs/` 目录
   - `docs/plans/` 目录

## 版本历史

### v1.2 (2026-01-19)
- ✨ 重构阶段3：采用文件上下文传递机制（方案C）
- ✅ 单任务执行循环，避免上下文丢失
- ✅ 新增 context.json 状态管理
- ✅ 新增 task-outputs/ 任务输出记录
- ✅ 新增 decisions.md 设计决策追踪
- ✅ 自动重试机制（失败自动重试1次）
- ✅ 用户可选择重试/跳过/终止
- ✅ 进度可视化报告
- ✅ 支持断点续传

### v1.1 (2026-01-19)
- ✨ 添加前端集成支持
- ✅ 自动检测前端需求
- ✅ 集成 ui-ux-pro-max 技能
- ✅ 添加前端需求识别指南
- ✅ 更新文档和示例

### v1.0 (2026-01-19)
- ✨ 初始版本发布
- ✅ 实现5阶段工作流
- ✅ 支持Codex集成
- ✅ 包含错误处理和状态管理
- ✅ 完整的文档和示例
