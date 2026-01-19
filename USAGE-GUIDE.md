# SparkCode Dev Superpower 使用指南

## 快速开始

### 前置条件检查

在使用前，请确保：

```bash
# 1. 检查Codex是否安装
which codex
# 应该输出: /path/to/codex

# 2. 检查Codex Superpowers
ls ~/.codex/superpowers/
# 应该看到superpowers相关文件

# 3. 检查项目目录
ls -la .sparkcode/logs docs/plans
# 如果不存在，运行: mkdir -p .sparkcode/logs docs/plans
```

## 使用场景

### 场景1：实现新功能

**用户输入：**
```
/sparkcode-dev-superpower 实现用户登录功能，支持邮箱和密码登录
```

**工作流：**

1. **阶段1：需求理解**
   - Claude会问你一系列问题来理解需求
   - 例如："需要支持记住登录状态吗？"
   - 例如："密码需要什么强度要求？"

2. **阶段2：编写计划**
   - Claude生成详细的实现计划
   - 保存到 `docs/plans/2026-01-19-user-login-plan.md`
   - 你可以审查并确认计划

3. **阶段3：Codex执行**
   - Claude调用Codex执行计划
   - Codex会：
     - 编写代码
     - 运行测试
     - 提交更改
   - 你会看到进度更新

4. **阶段4：代码审查**
   - Claude调用Codex审查代码
   - 显示审查结果：
     ```
     审查结果:
       ✗ Critical: 0
       ⚠ Major: 1
       ℹ Minor: 3
     ```
   - 你可以选择修复或继续

5. **阶段5：完成分支**
   - Claude协助你完成工作
   - 选项：merge、创建PR、或保持分支

### 场景2：修复Bug

**用户输入：**
```
/sparkcode-dev-superpower 修复登录页面在移动端显示错位的问题
```

工作流程相同，但计划会更聚焦于bug修复。

### 场景3：重构代码

**用户输入：**
```
/sparkcode-dev-superpower 重构用户认证模块，提取公共逻辑
```

## 高级用法

### 自定义计划粒度

如果任务很大，可以要求拆分：

```
/sparkcode-dev-superpower 实现完整的电商系统

（Claude会建议拆分成多个子任务）
```

### 处理执行失败

如果Codex执行失败，你会看到：

```
✗ Codex执行失败

选项:
  a) 重试执行
  b) 修改计划后重试
  c) 手动介入调试
  d) 放弃当前任务
```

选择合适的选项继续。

### 查看执行日志

```bash
# 查看最近的执行日志
ls -lt .sparkcode/logs/

# 查看具体日志
cat .sparkcode/logs/codex-output-20260119-103000.log
```

### 查看会话状态

```bash
# 查看当前会话状态
cat .sparkcode/session-state.json
```

## 常见问题

### Q1: Codex调用失败怎么办？

**症状：**
```
✗ Codex执行失败，退出码: 127
```

**解决方案：**
```bash
# 1. 检查Codex是否在PATH中
which codex

# 2. 检查Codex Superpowers
cd ~/.codex/superpowers && git status

# 3. 手动测试Codex调用
codex exec --prompt "echo test"
```

### Q2: 计划文件格式不兼容？

**症状：**
Codex无法读取计划文件

**解决方案：**
```bash
# 确保两端Superpowers版本一致
cd ~/.claude/plugins/cache/superpowers-marketplace/superpowers
git log -1

cd ~/.codex/superpowers
git log -1

# 如果版本不同，更新到相同版本
```

### Q3: 如何跳过某个阶段？

技能设计为完整工作流，不建议跳过阶段。但如果需要：

- **跳过brainstorming**: 直接提供详细需求
- **跳过planning**: 提供现成的计划文件
- **跳过execution**: 手动编写代码后，从review阶段开始

### Q4: 可以在现有分支上使用吗？

可以，但建议：

```bash
# 1. 先提交当前更改
git add . && git commit -m "WIP: current work"

# 2. 创建新分支用于sparkcode工作流
git checkout -b feature/sparkcode-task

# 3. 然后使用技能
```

### Q5: 如何自定义Codex超时时间？

编辑技能文件中的超时设置（默认30分钟）：

```bash
# 在sparkcode-dev-superpower.skill中查找
grep -n "timeout" sparkcode-dev-superpower.skill

# 修改相应的超时值
```

## 最佳实践

### 1. 任务粒度控制

✅ **好的任务：**
- "实现用户注册功能"
- "添加邮箱验证"
- "修复登录按钮样式"

❌ **避免的任务：**
- "实现整个系统"（太大）
- "改一下代码"（太模糊）

### 2. 提供清晰的需求

✅ **好的需求：**
```
实现用户登录功能：
- 支持邮箱+密码登录
- 密码至少8位，包含字母和数字
- 登录失败3次后锁定账户5分钟
- 支持"记住我"功能（7天有效）
```

❌ **模糊的需求：**
```
做一个登录
```

### 3. 及时审查计划

在阶段2完成后，仔细审查生成的计划：
- 检查任务分解是否合理
- 确认技术方案是否正确
- 验证验收标准是否明确

### 4. 监控执行过程

虽然Codex自动执行，但要关注：
- 执行日志中的警告
- 测试失败的原因
- 代码审查中的问题

### 5. 保持环境同步

定期更新两端的Superpowers：

```bash
# Claude端
cd ~/.claude/plugins/cache/superpowers-marketplace/superpowers
git pull

# Codex端
cd ~/.codex/superpowers
git pull
```

## 故障排除

### 问题：技能无法找到

```bash
# 检查技能文件位置
ls -la ~/.claude/skills/sparkcode-dev-superpower.skill

# 或检查项目根目录
ls -la sparkcode-dev-superpower.skill
```

### 问题：Codex权限错误

```bash
# 检查Codex配置
codex config list

# 确保有执行权限
chmod +x $(which codex)
```

### 问题：计划文件无法创建

```bash
# 检查目录权限
ls -ld docs/plans/

# 创建目录
mkdir -p docs/plans
chmod 755 docs/plans
```

## 性能优化

### 1. 使用缓存

```bash
# Codex会缓存依赖，确保缓存目录存在
mkdir -p ~/.codex/cache
```

### 2. 并行执行（未来功能）

当前版本串行执行任务。未来版本将支持并行执行独立任务。

### 3. 增量执行

如果执行失败，可以从失败点继续：

```bash
# 查看会话状态
cat .sparkcode/session-state.json

# 找到失败的任务，手动修复后继续
```

## 示例项目

### 示例1：Todo应用

```
/sparkcode-dev-superpower 实现一个Todo应用

需求：
- 添加、编辑、删除任务
- 标记任务完成/未完成
- 按状态筛选任务
- 数据持久化到本地存储
```

### 示例2：API服务

```
/sparkcode-dev-superpower 实现用户管理API

需求：
- RESTful API设计
- CRUD操作（创建、读取、更新、删除用户）
- JWT认证
- 输入验证
- 错误处理
- 单元测试覆盖率>80%
```

### 示例3：数据处理脚本

```
/sparkcode-dev-superpower 实现CSV数据清洗脚本

需求：
- 读取CSV文件
- 删除重复行
- 填充缺失值
- 数据类型转换
- 导出清洗后的数据
- 生成清洗报告
```

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

## 获取帮助

如果遇到问题：

1. **查看日志**：`.sparkcode/logs/`
2. **查看设计文档**：`docs/plans/2026-01-19-sparkcode-dev-superpower-design.md`
3. **查看实施计划**：`docs/plans/2026-01-19-sparkcode-dev-superpower-implementation-plan.md`
4. **提交Issue**：在项目仓库提交问题

## 版本信息

- **当前版本**：v1.0
- **发布日期**：2026-01-19
- **兼容性**：
  - Claude Code: 最新版本
  - Codex: 最新版本
  - Superpowers: v4.0+

## 更新日志

### v1.0 (2026-01-19)
- ✨ 初始版本发布
- ✅ 实现5阶段工作流
- ✅ 支持Codex集成
- ✅ 包含错误处理和状态管理
- ✅ 完整的文档和示例

---

**祝你使用愉快！** 🚀
