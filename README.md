# AOCI

> **AI-Oriented Code Indexing** — A protocol for compressing repository-scale code into a single structured text artifact that an LLM can ingest in one pass.

🌐 **Languages**: English | [中文](README.zh-CN.md)

[Platform](https://aoci.ai) · [Paper (arXiv)](https://arxiv.org/abs/2605.02421) · [Reproducible artifact (Zenodo)](https://doi.org/10.5281/zenodo.19677251)

---

## Overview

An AOCI index is a plain-text artifact that represents an entire repository through one structured entry per source file or database table. Each entry combines a discrete tag (architectural coordinates) with a continuous semantic description (function, relations, interface, design constraints).

The protocol targets a specific failure mode in LLM-assisted software engineering: degraded reasoning over long, low-entropy contexts. By relocating per-file information into a fixed schema before the task begins, AOCI allows an LLM to localize, reason about, and modify code without runtime exploration.

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

A repository of 100K+ LOC typically compresses to 200–400 entries — readable by an LLM in a single forward pass.

## Empirical results

Benchmarked across 5 production systems, 19 end-to-end development tasks, and 4 projects evaluated against 3 frontier LLMs under 6 context conditions (2,160 evaluations):

- **0 final-state defects** with AOCI vs **39 defects** across three agent-based tools (Claude Code, Cursor, OpenCode); Fisher's exact test *p* < 0.001
- **4–130× lower token consumption**; ratio scales with task complexity
- **97.67%** file-localization accuracy, 0.66pp below the Oracle upper bound
- Determinism: identical input produces identical output across runs

Methodology and full results: [arXiv:2605.02421](https://arxiv.org/abs/2605.02421). Reproduction artifacts: [Zenodo](https://doi.org/10.5281/zenodo.19677251).

## Usage

### Generate an index

Pass the protocol specification (below) and your repository to any frontier LLM. Recommended models: Claude Sonnet 4.6, GPT-5, Gemini 2.5 Pro.

Prompt template:

```
You are generating an AOCI index for a code repository.

Protocol specification:
[paste the "Protocol" section of this document]

Repository file tree:
[paste output of `tree` or `git ls-files`]

Key file contents:
[paste source of high-importance files identified from the tree]

Output: a single AOCI.md file. One entry per source file and per
database table. Follow the encoding rules strictly. Allocate the
S field to information that cannot be inferred from the file's
syntax alone.
```

Save the output as `AOCI.md` at the repository root.

### Use the index in coding sessions

Inject `AOCI.md` into the LLM context window at session start, prior to any task description. The LLM will resolve file localization, dependency chains, and architectural constraints from the index rather than through tool-based exploration.

### Maintain incrementally

Per-file independence permits *O(1)* updates per changed file. When file `X` is modified, regenerate only the entry for `X`; entries for unchanged files remain valid because the `R` field references files by name rather than by version-pinned interface.

### Automated platform

Manual generation, consistency validation, and version management are operationally feasible but tedious. The [aoci.ai](https://aoci.ai) platform automates index generation, drift detection, and incremental updates.

## Protocol

### Per-file entry

```
filename[ABCDE-tag]: F:function | R:relations | A:API | S:synopsis
```

The bracketed substring encodes the **discrete tag layer**. The colon-delimited remainder encodes the **continuous semantic layer** with four elements separated by `|`.

#### Tag layer

Five orthogonal dimensions:

| Dim | Meaning | Encoding |
|---|---|---|
| `A` | Architectural tier | Single-character code from project-defined dictionary (e.g., H=Handler, S=Service, R=Repository, M=Model, W=Middleware, U=Router) |
| `B` | Business module | Single-character code (e.g., A=Auth, O=Org, C=Credits) |
| `C` | Importance | One of {9, 8, 7, 5, 3, 1} — non-uniform six-level scale |
| `D` | Technical features (optional) | Concatenated single-character codes (e.g., J=JWT, R=RBAC, T=Tx, C=Crypto) |
| `E` | Code scale | One of {S, M, L, XL} — by line count |

Example: `WA9JM` denotes a Middleware-tier file in the Auth module of importance 9, with JWT feature, of medium size.

The `A`, `B`, `D` dictionaries are project-specific and declared in the index header.

#### Semantic layer

Four elements after the colon:

| Element | Content |
|---|---|
| `F` | Business role of the file. One concise phrase. |
| `R` | Comma-separated filename references to dependencies. |
| `A` | Exposed APIs or endpoints. `-` if none. |
| `S` | High-entropy design decisions: fallback logic, transactional boundaries, encryption schemes, counter-intuitive contracts, runtime invariants. Information not inferable from syntax alone. |

The `S` field carries the most evaluation-validated information density. Ablation studies show its removal reduces overall accuracy by 20.07pp (the largest single-component degradation). Treat `S` as the primary content vehicle.

### Database table entry

```
table_name[domain-table_type-scale_estimate-features]: field-level description
```

Four-dimensional tag separated by `-`:

- `domain` — functional area (U=User, P=Points, I=Indexing, A=Auditing, etc.)
- `table_type` — primary (M), association (A), log (L), config (C)
- `scale_estimate` — expected row count: S, M, L, XL
- `features` — concatenated property markers (GUID, JSONB, UNIQ, SOFT, FK)

Example: `users[U-M-M-GUID]` decodes as User-domain primary table of medium scale with GUID identifier.

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

## Citation

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

## License

This specification is released under the license in [LICENSE](LICENSE).
