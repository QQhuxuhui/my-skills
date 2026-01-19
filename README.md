# SparkCode Dev Superpower

完整的软件开发工作流技能，集成Claude端Superpowers和Codex端Superpowers。

## 快速开始

### 1. 安装

**重要：** 技能必须是目录结构，包含 `SKILL.md` 文件。

```bash
# 复制技能到Claude Code
cp -r sparkcode-dev-superpower ~/.claude/skills/

# 重启Claude Code以加载技能
```

**详细安装步骤请查看：** [INSTALL.md](INSTALL.md)

### 2. 准备Codex环境

```bash
# 确保Codex端已安装Superpowers
cd ~/.codex/superpowers
git pull
```

### 3. 使用

在Claude Code中调用:

```
/sparkcode-dev-superpower 实现XXX功能
```

### 4. 工作流

1. **需求理解**: Claude通过提问理解需求
2. **编写计划**: Claude生成详细实现计划
3. **代码实现**: Codex执行计划中的任务
4. **代码审查**: Codex审查代码质量
5. **分支完成**: Claude协助完成分支

## 文档

- **安装指南**: [INSTALL.md](INSTALL.md) - 详细的安装步骤和故障排除
- **使用指南**: [USAGE-GUIDE.md](USAGE-GUIDE.md) - 完整的使用说明和示例
- **设计文档**: [docs/plans/2026-01-19-sparkcode-dev-superpower-design.md](docs/plans/2026-01-19-sparkcode-dev-superpower-design.md)
- **实施计划**: [docs/plans/2026-01-19-sparkcode-dev-superpower-implementation-plan.md](docs/plans/2026-01-19-sparkcode-dev-superpower-implementation-plan.md)

## 技能文件

- [sparkcode-dev-superpower/SKILL.md](sparkcode-dev-superpower/SKILL.md)

## 许可

MIT
