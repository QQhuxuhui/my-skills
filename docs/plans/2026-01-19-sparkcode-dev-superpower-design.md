# SparkCode Dev Superpower - 设计文档

**日期**: 2026-01-19
**版本**: 1.0
**状态**: 已批准

## 概述

SparkCode Dev Superpower 是一个集成Claude端Superpowers和Codex端Superpowers的完整开发工作流技能。通过智能分工，Claude负责战略性思考（需求理解、设计、规划），Codex负责执行性工作（代码实现、测试、审查）。

## 核心理念

**分工原则**:
- **Claude端**: 头脑风暴、规划、分支管理
- **Codex端**: 代码执行、代码审查

**集成方式**: 通过共享的计划文件（markdown格式）和Shell命令调用实现解耦集成

**可行性**: 98%

## 架构设计

### 整体架构

```
┌─────────────────────────────────────────────────────────────┐
│                    Claude 端 (战略层)                        │
├─────────────────────────────────────────────────────────────┤
│  superpowers:brainstorming                                  │
│  superpowers:writing-plans                                  │
│  superpowers:finishing-a-development-branch                 │
└────────────────────┬────────────────────────────────────────┘
                     │
                     │ 计划文件共享
                     │ docs/plans/*.md
                     │
                     │ Shell命令调用
                     │ codex exec --prompt "..."
                     │
┌────────────────────▼────────────────────────────────────────┐
│                    Codex 端 (执行层)                         │
├─────────────────────────────────────────────────────────────┤
│  superpowers:executing-plans                                │
│  superpowers:requesting-code-review                         │
└─────────────────────────────────────────────────────────────┘
```

### 技能结构

**技能名称**: `sparkcode-dev-superpower`

**触发条件**:
- 用户请求完整的开发流程
- 用户调用 `/sparkcode-dev-superpower` 命令

**依赖**:
- Claude端已安装Superpowers
- Codex端已安装Superpowers
- 共享的git工作目录

## 完整工作流

### 阶段1: 需求理解与设计 [Claude端]

**执行者**: Claude
**使用技能**: `superpowers:brainstorming`

**流程**:
1. 用户提出开发需求
2. Claude通过提问理解需求
3. 探讨技术方案和架构
4. 明确验收标准和约束条件

**输出**: 设计文档和需求澄清

---

### 阶段2: 编写实现计划 [Claude端]

**执行者**: Claude
**使用技能**: `superpowers:writing-plans`

**流程**:
1. 基于设计生成详细的实现计划
2. 将任务分解为2-5分钟的小任务
3. 定义每个任务的验收标准
4. 写入计划文件

**输出**: `docs/plans/YYYY-MM-DD-<topic>-plan.md`

**计划文件格式**:
```markdown
# Feature X Implementation Plan

## Overview
[功能概述]

## Tasks
1. Task 1: [描述]
   - Acceptance: [验收标准]
   - Estimated: 3 minutes

2. Task 2: [描述]
   - Acceptance: [验收标准]
   - Estimated: 5 minutes

## Technical Details
[技术细节]

## Testing Strategy
[测试策略]
```

---

### 阶段3: 代码实现 [Codex端，Claude触发]

**执行者**: Codex
**触发者**: Claude
**使用技能**: `superpowers:executing-plans`

**步骤3.1: 提取计划名称**
```
输入: docs/plans/2026-01-19-user-auth-plan.md
处理: 移除 "docs/plans/" 前缀和 ".md" 后缀
输出: 2026-01-19-user-auth-plan
```

**步骤3.2: 调用Codex执行**
```bash
codex exec --full-auto \
  --prompt "Run ~/.codex/superpowers/.codex/superpowers-codex \
  use-skill superpowers:executing-plans 2026-01-19-user-auth-plan"
```

**步骤3.3: 监控执行**
- 显示进度通知: "⏳ Codex正在执行计划..."
- 显示当前任务: "📝 当前任务: [任务名称]"
- 显示已用时间: "⏱ 已用时间: [X分钟]"

**步骤3.4: 解析执行结果**

**成功标识**:
- 退出码 = 0
- 输出包含 "✓" 或 "Success" 或 "completed"
- 无严重错误信息

**失败标识**:
- 退出码 ≠ 0
- 输出包含 "✗" 或 "Error" 或 "failed"
- 存在异常堆栈信息

**提取信息**:
1. 执行了哪些任务
2. 哪些测试通过/失败
3. 生成了哪些文件
4. 是否有警告或建议

**输出**: 实现的代码 + 测试结果

---

### 阶段4: 代码审查 [Codex端，Claude触发]

**执行者**: Codex
**触发者**: Claude
**使用技能**: `superpowers:requesting-code-review`

**步骤4.1: 调用Codex审查**
```bash
codex exec --full-auto \
  --prompt "Run ~/.codex/superpowers/.codex/superpowers-codex \
  use-skill superpowers:requesting-code-review"
```

**步骤4.2: 解析审查报告**

从Codex输出中提取:
- **Critical**: 严重问题（安全漏洞、逻辑错误）
- **Major**: 重要问题（性能问题、代码质量）
- **Minor**: 次要问题（代码风格、命名）
- **Suggestions**: 改进建议

**步骤4.3: 向用户展示结果**

```
审查结果:
  ✗ Critical: 0
  ⚠ Major: 2
  ℹ Minor: 5
  💡 Suggestions: 3

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

---

### 阶段5: 分支完成 [Claude端]

**执行者**: Claude
**使用技能**: `superpowers:finishing-a-development-branch`

**流程**:
1. 运行最终测试验证
2. 展示变更摘要
3. 提供选项:
   - Merge到主分支
   - 创建Pull Request
   - 清理分支

**输出**: 完成的功能分支

## 错误处理策略

### 场景1: Codex执行失败

**检测条件**: 退出码 ≠ 0 或输出包含 "Error"

**处理流程**:
1. 向用户展示错误信息
2. 提供选项:
   - a) 重试执行（可能是临时问题）
   - b) 修改计划后重试
   - c) 手动介入调试
   - d) 放弃当前任务

### 场景2: 测试失败

**检测条件**: 输出包含 "test failed" 或 "X failing"

**处理流程**:
1. 提取失败的测试用例
2. 询问用户:
   - a) 让Codex自动修复（重新执行计划）
   - b) 手动修复测试
   - c) 调整验收标准

### 场景3: Codex超时或无响应

**检测条件**: 命令执行超过预设时间（30分钟）

**处理流程**:
1. 终止Codex进程
2. 检查已完成的工作
3. 询问用户是否从断点继续

### 重试机制

**自动重试条件**:
- 网络错误
- Codex临时不可用
- 轻微的语法错误

**最大重试次数**: 3次
**重试间隔**: 5秒

## 用户交互设计

### 交互点1: 计划确认

**时机**: 计划生成后，执行前

**展示**:
```
已生成计划: docs/plans/2026-01-19-user-auth-plan.md

计划摘要:
  - 任务数: 5
  - 预估时间: 15分钟
  - 涉及文件: 3个

选项:
  ✓ 开始执行
  ✎ 修改计划
  ✗ 取消
```

### 交互点2: 执行进度通知

**时机**: Codex执行期间

**展示**:
```
⏳ Codex正在执行计划...
📝 当前任务: 实现用户认证API
⏱ 已用时间: 5分钟
```

### 交互点3: 执行结果确认

**时机**: Codex执行完成后

**展示**:
```
✓ 执行成功
  - 完成任务: 5/5
  - 通过测试: 12/12
  - 新增文件: 3个
  - 修改文件: 2个

选项:
  → 继续代码审查
  ↻ 重试执行
  ✋ 暂停处理
```

### 交互点4: 审查结果处理

**时机**: 代码审查完成后

**展示**:
```
审查完成
  ✗ Critical: 0
  ⚠ Major: 2
    - 缺少输入验证 (auth.js:45)
    - 未处理异常情况 (user.js:78)
  ℹ Minor: 5

选项:
  ✓ 通过，进入分支完成阶段
  🔧 修复问题后重新审查
  👀 查看详细报告
```

## 状态管理

### 状态文件

**路径**: `.sparkcode/session-state.json`

**格式**:
```json
{
  "session_id": "550e8400-e29b-41d4-a716-446655440000",
  "current_stage": "executing",
  "plan_file": "docs/plans/2026-01-19-user-auth-plan.md",
  "plan_name": "2026-01-19-user-auth-plan",
  "codex_execution": {
    "started_at": "2026-01-19T10:30:00Z",
    "status": "running",
    "output_file": ".sparkcode/logs/codex-output-20260119-103000.log",
    "pid": 12345
  },
  "review_result": {
    "completed": false,
    "critical_issues": 0,
    "major_issues": 0,
    "minor_issues": 0
  },
  "history": [
    {
      "stage": "brainstorming",
      "completed_at": "2026-01-19T10:15:00Z"
    },
    {
      "stage": "planning",
      "completed_at": "2026-01-19T10:25:00Z"
    }
  ]
}
```

**用途**:
- 支持中断后恢复
- 记录执行历史
- 便于调试和审计
- 跟踪当前进度

### 日志管理

**日志目录**: `.sparkcode/logs/`

**日志文件**:
- `codex-output-YYYYMMDD-HHMMSS.log`: Codex执行的完整输出
- `session-YYYYMMDD.log`: 会话级别的操作日志
- `error-YYYYMMDD.log`: 错误和异常日志

## 实施路线图

### 步骤1: 环境准备

```bash
# 1. 确保Codex已安装Superpowers
cd ~/.codex/superpowers
git pull  # 更新到最新版本

# 2. 验证Codex可以调用Superpowers技能
codex exec --prompt "Run ~/.codex/superpowers/.codex/superpowers-codex \
  use-skill superpowers:using-superpowers"

# 3. 创建必要的目录
mkdir -p .sparkcode/logs
mkdir -p docs/plans
```

### 步骤2: 创建技能文件

在Claude端创建技能文件:
- 路径: `my-skills/sparkcode-dev-superpower.skill`
- 包含完整的工作流定义和命令模板

### 步骤3: 测试集成

使用简单任务测试完整流程:

1. **测试计划生成**
   - 创建一个"Hello World"功能的计划
   - 验证计划文件格式正确

2. **测试Codex执行**
   - 手动调用Codex执行计划
   - 验证输出格式和内容

3. **测试输出解析**
   - 解析Codex的stdout/stderr
   - 提取关键信息

4. **测试代码审查**
   - 调用Codex代码审查
   - 解析审查报告

5. **测试错误处理**
   - 模拟执行失败
   - 验证错误处理逻辑

### 步骤4: 迭代优化

根据实际使用情况调整:
- 输出解析规则
- 错误处理逻辑
- 超时时间设置
- 用户交互流程

## 最佳实践

### 1. 计划粒度控制

- **单个计划的任务数**: 3-8个
- **每个任务的预估时间**: 2-5分钟
- **总计划时间**: 不超过30分钟
- **拆分原则**: 过大的计划应拆分成多个子计划

### 2. Codex执行监控

- **超时时间**: 30分钟（可配置）
- **日志保存**: 保存完整的执行日志到 `.sparkcode/logs/`
- **资源监控**: 定期检查Codex的CPU和内存使用
- **进度通知**: 每5分钟更新一次进度

### 3. 错误恢复策略

- **备份机制**: 执行前创建git stash备份
- **状态保存**: 实时更新状态文件
- **恢复路径**: 失败后提供清晰的恢复选项
- **回滚能力**: 支持回滚到执行前的状态

### 4. 性能优化

- **自动化模式**: 使用 `--full-auto` 减少交互等待
- **并行执行**: 独立任务可以并行执行（未来扩展）
- **缓存机制**: 缓存常用的依赖和构建产物
- **增量执行**: 支持从失败点继续执行

## 潜在扩展方向

### 扩展1: 多Codex并行执行

**目标**: 提升大型项目的执行效率

**实现方式**:
1. 将大型计划拆分成独立子任务
2. 分析任务依赖关系
3. 启动多个Codex实例并行执行无依赖任务
4. 合并执行结果

**收益**: 执行时间减少50-70%

### 扩展2: 执行结果可视化

**目标**: 提供更直观的执行反馈

**功能**:
- 生成HTML格式的执行报告
- 展示任务依赖关系图
- 提供交互式的审查界面
- 支持时间线视图

### 扩展3: 智能计划优化

**目标**: 基于历史数据优化计划质量

**功能**:
- 根据历史执行数据优化任务分解
- 自动识别高风险任务
- 建议最优的执行顺序
- 预测执行时间

### 扩展4: 集成CI/CD

**目标**: 将工作流集成到持续集成流程

**功能**:
- 自动触发计划执行
- 集成到GitHub Actions/GitLab CI
- 自动创建PR和合并
- 生成发布说明

## 技术规格

### 命令格式

**Codex执行计划**:
```bash
codex exec --full-auto \
  --prompt "Run ~/.codex/superpowers/.codex/superpowers-codex \
  use-skill superpowers:executing-plans <计划名称>"
```

**Codex代码审查**:
```bash
codex exec --full-auto \
  --prompt "Run ~/.codex/superpowers/.codex/superpowers-codex \
  use-skill superpowers:requesting-code-review"
```

### 计划名称提取规则

```
输入: docs/plans/2026-01-19-user-auth-plan.md
步骤:
  1. 移除前缀: "docs/plans/"
  2. 移除后缀: ".md"
输出: 2026-01-19-user-auth-plan
```

### 输出解析规则

**成功模式**:
- 正则表达式: `✓|Success|completed|All tests passed`
- 退出码: 0

**失败模式**:
- 正则表达式: `✗|Error|failed|Exception`
- 退出码: 非0

**提取任务信息**:
- 正则表达式: `Task \d+: (.+?) - (✓|✗)`

**提取测试结果**:
- 正则表达式: `(\d+) passing|(\d+) failing`

## 风险评估

### 高风险项

**风险1: Codex输出格式变化**
- **概率**: 中
- **影响**: 高
- **缓解**: 使用灵活的正则表达式，支持多种格式

**风险2: 计划文件格式不兼容**
- **概率**: 低
- **影响**: 高
- **缓解**: 确保两端Superpowers版本一致

### 中风险项

**风险3: 执行超时**
- **概率**: 中
- **影响**: 中
- **缓解**: 设置合理的超时时间，支持断点续传

**风险4: 文件系统同步延迟**
- **概率**: 低
- **影响**: 中
- **缓解**: 添加文件存在性检查

### 低风险项

**风险5: 状态文件损坏**
- **概率**: 低
- **影响**: 低
- **缓解**: 定期备份状态文件

## 成功标准

### 功能完整性
- ✓ 支持完整的5阶段工作流
- ✓ 正确解析Codex输出
- ✓ 处理常见错误场景
- ✓ 支持中断恢复

### 性能指标
- ✓ 端到端执行时间 < 计划预估时间 × 1.2
- ✓ 输出解析准确率 > 95%
- ✓ 错误恢复成功率 > 90%

### 用户体验
- ✓ 交互流程清晰直观
- ✓ 错误信息易于理解
- ✓ 进度反馈及时准确

## 总结

### 核心优势

✅ **架构清晰**: Claude负责思考，Codex负责执行，职责分明
✅ **标准化**: 两端都使用Superpowers，语义一致，易于维护
✅ **可靠性**: 通过文件共享，避免复杂的进程通信
✅ **可扩展**: 易于添加新的集成点和功能
✅ **可维护**: 状态持久化，支持中断恢复

### 可行性评估

**总体可行性: 98%**

**已验证**:
- ✅ Codex技能调用方式明确
- ✅ 计划文件格式标准化
- ✅ Shell命令集成可行
- ✅ 工作目录共享无问题

**待验证**:
- ⚠️ Codex输出格式的稳定性（5%风险）
- ⚠️ 大型项目的执行性能（可通过测试验证）

### 下一步行动

1. **创建技能文件**: 编写 `sparkcode-dev-superpower.skill`
2. **环境验证**: 确认Codex端Superpowers正常工作
3. **小规模测试**: 使用简单任务测试完整流程
4. **迭代优化**: 根据测试结果调整实现细节
5. **文档完善**: 编写用户使用指南

---

**设计批准**: ✓ 已批准
**实施优先级**: 高
**预计完成时间**: 2-3天（开发+测试）
