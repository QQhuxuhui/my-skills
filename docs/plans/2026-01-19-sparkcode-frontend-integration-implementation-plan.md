# SparkCode Dev Superpower 前端集成实施计划

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** 优化 sparkcode-dev-superpower 技能，添加 ui-ux-pro-max 前端集成支持，使其在检测到前端需求时自动在计划中标注使用该技能

**Architecture:** 修改技能文件的阶段2部分，添加前端检测逻辑和任务标注规则；新增"前端需求识别指南"章节；更新相关文档说明前端支持

**Tech Stack:** Markdown (技能定义)、Git (版本控制)

---

## Task 1: 更新阶段2 - 添加前端检测逻辑

**Files:**
- Modify: `sparkcode-dev-superpower/SKILL.md:42-62`

**Step 1: 读取当前的阶段2内容**

```bash
sed -n '42,62p' sparkcode-dev-superpower/SKILL.md
```

Expected: 显示当前阶段2的内容

**Step 2: 备份当前内容**

```bash
cp sparkcode-dev-superpower/SKILL.md sparkcode-dev-superpower/SKILL.md.backup
```

Expected: 创建备份文件

**Step 3: 替换阶段2内容**

使用 Edit 工具替换第42-62行的内容为优化后的版本：

```markdown
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

**下一步**: 用户确��计划后，进入阶段3
```

**Step 4: 验证修改**

```bash
sed -n '42,95p' sparkcode-dev-superpower/SKILL.md
```

Expected: 显示更新后的阶段2内容，包含前端检测逻辑

**Step 5: Commit**

```bash
git add sparkcode-dev-superpower/SKILL.md
git commit -m "feat: add frontend detection logic to stage 2"
```

---

## Task 2: 添加前端需求识别指南章节

**Files:**
- Modify: `sparkcode-dev-superpower/SKILL.md` (在阶段2后插入新章节)

**Step 1: 确定插入位置**

```bash
grep -n "### 阶段3" sparkcode-dev-superpower/SKILL.md
```

Expected: 找到阶段3的行号（应该在新的阶段2之后）

**Step 2: 准备新章节内容**

创建包含前端识别指南的内容（在阶段2和阶段3之间插入）

**Step 3: 插入新章节**

使用 Edit 工具在阶段2结束后、阶段3开始前插入以下内容：

```markdown

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

```

**Step 4: 验证插入**

```bash
grep -A 5 "## 前端需求识别指南" sparkcode-dev-superpower/SKILL.md
```

Expected: 显示新插入的章节标题和前几行内容

**Step 5: Commit**

```bash
git add sparkcode-dev-superpower/SKILL.md
git commit -m "feat: add frontend requirement identification guide"
```

---

## Task 3: 更新示例用法章节

**Files:**
- Modify: `sparkcode-dev-superpower/SKILL.md` (示例用法章节)

**Step 1: 找到示例用法章节**

```bash
grep -n "## 示例用法" sparkcode-dev-superpower/SKILL.md
```

Expected: 找到示例用法章节的行号

**Step 2: 添加前端项目示例**

在现有示例后添加新的前端项目示例：

```markdown

### 前端项目示例

\`\`\`
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
\`\`\`

### 前后端混合项目示例

\`\`\`
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
\`\`\`
```

**Step 3: 验证添加**

```bash
grep -A 10 "### 前端项目示例" sparkcode-dev-superpower/SKILL.md
```

Expected: 显示新添加的前端示例

**Step 4: Commit**

```bash
git add sparkcode-dev-superpower/SKILL.md
git commit -m "docs: add frontend project examples to skill"
```

---

## Task 4: 更新前置条件说明

**Files:**
- Modify: `sparkcode-dev-superpower/SKILL.md:14-19`

**Step 1: 读取当前前置条件**

```bash
sed -n '14,19p' sparkcode-dev-superpower/SKILL.md
```

Expected: 显示当前的前置条件列表

**Step 2: 添加 ui-ux-pro-max 前置条件**

使用 Edit 工具更新前置条件部分：

```markdown
## 前置条件

使用此技能前，请确保：
1. Codex端已安装Superpowers
2. Codex端已安装 ui-ux-pro-max 技能（用于前端开发）
3. 工作在共享的git仓库中
4. 已创建 `.sparkcode/logs/` 目录
```

**Step 3: 验证修改**

```bash
sed -n '14,22p' sparkcode-dev-superpower/SKILL.md
```

Expected: 显示更新后的前置条件，包含 ui-ux-pro-max

**Step 4: Commit**

```bash
git add sparkcode-dev-superpower/SKILL.md
git commit -m "docs: add ui-ux-pro-max to prerequisites"
```

---

## Task 5: 更新 README.md

**Files:**
- Modify: `README.md`

**Step 1: 读取当前 README**

```bash
cat README.md
```

Expected: 显示当前README内容

**Step 2: 更新工作流说明**

在"工作流"部分添加前端支持说明：

```markdown
### 4. 工作流

1. **需求理解**: Claude通过提问理解需求
2. **编写计划**: Claude生成详细实现计划，自动检测前端需求
3. **代码实现**: Codex执行计划中的任务
   - 前端任务：自动调用 ui-ux-pro-max 技能
   - 后端任务：使用标准开发流程
4. **代码审查**: Codex审查代码质量
5. **分支完成**: Claude协助完成分支
```

**Step 3: 更新前置条件**

在"准备Codex环境"部分添加：

```markdown
### 2. 准备Codex环境

```bash
# 确保Codex端已安装Superpowers
cd ~/.codex/superpowers
git pull

# 确保Codex端已安装 ui-ux-pro-max（用于前端开发）
# 如果未安装，请参考 ui-ux-pro-max 的安装文档
```
```

**Step 4: 验证修改**

```bash
grep -A 5 "ui-ux-pro-max" README.md
```

Expected: 显示包含 ui-ux-pro-max 的相关说明

**Step 5: Commit**

```bash
git add README.md
git commit -m "docs: update README with frontend support information"
```

---

## Task 6: 更新 USAGE-GUIDE.md

**Files:**
- Modify: `USAGE-GUIDE.md`

**Step 1: 添加前端开发场景章节**

在"使用场景"部分后添加新的"前端开发场景"章节：

```markdown
## 前端开发场景

### 场景4：纯前端项目

**用户输入：**
```
/sparkcode-dev-superpower 创建一个Todo应用的前端界面
```

**工作流：**

1. **阶段1：需求理解**
   - Claude询问UI/UX需求
   - 例如："需要什么样的视觉风格？"
   - 例如："支持哪些交互功能？"

2. **阶段2：编写计划**
   - Claude检测到前端需求
   - 生成的计划中所有任务都标注 `@ui-ux-pro-max`
   - 保存到 `docs/plans/2026-01-19-todo-app-frontend-plan.md`

3. **阶段3：Codex执行**
   - Codex读取计划，识别 `@ui-ux-pro-max` 标记
   - 自动调用 ui-ux-pro-max 技能实现前端
   - 生成精美的UI界面和交互逻辑

4. **阶段4：代码审查**
   - Codex审查前端代码质量
   - 检查响应式设计、可访问性等

5. **阶段5：完成分支**
   - 合并或创建PR

### 场景5：前后端混合项目

**用户输入：**
```
/sparkcode-dev-superpower 实现用户管理系统（包含界面和API）
```

**计划示例：**
```
Task 1: 实现用户列表界面
**使用 @ui-ux-pro-max 技能实现前端界面**
- 用户列表组件
- 搜索和筛选
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

**执行流程：**
- Task 1: Codex调用 ui-ux-pro-max 实现前端
- Task 2: Codex使用标准流程实现后端
- Task 3: Codex集成前后端

## 前端集成说明

### ui-ux-pro-max 技能的作用

**自动调用时机**：
- 当计划中的任务包含 `@ui-ux-pro-max` 标记时
- Codex会自动识别并调用该技能

**ui-ux-pro-max 的优势**：
- 生成高质量的UI设计
- 避免通用的AI美学
- 创造性的视觉方案
- 精致的交互细节

### 前端任务识别

**自动识别的关键词**：
- UI、界面、页面、组件
- 表单、按钮、菜单、导航
- 样式、主题、动画
- React、Vue、HTML/CSS

**手动调整**：
如果自动检测不准确，可以在阶段2确认计划时手动调整任务标注。
```

**Step 2: 验证添加**

```bash
grep -A 5 "## 前端开发场景" USAGE-GUIDE.md
```

Expected: 显示新添加的前端场景章节

**Step 3: Commit**

```bash
git add USAGE-GUIDE.md
git commit -m "docs: add frontend development scenarios to usage guide"
```

---

## Task 7: 更新 INSTALL.md

**Files:**
- Modify: `INSTALL.md`

**Step 1: 添加 ui-ux-pro-max 安装说明**

在"步骤2：准备Codex环境"部分添加：

```markdown
### 步骤2：准备Codex环境

```bash
# 1. 确保Codex已安装
which codex
# 应该输出: /path/to/codex

# 2. 确保Codex端已安装Superpowers
cd ~/.codex/superpowers
git pull  # 更新到最新版本

# 3. 确保Codex端已安装 ui-ux-pro-max（用于前端开发）
# 检查是否已安装
ls ~/.codex/skills/ | grep ui-ux-pro-max

# 如果未安装，请参考 ui-ux-pro-max 的安装文档
# 通常安装方式：
# cp -r /path/to/ui-ux-pro-max ~/.codex/skills/

# 4. 验证Codex可以调用Superpowers技能
codex exec --prompt "Run ~/.codex/superpowers/.codex/superpowers-codex use-skill superpowers:using-superpowers"
```
```

**Step 2: 更新前置条件检查**

在"前置条件检查"部分添加：

```markdown
# 3. 检查 ui-ux-pro-max 技能
ls ~/.codex/skills/ui-ux-pro-max/
# 应该看到 SKILL.md 文件
```

**Step 3: 验证修改**

```bash
grep -A 3 "ui-ux-pro-max" INSTALL.md
```

Expected: 显示包含 ui-ux-pro-max 的安装说明

**Step 4: Commit**

```bash
git add INSTALL.md
git commit -m "docs: add ui-ux-pro-max installation instructions"
```

---

## Task 8: 更新版本号和版本历史

**Files:**
- Modify: `sparkcode-dev-superpower/SKILL.md` (版本历史章节)

**Step 1: 找到版本历史章节**

```bash
grep -n "## 版本历史" sparkcode-dev-superpower/SKILL.md
```

Expected: 找到版本历史章节的行号

**Step 2: 更新版本历史**

在版本历史章节添加 v1.1 的更新记录：

```markdown
## 版本历史

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
```

**Step 3: 更新技能描述中的版本号**

在文件开头的描述中提及版本：

```markdown
# SparkCode Dev Superpower

**版本**: v1.1
**更新日期**: 2026-01-19

## 概述

这个技能协调Claude和Codex，实现端到端的开发工作流：
- **Claude负责**: 需求理解、设计、规划、分支管理
- **Codex负责**: 代码实现、测试、代码审查
- **前端支持**: 自动集成 ui-ux-pro-max 技能处理前端开发
```

**Step 4: 验证修改**

```bash
grep -A 5 "### v1.1" sparkcode-dev-superpower/SKILL.md
```

Expected: 显示 v1.1 的版本历史记录

**Step 5: Commit**

```bash
git add sparkcode-dev-superpower/SKILL.md
git commit -m "chore: bump version to v1.1 with frontend integration"
```

---

## 验收标准

### 功能完整性
- ✓ 阶段2包含前端检测逻辑
- ✓ 添加了前端需求识别指南章节
- ✓ 更新了示例用法，包含前端项目
- ✓ 前置条件包含 ui-ux-pro-max
- ✓ 所有文档都已更新

### 文档质量
- ✓ README.md 说明前端支持
- ✓ USAGE-GUIDE.md 包含前端场景
- ✓ INSTALL.md 包含 ui-ux-pro-max 安装说明
- ✓ 版本历史已更新

### 一致性
- ✓ 所有文档中的说明一致
- ✓ 示例格式统一
- ✓ 技能调用语法正确（@ui-ux-pro-max）

## 测试计划

### 手动测试

1. **测试前端检测**
   - 阅读更新后的阶段2内容
   - 验证前端检测规则清晰明确

2. **测试文档完整性**
   - 检查所有文档链接有效
   - 验证示例代码格式正确

3. **测试版本信息**
   - 确认版本号更新到 v1.1
   - 验证版本历史记录完整

### 集成测试（后续）

在实际使用中测试：
1. 创建纯前端项目，验证计划中正确标注 @ui-ux-pro-max
2. 创建前后端混合项目，验证前后端任务正确分离
3. 创建纯后端项目，验证不会错误标注前端技能

## 下一步

完成实施后：
1. 在实际项目中测试前端集成
2. 收集用户反馈
3. 根据反馈优化前端检测规则
4. 考虑添加更多技能集成（如果需要）
