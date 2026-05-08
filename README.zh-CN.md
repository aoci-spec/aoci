# AOCI

> **AI-Oriented Code Indexing**（面向 AI 的代码索引）—— 一个把仓库级代码压缩为单一结构化文本工件的协议，LLM 可一次读完。

🌐 **语言**：[English](README.md) | 中文

[平台](https://aoci.ai) · [论文 (arXiv)](https://arxiv.org/abs/2605.02421) · [复现包 (Zenodo)](https://doi.org/10.5281/zenodo.19677251)

---

## 概述

AOCI 索引是一个纯文本工件，通过每个源文件或数据库表对应一个结构化条目，表达整个仓库。每个条目结合一个离散标签（架构坐标）和一段连续语义描述（功能、关系、接口、设计约束）。

该协议针对 LLM 辅助软件工程中的一个具体失效模式：长、低熵上下文中的推理退化。AOCI 通过在任务开始之前将文件级信息按固定 schema 重新组织，使 LLM 无需运行时探索即可定位、理解、修改代码。

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

10 万行级别的仓库通常压缩为 200–400 个条目——LLM 可单次前向传播读完。

## 实证结果

在 5 个生产系统的 19 个端到端开发任务，以及 4 个项目 × 3 个前沿 LLM × 6 种上下文条件（共 2,160 次评估）的基准上：

- AOCI **最终零缺陷** vs 三个 agent 工具 (Claude Code、Cursor、OpenCode) 共 **39 个缺陷**；Fisher 精确检验 *p* < 0.001
- Token 消耗低 **4–130 倍**；比率随任务复杂度上升
- 文件定位准确率 **97.67%**，仅比 Oracle 上限低 0.66 个百分点
- 确定性：相同输入跨次产出相同结果

方法学与完整结果见 [arXiv:2605.02421](https://arxiv.org/abs/2605.02421)。复现工件见 [Zenodo](https://doi.org/10.5281/zenodo.19677251)。

## 使用

### 生成索引

将下方协议规范与你的仓库一并交给任意前沿 LLM。推荐模型：Claude Sonnet 4.6、GPT-5、Gemini 2.5 Pro。

提示词模板：

```
你正在为一个代码仓库生成 AOCI 索引。

协议规范：
[粘贴本文档"协议"部分]

仓库文件树：
[粘贴 `tree` 或 `git ls-files` 的输出]

关键文件内容：
[粘贴文件树中识别出的高重要性文件源码]

输出：一份 AOCI.md 文件。每个源文件和数据库表对应一个条目。
严格遵循编码规则。S 字段用于承载无法从文件语法本身推断的信息。
```

将输出保存为仓库根目录下的 `AOCI.md`。

### 在编程会话中使用

会话开始时、任务描述之前，将 `AOCI.md` 注入 LLM 上下文窗口。LLM 通过索引而非工具探索完成文件定位、依赖追踪、架构约束的解析。

### 增量维护

文件级独立性允许每次变更以 *O(1)* 成本更新单个条目。当文件 `X` 被修改时，仅重新生成 `X` 的条目；其他文件的条目仍然有效，因为 `R` 字段使用文件名引用而非版本固定的接口。

### 自动化平台

手工生成、一致性校验、版本管理在工程上可行但繁琐。[aoci.ai](https://aoci.ai) 平台自动化处理索引生成、漂移检测、增量更新。

## 协议

### 文件级条目

```
filename[ABCDE-tag]: F:function | R:relations | A:API | S:synopsis
```

方括号内编码**离散标签层**。冒号后编码**连续语义层**，四个元素用 `|` 分隔。

#### 标签层

五个正交维度：

| 维度 | 含义 | 编码 |
|---|---|---|
| `A` | 架构层级 | 项目自定义字典中的单字符（例如 H=Handler、S=Service、R=Repository、M=Model、W=Middleware、U=Router） |
| `B` | 业务模块 | 单字符（例如 A=Auth、O=Org、C=Credits） |
| `C` | 重要性 | 取值于 {9, 8, 7, 5, 3, 1}——非均匀六档 |
| `D` | 技术特征（可选） | 单字符串联（例如 J=JWT、R=RBAC、T=Tx、C=Crypto） |
| `E` | 代码规模 | 取值于 {S, M, L, XL}——按行数分档 |

示例：`WA9JM` 表示 Middleware 层级、Auth 模块、重要性 9、含 JWT 特征、中等规模的文件。

`A`、`B`、`D` 字典由项目自定义，在索引头部声明。

#### 语义层

冒号后的四个元素：

| 元素 | 内容 |
|---|---|
| `F` | 文件的业务角色，一句简短描述 |
| `R` | 逗号分隔的依赖文件名引用 |
| `A` | 暴露的 API 或端点；无则填 `-` |
| `S` | 高熵设计决策：降级逻辑、事务边界、加密方案、反直觉契约、运行时不变式。无法从语法本身推断的信息。 |

`S` 字段是经实证验证信息密度最高的字段。消融实验显示移除 S 会使整体准确率下降 20.07 个百分点（单组件最大降幅）。S 应作为主要内容载体。

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

## 引用

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

## 许可

本规范以 [LICENSE](LICENSE) 文件中的许可发布。
