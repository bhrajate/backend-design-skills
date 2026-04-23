# Backend Design Skills

[English](../../README.md) | [中文](./README.zh.md)

一个 Claude Code skill，为后端系统设计提供专家级、以原则驱动的指导。当你询问
架构设计、API 设计、服务拆分、领域建模或代码组织等问题时，Claude 会先查阅本
skill 再给出建议，从而让回答基于业界公认的设计原则，而非临时拍脑袋的意见。

## 功能介绍

`backend-design` skill 会让 Claude 在回答后端问题时应用以下 **黄金原则**：

| 原则 | 适用范围 | 核心思想 |
|---|---|---|
| S.O.L.I.D | 类 / 模块级 | 面向对象的可维护性设计规则 |
| DRY | 任意层级 | 消除重复 |
| KISS | 任意层级 | 保持简单 |
| YAGNI | 特性级 | 不为未来尚未出现的需求编码 |
| Clean Architecture | 应用级 | 解耦业务逻辑与基础设施 |
| DDD（领域驱动设计） | 领域级 | 围绕业务领域建模软件 |
| 微服务 | 系统级 | 按业务能力拆分服务 |
| 12-Factor App | 部署级 | 构建可移植、可伸缩的云服务 |

S.O.L.I.D、DDD、微服务、Clean Architecture 及 API 设计的深入参考材料位于
`skills/backend-design/references/` 目录，按需加载。

### 何时会自动触发

当你的问题涉及以下场景时，该 skill 会被自动调用：

- "项目 / 服务应该怎么组织？"
- "这个设计合理吗？" / "我该用什么模式？"
- API 设计、服务边界、领域建模
- 为提高可维护性做的重构
- 任何后端架构或系统设计问题

## 安装方式

### 方式一 —— 通过 Plugin Marketplace 安装（推荐）

在 Claude Code 中将本仓库作为 marketplace 添加，并安装插件：

```
/plugin marketplace add bhrajate/backend-design-skills
/plugin install backend-design@backend-design-skills
```

其它常用命令：

```
/plugin marketplace list          # 列出已添加的 marketplace
/plugin marketplace update        # 拉取最新插件定义
/plugin marketplace remove <name> # 移除某个 marketplace
```

### 方式二 —— 从本地克隆安装

```bash
git clone git@github.com:bhrajate/backend-design-skills.git backend-design-skills
```

然后在 Claude Code 中：

```
/plugin marketplace add ./backend-design-skills
/plugin install backend-design@backend-design-skills
```

### 方式三 —— 手动安装（用户级 skill）

把 skill 目录拷贝到用户级 skills 目录：

```bash
mkdir -p ~/.claude/skills
cp -r skills/backend-design ~/.claude/skills/
```

Claude Code 会自动发现 `~/.claude/skills/<name>/SKILL.md`，重启会话后 skill
即可生效。

## 验证安装

安装完成后，向 Claude 提一个设计问题，例如：

> "我应该如何把单体拆成多个服务？"

Claude 的回答应该会引用 `backend-design` skill，并基于相关原则（例如 YAGNI、
DDD 限界上下文等）展开。

## 仓库目录结构

```
.
├── .claude-plugin/
│   └── marketplace.json     # Marketplace 清单（声明插件及其 skills）
├── skills/
│   └── backend-design/
│       ├── SKILL.md         # Skill 入口
│       └── references/      # 按需加载的深度文档
│           ├── solid.md
│           ├── ddd.md
│           ├── microservices.md
│           ├── clean-architecture.md
│           └── api-design.md
└── docs/
    ├── assets/
    └── lang/
        └── README.zh.md
```

## 许可证

请查看仓库根目录下的 `LICENSE` 文件。
