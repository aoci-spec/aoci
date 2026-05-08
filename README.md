<div align="center">

# AOCI

### AI-Oriented Code Indexing

**A protocol for compressing repository-scale code into a single structured text artifact that an LLM can ingest in one pass.**

[![Spec Version](https://img.shields.io/badge/spec-v1.0-blue.svg?style=flat-square)](https://github.com/aoci-spec/aoci)
[![License](https://img.shields.io/badge/license-pending-lightgrey.svg?style=flat-square)](NOTICE.md)
[![Paper](https://img.shields.io/badge/paper-arXiv-b31b1b.svg?style=flat-square)](https://arxiv.org/abs/2605.02421)
[![Platform](https://img.shields.io/badge/platform-aoci.ai-7B3FE4.svg?style=flat-square)](https://aoci.ai)

[**Platform**](https://aoci.ai) · [**Paper (arXiv)**](https://arxiv.org/abs/2605.02421) · [**中文文档**](README.zh-CN.md)

</div>

---

## 📖 Overview

An AOCI index is a plain-text artifact that represents an entire repository through one structured entry per source file or database table. Each entry combines a discrete tag (architectural coordinates) with a continuous semantic description (function, relations, interface, design constraints).

The protocol targets a specific failure mode in LLM-assisted software engineering: degraded reasoning over long, low-entropy contexts. By relocating per-file information into a fixed schema before the task begins, AOCI lets an LLM localize, reason about, and modify code without runtime exploration.

> A repository of **~100K LOC** typically compresses to **600–800 lines of index** — a token-compression ratio of approximately **1:100**.

A representative entry:

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

For methodology, evaluation, and theoretical analysis, see the [paper](https://arxiv.org/abs/2605.02421).

---

## 🚀 What AOCI enables

AOCI supports two complementary development modes — both validated on production systems.

### Mode 1 — Index-first development (greenfield)

> *Build new systems by writing the index first, then generating code from it.*

A developer describes requirements in natural language; an LLM produces a complete AOCI index as architectural blueprint; code is generated file-by-file under the constraints of each entry. Architecture is reviewed at the **index level (hundreds of lines)** — *before* any code is generated.

**Demonstrated capability**: end-to-end development of production systems up to **~200K LOC**, across stacks including Node.js + React, Go + Gin + Vue 3, and Go + React 19 + TypeScript, achieving **SonarQube quad-A ratings** (Reliability · Security · Maintainability · Coverage) under continuous deployment.

### Mode 2 — Iterative development on existing repositories (brownfield)

> *Compress an existing codebase into an index, then iterate on it continuously.*

An LLM reads an existing repository and generates the AOCI index. The index becomes the working blueprint for all subsequent feature additions, refactors, and bug fixes. Each change updates the index *first*, then propagates to code — keeping the architectural map continuously aligned with the implementation.

**Demonstrated capability**: 8+ months of continuous iterative development on production systems of 100K+ LOC, with the index serving as the single source of architectural truth across every development task.

---

## 🔓 No toolchain lock-in

AOCI is a **format specification**. It does not depend on any specific:

- **LLM model** — works with any model capable of structured generation
- **Agent tool** — orthogonal to Claude Code, Cursor, Cline, Aider, Copilot, etc.
- **IDE or editor** — the index is plain text, edit it anywhere
- **Build system, language, or framework** — stack-agnostic by design
- **Developer background** — usable by professional engineers and domain experts without traditional programming training alike

The index file goes anywhere a text file can go.

---

## ⚙️ Usage

### Generate an index

Pass the protocol specification (below) and your repository to any frontier LLM. Prompt template:

```
You are generating an AOCI index for a code repository.

Protocol specification:
[paste the "Protocol" section of this document]

Repository file tree:
[paste output of `tree` or `git ls-files`]

Key file contents:
[paste source of high-importance files]

Output: a single AOCI index. One entry per source file and per
database table. Follow the encoding rules strictly. Allocate the
S field to information that cannot be inferred from syntax alone.
```

Save the output anywhere — `AOCI.md`, `aoci.txt`, `index.aoci`, or hosted on the [aoci.ai](https://aoci.ai) platform. The protocol does not mandate a filename or storage location.

### Use the index in coding sessions

Inject the index content into the LLM context window at session start, prior to any task description. The LLM resolves file localization, dependency chains, and architectural constraints from the index rather than through tool-based exploration.

### Maintain incrementally

Per-file independence permits *O(1)* updates per changed file. When file `X` is modified, regenerate only the entry for `X`; entries for unchanged files remain valid because the `R` field references files by name rather than by version-pinned interface.

---

## 🛠 Platform — aoci.ai

Manual generation, drift detection, and version management are operationally feasible but tedious for production codebases. The [**aoci.ai**](https://aoci.ai) platform automates the protocol end-to-end:

| Feature | Description |
|---|---|
| 🔌 **Bring your own model** | Plug in any LLM endpoint (OpenAI-compatible, Anthropic, Gemini, self-hosted, custom proxies). No vendor lock-in. |
| ⚙️ **Custom protocol configuration** | Define your project's A/B/D dictionaries and token budgets via UI; platform enforces them across all entries. |
| 🔄 **GitHub sync** | Connect a repo, get an index automatically; subsequent commits trigger incremental updates. |
| 📊 **Drift detection** | Flags entries inconsistent with current source. |
| 📚 **Version history** | Every index generation is versioned and diffable. |

The platform is one implementation of the AOCI protocol. The protocol itself remains open and tool-independent — anyone can build alternative implementations.

---

## 🏛 Reference implementations

Production systems built and continuously maintained under AOCI-driven development:

| System | Stack | Scale | Verification |
|---|---|---|---|
| AI Practice Platform | Node.js + React | ~148K LOC | SonarQube 2A · 0 bugs / 0 vulnerabilities · 539 tests passing |
| AI Education Platform | Go + Gin + Vue 3 | ~82K LOC | SonarQube 4A · 595 tests passing · 14,085 QPS stress · 12 security pen-checks passed · 100K+ registered users |
| TE-DNA 2.0 | Go + React + TypeScript | ~56K LOC | SonarQube 4A · production deployment |
| LegalMind | Go + React 19 + TypeScript | ~42K LOC | SonarQube 4A |
| AI Hedge Fund Pro | Go + React + TypeScript | ~39K LOC | SonarQube 4A |

> SonarQube 4A = A-grade across all four dimensions: **Reliability**, **Security**, **Maintainability**, **Coverage**.

These are not toy demonstrations. They are production systems serving real users, with eight-month deployment histories and zero AOCI-attributable defects across all benchmarked development tasks.

---

## 📐 Protocol

### Per-file entry

```
filename[ABCDE-tag]: F:function | R:relations | A:API | S:synopsis
```

The bracketed substring encodes the **discrete tag layer**. The colon-delimited remainder encodes the **continuous semantic layer** with four elements separated by `|`.

#### Tag layer — five orthogonal dimensions

| Dim | Meaning | Encoding |
|---|---|---|
| `A` | Architectural tier | Single-character code from project-defined dictionary (e.g., H=Handler, S=Service, R=Repository, M=Model, W=Middleware, U=Router) |
| `B` | Business module | Single-character code (e.g., A=Auth, O=Org, C=Credits) |
| `C` | Importance | One of {9, 8, 7, 5, 3, 1} — non-uniform six-level scale |
| `D` | Technical features (optional) | Concatenated single-character codes (e.g., J=JWT, R=RBAC, T=Tx, C=Crypto) |
| `E` | Code scale | One of {S, M, L, XL} — by line count |

> Example: `WA9JM` denotes a Middleware-tier file in the Auth module of importance 9, with JWT feature, of medium size.

The `A`, `B`, `D` dictionaries are project-specific and declared in the index header.

#### Semantic layer — four elements

| Element | Content |
|---|---|
| `F` | Business role of the file. One concise phrase. |
| `R` | Comma-separated filename references to dependencies. |
| `A` | Exposed APIs or endpoints. `-` if none. |
| `S` | High-entropy design decisions: fallback logic, transactional boundaries, encryption schemes, counter-intuitive contracts, runtime invariants. Information not inferable from syntax alone. |

> Treat `S` as the primary content vehicle — it carries the highest information density per token in the protocol.

### Database table entry

```
table_name[domain-table_type-scale_estimate-features]: field-level description
```

Four-dimensional tag separated by `-`:

- `domain` — functional area (U=User, P=Points, I=Indexing, A=Auditing, etc.)
- `table_type` — primary (M), association (A), log (L), config (C)
- `scale_estimate` — expected row count: S, M, L, XL
- `features` — concatenated property markers (GUID, JSONB, UNIQ, SOFT, FK)

> Example: `users[U-M-M-GUID]` decodes as User-domain primary table of medium scale with GUID identifier.

The remainder is comma-separated field-level description: column semantics, constraints, encoding choices.

### Index header

The index begins with a project-specific header that declares dimension dictionaries and budget allocation. Recommended structure:

```
# AOCI Index — [project name]
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
#   - One entry per source file or database table
#   - R uses filename references, not version-pinned interfaces
#   - S records what cannot be derived from syntax
```

### Entry independence

Each entry depends only on its source file. Inter-entry references occur exclusively through filename mentions in the `R` field; no entry embeds content from another. This permits incremental regeneration with cost proportional to the change set rather than total repository size.

---

## 📚 Citation

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

---

## 📜 License & Patent

The AOCI Protocol Specification is currently provided **all rights reserved**. A formal license is being finalized — see [NOTICE.md](NOTICE.md) for current terms.

The AOCI Protocol itself is covered by **patents pending** with the China National Intellectual Property Administration. Commercial implementation may require a patent license — see [aoci.ai/license](https://aoci.ai/license) for inquiries.

---

<div align="center">
<sub>Built with the AOCI protocol · Maintained by the AOCI community</sub>
</div>
