# Evidence — Real Execution Records

A durable, auditable record of running every module of **Customizing LLMs on AWS EKS** on a live cluster.

- **Cluster:** `genai-workshop` (EKS Auto Mode, k8s v1.36)
- **Account / Region:** `215808926717` / `us-east-2`
- **Model:** Ministral-3-8B-Instruct-2512
- **Captured:** 2026-10-20 (live, read-only re-verification + per-module execution output)

Each record contains the exact commands run and the **real output / results** observed on the cluster. Each has a matching SVG flow diagram under [`diagrams/`](./diagrams).

---

## Records

| # | Module | Record | Diagram | Status |
|---|--------|--------|---------|--------|
| 0 | Foundation (EKS + Karpenter + GPU) | [00-foundation.md](./00-foundation.md) | [foundation.svg](./diagrams/foundation.svg) | ✅ verified |
| 1 | 100 — Serve with vLLM | [100-vllm.md](./100-vllm.md) | [100-vllm.svg](./diagrams/100-vllm.svg) | ✅ verified |
| 2 | 200 — Agent (tool calling) | [200-agent.md](./200-agent.md) | [200-agent.svg](./diagrams/200-agent.svg) | ✅ core / ⚠️ container |
| 3 | 300 — Benchmark | [300-benchmark.md](./300-benchmark.md) | [300-benchmark.svg](./diagrams/300-benchmark.svg) | ✅ verified |
| 4 | 400 — LMCache (L1+L2) | [400-lmcache.md](./400-lmcache.md) | [400-lmcache.svg](./diagrams/400-lmcache.svg) | ✅ verified |
| 5 | 600 — LoRA fine-tune | [600-finetune.md](./600-finetune.md) | [600-finetune.svg](./diagrams/600-finetune.svg) | ✅ verified |
| 6 | 700 — RAG | [700-rag.md](./700-rag.md) | [700-rag.svg](./diagrams/700-rag.svg) | ✅ verified |
| 7 | 800 — Ray distributed | [800-ray.md](./800-ray.md) | [800-ray.svg](./diagrams/800-ray.svg) | ✅ infra / ⚠️ serve |
| — | End-to-end | (see [../WORKSHOP-FLOW.md](../WORKSHOP-FLOW.md)) | [end-to-end.svg](./diagrams/end-to-end.svg) | — |

**Legend:** ✅ = fully verified with live output; ⚠️ = one sub-component blocked by a genuine environment/version constraint, documented transparently in the record.

---

## Summary of verified results

| Module | Headline evidence |
|--------|-------------------|
| Foundation | 3 NodePools (gpu capped at 1); model 9.7 GiB in S3; GPU node `nvidia.com/gpu:1` |
| 100 | vLLM 1/1 Running on L40S; "what is EKS?" answered (40 tokens); public ALB |
| 200 | `tool_calls=[current_weather({"location":"Tokyo"})]`, `finish_reason:tool_calls` |
| 300 | 60 req / 0 fail; TTFT 85.9 ms, TPOT 25.1 ms; results in S3 `benchmarks/` |
| 400 | L1 `LocalCPUBackend` + L2 `rediss://lmcache-valkey-eks-...:6379` both active |
| 600 | loss 2.77→0.26, acc 46%→97.7%; adapter 67.3 MiB in S3; served base+adapter |
| 700 | 25 vectors in S3 Vectors; grounded headphone answer; Gradio ALB |
| 800 | RayCluster `ready`, head+GPU worker 1/1; serve blocked by `ministral3` arch gap |

---

## How this was produced

1. Patched manifests (correct bucket/region, EKS-Auto-Mode node selectors, KubeRay RBAC) live in [`../.deploy/`](../.deploy); workshop originals left untouched.
2. All AWS/kubectl calls were run against the live cluster; output captured verbatim into the records above.
3. SVG diagrams are hand-authored (no Mermaid/JS dependency) so they render natively on GitHub.
4. Environment values are in [`../.env`](../.env) — **git-ignore it** (contains credentials).

> These two caveats are stated in their records, not hidden: Module 200's agent *container* needs Docker+ECR (unavailable here — the tool-calling mechanism itself is verified); Module 800's Ray *serve layer* fails because the brand-new `ministral3` architecture is unknown to the transformers version bundled in Ray's image (the distributed cluster itself is fully verified).
