# SparkCode Dev Superpower

完整的软件开发工作流技能，集成Claude端Superpowers和Codex端Superpowers。

## 快速开始

### 1. 安装

确保Codex端已安装Superpowers:

```bash
cd ~/.codex/superpowers
git pull
```

### 2. 使用

在Claude中调用:

```
/sparkcode-dev-superpower 实现XXX功能
```

### 3. 工作流

1. **需求理解**: Claude通过提问理解需求
2. **编写计划**: Claude生成详细实现计划
3. **代码实现**: Codex执行计划中的任务
4. **代码审查**: Codex审查代码质量
5. **分支完成**: Claude协助完成分支

## 文档

- [设计文档](docs/plans/2026-01-19-sparkcode-dev-superpower-design.md)
- [实施计划](docs/plans/2026-01-19-sparkcode-dev-superpower-implementation-plan.md)

## 技能文件

- [sparkcode-dev-superpower.skill](sparkcode-dev-superpower.skill)

## 许可

MIT
