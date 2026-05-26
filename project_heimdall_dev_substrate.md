---
name: heimdall-dev-substrate-already-exists-on-arcaivm-latent-infrastructure
description: "ArcAiVM's open-webui + Ollama + RTX 3050 + Neo4j stack is Heimdall's complete development environment — was set up before the product existed; treat as the testbed when Heimdall build starts."
metadata: 
  node_type: memory
  type: project
  originSessionId: 25aa75a3-911b-4fba-aa2c-3ced07920f2a
---

**Decision date:** 2026-05-26 (strategic conversation following AI-stack audit on ArcAiVM).

**Constraint discovered 2026-05-26 (afternoon):** ArcAiVM is only **4GB RAM (3GB ballooned)**, not 16GB as earlier memory implied. Llama 3.1 8B OOM-killed when attempting to memory-map its 5GB GGUF during load. PVE cluster had 595MiB free, no headroom to raise. So the substrate as currently configured **cannot run a 7-8B model**. Either the VM gets bumped to ≥8GB (steal from ciphertrust-ce 8→6GB or kryptvakt-dev 4→3GB) before Heimdall dev begins, OR Heimdall dev iteration uses a 3B-class model (Llama 3.2 3B, Qwen 2.5 3B), OR dev happens on a different machine. Don't repeat the "ArcAiVM as substrate" assumption without re-checking RAM allocation first.

The infrastructure Mitch set up on ArcAiVM (VM 103) over a year ago is
*not* stranded as the morning audit suggested — it's **latent
Heimdall-dev substrate**, *modulo the RAM constraint above*. The stack
predates the product but maps cleanly onto the development workflow
Heimdall will need, provided the RAM is sorted first.

## What's on ArcAiVM and how it maps to Heimdall

| Component | Today | Heimdall dev role |
|---|---|---|
| Ollama (llama3.1:8b + qwen2.5-coder:7b) | 0% GPU utilization, models downloaded but unused | LLM endpoint for playbook iteration; OpenAI-compatible tool-use; iterate dialogue patterns against Qwen for £0 instead of burning Anthropic API tokens |
| RTX 3050 8GB (passthrough) | Idle | Required for Ollama inference at usable speed; can drive up to ~14B 4-bit-quant models |
| open-webui | Mitch's personal AI chat surface | Pattern reference (NOT a copy) for the kryptvakt-native Heimdall chat panel; ALSO usable as a dev testbed — register Heimdall's tool-use surface as OpenAI function schemas, point at local Ollama, iterate the playbook → response loop with real chat UX |
| Neo4j | Set up to "drive determinism into the stack" | **v1.5+ knowledge-graph substrate for multi-framework reasoning** — encode compliance-framework requirements as graph nodes, relationship traversals replace multi-step tool-call chains. NOT for v1 (focused SQLite tools sufficient for single-framework PCI-DSS audit prep). |
| arcai-bot Telegram interface | Direct Anthropic Sonnet API | Pattern for Heimdall's customer-side chat surface (Telegram/Slack/Teams bot for ops on mobile) |
| OpenClaw orchestrator | Configured Qwen-primary + Sonnet-fallback, currently bypassed by bot | Pattern for Heimdall's playbook-router (dispatch operator question to right playbook) |

## Heimdall v1 vs v1.5+ architecture

- **v1 (Tier 2 launch — PCI-DSS audit prep)**: ship with focused
  SQLite-backed tools per ADR-004/005. Local Ollama as dev iteration
  loop. open-webui as the UX reference for the kryptvakt-native chat
  panel. Don't introduce Neo4j into the product.
- **v1.5+ (multi-framework correlation)**: when Heimdall has to
  reason across PCI + SOC 2 + DORA simultaneously, the
  relational-query overhead of SQLite + multi-tool-chains blows up.
  Neo4j becomes structurally necessary. Prototype the graph schema
  against the existing ArcAiVM Neo4j instance before deciding
  whether to ship it as part of the kryptvakt distribution.

## The unobvious insight from Mitch (2026-05-26)

> "if heimdall didn't have to load all user inventory in every prompt
> but able to quickly find info from a knowledge graph"

This is the correct architectural intuition. The naive ADR-004
"focused tools" path works for v1 (narrow questions, single
framework). For v1.5+ relational questions ("what's the audit risk
if we lose AD CS?"), a graph-augmented retrieval pattern beats
multi-tool-call stitching on both token spend AND determinism. The
ArcAiVM Neo4j Mitch set up >12 months ago for unrelated reasons is
exactly the right place to prototype this v1.5 capability.

## Apply when:

- Starting Heimdall build (Phase 3 — currently deferred per ADR-005)
  → activate the local Ollama as the dev LLM, write playbooks against
  open-webui as the iteration surface
- Designing v1.5 multi-framework correlation → prototype against
  ArcAiVM Neo4j before deciding to ship a graph component
- Sizing GPU upgrades → RTX 3050 8GB handles dev iteration; if Mitch
  wants to dogfood Heimdall against bigger models, larger GPU may
  become justified post-AB

See `kryptvakt/ADR.md` for ADR-004 (security) and ADR-005 (product
structure). ADR-006 (queued) should incorporate the graph-as-v1.5
note and the open-webui-as-dev-testbed note.
