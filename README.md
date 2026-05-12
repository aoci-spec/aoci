<div align="center">

# AOCI

### AI-Oriented Code Indexing

**A protocol for compressing repository-scale code into a single structured text artifact that an LLM can ingest in one pass.**

[![Spec Version](https://img.shields.io/badge/spec-v1.0-blue.svg?style=flat-square)](https://github.com/aoci-spec/aoci)
[![License](https://img.shields.io/badge/license-pending-lightgrey.svg?style=flat-square)](NOTICE.md)
[![Paper](https://img.shields.io/badge/paper-arXiv-b31b1b.svg?style=flat-square)](https://arxiv.org/abs/2605.02421)
[![Platform](https://img.shields.io/badge/platform-aoci.ai-7B3FE4.svg?style=flat-square)](https://aoci.ai)

[**Platform**](https://aoci.ai) · [**Paper (arXiv)**](https://arxiv.org/abs/2605.02421) · [**中文**](README.zh-CN.md)

</div>

---

## 📖 Overview

An AOCI index is a plain-text artifact that represents an entire repository through one structured entry per source file or database table. Each entry combines a discrete tag (architectural coordinates) with a continuous semantic description (function, relations, interface, design constraints).

The protocol targets a specific failure mode in LLM-assisted software engineering: degraded reasoning over long, low-entropy contexts. By relocating per-file information into a fixed schema before the task begins, AOCI lets an LLM localize, reason about, and modify code without runtime exploration.

> A repository of **~100K LOC** typically compresses to **600–800 lines of index** — a token-compression ratio of approximately **1:100 to 1:300**, with repository-level information recall of **70% to 90%**.

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

### Mode 1 — Index-first development

> *Write the index first, then generate code from it, to build entirely new systems.*

A developer describes requirements in natural language; an LLM produces a complete AOCI index as architectural blueprint; code is generated file-by-file under the constraints of each entry. Architecture is reviewed at the **index level (hundreds of lines)** — *before* any code is generated.

**Demonstrated capability**: end-to-end development and maintenance, by a single developer, of production systems up to **~200K LOC**, across stacks including Node.js + React, Go + Gin + Vue 3, and Go + React 19 + TypeScript, achieving **SonarQube quad-A ratings** (Reliability · Security · Maintainability · Coverage) under continuous deployment.

### Mode 2 — Iterative development on existing repositories

> *Compress an existing codebase into an index, then iterate continuously based on the index.*

An LLM reads an existing repository and generates the AOCI index. The index becomes the working blueprint for all subsequent feature additions, refactors, and bug fixes. Each change updates the index *first*, then propagates to code — keeping the architectural map continuously aligned with the implementation.

**Demonstrated capability**: **8+ months** of continuous iterative development on production systems of 100K+ LOC, with the index serving as the single source of architectural truth across every development task, with zero architectural debt or regression defects across tasks.

---

## 🔓 No toolchain lock-in

AOCI is a **format specification only**. It does not depend on any specific:

- **LLM model** — works with any model capable of structured generation and long-context reasoning
- **Agent tool** — orthogonal to Claude Code, Cursor, Cline, Aider, Copilot, etc.; can be layered alongside them
- **IDE or editor** — the index is plain text, editable in any environment
- **Build system, language, or framework** — stack-agnostic by design
- **Developer background** — usable by professional engineers and domain experts without traditional programming training alike

The index file goes anywhere a text file can go — local disk, Git repository, documentation system, or the [aoci.ai](https://aoci.ai) platform.

---

## 🧭 End-to-end workflow

This section describes the standard workflow for building a complete web system from scratch using the AOCI protocol. The workflow has been validated multiple times in production environments and can serve as a reference path for new projects.

> Scope: this workflow assumes a developer with no existing runtime environment or project code, aiming to complete the full "requirements → index → code → deployment" cycle end-to-end.

---

### Phase 1 — Infrastructure preparation

Building an internet-accessible production system requires three coordinated service layers — frontend, backend, and database — and therefore a remotely accessible server.

#### 1.1 Choose a cloud provider

Any mainstream cloud provider works:

AWS EC2, GCP Compute Engine, Azure VM, DigitalOcean, Linode, Alibaba Cloud, Tencent Cloud, etc.

#### 1.2 Server configuration

Recommended starter configuration:

- **OS**: Ubuntu LTS (22.04 or higher) — best command compatibility
- **Specs**: 2 vCPU / 2 GB RAM (sufficient for learning and small production systems)
- **Storage**: default 40 GB system disk
- **Network**: fixed public IP; pay-as-you-go bandwidth (minimal prepayment is enough)

No need to purchase elastic IPs or custom VPCs — defaults are fine.

#### 1.3 Port configuration

In the cloud console's security group rules, allow the following inbound ports:

| Port | Protocol | Purpose |
|---|---|---|
| 22 | TCP | SSH remote login (usually open by default) |
| 80 | TCP | HTTP service |
| 443 | TCP | HTTPS service |

#### 1.4 Remote login

Connect to the server via SSH. After setting a root password, log in with:

```bash
ssh root@<your-server-ip>
```

---

### Phase 2 — Environment setup

Install the technology stack required by the target system. The AOCI protocol itself is stack-agnostic; this section uses the most common web application stack as an example.

#### 2.1 Default reference stack

```
Frontend  ─── React
Backend   ─── Go (with Gin framework)
Database  ─── PostgreSQL
Proxy     ─── Nginx (reverse proxy + SSL termination)
```

#### 2.2 Installation method

The [aoci.ai](https://aoci.ai) platform provides validated system-level installation command blocks — developers can skip step-by-step configuration in three steps:

1. Retrieve the command block for your target stack from the platform
2. Copy the entire block to the server terminal
3. Press Enter and wait for automatic installation to complete

Other stacks (Node.js + Express, Python + Django, Java + Spring Boot, Rust + Axum, etc.) can be configured manually. The AOCI protocol works consistently across all stacks.

---

### Phase 3 — Model selection

AOCI index generation quality and index-driven development effectiveness are determined by the host model's **structured generation stability** and **long-context reasoning ability**. Capability tiers observed in production practice:

| Tier | Example models | Practical behavior |
|---|---|---|
| Recommended | Claude Opus | Highest index generation quality, stable long-repository reasoning, strict protocol adherence, first choice for production |
| Recommended | Claude Sonnet | Best price-performance ratio, well-rounded experience, recommended for daily use |
| Acceptable | GPT-5.5 series | Workable for most tasks; complex cross-module reasoning may need multiple iterations |
| Not recommended | Models lacking reasoning or long-context capability | Unstable protocol adherence, high index drift rate, not recommended for production |

The AOCI protocol itself **is not bound to any model**. Developers can choose:

- Plug in their own API keys via the [aoci.ai](https://aoci.ai) platform
- Use any long-context-capable independent model client (Claude web client, ChatGPT, Google AI Studio, etc.)

---

### Phase 4 — Requirements document

Open a conversation with the selected model and clarify three questions:

1. **Core positioning** — what software is being built; what problem does it solve
2. **Target users** — who will use it; typical scenarios
3. **Functional scope** — what features are expected; what non-functional constraints apply (performance, security, compliance)

Have the model produce a concise requirements document from the conversation; iterate until satisfied.

> **Key practice**: the first-version requirements should focus on the core feature set, avoiding covering the complete product form all at once.

Reasonable first-version scope: **3–8 core business modules** covering **the main workflows**. Other features are added incrementally in later iterations.

---

### Phase 5 — Index generation

Submit the requirements document together with a sample AOCI index to the model, asking it to generate the corresponding AOCI index.

#### 5.1 Prompt essentials

The prompt should specify:

- **Target stack** — backend language/framework, frontend framework, database type
- **File count constraint** — first-time users are advised to **keep under 50 source files**
- **Three-section output format** — index header rules, code index, database index

#### 5.2 First-time recommendations

> Index size correlates directly with first-time success rate: shorter index, simpler system, higher probability of getting the first run working, faster feedback loop, and easier to build understanding of and confidence in the protocol.

Suggested parameters for your first project:

| Project dimension | First-time recommendation |
|---|---|
| Total source files | ≤ 50 |
| Database tables | ≤ 15 |
| Business modules | 3–5 |
| Estimated LOC | 10K – 30K |

---

### Phase 6 — Index ingestion and project initialization

The model-generated index is organized in three sections; each must be ingested into the development environment separately.

#### 6.1 The three sections

| Section | Content |
|---|---|
| Index header | Tag dictionary definitions, dimension declarations, conventions |
| Code index | One entry per source file |
| Database index | One entry per database table |

#### 6.2 Ingestion methods

**Method A: Use the [aoci.ai](https://aoci.ai) platform (recommended)**

1. Create a new project on the platform (project name is arbitrary)
2. Paste the three sections into their corresponding areas: index header, code index, database index
3. Save — the platform automatically composes the complete index

The composed complete index on the platform is read-only, preventing accidental edits that would cause drift. Only the three input sections require manual maintenance.

**Method B: Plain text file**

Save the complete index as a single text file. The filename is arbitrary: `AOCI.md` / `aoci.txt` / `index.aoci` / `system-index.md` all work. Place the file at the repository root or anywhere else convenient.

---

### Phase 7 — First development loop

Enter a development session and start the **three-party collaboration loop**:

```
            ┌────────────────────┐
            │  Developer (you)   │
            │  describes need    │
            └─────────┬──────────┘
                      │
                      ▼
            ┌────────────────────┐
            │   LLM model        │
            │   produces command │
            │   or code          │
            └─────────┬──────────┘
                      │
                      ▼
            ┌────────────────────┐
            │  Server terminal   │
            │  executes, returns │
            │  result            │
            └─────────┬──────────┘
                      │
                      └──→ Result fed back to the LLM
                           (loop until the main flow runs)
```

#### 7.1 Loop steps

1. Developer describes the current need or next task
2. Model produces the corresponding command, config, or code
3. Developer copies the output to the terminal and executes
4. Terminal returns the execution result
5. Developer copies the relevant result back into the model's context
6. Model produces the next action based on the feedback
7. Repeat 1–6 until the system's main flow is accessible

#### 7.2 Platform assistance

The [aoci.ai](https://aoci.ai) platform provides a **development-mode dialog** — with the AOCI protocol prompt built in — so that commands produced by the model are directly executable, reducing manual cleanup. Development mode also automatically maintains protocol adherence in conversation based on the current project's index context.

---

### Phase 8 — First-run completion check

Once the main flow runs (login works, core pages are accessible, key workflows are complete), perform a final consistency check:

1. Send the complete index back to the model
2. Ask the model to verify that all files declared in the index have been generated and all database tables have been created
3. Patch any missing files, fields, or tables

The first development cycle is now complete.

---

### Phase 9 — Iterative development

Subsequent bug fixes and feature additions follow this flow:

#### 9.1 Start a new conversation

> Do not continue in the old session. Long sessions degrade significantly in context quality (context rot), and token cost grows linearly with history length. Starting fresh is most efficient.

#### 9.2 Inject the current full index

Send the latest complete index to the new conversation.

#### 9.3 Capability check

Ask the model to assess its grasp of the current system:

```
This is the AOCI index of the system you are about to develop.
Please assess: what percentage of the system's information have
you grasped? Are you ready to take over subsequent development
based on this index?
```

Proceed once the model confirms.

#### 9.4 Enter the development loop

State the iteration's requirement or bug description and enter the three-party loop from Phase 7.

---

### Phase 10 — Index-code synchronization

> **Core principle**: after every code change (feature addition, refactor, bug fix), the index must be updated synchronously. Changes update the index **first**, then propagate to code.

A persistent divergence between index and code causes the LLM to reason against an outdated architectural map, introducing changes that do not match the production system's true state.

#### 10.1 Maintenance operations

| Change type | Operation |
|---|---|
| New source file | Have the model generate the entry; locate the insertion point by directory grouping and add the new entry |
| Modified file | Have the model generate the updated entry; replace the original entry at the same position |
| Deleted source file | Remove the corresponding entry |
| New database table | Have the model generate the entry; append to the database index section |
| Modified table schema | Have the model generate the updated entry; replace the original entry at the same position |

#### 10.2 Modifying index header rules

Generally, do not modify the index header (tag dictionaries, dimension definitions, etc.). These rules define the "grammar" of the entire index; frequent changes invalidate previously generated entries.

Only consider modifying the header when:
- The project introduces new architectural tiers or business modules (extend the A/B dictionaries)
- The project introduces new key technical features (extend the D dictionary)
- You have a deep grasp of the protocol and need to adjust token budget strategy

After header changes, use the model to systematically check whether affected entries are still compliant.

---

### Phase 11 — Code hosting

Serious commercial systems should host code in a Git repository. GitHub, GitLab, Gitee, Bitbucket, self-hosted Gitea — any work.

#### 11.1 Initialization and first push

```bash
cd /path/to/your/project
git init
git remote add origin https://github.com/<username>/<repo>.git
git add .
git commit -m "Initial commit"
git branch -M main
git push -u origin main
```

#### 11.2 Pushing after each iteration

After each iteration, execute:

```bash
git status              # view changes
git add .
git commit -m "iteration description"
git push origin main
git log --oneline -5    # confirm recent commits
```

#### 11.3 Discarding uncommitted changes

If the iteration introduced serious problems that need to be thrown out:

```bash
git reset --hard        # discard all uncommitted changes
git clean -fd           # remove untracked new files
git status              # confirm working tree is clean
```

> ⚠ This operation is **irreversible**. Uncommitted modifications are permanently lost. **Database changes (table creation, schema alterations, data writes) are not rolled back by Git operations** — these require a separate database migration tool or manual scripts.

---

### Workflow summary

The entire workflow in one line:

> Provision server → install environment → choose model → generate requirements → generate AOCI index → ingest into development environment → enter development loop → main flow running → continuous iteration → synchronize index → push to Git.

**Core principle**: index and code must stay continuously synchronized. Once they diverge persistently, AOCI's capability degrades, and development falls back into the same failure modes as traditional LLM-assisted coding.

---

### Learning path recommendations

For first-time AOCI users, the suggested learning path is:

1. **Start with a small project** — choose a 3–5 module, 10K–30K LOC project as practice. A simple CRUD system, blog backend, or internal tool works well.
2. **Run through the full workflow once** — even on a very simple system, walk from Phase 1 to Phase 11 to build complete workflow muscle memory.
3. **Master the synchronization rhythm** — develop the habit of "code change immediately followed by index sync" on the small project. This is the foundation for sustaining 200K-LOC systems later.
4. **Then enter serious projects** — after 1–2 small projects build intuition for the protocol, launch your formal commercial project.

Starting directly with a large project as your first practice tends to lead to mid-project confidence collapse. The AOCI capability curve grows steeply with experience — the second project is usually significantly more efficient than the first.

---

## 🆘 FAQ

#### Q1: The model says the system is accessible, but the browser can't open the page?

Usually because the cloud provider's security group hasn't allowed HTTP/HTTPS ports. Check the security group rules in the cloud console and confirm that ports 80 and 443 are open. The exact configuration location varies by provider — screenshot the current console UI and send it to the model for specific guidance.

#### Q2: Which part of the terminal output should I copy back to the model?

Depends on the situation:

| Model's request | What to copy |
|---|---|
| Asks to view code (e.g., `cat <file>`) | Copy the entire terminal output of this command |
| Asks to execute a command and see the result | Copy **only the execution result** (typically the numbered output section); do not also copy the input command (it's redundant) |
| Output too long to copy fully | Copy the last 10–30 lines, or screenshot the terminal. Explicitly tell the model "output is too long, only the trailing section is shown" — the model will adjust strategy accordingly |

#### Q3: Can a less capable model complete AOCI-driven development?

Technically yes, but with significantly degraded results. AOCI demands strong **long-context reasoning** and **protocol adherence**. Less capable models tend to fail in these scenarios:

- Missing files or mis-classifying tags during index generation
- Losing constraint relationships during cross-file reasoning
- Unstable protocol adherence, output format drift

The recommended minimum is Claude Sonnet 4.6 or an equivalent-class model. Specific model selection can be A/B tested on the [aoci.ai](https://aoci.ai) platform before commitment.

#### Q4: The model's replies become incoherent during development — what do I do?

Usually a context quality degradation. Handle as follows:

1. Resend the latest complete index to the model
2. If still incoherent, start a new conversation, inject the index, and continue (see Phase 9.1–9.3)

#### Q5: Where should the index file be saved?

The AOCI protocol does not mandate a location. Common options:

- **Repository root** — e.g., `AOCI.md`, versioned alongside code
- **Independent documentation system** — Notion, Confluence, Logseq
- **[aoci.ai](https://aoci.ai) platform** — automatic versioning and drift detection

Regardless of where, what matters is that **the index stays synchronized with the code**.

#### Q6: The terminal hangs or stops responding — what do I do?

| Situation | Response |
|---|---|
| Terminal hangs (cursor frozen, command not completing) | Press `Ctrl+C` to interrupt; if ineffective, try `Ctrl+Z` to suspend |
| Terminal completely unresponsive | Close the window, re-establish the SSH connection |
| Error message unintelligible | Copy the complete error output or screenshot (including stack trace) to the model for analysis |

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

## 🛠 Platform — aoci.ai

Manual generation, drift detection, and version management are operationally feasible but tedious for production codebases. The [**aoci.ai**](https://aoci.ai) platform automates the protocol end-to-end:

| Feature | Description |
|---|---|
| 🔌 **Bring your own model** | Plug in any LLM endpoint (OpenAI-compatible, Anthropic, Gemini, self-hosted, custom proxies). No vendor lock-in. |
| ⚙️ **Custom protocol configuration** | Define your project's A/B/D dictionaries and token budgets via UI; platform enforces them across all entries. |
| 🔄 **GitHub sync** | Connect a repo, get an index automatically; subsequent commits trigger incremental updates. |
| 📊 **Drift detection** | Flags entries inconsistent with current source, preventing index-code divergence. |
| 📚 **Version history** | Every index generation is versioned and diffable. |
| 🧩 **Development-mode dialog** | Built-in AOCI protocol prompts make model-generated commands directly executable. |

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
