---
name: sparkcode-dev-superpower
description: 完整的软件开发工作流，集成Claude端Superpowers（设计+规划）和Codex端Superpowers（执行+审查）。当用户请求完整的开发流程或调用 /sparkcode-dev-superpower 时使用。
---

# SparkCode Dev Superpower

## 概述

这个技能协调Claude和Codex，实现端到端的开发工作流：
- **Claude负责**: 需求理解、设计、规划、分支管理
- **Codex负责**: 代码实现、测试、代码审查

## 前置条件

使用此技能前，请确保：
1. Codex端已安装Superpowers
2. 工作在共享的git仓库中
3. 已创建 `.sparkcode/logs/` 目录

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

### 阶段3: 代码实现（Codex执行）

**目标**: 调用Codex执行计划中的所有任务

**步骤3.1: 提取计划名称**

从计划文件路径提取名称：

```bash
# 输入: docs/plans/2026-01-19-feature-x-plan.md
# 输出: 2026-01-19-feature-x-plan

plan_file="docs/plans/2026-01-19-feature-x-plan.md"
plan_name=$(basename "$plan_file" .md)
echo "计划名称: $plan_name"
```

**步骤3.2: 创建状态文件**

```bash
mkdir -p .sparkcode/logs
cat > .sparkcode/session-state.json <<EOF
{
  "session_id": "$(uuidgen)",
  "current_stage": "executing",
  "plan_file": "$plan_file",
  "plan_name": "$plan_name",
  "codex_execution": {
    "started_at": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
    "status": "running"
  }
}
EOF
```

**步骤3.3: 调用Codex执行**

使用 Bash 工具执行：

```bash
codex exec --full-auto \
  --prompt "Run ~/.codex/superpowers/.codex/superpowers-codex \
  use-skill superpowers:executing-plans $plan_name"
```

**步骤3.4: 解析执行结果**

检查Codex的输出：

```bash
# 检查退出码
if [ $? -eq 0 ]; then
  echo "✓ Codex执行成功"
else
  echo "✗ Codex执行失败，退出码: $?"
fi
```

**成功标识**:
- 退出码 = 0
- 输出包含 "✓" 或 "Success" 或 "completed"

**失败标识**:
- 退出码 ≠ 0
- 输出包含 "✗" 或 "Error" 或 "failed"

**步骤3.5: 向用户展示结果**

提取并展示关键信息：
- 完成的任务数
- 通过的测试数
- 新增/修改的文件

**错误处理**:

如果执行失败：
1. 展示错误信息
2. 询问用户：
   - a) 重试执行
   - b) 修改计划后重试
   - c) 手动介入调试
   - d) 放弃当前任务

**输出**: 实现的代码 + 测试结果

**下一步**: 执行成功后，进入阶段4

### 阶段4: 代码审查（Codex审查）

**目标**: 调用Codex审查未提交的代码变更

**步骤4.1: 调用Codex审查**

使用 Bash 工具执行：

```bash
codex exec --full-auto \
  --prompt "Run ~/.codex/superpowers/.codex/superpowers-codex \
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

### 提取计划名称

```bash
extract_plan_name() {
  local plan_file="$1"
  basename "$plan_file" .md
}

# 使用示例
plan_name=$(extract_plan_name "docs/plans/2026-01-19-feature-x-plan.md")
echo "$plan_name"  # 输出: 2026-01-19-feature-x-plan
```

### 解析Codex输出

```bash
parse_codex_output() {
  local output="$1"

  # 检查成功标识
  if echo "$output" | grep -qE "✓|Success|completed"; then
    echo "SUCCESS"
    return 0
  fi

  # 检查失败标识
  if echo "$output" | grep -qE "✗|Error|failed"; then
    echo "FAILED"
    return 1
  fi

  echo "UNKNOWN"
  return 2
}
```

### 创建状态文件

```bash
create_state_file() {
  local plan_file="$1"
  local plan_name="$2"

  mkdir -p .sparkcode/logs

  cat > .sparkcode/session-state.json <<EOF
{
  "session_id": "$(uuidgen || echo "manual-$(date +%s)")",
  "current_stage": "executing",
  "plan_file": "$plan_file",
  "plan_name": "$plan_name",
  "codex_execution": {
    "started_at": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
    "status": "running"
  }
}
EOF
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

## 技术规格

### Codex命令格式

**执行计划**:
```bash
codex exec --full-auto \
  --prompt "Run ~/.codex/superpowers/.codex/superpowers-codex \
  use-skill superpowers:executing-plans <计划名称>"
```

**代码审查**:
```bash
codex exec --full-auto \
  --prompt "Run ~/.codex/superpowers/.codex/superpowers-codex \
  use-skill superpowers:requesting-code-review"
```

### 输出解析规则

**成功模式**:
- 正则表达式: `✓|Success|completed|All tests passed`
- 退出码: 0

**失败模式**:
- 正则表达式: `✗|Error|failed|Exception`
- 退出码: 非0

### 状态文件格式

路径: `.sparkcode/session-state.json`

```json
{
  "session_id": "uuid",
  "current_stage": "executing|reviewing|finishing",
  "plan_file": "docs/plans/YYYY-MM-DD-<topic>-plan.md",
  "plan_name": "YYYY-MM-DD-<topic>-plan",
  "codex_execution": {
    "started_at": "ISO8601 timestamp",
    "status": "running|completed|failed"
  }
}
```

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

- **v1.0** (2026-01-19): 初始版本
  - 实现5阶段工作流
  - 支持Codex集成
  - 包含错误处理和状态管理
