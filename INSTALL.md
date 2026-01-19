# SparkCode Dev Superpower - 安装指南

## ✅ 问题已解决！

**重要发现：** Claude Code技能的正确格式是**目录结构**，不是单个 `.skill` 文件。

## 📁 正确的技能格式

```
~/.claude/skills/
└── sparkcode-dev-superpower/    # 目录（技能名称）
    └── SKILL.md                  # 文件（必须大写）
```

**错误格式（不会被识别）：**
```
~/.claude/skills/
└── sparkcode-dev-superpower.skill  # ❌ 单个文件不会被识别
```

## 🚀 安装步骤

### 步骤1：安装技能到Claude Code

```bash
# 方法A：从项目复制（推荐）
cp -r /usr/src/workspace/github/QQhuxuhui/my-skills/sparkcode-dev-superpower ~/.claude/skills/

# 方法B：创建符号链接
ln -s /usr/src/workspace/github/QQhuxuhui/my-skills/sparkcode-dev-superpower ~/.claude/skills/

# 验证安装
ls -la ~/.claude/skills/sparkcode-dev-superpower/SKILL.md
# 应该看到 SKILL.md 文件
```

### 步骤2：准备Codex环境

```bash
# 1. 确保Codex已安装
which codex
# 应该输出: /path/to/codex

# 2. 确保Codex端已安装Superpowers
cd ~/.codex/superpowers
git pull  # 更新到最新版本

# 3. 验证Codex可以调用Superpowers技能
codex exec --prompt "Run ~/.codex/superpowers/.codex/superpowers-codex use-skill superpowers:using-superpowers"
```

### 步骤3：创建必要目录

```bash
# 在你的项目中创建这些目录
mkdir -p .sparkcode/logs
mkdir -p docs/plans
```

### 步骤4：重启Claude Code

**这是关键步骤！** Claude Code在启动时加载技能。

```bash
# 完全退出Claude Code
# 然后重新启动
```

## ✅ 验证安装

重启Claude Code后，测试技能是否可用：

```
/sparkcode-dev-superpower 创建一个简单的Hello World程序
```

如果技能被正确识别，你应该看到：
1. Claude开始执行brainstorming阶段
2. 询问你关于需求的问题

## 📖 技能目录结构说明

根据Claude Code官方文档，标准的技能结构是：

```
plugin-name/
└── skills/
    └── skill-name/
        ├── SKILL.md          # 必需：技能定义
        ├── references/       # 可选：参考文档
        ├── examples/         # 可选：示例代码
        └── scripts/          # 可选：辅助脚本
```

我们的技能目前只有 `SKILL.md`，这是最基本的要求。

## 🔍 故障排除

### 问题1：技能仍然无法识别

**检查清单：**

```bash
# 1. 确认目录结构正确
ls -la ~/.claude/skills/sparkcode-dev-superpower/
# 应该看到 SKILL.md 文件

# 2. 确认文件名大写
ls ~/.claude/skills/sparkcode-dev-superpower/SKILL.md
# 必须是 SKILL.md，不是 skill.md 或 Skill.md

# 3. 确认YAML frontmatter格式
head -5 ~/.claude/skills/sparkcode-dev-superpower/SKILL.md
# 应该看到：
# ---
# name: sparkcode-dev-superpower
# description: ...
# ---
```

### 问题2：Codex调用失败

```bash
# 检查Codex是否在PATH中
which codex

# 检查Codex Superpowers版本
cd ~/.codex/superpowers && git log -1

# 手动测试Codex
codex exec --prompt "echo test"
```

### 问题3：权限问题

```bash
# 确保技能目录有正确的权限
chmod -R 755 ~/.claude/skills/sparkcode-dev-superpower
```

## 💡 使用示例

安装成功后，你可以这样使用：

```
/sparkcode-dev-superpower 实现用户登录功能，支持邮箱和密码登录
```

或者在对话中提到：

```
请使用 sparkcode-dev-superpower 技能来实现一个博客系统
```

## 📚 相关文档

- **详细使用指南**：`USAGE-GUIDE.md`
- **README**：`README.md`
- **设计文档**：`docs/plans/2026-01-19-sparkcode-dev-superpower-design.md`
- **实施计划**：`docs/plans/2026-01-19-sparkcode-dev-superpower-implementation-plan.md`

## 🎯 下一步

1. ✅ 按照上述步骤安装技能
2. ✅ 重启Claude Code
3. ✅ 测试技能是否可用
4. 📖 查看 `USAGE-GUIDE.md` 了解详细用法

## 🙏 感谢

感谢你发现了格式问题！这帮助我们修正了技能的结构，现在应该可以正常工作了。

---

**最后更新**：2026-01-19
**版本**：v1.0.1（修正了技能格式）
