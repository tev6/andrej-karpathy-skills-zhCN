# Andrej Karpathy Skills — 中文版

[andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills) 的中文翻译，将 Andrej Karpathy 提出的 LLM 编程行为准则翻译为中文，供中文开发者使用。

## 这是什么？

一份面向 AI 编程助手（如 Claude Code）的行为指南，以 `CLAUDE.md` 和 `SKILL.md` 的形式存在，核心思路源于 [Andrej Karpathy 对 LLM 编程常见问题的观察](https://x.com/karpathy/status/2015883857489522876)。

它定义了四个核心原则：

| 原则          | 解决什么问题            |
| ----------- | ----------------- |
| **先思考，后编码** | 错误假设、隐藏困惑、缺失的权衡考量 |
| **简洁至上**    | 过度复杂化、臃肿的抽象层      |
| **精准修改**    | 顺便重构、改动无关代码       |
| **目标驱动执行**  | 缺乏可验证的成功标准        |

## 为什么有用？

LLM 在编程时有一些常见的不良倾向：

- 替你做假设，然后一路走到底
- 把代码做得过于复杂，引入不必要的抽象
- 顺手改掉不该改的代码
- 在没有明确目标的情况下盲目修改

这组准则正是针对这些问题设计的，让 AI 在编码时更谨慎、更精确、更可控。

## 应用场景

- **日常编码** — 在编写或修改代码时，让 AI 遵循更专业的编码习惯
- **代码审查** — 规范 AI 的审查行为，避免过度优化
- **重构** — 确保 AI 只做被要求的改动，不顺手牵羊
- **调试** — 通过"测试先行"的方式定位和修复问题

## 安装使用

将 `CLAUDE.md` 放入项目根目录，AI 编程助手会自动读取并遵循其中的准则。

### 方案 A：curl 下载

**新项目：**

```bash
curl -o CLAUDE.md https://raw.githubusercontent.com/tev6/andrej-karpathy-skills-zhCN/main/CLAUDE.md
```

**已有项目（追加到现有 CLAUDE.md 末尾）：**

```bash
echo "" >> CLAUDE.md
curl https://raw.githubusercontent.com/tev6/andrej-karpathy-skills-zhCN/main/CLAUDE.md >> CLAUDE.md
```

### 方案 B：手动复制

1. 打开 [CLAUDE.md](https://raw.githubusercontent.com/tev6/andrej-karpathy-skills-zhCN/main/CLAUDE.md)
2. 复制全部内容
3. 粘贴到项目根目录的 `CLAUDE.md` 文件中

## 文件结构

```
├── CLAUDE.md              # 核心准则（项目级配置）
├── EXAMPLES.md             # 四大原则的示例说明
├── README.translate.md     # 原英文版 README 的中文翻译
├── .claude-plugin/
│   ├── marketplace.json    # 插件市场配置
│   └── plugin.json         # 插件定义
└── skills/
    └── karpathy-guidelines/
        └── SKILL.md        # 技能定义文件
```

<br />

## 许可证

MIT
