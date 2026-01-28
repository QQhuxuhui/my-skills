# My Skills

个人 Claude Code 技能集合。

## 技能列表

| 技能 | 描述 |
|------|------|
| [sparkcode-dev-superpower](sparkcode-dev-superpower/SKILL.md) | 完整的软件开发工作流，集成Claude端和Codex端Superpowers |
| [technical-bid-writer](technical-bid-writer/SKILL.md) | 软件/IT系统开发类技术标书写作助手 |

---

## SparkCode Dev Superpower

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

# 确保Codex端已安装 ui-ux-pro-max（用于前端开发）
# 如果未安装，请参考 ui-ux-pro-max 的安装文档
```

### 3. 使用

在Claude Code中调用:

```
/sparkcode-dev-superpower 实现XXX功能
```

### 4. 工作流

1. **需求理解**: Claude通过提问理解需求
2. **编写计划**: Claude生成详细实现计划，自动检测前端需求
3. **代码实现**: Codex执行计划中的任务
   - 前端任务：自动调用 ui-ux-pro-max 技能
   - 后端任务：使用标准开发流程
4. **代码审查**: Codex审查代码质量
5. **分支完成**: Claude协助完成分支

## 文档

- **安装指南**: [INSTALL.md](INSTALL.md) - 详细的安装步骤和故障排除
- **使用指南**: [USAGE-GUIDE.md](USAGE-GUIDE.md) - 完整的使用说明和示例
- **设计文档**: [docs/plans/2026-01-19-sparkcode-dev-superpower-design.md](docs/plans/2026-01-19-sparkcode-dev-superpower-design.md)
- **实施计划**: [docs/plans/2026-01-19-sparkcode-dev-superpower-implementation-plan.md](docs/plans/2026-01-19-sparkcode-dev-superpower-implementation-plan.md)

## 技能文件

- [sparkcode-dev-superpower/SKILL.md](sparkcode-dev-superpower/SKILL.md)

---

## Technical Bid Writer

软件/IT系统开发类技术标书写作助手，帮助研发负责人高效撰写技术标书。

### 功能

- 结构化写作指导（8大章节模板）
- 技术方案设计（架构、选型建议）
- 内容润色优化
- 全流程辅助

### 安装

```bash
cp -r technical-bid-writer ~/.claude/skills/
```

### 使用

在 Claude Code 中说：
- "帮我写一个XX系统的技术标书"
- "我需要准备招标响应的技术方案"

### 技能文件

- [technical-bid-writer/SKILL.md](technical-bid-writer/SKILL.md)

---

## 许可

MIT
