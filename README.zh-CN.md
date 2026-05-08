# AOCI

[![Spec Version](https://img.shields.io/badge/spec-v1.0-blue)](https://github.com/aoci-spec/aoci)
[![License](https://img.shields.io/badge/license-CC0-green)](LICENSE)
[![Paper](https://img.shields.io/badge/paper-arXiv-b31b1b)](https://arxiv.org/abs/2605.02421)
[![Platform](https://img.shields.io/badge/platform-aoci.ai-purple)](https://aoci.ai)

> **AI-Oriented Code Indexing**（面向 AI 的代码索引）—— 一个把仓库级代码压缩为单一结构化文本工件的协议，LLM 可一次读完。

🌐 **语言**：[English](README.md) · 中文

---

## 📖 概述

AOCI 索引是一个纯文本工件，通过每个源文件或数据库表对应一个结构化条目，表达整个仓库。每个条目结合一个离散标签（架构坐标）和一段连续语义描述（功能、关系、接口、设计约束）。

该协议针对 LLM 辅助软件工程中的一个具体失效模式：长、低熵上下文中的推理退化。AOCI 通过在任务开始之前将文件级信息按固定 schema 重新组织，使 LLM 无需运行时探索即可定位、理解、修改代码。

**约 10 万行 LOC 的仓库通常压缩为 600–800 行索引——token 压缩比约 1:100。**

一个典型条目：

```
auth.go[WA9JM]: F:JWT authentication middleware |
R:pkg/jwt,model/user | A:- |
S:extract Bearer token, verify JWT, inject user_id
into gin.Context, API key fallback via SHA256

org_repo.go[PO9NTM]: F:organizational data access |
R:model/org | A:- |
S:closure-table for GetTree/GetAncestors,
MoveNode reconstructs subtree relations

users[U-M-M-GUID]: user primary table,
uuid/username/email unique, password_hash bcrypt,
status, is_superadmin, preferences JSONB
```

方法学、评估结果与理论分析见[论文](https://arxiv.org/abs/2605.02421)。

---

## 🔓 不依赖任何工具链

AOCI **仅是一种格式规范**。它不依赖于任何特定：

- **LLM 模型** — 任何具备结构化生成能力的模型均可
- **Agent 工具** — 与 Claude Code、Cursor、Cline、Aider、Copilot 等正交
- **IDE 或编辑器** — 索引就是纯文本
- **构建系统、语言、框架** — 协议技术栈无关

你可以在任何已有的技术栈上使用 AOCI。索引文件可以放在任何能存放文本文件的地方。

---

## ⚙️ 使用

### 生成索引

将下方协议规范与你的仓库一并交给任意前沿 LLM。提示词模板：

```
你正在为一个代码仓库生成 AOCI 索引。

协议规范：
[粘贴本文档"协议"部分]

仓库文件树：
[粘贴 `tree` 或 `git ls-files` 的输出]

关键文件内容：
[粘贴高重要性文件的源码]

输出：一份 AOCI 索引。每个源文件和数据库表对应一个条目。
严格遵循编码规则。S 字段用于承载无法从语法本身推断的信息。
```

输出可保存为任意位置——`AOCI.md`、`aoci.txt`、`index.aoci`，或托管在 [aoci.ai](https://aoci.ai) 平台。协议不强制规定文件名或存储位置。

### 在编程会话中使用

会话开始时、任务描述之前，将索引内容注入 LLM 上下文窗口。LLM 通过索引而非工具探索完成文件定位、依赖追踪、架构约束的解析。

### 增量维护

文件级独立性允许每次变更以 *O(1)* 成本更新单个条目。当文件 `X` 被修改时，仅重新生成 `X` 的条目；其他文件的条目仍然有效，因为 `R` 字段使用文件名引用而非版本固定的接口。

---

## 🛠 平台 — aoci.ai

手工生成、漂移检测、版本管理在工程上可行，但对生产级代码库而言繁琐。[**aoci.ai**](https://aoci.ai) 平台端到端自动化协议执行：

- **自带模型** — 可接入任何 LLM 端点 (OpenAI 兼容、Anthropic、Gemini、自托管、自定义代理)。平台不绑定任何厂商。
- **自定义协议配置** — 通过界面定义你项目的 A/B/D 字典和 token 预算；平台在所有生成条目上强制执行。
- **GitHub 同步** — 连接仓库即自动生成索引；后续 commit 触发增量更新。
- **漂移检测** — 标记与当前源码不一致的条目。
- **版本历史** — 每次索引生成均版本化、可 diff。

平台是 AOCI 协议的一种实现。协议本身保持开放、工具无关——任何人均可构建替代实现。

---

## 🏛 参考实现

在 AOCI 驱动开发模式下构建并持续维护的生产系统：

| 系统 | 技术栈 | 规模 | 验证 |
|---|---|---|---|
| AI Practice Platform | Node.js + React | ~148K LOC | SonarQube 2A · 0 bugs / 0 漏洞 · 539 测试通过 |
| AI Education Platform | Go + Gin + Vue 3 | ~82K LOC | SonarQube 4A · 595 测试通过 · 14,085 QPS 压测 · 12 项安全渗透测试通过 |
| TE-DNA 2.0 | Go + React + TypeScript | ~56K LOC | SonarQube 4A · 生产部署 |
| LegalMind | Go + React 19 + TypeScript | ~42K LOC | SonarQube 4A |
| AI Hedge Fund Pro | Go + React + TypeScript | ~39K LOC | SonarQube 4A |

SonarQube 4A = 四个核心维度（可靠性、安全性、可维护性、覆盖率）均为 A 级。

这些不是 demo 项目，而是服务真实用户的生产系统（AI Education Platform 注册用户超过 10 万），具有八个月以上的部署历史，在所有 benchmark 任务中 AOCI 引入零缺陷。

---

## 📐 协议

### 文件级条目

```
filename[ABCDE-tag]: F:function | R:relations | A:API | S:synopsis
```

方括号内编码**离散标签层**。冒号后编码**连续语义层**，四个元素用 `|` 分隔。

#### 标签层 — 五个正交维度

| 维度 | 含义 | 编码 |
|---|---|---|
| `A` | 架构层级 | 项目自定义字典中的单字符（例如 H=Handler、S=Service、R=Repository、M=Model、W=Middleware、U=Router） |
| `B` | 业务模块 | 单字符（例如 A=Auth、O=Org、C=Credits） |
| `C` | 重要性 | 取值于 {9, 8, 7, 5, 3, 1}——非均匀六档 |
| `D` | 技术特征（可选） | 单字符串联（例如 J=JWT、R=RBAC、T=Tx、C=Crypto） |
| `E` | 代码规模 | 取值于 {S, M, L, XL}——按行数分档 |

示例：`WA9JM` 表示 Middleware 层级、Auth 模块、重要性 9、含 JWT 特征、中等规模的文件。

`A`、`B`、`D` 字典由项目自定义，在索引头部声明。

#### 语义层 — 四个元素

| 元素 | 内容 |
|---|---|
| `F` | 文件的业务角色，一句简短描述 |
| `R` | 逗号分隔的依赖文件名引用 |
| `A` | 暴露的 API 或端点；无则填 `-` |
| `S` | 高熵设计决策：降级逻辑、事务边界、加密方案、反直觉契约、运行时不变式。无法从语法本身推断的信息。 |

`S` 应作为主要内容载体——它在协议中承载每 token 信息密度最高的内容。

### 数据库表条目

```
table_name[domain-table_type-scale_estimate-features]: 字段级描述
```

四维标签用 `-` 分隔：

- `domain` — 功能领域 (U=User、P=Points、I=Indexing、A=Auditing 等)
- `table_type` — 主表 (M)、关联表 (A)、日志表 (L)、配置表 (C)
- `scale_estimate` — 预期行数：S、M、L、XL
- `features` — 串联的属性标记（GUID、JSONB、UNIQ、SOFT、FK 等）

示例：`users[U-M-M-GUID]` 解码为用户域主表，中等规模，使用 GUID 标识符。

冒号后是逗号分隔的字段级描述：列语义、约束、编码选择。

### 索引头部

索引以项目特定的头部开始，声明维度字典与预算分配。推荐结构：

```
# AOCI Index — [项目名]
# Spec version: 1.0
#
# Tag dictionaries:
#   A: H=Handler S=Service R=Repository M=Model W=Middleware U=Router
#   B: A=Auth O=Org R=Role C=Credits I=Index ...
#   D: J=JWT R=RBAC T=Tx C=Crypto W=WS S=SSE ...
#
# Importance levels (C): 9 > 8 > 7 > 5 > 3 > 1
#
# Token budget per entry:
#   C=9 : 80–150 tokens
#   C=7 : 50–80 tokens
#   C<=3: 20–40 tokens
#
# Conventions:
#   - 每个源文件或数据库表对应一个条目
#   - R 使用文件名引用，不绑定版本化接口
#   - S 记录无法从语法推导的信息
```

### 条目独立性

每个条目仅依赖其源文件。条目间引用仅通过 `R` 字段中的文件名提及实现；条目不内嵌其他条目的内容。这允许增量重新生成的成本与变更集而非仓库总大小成正比。

---

## 📚 引用

```bibtex
@misc{aoci2026,
  author       = {Jinshi Liu and others},
  title        = {{AOCI}: Symbolic-Semantic Indexing for Practical Repository-Scale Code Understanding with LLMs},
  year         = {2026},
  eprint       = {2605.02421},
  archivePrefix= {arXiv},
  url          = {https://arxiv.org/abs/2605.02421}
}
```

## 📜 许可

本规范以 [LICENSE](LICENSE) 文件中的许可发布。
