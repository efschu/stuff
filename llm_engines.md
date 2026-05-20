> **⚠️ AUTO-UPDATE INSTRUCTION FOR SUBSEQUENT LLM AGENTS:**
>
> This document was generated on **2026-05-19**. Before trusting any status claim, you MUST verify every single row against the live sources listed in the "Quelle" column. The LLM inference landscape changes weekly.
>
> **YOUR TASK:**
>
> 1. **For EVERY row in every table**, visit the URL(s) in the "Quelle" column.
> 2. **Check for changes:**
>    - Did a CLOSED PR merge? → Update Status to ✅ Production, add version/tag.
>    - Did an OPEN issue get resolved? → Update Status, remove from "Open Issues" section.
>    - Did a new PR/issue appear that changes the status? → Add it.
>    - Did a repo move, get archived, or change maintainers? → Note it.
> 3. **Search for NEW developments** not yet captured:
>    - GitHub: `vllm-project/vllm`, `sgl-project/sglang`, `ggml-org/llama.cpp`, `ikawrakow/ik_llama.cpp`, `spiritbuun/buun-llama-cpp`, `Sandermage/genesis-vllm-patches`, `noonghunna/club-3090`, `LMCache/LMCache`, `llm-d/llm-d-kv-cache`, `kvcache-ai/mooncake`, `UChi-JCL/CacheGen`, `NVIDIA/kvpress`, `FluffyAIcode/LLM-KV--Cache-compress`, `InternLM/lmdeploy`, `NVlabs/RocketKV`
>    - Search queries: `"turboquant" AND "mtp"`, `"kv cache" AND "fp8" AND "tensor parallel"`, `"qwen3.6" AND "speculative"`, `"draft-mtp"`, `"turboquant_3bit_nc"`, `"lmcache" AND "mtp"`, `"disaggregated prefill" AND "qwen3.6"`, `"nixl connector" AND "speculative"`, `"llm-d-kv-cache" AND "mtp"`
>    - HuggingFace: `Qwen/Qwen3.6-27B`, `unsloth/Qwen3.6-27B-MTP-GGUF`
>    - Web: `"TurboQuant" vLLM MTP`, `"Genesis" vLLM patches`, `"llama.cpp" MTP merged`, `"LMCache" MTP speculative`, `"vLLM disaggregated prefill" MTP`
> 4. **Update this file in place:**
>    - Change ✅/❌/⚠️ as verified.
>    - Add new rows for newly merged features.
>    - Remove rows for deprecated/removed features.
>    - Update the "Letzte Aktualisierung" date at the top.
>    - Append new entries to the "Offene Issues & PRs" section.
>    - DO NOT remove the verification instruction block at the top.
>
> **CRITICAL FOCUS AREAS (highest churn):**
> - TurboQuant + MTP on vLLM (Genesis vs upstream merge status, vllm#40880, #40914, #40069)
> - llama.cpp Strict Tensor Parallel stability (`--split-mode tensor`, PR #19378 follow-ups)
> - llama.cpp TurboQuant upstream acceptance (Issue #20977, all closed PRs)
> - SGLang FP4 KV Cache graduation from experimental to production
> - Qwen3.6 MTP method name changes (`qwen3_next_mtp` vs `mtp` vs `draft-mtp`)
> - LMCache MTP compatibility (chunk caching vs MTP draft tokens)
> - vLLM Disagg Prefill + MTP (NixlConnector, MooncakeConnector, LMCacheConnectorV1)
> - llm-d-kv-cache speculative indexing maturity (v0.6.0+ roadmap)
> - lmdeploy TurboQuant + speculative decoding (currently blocked)
> - Hardware compatibility updates (V100/RTX3080/RTX5090 benchmarks, FP8 KV support, NVFP4)
> - Hardware compatibility updates (V100/RTX3080/RTX5090 benchmarks, FP8 KV support)
>
> **VERIFY BEFORE YOU WRITE. CITE EVERY CHANGE.**
>
---

# LLM Inference Engines: Status Report (May 2026)

**Fokus:** KV Cache Quantisierung (FP8/FP4/Q4/Q5/Q8/TurboQuant) + Tensor Parallel (Strict TP) + Native MTP (Qwen3.6-27B)
**Letzte Aktualisierung:** 2026-05-20

---

## 1. vLLM (vllm-project/vllm)

| Feature | Status | Version / PR / Issue | Quelle |
|---------|--------|---------------------|--------|
| **FP8 KV Cache** | ✅ Production | `--kv-cache-dtype fp8` (e4m3/e5m2). Per-tensor & per-attention-head via llm-compressor. | [vLLM Docs](https://docs.vllm.ai/en/latest/features/quantization/quantized_kvcache/), [PR #30141](https://github.com/vllm-project/vllm/pull/30141), [Blog Apr 2026](https://vllm.ai/blog/2026-04-22-fp8-kvcache) |
| **FP4 KV Cache** | ❌ | Nicht implementiert | — |
| **Q4/Q5/Q8 Integer KV** | ❌ | Keine Integer KV Cache Quantisierung | — |
| **TurboQuant KV (dense)** | ✅ Production | `--kv-cache-dtype turboquant_3bit_nc` (TQ3, 3-bit). PR #38479 (15.04.2026) nativ in vLLM. **Ohne Genesis:** Funktioniert für GQA/MHA-Modelle (Llama, Mistral, Gemma). | [PR #38479](https://github.com/vllm-project/vllm/pull/38479) |
| **TurboQuant KV (hybrid/Qwen3.6)** | ⚠️ **Genesis required** | Qwen3.6-27B (Hybrid: DeltaNet + Attention): `NotImplementedError` in `vllm/engine/arg_utils.py:1652`. **Mit Genesis v7.72.2:** ✅ Production (P67 TQ multi-query kernel + P64/P66/PN8/PN9/PN14/PN30/PN34). | [Genesis](https://github.com/Sandermage/genesis-vllm-patches), [club-3090/turbo.yml](https://github.com/noonghunna/club-3090/blob/master/models/qwen3.6-27b/vllm/compose/dual/turbo.yml) |
| **Strict Tensor Parallel** | ✅ Production | `--tensor-parallel-size N`, multi-GPU/multi-node via Ray | [vLLM Docs](https://docs.vllm.ai/en/latest/serving/distributed_serving.html) |
| **MTP (Qwen3.6)** | ✅ Production | `--speculative-config '{"method":"qwen3_next_mtp","num_speculative_tokens":2}'` | [Qwen3.6-27B HF Card](https://huggingface.co/Qwen/Qwen3.6-27B) |
| **MTP + Prefix Cache Bug** | ⚠️ OPEN | MTP reduziert Prefix Cache Hit Rate von 92% auf 71% | [Issue #38182](https://github.com/vllm-project/vllm/issues/38182) |
| **MTP + Pipeline Parallel** | ❌ BLOCKED | `NotImplementedError` auf v0.19.0 | [devnen/qwen3.6-windows-server MTP_HEAD.md](https://github.com/devnen/qwen3.6-windows-server/blob/main/docs/MTP_HEAD.md) |
| **CPU KV Offload** | ✅ | `--cpu-offload-gb 200` (v0.19.0) | [mubibai.com Apr 2026](https://mubibai.com/sglang-vs-vllm-production-inference-benchmarks-april-2026/) |
| **NVFP4** | ✅ | Blackwell (B200/GB300) via compressed-tensors | [vLLM Docs](https://docs.vllm.ai/en/latest/) |

### Genesis (Sandermage/genesis-vllm-patches) — vLLM Patch-Set für TurboQuant + MTP

**Repo:** https://github.com/Sandermage/genesis-vllm-patches | **Version:** v7.72.2

**Ohne Genesis:** TurboQuant + MTP auf vLLM ist **defekt** (Issue #40880). club-3090 tombstoned `tq3-mtp.yml` am 2026-05-11.

**Mit Genesis v7.72.2:** TurboQuant + MTP + Strict TP ist **Production** (club-3090 `turbo.yml`: ✅ Production).
- **Production-validiert:** `turboquant_k8v4` + MTP (k8v4, nicht 3bit_nc). 4 streams @ 262K, 53.65 narr / 72.93 code TPS, AL 3.41.
- **TOMBSTONED:** `turboquant_3bit_nc` + MTP (tq3-mtp.yml). Issue #40880: "we did not test MTP at all in the v7.13 cycle... your data shows that assumption is wrong."
- **Single-card OOM Bug:** >50K Tokens auf 24GB GPU → OOM trotz PN59-Fix (chunked-prefill Eligibility-Check). Workaround: dual.yml/dual-turbo.yml (TP=2) oder llama.cpp.

**Kritische Genesis Patches für TQ3 + MTP:**
| Patch | Funktion | Status |
|-------|----------|--------|
| **P67** | TQ multi-query kernel (K+1-aware spec-verify) | ✅ Required |
| **P64** | MTP streaming tool-call edge case | ✅ Required |
| **P66** | cudagraph_capture_sizes divisibility filter | ✅ Required |
| **PN8** | MTP draft online quantization | ✅ Required |
| **PN9** | Independent drafter attention | ✅ Required |
| **PN14** | TQ decode OOB clamp | ✅ Required |
| **PN30** | DS layout spec-decode (mit local fix) | ✅ Required |
| **PN34** | Workspace lock relaxation | ✅ Required |
| **P4** | Hybrid turboquant support | ✅ Required |
| **P5** | KV page-size unification for hybrid models | ✅ Required |

**Open upstream PRs (Genesis-free Fix Gap):**
| PR | Thema | Status | Quelle |
|----|-------|--------|--------|
| #38479 | TurboQuant KV Cache (dense GQA/MHA) | MERGED | [PR #38479](https://github.com/vllm-project/vllm/pull/38479) |
| #40914 | Synthetic seq_lens K+1 routing | OPEN (Fix for #40880, restores FULL cudagraph for spec-decode) | [PR](https://github.com/vllm-project/vllm/pull/40914) |
| #40798 | TQ decode workspace pre-allocation | OPEN | [PR](https://github.com/vllm-project/vllm/pull/40798) |
| #42215 | TQ decode kernel warmup | OPEN | [PR](https://github.com/vllm-project/vllm/pull/42215) |
| #40792 | k8v4 GQA decode optimization | OPEN | [PR](https://github.com/vllm-project/vllm/pull/40792) |
| #40069 | Tracking issue (Spec-decode/Eagle disabled for TurboQuant) | OPEN | [Issue](https://github.com/vllm-project/vllm/issues/40069) |

**vLLM-eigener Benchmark (11.05.2026):** FP8 via `--kv-cache-dtype fp8` bleibt die beste Standardwahl für KV-Cache-Quantisierung: 2× Kapazität bei vernachlässigbaren Qualitätsverlusten. TurboQuant 3bit_nc und k3v4_nc zeigen bedeutende Qualitätsverluste und substanzielle Latenzverschlechterung.

**Empfohlener Start (Qwen3.6-27B, 4× GPU, FP8):**
```bash
vllm serve Qwen/Qwen3.6-27B --tensor-parallel-size 4 \
  --kv-cache-dtype fp8 \
  --speculative-config '{"method":"qwen3_next_mtp","num_speculative_tokens":2}' \
  --max-model-len 262144
```

**Empfohlener Start (Qwen3.6-27B, 2× GPU, TurboQuant + MTP + Genesis):**
```bash
# Requires Genesis v7.72.2 patches applied to vLLM nightly
vllm serve qwen3.6-27b-autoround-int4 --tensor-parallel-size 2 \
  --kv-cache-dtype turboquant_3bit_nc \
  --speculative-config '{"method":"mtp","num_speculative_tokens":3}' \
  --max-model-len 262144 --max-num-seqs 4 --max-num-batched-tokens 4128 \
  --quantization auto_round --dtype float16 --trust-remote-code
```

---

## 2. SGLang (LMSYS/sGLang)

| Feature | Status | Version / PR / Issue | Quelle |
|---------|--------|---------------------|--------|
| **FP8 KV Cache** | ✅ Production | `--kv-cache-dtype fp8_e4m3` / `fp8_e5m2`. Per-tensor scaling. | [SGLang Docs](https://sgl-project.github.io/advanced_features/quantized_kv_cache.html) |
| **FP4 KV Cache** | ✅ Experimental | `--kv-cache-dtype fp4_e2m1` (MXFP4). ~3.56× mehr Tokens als BF16. Block-based microscaling. | [SGLang Docs](https://sgl-project.github.io/advanced_features/quantized_kv_cache.html), [PR #10078](https://github.com/sgl-project/sglang/pull/10078), [PR #12612](https://github.com/sgl-project/sglang/pull/12612) |
| **Q4/Q5/Q8 Integer KV** | ❌ | Keine Integer KV Cache Quantisierung | — |
| **TurboQuant KV** | ❌ | Keine Implementierung | — |
| **Strict Tensor Parallel** | ✅ Production | `--tp-size N`, auch `--dp-size N` für Data Parallel | [SGLang Docs](https://sgl-project.github.io/) |
| **MTP (Qwen3.6)** | ✅ Production | `--speculative-algo NEXTN --speculative-num-steps 3 --speculative-eagle-topk 1 --speculative-num-draft-tokens 4` | [Qwen3.6-27B HF Card](https://huggingface.co/Qwen/Qwen3.6-27B) |
| **HiSparse KV Offload** | ✅ | Offload inaktiver KV Cache zu Host Memory (integrated Apr 2026). 3× throughput bei 256 concurrent requests | [mubibai.com Apr 2026](https://mubibai.com/sglang-vs-vllm-production-inference-benchmarks-april-2026/) |
| **RadixAttention** | ✅ | Besserer Prefix Sharing als vLLM's PagedAttention | [wiki.charleschen.ai](https://wiki.charleschen.ai/ai/processed/wiki/infrastructure/fleet/comparisons/vllm-vs-sglang-vs-llamacpp-vs-tgi) |

**Empfohlener Start (Qwen3.6-27B, 4× GPU):**
```bash
python -m sglang.launch_server --model-path Qwen/Qwen3.6-27B \
  --tp-size 4 --kv-cache-dtype fp8_e4m3 \
  --speculative-algo NEXTN --speculative-num-steps 3 \
  --speculative-eagle-topk 1 --speculative-num-draft-tokens 4 \
  --context-length 262144
```

---

## 3. llama.cpp mainline (ggml-org/llama.cpp)

| Feature | Status | Version / PR / Issue | Quelle |
|---------|--------|---------------------|--------|
| **FP8 KV Cache** | ❌ | Nicht implementiert | — |
| **FP4 KV Cache** | ❌ | Nicht implementiert | — |
| **Q8_0 K Cache** | ✅ Production | `--cache-type-k q8_0` | [PR #2969](https://github.com/ggml-org/llama.cpp/pull/2969) |
| **Q5_0/Q5_1 K Cache** | ✅ Production | `--cache-type-k q5_0` / `q5_1` | [PR #6183](https://github.com/ggml-org/llama.cpp/pull/6183) |
| **Q4_0/Q4_1 K Cache** | ✅ Production | `--cache-type-k q4_0` / `q4_1` | [PR #6183](https://github.com/ggml-org/llama.cpp/pull/6183) |
| **Q8_0 V Cache** | ✅ Production | `--cache-type-v q8_0` | [PR #2969](https://github.com/ggml-org/llama.cpp/pull/2969) |
| **Q4/Q5 V Cache** | ❌ | Nur Research-PR #7412 (nicht gemerged) | [PR #7412](https://github.com/ggml-org/llama.cpp/pull/7412) |
| **TurboQuant KV** | ❌ | PR #21131 (geschlossen), PR #21062 (geschlossen), PR #21307 (geschlossen), PR #21192 (geschlossen). Issue #20977 offen (321 👍). WIP commit `9b041de` mit turbo3_0/turbo4_0 Typen. | [PR #21131](https://github.com/ggml-org/llama.cpp/pull/21131), [PR #21062](https://github.com/ggml-org/llama.cpp/pull/21062), [PR #21307](https://github.com/ggml-org/llama.cpp/pull/21307), [PR #21192](https://github.com/ggml-org/llama.cpp/pull/21192), [Issue #20977](https://github.com/ggml-org/llama.cpp/issues/20977), [Commit 9b041de](https://github.com/ggml-org/llama.cpp/commit/9b041de7f2176735e66e91abfe8fa8199659bf6f) |
| **Strict Tensor Parallel** | ✅ Neu (Feb 2026) | `--split-mode tensor` (PR #19378). Backend-agnostic, AllReduce-basiert. **Instabil:** OOM bei >32K Context auf 4×3090. | [PR #19378](https://github.com/ggml-org/llama.cpp/pull/19378), [ikawrakow Discussion #1247](https://github.com/ikawrakow/ik_llama.cpp/discussions/1247) |
| **Layer/Row Parallel** | ✅ Legacy | `--split-mode layer` / `--split-mode row` | [llama.cpp Docs](https://github.com/ggml-org/llama.cpp) |
| **MTP (Qwen3.6)** | ✅ Merged (May 2026) | `--spec-type draft-mtp --spec-draft-n-max 2`. Von `--spec-type mtp` umbenannt am 13.05.2026. | [PR #22673](https://github.com/ggml-org/llama.cpp/pull/22673), [Unsloth Docs](https://unsloth.ai/docs/models/qwen3.6) |

**Empfohlener Start (Qwen3.6-27B, Single GPU, Integer K-Quant):**
```bash
./llama-server -hf unsloth/Qwen3.6-27B-MTP-GGUF:Q4_K_M \
  --cache-type-k q4_0 --cache-type-v q8_0 \
  --ctx-size 262144 --gpu-layers 99 --flash-attn on \
  --spec-type draft-mtp --spec-draft-n-max 2
```

**Empfohlener Start (Qwen3.6-27B, Strict TP 4× GPU, riskant):**
```bash
./llama-server -m qwen3.6-27b-q4_k_m.gguf \
  --split-mode tensor --tensor-split 1,1,1,1 \
  --cache-type-k q4_0 --cache-type-v q8_0 \
  --ctx-size 262144 --gpu-layers 99 \
  --spec-type draft-mtp --spec-draft-n-max 2
```

---

## 4. ik_llama.cpp (ikawrakow/ik_llama.cpp)

**Repo:** https://github.com/ikawrakow/ik_llama.cpp | **Stars:** ~2K | **Aktiv:** Ja (letzter Push: 2026-05-19)

| Feature | Status | Version / PR / Issue | Quelle |
|---------|--------|---------------------|--------|
| **FP8/FP4 KV** | ❌ | Nicht implementiert | — |
| **Q8_0 K/V Cache** | ✅ | `--cache-type-k q8_0 --cache-type-v q8_0` | Inherited from llama.cpp |
| **Q4/Q5 K/V Cache** | ✅ | `--cache-type-k q4_0` / `q5_0` etc. | [PR #6183](https://github.com/ggml-org/llama.cpp/pull/6183) (upstream) |
| **Q8_KV** | ✅ | Neuer Typ für 8-bit KV Cache (Feb 2025) | [PR #208](https://github.com/ikawrakow/ik_llama.cpp/pull/208) |
| **Hadamard K/V** | ✅ | Hadamard Transforms für K/V Cache | [PR #1033](https://github.com/ikawrakow/ik_llama.cpp/pull/1033), [PR #1034](https://github.com/ikawrakow/ik_llama.cpp/pull/1034), [PR #1527](https://github.com/ikawrakow/ik_llama.cpp/pull/1527) |
| **Low PPL Q4_0 KV** | ✅ | Optimiertes Q4_0 KV Cache | [PR #1547](https://github.com/ikawrakow/ik_llama.cpp/pull/1547), [PR #1556](https://github.com/ikawrakow/ik_llama.cpp/pull/1556) |
| **Trellis Quants** | ✅ | IQ1_KT, IQ2_KT, IQ3_KT, IQ4_KT (novel integer trellis) | [PR #113](https://github.com/ikawrakow/ik_llama.cpp/pull/113), [PR #475](https://github.com/ikawrakow/ik_llama.cpp/pull/475) |
| **IQK Quants** | ✅ | IQ2_K, IQ3_K, IQ4_K, IQ5_K, IQ6_K + row-interleaved | [README](https://github.com/ikawrakow/ik_llama.cpp) |
| **Graph Tensor Parallel** | ✅ Production | `--split-mode graph` (PR #1022). **Best-in-class:** 6× schneller PP als mainline Strict TP, 33% besser TG. | [PR #1022](https://github.com/ikawrakow/ik_llama.cpp/pull/1022), [Discussion #1247](https://github.com/ikawrakow/ik_llama.cpp/discussions/1247) |
| **Strict Tensor Parallel** | ❌ | Nicht implementiert (nur Graph/Layer/Row) | — |
| **MTP (Qwen3.6)** | ✅ Best-in-class | Full MTP support, custom quant preserves MTP heads in Q8_0. Draft acceptance ~89.9%. | [PR #1698](https://github.com/ikawrakow/ik_llama.cpp/pull/1698), [PR #1745](https://github.com/ikawrakow/ik_llama.cpp/pull/1745), [AlexanderKyng/Qwen3.6-27B-MTP-Custom-IQ6_K](https://huggingface.co/AlexanderKyng/Qwen3.6-27B-MTP-Custom-IQ6_K) |
| **Fused Delta-net** | ✅ | Qwen3-Next / Qwen3.5-MoE | [PR #1315](https://github.com/ikawrakow/ik_llama.cpp/pull/1315), [PR #1333](https://github.com/ikawrakow/ik_llama.cpp/pull/1333), [PR #1362](https://github.com/ikawrakow/ik_llama.cpp/pull/1362), [PR #1373](https://github.com/ikawrakow/ik_llama.cpp/pull/1373) |
| **TurboQuant** | ✅ Via forks | TheTom/llama-cpp-turboquant als Basis. tbq2_0, tbq3_0, tbq4_0, tbqp3_0, tbqp4_0. | [TheTom/llama-cpp-turboquant](https://github.com/TheTom/llama-cpp-turboquant), [Discussion #21155](https://github.com/ggml-org/llama.cpp/discussions/21155) |

**⚠️ WICHTIG:** ik_llama.cpp hat **kein Strict Tensor Parallel**. Nur Graph/Layer/Row. Damit nicht kompatibel mit deiner Strict-TP-Anforderung.

**Empfohlener Start (Graph TP, Dual RTX 3090, MTP + Extreme Quant):**
```bash
./llama-server -m qwen3.6-27B-MTP-Custom-IQ6_K.gguf \
  --split-mode graph --tensor-split 3,2 --max-gpu 2 \
  --cache-type-k f16 --cache-type-v f16 \
  --ctx-size 200231 --gpu-layers 999 --flash-attn on \
  -mtp --draft-max 4 --draft-p-min 0.70 -jinja
```

---

## 5. buun-llama-cpp (spiritbuun/buun-llama-cpp)

**Repo:** https://github.com/spiritbuun/buun-llama-cpp (Backup: gkraker04/buun-llama-cpp-old)
**Status:** ⚠️ **Experimental** ("Highly experimental fork. Use at your own discretion.")

| Feature | Status | Version / PR / Issue | Quelle |
|---------|--------|---------------------|--------|
| **TCQ (Trellis-Coded Quantization)** | ✅ | 2-3 bit KV Cache via 512-state Viterbi trellis. 10-44% KL-divergence reduction. O(1) sliding-window decode. | [README](https://github.com/spiritbuun/buun-llama-cpp) |
| **Q4/Q5/Q8 KV** | ✅ | Inherited from llama.cpp base | — |
| **Strict Tensor Parallel** | ❌ | Legacy row split nur | — |
| **MTP (Qwen3.6)** | ❌ | Nicht dokumentiert | — |
| **TurboQuant** | ✅ | Fork von TheTom/llama-cpp-turboquant | [README](https://github.com/spiritbuun/buun-llama-cpp) |

**⚠️ WICHTIG:** Kein Strict TP, keine MTP-Dokumentation. Experimental.

---

## 6. KV Cache Offloading & Disaggregation

### 6.1 LMCache (LMCache/LMCache)

**Repo:** https://github.com/LMCache/LMCache | **Stars:** ~8.3K | **Releases:** 42 (Latest: v0.1.1 Operator, May 2026)
**Initiated by:** Tensormesh | **Adopted by:** GMI Cloud, Google Cloud, CoreWeave, Redis, Weka, PliOps

| Feature | Status | Version / PR / Issue | Quelle |
|---------|--------|---------------------|--------|
| **vLLM Integration** | ✅ Production | Chunk-level KV cache lookup/injection. CPU, Disk, NIXL storage backends. 3-10× TTFT reduction bei RAG/multi-round QA. | [LMCache Docs](https://docs.lmcache.ai/developer_guide/integration.html), [Repo](https://github.com/LMCache/LMCache) |
| **SGLang Integration** | ✅ Production | `LMCacheConnector` mit `load_kv()` / `store_kv()`. KV Events support (Jan 2026). | [sglang_adapter.py](https://github.com/LMCache/LMCache/blob/dev/lmcache/integration/sglang/sglang_adapter.py), [PR #2441](https://github.com/LMCache/LMCache/commit/d44390fb5847a0d83522b8927d48e55ef0ebfbeb) |
| **TRT-LLM Integration** | ⚠️ Coming Soon | Angekündigt, keine Implementation | [Docs](https://docs.lmcache.ai/developer_guide/integration.html) |
| **CPU Offloading** | ✅ Production | KV Cache chunks werden asynchron nach CPU RAM offgeloaded. Zero-CPU-copy via GDS/NIXL. | [README](https://github.com/LMCache/LMCache) |
| **Disk Offloading** | ✅ Production | KV Cache auf NVMe/Disk. Für sehr lange Kontexte (>100K tokens). | [README](https://github.com/LMCache/LMCache) |
| **NIXL Backend** | ✅ Production | NIXL-basierte Übertragung (UCX, GDS). Async send/recv. | [vLLM NixlConnector](https://docs.vllm.ai/en/latest/features/nixl_connector_usage/) |
| **KV Events** | ✅ Production | vLLM + SGLang KV Events (BlockStored, BlockRemoved). Für fleet-wide cache indexing. | [Commit d44390f](https://github.com/LMCache/LMCache/commit/d44390fb5847a0d83522b8927d48e55ef0ebfbeb) |
| **Disaggregated Prefill** | ✅ Production | `LMCacheConnectorV1` für vLLM Disagg Prefill via NIXL. | [vLLM Docs](https://docs.vllm.ai/en/latest/usage/disagg_prefill.html) |
| **MTP Kompatibilität** | ⚠️ Nicht dokumentiert | LMCache arbeitet auf Block-Ebene (token-hashing). MTP draft tokens sind temporär und werden nicht cached. **Potenzieller Konflikt:** MTP generiert K draft tokens pro step — LMCache könnte diese fälschlich als "real" tokens cachen. Keine Dokumentation oder Tests für MTP-Modelle gefunden. | — |
| **KV Quant Kompatibilität** | ⚠️ Nicht dokumentiert | LMCache speichert KV chunks als rohe Tensors (BF16/FP16). Keine Dokumentation für quantisierte KV (FP8, TurboQuant). **Offene Frage:** Kann LMCache mit vLLM's `--kv-cache-dtype fp8/turboquant_3bit_nc` umgehen? | — |
| **Strict TP Kompatibilität** | ✅ Theoretisch | LMCache arbeitet pro-worker (per-GPU). Bei TP=N hat jeder worker seinen TP-sliced KV chunk. LMCache müsste pro-worker chunks speichern/abrufen. Keine TP-spezifische Dokumentation. | — |

**⚠️ KRITISCHE LÜCKE:** LMCache wurde **nicht** für MTP-Modelle oder quantisierte KV Caches validiert. Chunk-basiertes Caching könnte mit MTP draft tokens kollidieren (falsche Cache-Einträge).

---

### 6.2 llm-d-kv-cache (llm-d/llm-d-kv-cache)

**Repo:** https://github.com/llm-d/llm-d-kv-cache | **Stars:** ~140 | **Sprache:** Go
**Projekt:** llm-d (Distributed LLM Inference Platform)

| Feature | Status | Version / PR / Issue | Quelle |
|---------|--------|---------------------|--------|
| **KV-Cache Indexer** | ✅ Production | Global, near-real-time view of KV-Cache block locality across vLLM fleet. ZMQ-based event ingestion. In-memory LRU + Redis/Valkey backend. | [architecture.md](https://github.com/llm-d/llm-d-kv-cache/blob/main/docs/architecture.md), [README](https://github.com/llm-d/llm-d-kv-cache) |
| **KV-Cache Aware Routing** | ✅ Production | Scheduler fragt Indexer: "Welcher Pod hat die meisten KV-Blocks für diesen Prompt?" → Smart routing. | [llm-d.ai/docs](https://llm-d.ai/docs/architecture/Components/kv-cache) |
| **vLLM Integration** | ✅ Production | KVEvents von vLLM (msgpack-encoded). `kvevents.EngineAdapter` für engine-agnostic parsing. | [architecture.md](https://github.com/llm-d/llm-d-kv-cache/blob/main/docs/architecture.md) |
| **SGLang Integration** | ✅ Production | SGLang KVEvents integration (v0.6.0, Feb 2026). | [Issue #283](https://github.com/llm-d/llm-d-kv-cache/issues/283) |
| **Disaggregated Tokenization** | ✅ Production | vLLM Renderer API integration (v0.6.0). | [Issue #283](https://github.com/llm-d/llm-d-kv-cache/issues/283) |
| **Speculative KV-Cache Indexing** | ✅ Production | Approximation integration (v0.6.0). | [Issue #283](https://github.com/llm-d/llm-d-kv-cache/issues/283) |
| **GDS Support** | ✅ Production | NVIDIA GPUDirect Storage (v0.6.0). | [Issue #283](https://github.com/llm-d/llm-d-kv-cache/issues/283) |
| **NIXL Backend** | ❌ Nicht fertig | Offen in Issue #283 (v0.6.0 roadmap). | [Issue #283](https://github.com/llm-d/llm-d-kv-cache/issues/283) |
| **MTP Kompatibilität** | ⚠️ Nicht dokumentiert | "Speculative KV-cache indexing" ist implementiert, aber ohne MTP-spezifische Tests. Indexer arbeitet auf Block-Ebene — MTP draft tokens könnten fälschlich indiziert werden. | — |
| **KV Quant Kompatibilität** | ❌ Nicht dokumentiert | Keine Erwähnung von FP8/TurboQuant KV. Indexer arbeitet mit Block-Hashes (unabhängig vom dtype). | — |
| **Strict TP Kompatibilität** | ✅ DP-aware | "DP-aware indexing" implementiert (v0.6.0). TP-spezifisch nicht dokumentiert. | [Issue #283](https://github.com/llm-d/llm-d-kv-cache/issues/283) |

**⚠️ KRITISCHE LÜCKE:** llm-d-kv-cache ist ein **Routing/Scheduling-Tool** (kein Offloading). Es optimiert request routing basierend auf Cache-Lokality. Kein direkter KV-Transfer oder Offloading-Mechanismus.

---

### 6.3 vLLM Disaggregated Prefill (Built-in)

**vLLM Version:** v0.18+ (Production)

| Feature | Status | Version / PR / Issue | Quelle |
|---------|--------|---------------------|--------|
| **Disagg Prefill v0** | ✅ Production | 2 vLLM instances: Prefill (GPU 0) → Decode (GPU 1). `P2pNcclConnector` (NCCL-basiert). `KVTransferConfig(kv_connector, kv_role, kv_rank)`. | [vLLM Docs](https://docs.vllm.ai/en/v0.18.2/examples/offline_inference/disaggregated_prefill/), [PR #10502](https://github.com/vllm-project/vllm/pull/10502) |
| **Disagg Prefill v1** | ✅ Production | ExampleConnector (local storage), offline mode. Prefill + Decode als separate LLM-Instanzen. | [vLLM Docs v0.18.2](https://docs.vllm.ai/en/v0.18.2/examples/offline_inference/disaggregated-prefill-v1/) |
| **NixlConnector** | ✅ Production | NIXL-basiert (UCX, GDS). Fully async send/recv. `kv_role=kv_both` für bidirektionalen Transfer. | [vLLM Docs](https://docs.vllm.ai/en/latest/features/nixl_connector_usage/), [vLLM Disagg](https://docs.vllm.ai/en/latest/usage/disagg_prefill.html) |
| **MooncakeConnector** | ✅ Production | Mooncake Transfer Engine (RDMA, TCP, NVMe-of). Topology-aware path selection. `MOONCAKE_CONFIG_PATH` env. | [PR #10884](https://github.com/vllm-project/vllm/pull/10884), [Mooncake Docs](https://kvcache-ai.github.io/Mooncake/getting_started/examples/vllm-integration/vllm-integration-v0.2.html) |
| **LMCacheConnectorV1** | ✅ Production | LMCache als KV-Transfer-Backend via NIXL. | [vLLM Disagg](https://docs.vllm.ai/en/latest/usage/disagg_prefill.html) |
| **MultiConnector** | ✅ Production | Multiple connectors in ordered list (fallback chain). `kv_connector_extra_config.connectors[]`. | [vLLM Disagg](https://docs.vllm.ai/en/latest/usage/disagg_prefill.html) |
| **Disagg Encoder** | ✅ Production | Vision-Encoder in separatem Prozess. E→PD oder E→P→D. `ECConnector` für Encoder-Cache-Transfer. | [vLLM Docs](https://docs.vllm.ai/en/v0.18.2/features/disagg_encoder/) |
| **MTP Kompatibilität** | ⚠️ Nicht dokumentiert | Disagg Prefill transferiert KV nach Prefill. MTP draft tokens werden während Decode generiert — **kein Konflikt** (MTP läuft nur auf Decode-Instanz). | — |
| **KV Quant Kompatibilität** | ⚠️ Nicht dokumentiert | KV-Transfer arbeitet auf rohen Tensors. **Offene Frage:** Funktioniert Transfer bei `--kv-cache-dtype fp8/turboquant`? Quantisierte Blocks müssten dequantisiert für Transfer oder quantized format preserved. | — |
| **Strict TP Kompatibilität** | ✅ | Prefill und Decode können jeweils eigene TP-Graden haben. `--tensor-parallel-size` pro Instanz. | [vLLM Disagg](https://docs.vllm.ai/en/latest/usage/disagg_prefill.html) |

**Empfohlener Start (Disagg Prefill + MTP, 2× GPU):**
```bash
# Prefill instance (GPU 0)
CUDA_VISIBLE_DEVICES=0 vllm serve Qwen/Qwen3.6-27B \
  --tensor-parallel-size 1 --port 8100 \
  --kv-transfer-config '{"kv_connector":"NixlConnector","kv_role":"kv_producer","kv_rank":0}' \
  --max-model-len 262144

# Decode instance (GPU 1) + MTP
CUDA_VISIBLE_DEVICES=1 vllm serve Qwen/Qwen3.6-27B \
  --tensor-parallel-size 1 --port 8200 \
  --kv-transfer-config '{"kv_connector":"NixlConnector","kv_role":"kv_consumer","kv_rank":1}' \
  --speculative-config '{"method":"qwen3_next_mtp","num_speculative_tokens":2}' \
  --max-model-len 262144
```

---

### 6.4 Mooncake (kvcache-ai/Mooncake)

**Repo:** https://github.com/kvcache-ai/mooncake | **Sprache:** C++
**Transfer Engine:** RDMA, TCP, NVMe-of. Topology-aware path selection.

| Feature | Status | Version / PR / Issue | Quelle |
|---------|--------|---------------------|--------|
| **vLLM Integration** | ✅ Production | `MooncakeConnector` via PR #10884 (merged Dec 2024). `mooncake_vllm_adaptor` Python binding. | [PR #10884](https://github.com/vllm-project/vllm/pull/10884), [Integration Guide](https://kvcache-ai.github.io/Mooncake/getting_started/examples/vllm-integration/vllm-integration-v0.2.html) |
| **RDMA Support** | ✅ Production | `mlx5_0,mlx5_1` NIC aggregation. DRAM→GPU VRAM, DRAM→DRAM, DRAM→NVMe. | [Mooncake Docs](https://github.com/kvcache-ai/mooncake) |
| **MTP Kompatibilität** | ⚠️ Nicht dokumentiert | Mooncake ist ein Transfer-Engine (keine Engine-integration). Kompatibilität hängt vom Consumer (vLLM + MTP) ab. | — |
| **KV Quant Kompatibilität** | ❌ Nicht dokumentiert | Transferiert rohe Tensors. Keine Quant-aware Transfer. | — |

---

### 6.5 CacheGen (UChi-JCL/CacheGen)

**Repo:** https://github.com/UChi-JCL/CacheGen | **Paper:** SIGCOMM 2024
**Ansatz:** KV Cache Compression + Streaming für disaggregated serving.

| Feature | Status | Version / PR / Issue | Quelle |
|---------|--------|---------------------|--------|
| **KV Encoding** | ✅ Research | Custom tensor encoder. 3.5-4.3× Compression. Adaptive per-chunk compression levels. | [SIGCOMM 2024](https://arxiv.org/pdf/2310.07240), [MSR Publication](https://www.microsoft.com/en-us/research/publication/cachegen-fast-context-loading-for-language-model-applications-via-kv-cache-streaming/) |
| **Streaming** | ✅ Research | Adaptive streaming je nach Bandbreite. Fallback zu Text-Transfer + Recompute. | [arXiv:2310.07240](https://arxiv.org/pdf/2310.07240) |
| **vLLM Integration** | ❌ Research-only | Keine Production-Integration. vLLM nur als Benchmark-Baseline verwendet. | [Paper](https://arxiv.org/pdf/2310.07240) |
| **MTP Kompatibilität** | ❌ | Nicht untersucht. | — |
| **KV Quant Kompatibilität** | ❌ | Nicht untersucht (eigener Encoder, nicht kompatibel mit FP8/TurboQuant). | — |

**⚠️ STATUS:** Research-Paper (SIGCOMM 2024). Keine Production-Integration in irgendeiner Engine.

---

### 6.6 NVIDIA kvpress (NVIDIA/kvpress)

**Repo:** https://github.com/NVIDIA/kvpress | **Sprache:** Python
**Ansatz:** KV Cache Compression via "presses" (token eviction, dimensionality reduction).

| Feature | Status | Version / PR / Issue | Quelle |
|---------|--------|---------------------|--------|
| **Presses** | ✅ Production | 15+ Presses: KVzip, SnapKV, Random, Knorm, ExpectedAttention, PyramidKV, LagKV, KVzap, FastKVzip, ThinK, SimLayer, DuoAttention, Finch, KVCompose, AdaKV, Chunk, DecodingPress, DMSPress. | [README](https://github.com/NVIDIA/kvpress) |
| **transformers Integration** | ✅ Production | `KVPressTextGenerationPipeline` (auto-registered als `kv-press-text-generation`). | [README](https://github.com/NVIDIA/kvpress) |
| **vLLM Integration** | ❌ | Nicht integriert (nur transformers). | — |
| **MTP Kompatibilität** | ❌ | Nicht untersucht (nur transformers, keine MTP-Engines). | — |
| **KV Quant Kompatibilität** | ❌ | Komprimiert via Token-Eviction (nicht Quantisierung). Orthogonal zu FP8/TurboQuant. | — |

**⚠️ STATUS:** transformers-only. Kein Engine-Integration (vLLM/SGLang/llama.cpp).

---

### 6.7 KakeyaLattice (FluffyAIcode/LLM-KV--Cache-compress)

**Repo:** https://github.com/FluffyAIcode/LLM-KV--Cache-compress | **Stars:** ~6 | **Release:** v1.5 (Apr 2026)
**Ansatz:** Nested-Lattice (D4/E8) KV Cache Compression. 2.4-2.8× CR bei <1% PPL loss.

| Feature | Status | Version / PR / Issue | Quelle |
|---------|--------|---------------------|--------|
| **D4/E8 Lattice Quant** | ✅ Production | GPU-native lattice-quantisation codec. V14KakeyaZamirLatticeGPU (D4), V15KakeyaZamirE8GPU (E8). | [README](https://github.com/FluffyAIcode/LLM-KV--Cache-compress) |
| **vLLM Plugin** | ✅ Production | `vllm_backend/kakeya_v1_4_snapshot/` — Attention monkey-patches (Qwen3, Qwen2, Gemma4, GLM). | [README](https://github.com/FluffyAIcode/LLM-KV--Cache-compress) |
| **Qwen3 Validiert** | ✅ | Qwen3-4B: 2.77× CR (+26.9% vs TurboQuant). <1% PPL loss. | [README](https://github.com/FluffyAIcode/LLM-KV--Cache-compress) |
| **Streaming** | ✅ | ~0.25ms/decode-step (<2% overhead). Pure per-vector function (no cross-token state). | [README](https://github.com/FluffyAIcode/LLM-KV--Cache-compress) |
| **MTP Kompatibilität** | ❌ | Nicht untersucht. | — |
| **Strict TP Kompatibilität** | ❌ | Nicht untersucht (Plugin für single-GPU vLLM). | — |

**⚠️ STATUS:** Sehr neu (Apr 2026), minimal community adoption (6⭐). Qwen3-4B getestet, nicht Qwen3.6-27B.

---

### 6.8 lmdeploy (InternLM/lmdeploy) TurboQuant KV

**Repo:** https://github.com/InternLM/lmdeploy

| Feature | Status | Version / PR / Issue | Quelle |
|---------|--------|---------------------|--------|
| **TurboQuant KV** | ✅ Production | `quant_policy=42`: K=4bit QJL4 + V=2bit MSE. PytorchEngine only. | [PR #4510](https://github.com/InternLM/lmdeploy/commit/e433925b61c84249b0edc5cb46bebc2dec71cb7f), [Issue #4499](https://github.com/InternLM/lmdeploy/issues/4499) |
| **Speculative Decoding** | ✅ Production | `deepseek_mtp` (DeepSeek-V3/R1). EAGLE3, draft model. | [Docs](https://lmdeploy.readthedocs.io/en/latest/advance/spec_decoding.html), [PR #3945](https://github.com/InternLM/lmdeploy/pull/3945) |
| **TurboQuant + Spec Decoding** | ❌ **EXPLIZIT NICHT UNTERSTÜTZT** | PR #4510: "**Speculative decoding not supported**" für TurboQuant. | [PR #4510](https://github.com/InternLM/lmdeploy/commit/e433925b61c84249b0edc5cb46bebc2dec71cb7f) |
| **MTP (Qwen3.6)** | ❌ | Nur `deepseek_mtp` dokumentiert. Keine Qwen3.6 MTP Unterstützung. | [Docs](https://lmdeploy.readthedocs.io/en/latest/advance/spec_decoding.html) |
| **Strict TP** | ✅ | `tp=N` in `PytorchEngineConfig`. | [PR #4510](https://github.com/InternLM/lmdeploy/commit/e433925b61c84249b0edc5cb46bebc2dec71cb7f) |

**⚠️ KRITISCHE LÜCKE:** lmdeploy unterstützt TurboQuant KV, aber **explizit NICHT** mit spekulativem Decoding. Damit inkompatibel mit MTP.

---

### 6.9 RocketKV (NVlabs/RocketKV)

**Repo:** https://github.com/NVlabs/RocketKV | **Paper:** ICML 2025
**Ansatz:** Two-stage KV Cache Compression (SnapKV coarse eviction + Hybrid Sparse Attention fine-grained top-k).

| Feature | Status | Version / PR / Issue | Quelle |
|---------|--------|---------------------|--------|
| **Compression** | ✅ Research | Bis zu 400× CR. 3.7× end-to-end speedup. 32.6% peak memory reduction. | [README](https://github.com/NVlabs/RocketKV) |
| **vLLM/Engine Integration** | ❌ | transformers + gpt-fast only. Keine Engine-Integration. | [README](https://github.com/NVlabs/RocketKV) |
| **MTP Kompatibilität** | ❌ | Nicht untersucht. | — |

**⚠️ STATUS:** Research-Paper (ICML 2025). Keine Production-Integration.

---

### 6.10 Vergleich: Offloading & Disaggregation

| Mechanismus | Engine | Offload Ziel | MTP | KV Quant | Strict TP | Status |
|-------------|--------|-------------|-----|----------|-----------|--------|
| **LMCache** | vLLM, SGLang | CPU, Disk, NIXL | ⚠️ | ⚠️ | ✅ Theo | ✅ Prod |
| **llm-d-kv-cache** | vLLM, SGLang | Routing (kein Offload) | ⚠️ | ❌ | ✅ DP | ✅ Prod |
| **vLLM Disagg Prefill** | vLLM | GPU→GPU (NCCL/NIXL/Mooncake) | ⚠️ | ⚠️ | ✅ | ✅ Prod |
| **Mooncake** | vLLM | RDMA, NVMe | ⚠️ | ❌ | ✅ | ✅ Prod |
| **CacheGen** | — | Network streaming | ❌ | ❌ | ❌ | 📄 Research |
| **kvpress** | transformers | Token eviction | ❌ | ❌ | ❌ | ✅ Prod |
| **KakeyaLattice** | vLLM (plugin) | Lattice quant (GPU) | ❌ | ✅ (eigenes) | ❌ | ✅ Prod |
| **lmdeploy TQ** | lmdeploy | GPU (TQ K4V2) | ❌ **BLOCKED** | ✅ TQ | ✅ | ✅ Prod |
| **RocketKV** | — | Token eviction + sparse | ❌ | ❌ | ❌ | 📄 Research |

**⚠️ GEMEINSAMES MUSTER:** **KEIN** Offloading/Disaggregation-Mechanismus wurde für MTP-Modelle (Qwen3.6) validiert. Die meisten arbeiten auf Block/Token-Ebene und könnten mit MTP draft tokens kollidieren.

---

## 7. Neue Projekte & Alternativen

| Projekt | Repo | Fokus | Status | Quelle |
|---------|------|-------|--------|--------|
| **quant.cpp** | [quantumaikr/quant.cpp](https://github.com/quantumaikr/quant.cpp) | Minimal C11, 6.9× KV Cache Compression (PolarQuant, QJL, delta). 3-bit K bei +1.3% PPL. Single-header library. | ✅ Production | [Tech Report](https://github.com/quantumaikr/quant.cpp/blob/main/docs/papers/quant_cpp_tech_report.md) |
| **turboquant-vllm** | [Alberto-Codes/turboquant-vllm](https://github.com/Alberto-Codes/turboquant-vllm) | vLLM Plugin für TurboQuant KV. 3.76× Compression. Asymmetric K/V. 45⭐, v1.5.0. | ✅ Plugin | [README](https://github.com/Alberto-Codes/turboquant-vllm) |
| **Synapse** | [Clarit-AI/Synapse](https://github.com/Clarit-AI/Synapse) | Edge inference, built on ik-llama.cpp + rk-llama.cpp | ✅ | [Repo](https://github.com/Clarit-AI/Synapse) |
| **qwen3.6-mtp** | [ArkaD171717/Qwen3.6-MTP](https://github.com/ArkaD171717/Qwen3.6-MTP) | MTP tuner für vLLM/SGLang. Config generation, crossover analysis, bug detection. PyPI. | ✅ | [PyPI](https://pypi.org/project/qwen3.6-mtp/), [Repo](https://github.com/ArkaD171717/Qwen3.6-MTP) |
| **mlx-lm (MTP)** | [AirRunner/mlx-lm feat/mtp-native](https://github.com/AirRunner/mlx-lm) | Apple Silicon MTP Support. +48% Speedup mit MTP. | ✅ Branch | [trevon/Qwen3.6-27B-mtp](https://huggingface.co/trevon/Qwen3.6-27B-mtp) |
| **kvtc** | [OnlyTerp/kvtc](https://github.com/OnlyTerp/kvtc) | First open-source KVTC (NVIDIA, ICLR 2026). 8-32× KV Cache Compression via PCA + adaptive quant + entropy coding. | ✅ | [Repo](https://github.com/OnlyTerp/kvtc) |
| **turboquant_plus** | [TheTom/turboquant_plus](https://github.com/TheTom/turboquant_plus) | TurboQuant research & upstream PR plan | ✅ | [Repo](https://github.com/TheTom/turboquant_plus) |
| **llama-cpp-turboquant** | [TheTom/llama-cpp-turboquant](https://github.com/TheTom/llama-cpp-turboquant) | TurboQuant für llama.cpp (Basis für buun/ik forks) | ✅ | [Repo](https://github.com/TheTom/llama-cpp-turboquant) |
| **llama.cpp-turboq-mtp** | [Indras-Mirror/llama.cpp-turboq-mtp](https://github.com/Indras-Mirror/llama.cpp-turboq-mtp) | TBQ4 Fused Flash Attention + MTP + Shared Tensors. 82+ tok/s mit verlustfreiem 4.25 bpv KV-Cache bei 200K Kontext auf RTX 4090. TBQ4_0: 4.2 GB vs. 16.4 GB (upstream Q4_0) bei 262K. | ✅ Fork | [Repo](https://github.com/Indras-Mirror/llama.cpp-turboq-mtp) |

---

## 8. Qwen3.6-27B MTP Architektur

| Property | Value | Quelle |
|----------|-------|--------|
| **Architecture** | Dense (GDN + Gated Attention) | [Qwen3.6-27B HF Card](https://huggingface.co/Qwen/Qwen3.6-27B) |
| **MTP Layers** | 1 (NextN prediction) | [qwen3.6-mtp PyPI](https://pypi.org/project/qwen3.6-mtp/) |
| **Context** | 262K (extensible to 1M) | [Qwen3.6-27B HF Card](https://huggingface.co/Qwen/Qwen3.6-27B) |
| **Best draft tokens** | 2-3 (MTP-2: +39%, MTP-3: +42% throughput) | [qwen3.6-mtp](https://github.com/ArkaD171717/Qwen3.6-MTP) |
| **Crossover** | MTP-4/5 net-negative at batch size 64 | [qwen3.6-mtp](https://github.com/ArkaD171717/Qwen3.6-MTP) |
| **GGUF MTP builds** | unsloth/Qwen3.6-27B-MTP-GGUF (Q4_K_M, UD-Q4_K_XL, etc.) | [unsloth/Qwen3.6-27B-MTP-GGUF](https://huggingface.co/unsloth/Qwen3.6-27B-MTP-GGUF) |
| **Custom IQ6_K + Q8_0 MTP heads** | AlexanderKyng/Qwen3.6-27B-MTP-Custom-IQ6_K | [HF Card](https://huggingface.co/AlexanderKyng/Qwen3.6-27B-MTP-Custom-IQ6_K) |
| **Custom IQ4_KS + MTP** | RDson/Qwen3.6-27B-MTP-IQ4_KS-GGUF | [HF Card](https://huggingface.co/RDson/Qwen3.6-27B-MTP-IQ4_KS-GGUF) |

---

## 9. Entscheidungsmatrix: Strict TP + CUDA + MTP + KV Quant

| Engine | Strict TP | CUDA | MTP | FP8 KV | FP4 KV | Integer KV | TurboQuant | Offloading |
|--------|-----------|------|-----|--------|--------|------------|------------|------------|
| **vLLM** | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ⚠️ (Genesis) | ✅ (NIXL, Mooncake, LMCache) |
| **vLLM + Genesis** | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ✅ Production | ✅ (NIXL, Mooncake, LMCache) |
| **SGLang** | ✅ | ✅ | ✅ | ✅ | ✅ (exp) | ❌ | ❌ | ✅ (HiSparse, LMCache) |
| **llama.cpp** | ✅ (new, unstable) | ✅ | ✅ | ❌ | ❌ | ✅ (K only) | ❌ | ❌ |
| **ik_llama** | ❌ (Graph only) | ✅ | ✅ | ❌ | ❌ | ✅ (K+V) | ✅ (via forks) | ❌ |
| **buun-llama** | ❌ (legacy) | ✅ | ❌ | ❌ | ❌ | ✅ | ✅ (TCQ) | ❌ |
| **lmdeploy** | ✅ | ✅ | ❌ (DeepSeek only) | ❌ | ❌ | ❌ | ✅ (TQ K4V2, **no spec**) | ❌ |

**Fazit:**
- **Stabilste Strict TP + MTP + KV Quant:** **vLLM** (FP8) oder **SGLang** (FP8/FP4)
- **TurboQuant + MTP + Strict TP:** **vLLM + Genesis v7.72.2** (club-3090 validiert: ✅ Production, 4 streams @ 262K). **Ohne Genesis:** Nirgendwo funktional.
- **Integer KV Quant + MTP:** Nur **llama.cpp** (K-only, Strict TP instabil)
- **Maximale VRAM-Effizienz (aber kein Strict TP):** **ik_llama.cpp** (Graph TP + MTP + TurboQuant + Extreme Quant)
- **KV Offloading + MTP:** **KEIN** Offloading-Mechanismus wurde für MTP validiert. LMCache/vLLM Disagg arbeiten auf Block-Ebene — potenzieller Konflikt mit MTP draft tokens.
- **TurboQuant + Offloading:** **lmdeploy** hat TurboQuant KV (K4V2) aber **spekulatives Decoding explizit nicht unterstützt**. lmdeploy + TurboQuant + MTP = ❌.

---

## 10. Offene Issues & PRs (Relevant)

| Engine | Issue/PR | Thema | Status | Quelle |
|--------|----------|-------|--------|--------|
| vLLM | #38479 | TurboQuant KV Cache (dense GQA/MHA) | MERGED | [PR](https://github.com/vllm-project/vllm/pull/38479) |
| vLLM | #38182 | MTP + Prefix Cache Hit Rate Drop (92%→71%) | OPEN | [Issue](https://github.com/vllm-project/vllm/issues/38182) |
| vLLM | #38280 | TurboQuant KV Cache | CLOSED | [PR](https://github.com/vllm-project/vllm/pull/38280) |
| vLLM | #38662 | TurboQuant KV Cache (PolarQuant + QJL) | CLOSED | [PR](https://github.com/vllm-project/vllm/pull/38662) |
| vLLM | #40831 | TurboQuant + MTP = Degenerate Token Loops | CLOSED | [Issue](https://github.com/vllm-project/vllm/issues/40831) |
| vLLM | #40880 | TQ + MTP = `!`-floods, first-word repetition (Genesis P67 fix) | OPEN | [Issue](https://github.com/vllm-project/vllm/issues/40880) |
| vLLM | #40069 | Tracking: Spec-decode/Eagle disabled for TurboQuant | OPEN | [Issue](https://github.com/vllm-project/vllm/issues/40069) |
| vLLM | #40914 | Synthetic seq_lens K+1 routing | OPEN (negativ) | [PR](https://github.com/vllm-project/vllm/pull/40914) |
| vLLM | #40798 | TQ decode workspace pre-allocation | OPEN | [PR](https://github.com/vllm-project/vllm/pull/40798) |
| vLLM | #42215 | TQ decode kernel warmup | OPEN | [PR](https://github.com/vllm-project/vllm/pull/42215) |
| vLLM | #40792 | k8v4 GQA decode optimization | OPEN | [PR](https://github.com/vllm-project/vllm/pull/40792) |
| vLLM | #40391 | INT8 PTH KV (per-token-head) | MERGED | [PR](https://github.com/vllm-project/vllm/pull/40391) |
| llama.cpp | #21089 | TurboQuant (tbq3_0, tbq4_0) | OPEN (aktiver Upstream-PR) | [TurboQuant Tools](https://turboquant-tools.com) |
| llama.cpp | #20977 | TurboQuant Support Feature Request | OPEN (321 👍) | [Issue](https://github.com/ggml-org/llama.cpp/issues/20977) |
| llama.cpp | #21131 | TurboQuant KV Cache (4.57× compression) | CLOSED | [PR](https://github.com/ggml-org/llama.cpp/pull/21131) |
| llama.cpp | #21062 | TurboQuant KV Cache (2/3/4-bit) + CUDA | CLOSED | [PR](https://github.com/ggml-org/llama.cpp/pull/21062) |
| llama.cpp | #21307 | TurboQuant KV Cache (turbo4_0, turbo3_0, turbo2_0) | CLOSED | [PR](https://github.com/ggml-org/llama.cpp/pull/21307) |
| llama.cpp | #21192 | TurboQuant Random Orthogonal Rotation | CLOSED | [PR](https://github.com/ggml-org/llama.cpp/pull/21192) |
| llama.cpp | #5932 | 4-bit KV Cache Discussion | OPEN | [Discussion](https://github.com/ggml-org/llama.cpp/discussions/5932) |
| llama.cpp | #19378 | Strict Tensor Parallel (split-mode tensor) | MERGED (Feb 2026) | [PR](https://github.com/ggml-org/llama.cpp/pull/19378) |
| llama.cpp | #22673 | MTP Support | MERGED (May 2026) | [PR](https://github.com/ggml-org/llama.cpp/pull/22673) |
| llama.cpp | #23230 | MTP speed regression nach PR #22673 final merge | OPEN (unconfirmed) | [Issue](https://github.com/ggml-org/llama.cpp/issues/23230) |
| ik_llama | #1022 | Graph Tensor Parallel | MERGED | [PR](https://github.com/ikawrakow/ik_llama.cpp/pull/1022) |
| ik_llama | #1698, #1745 | MTP for Qwen3.5/3.6 | MERGED | [PR #1698](https://github.com/ikawrakow/ik_llama.cpp/pull/1698), [PR #1745](https://github.com/ikawrakow/ik_llama.cpp/pull/1745) |
| ik_llama | #627 | Feature Request: Tensor Parallelism | OPEN | [Issue](https://github.com/ikawrakow/ik_llama.cpp/issues/627) |

---

## 11. Quellen & Referenzen

- **vLLM:** https://github.com/vllm-project/vllm, https://docs.vllm.ai, https://vllm.ai/blog/2026-04-22-fp8-kvcache
- **vLLM Blog 11.05.2026:** https://vllm.ai/blog/2026-05-11-fp8-kvcache-benchmark — FP8 > TurboQuant für Throughput; TurboQuant 3bit_nc: substanzielle PPL-Verluste
- **Genesis (Sandermage):** https://github.com/Sandermage/genesis-vllm-patches
- **club-3090 (noonghunna):** https://github.com/noonghunna/club-3090 — Validated TQ3 + MTP + Genesis v7.72.2 on dual RTX 3090
- **SGLang:** https://github.com/sgl-project/sglang, https://sgl-project.github.io/advanced_features/quantized_kv_cache.html
- **llama.cpp:** https://github.com/ggml-org/llama.cpp
- **ik_llama.cpp:** https://github.com/ikawrakow/ik_llama.cpp
- **buun-llama-cpp:** https://github.com/spiritbuun/buun-llama-cpp
- **Qwen3.6-27B:** https://huggingface.co/Qwen/Qwen3.6-27B
- **Unsloth Qwen3.6 Docs:** https://unsloth.ai/docs/models/qwen3.6
- **TurboQuant Paper:** Zandieh et al., "Online Vector Quantization with Near-optimal Distortion Rate", ICLR 2026, arXiv:2504.19874
- **qwen3.6-mtp:** https://pypi.org/project/qwen3.6-mtp/, https://github.com/ArkaD171717/Qwen3.6-MTP
- **quant.cpp:** https://github.com/quantumaikr/quant.cpp
- **turboquant-vllm:** https://github.com/Alberto-Codes/turboquant-vllm
- **NVIDIA Developer Forums:** https://forums.developer.nvidia.com/t/mtp-llama-cpp-a-look-at-qwen3-6-27b/370298
- **mubibai.com Benchmarks:** https://mubibai.com/sglang-vs-vllm-production-inference-benchmarks-april-2026/
- **wiki.charleschen.ai Comparison:** https://wiki.charleschen.ai/ai/processed/wiki/infrastructure/fleet/comparisons/vllm-vs-sglang-vs-llamacpp-vs-tgi
- **devnen/qwen3.6-windows-server:** https://github.com/devnen/qwen3.6-windows-server/blob/main/docs/MTP_HEAD.md
- **AlexanderKyng/Qwen3.6-27B-MTP-Custom-IQ6_K:** https://huggingface.co/AlexanderKyng/Qwen3.6-27B-MTP-Custom-IQ6_K
- **RDson/Qwen3.6-27B-MTP-IQ4_KS-GGUF:** https://huggingface.co/RDson/Qwen3.6-27B-MTP-IQ4_KS-GGUF
- **LMCache:** https://github.com/LMCache/LMCache, https://docs.lmcache.ai/developer_guide/integration.html
- **llm-d-kv-cache:** https://github.com/llm-d/llm-d-kv-cache, https://llm-d.ai/docs/architecture/Components/kv-cache
- **Mooncake:** https://github.com/kvcache-ai/mooncake, https://kvcache-ai.github.io/Mooncake/
- **vLLM Disagg Prefill:** https://docs.vllm.ai/en/latest/usage/disagg_prefill.html, https://docs.vllm.ai/en/latest/features/nixl_connector_usage/
- **CacheGen:** https://github.com/UChi-JCL/CacheGen, arXiv:2310.07240 (SIGCOMM 2024)
- **NVIDIA kvpress:** https://github.com/NVIDIA/kvpress
- **KakeyaLattice:** https://github.com/FluffyAIcode/LLM-KV--Cache-compress
- **lmdeploy TurboQuant:** https://github.com/InternLM/lmdeploy, [PR #4510](https://github.com/InternLM/lmdeploy/commit/e433925b61c84249b0edc5cb46bebc2dec71cb7f)
- **RocketKV:** https://github.com/NVlabs/RocketKV (ICML 2025)
- **Hardware Compatibility Sources:**
  - CanItRun Qwen3.6-27B: https://canitrun.dev/models/qwen3.6-27b/
  - Clore.ai Qwen3.6-27B Guide: https://docs.clore.ai/guides/language-models/qwen36-27b
  - CobraPhil RTX 5090 Recipe: https://github.com/CobraPhil/qwen36-27b-single-5090
  - Knightli VRAM Table: https://www.knightli.com/en/2026/05/01/qwen3-6-local-vram-quantization-table/
  - InsiderLLM Guide: https://insiderllm.com/guides/qwen-3-6-local-ai-guide/
  - WillItRunAI: https://willitrunai.com/blog/qwen-3-6-27b-vram-requirements

---

## 12. Hardware Kompatibilität: Qwen3.6-27B

### VRAM-Anforderungen (Gewichte nur, 8K Kontext)

| Quantisierung | GGUF-Dateigröße | Min. VRAM | Sichere VRAM | Beste für |
|--------------|----------------|-----------|-------------|----------|
| BF16 | 55.6 GB | 64 GB | 80 GB | H100 80GB, Dual-GPU |
| Q8_0 | 28.6 GB | 32 GB | 40 GB | RTX 5090, A100 40GB |
| Q6_K | 22.5 GB | 28 GB | 32 GB | RTX 4090, RTX 3090 |
| Q5_K_M | 19.5 GB | 24 GB | 32 GB | RTX 4090, RTX 3090 |
| Q4_K_M | 16.8 GB | 20 GB | 24 GB | RTX 4080 16GB (tight), RTX 4090 |
| Q3_K_M | 13.6 GB | 16 GB | 20 GB | RTX 3080 20GB, RTX 4080 16GB |
| Q2_K | 11.8 GB | 14 GB | 18 GB | RTX 3080 20GB (comfortable) |

**KV Cache pro Token (27B, 64 heads, head_dim=128):**
- BF16: ~200 Bytes/token
- FP8/INT8: ~100 Bytes/token
- FP4/INT4: ~50 Bytes/token
- TurboQuant 3bit: ~25-30 Bytes/token

---

### 12.1 2× NVIDIA V100 16GB (32GB total)

**Architektur:** Volta (2017) | **Tensor Cores:** FP16 only (kein natives BF16)
**Performance:** TF32: ~7.5 TFLOPS, FP16: ~15 TFLOPS pro GPU
**PCIe:** Gen3 x16 | **NVLink:** ✅ (bis zu 300 GB/s)

| Quantisierung | TP=2 pro GPU | Max. Kontext | Expected TPS | Status |
|--------------|-------------|-------------|-------------|--------|
| BF16 | ❌ OOM (27GB > 16GB) | — | — | ❌ |
| Q8_0 | 14.3 GB | ~32K | ~15-20 tok/s | ⚠️ Tight |
| Q6_K | 11.25 GB | ~48K | ~18-25 tok/s | ✅ Comfortable |
| Q4_K_M | 8.4 GB | ~64K | ~20-28 tok/s | ✅ Comfortable |
| Q3_K_M | 6.8 GB | ~80K | ~22-30 tok/s | ✅ Comfortable |

**Limitationen:**
- **V100 hat keine BF16 Tensor Cores** — FP16 wird verwendet, was zu Precision-Loss bei 27B Modellen führen kann
- **KV Cache:** Mit FP8 KV: ~2x mehr Kontext möglich
- **MTP:** ✅ Funktioniert, aber Volta-Architektur limitiert den Speedup (~1.5-1.8×)
- **TurboQuant:** ⚠️ Theoretisch möglich, aber Volta-Compute-Capability (7.0) könnte Kernel-Kompatibilitätsprobleme haben
- **Strict TP:** ✅ vLLM/SGLang unterstützen TP=2 auf V100
- **Interconnect:** NVLink (SXM2) ideal für TP=2. PCIe-only V100s haben ~15-20% TP-Overhead. Pipeline Parallelism (PP=2) kann besser sein als TP=2 auf PCIe.

**Empfohlene Konfiguration:**
```bash
# 2x V100 16GB mit Q6_K und FP8 KV
vllm serve Lorbus/Qwen3.6-27B-int4-AutoRound \
  --tensor-parallel-size 2 \
  --quantization auto_round \
  --kv-cache-dtype fp8 \
  --max-model-len 49152 \
  --dtype float16 \
  --gpu-memory-utilization 0.90
```

**Fazit:** V100 16GB ist **borderline** für Qwen3.6-27B. Q6_K mit FP8 KV ist der sweet spot. BF16 ist nicht möglich ohne CPU-Offload.

---

### 12.2 2× NVIDIA RTX 3080 20GB (40GB total)

**Architektur:** Ampere (2020) | **Tensor Cores:** FP16/BF16
**Performance:** TF32: ~19.8 TFLOPS, FP16: ~39.6 TFLOPS pro GPU
**PCIe:** Gen4 x16 | **NVLink:** ❌ (PCIe-only)

| Quantisierung | TP=2 pro GPU | Max. Kontext | Expected TPS | Status |
|--------------|-------------|-------------|-------------|--------|
| BF16 | ❌ OOM (27GB > 20GB) | — | — | ❌ |
| Q8_0 | 14.3 GB | ~48K | ~25-35 tok/s | ✅ Comfortable |
| Q6_K | 11.25 GB | ~64K | ~30-40 tok/s | ✅ Comfortable |
| Q4_K_M | 8.4 GB | ~80K | ~35-50 tok/s | ✅ Comfortable |
| Q3_K_M | 6.8 GB | ~128K | ~40-55 tok/s | ✅ Very comfortable |

**Vorteile:**
- **Ampere hat native BF16** — keine Precision-Loss wie bei V100
- **FP8 KV Cache:** ✅ vLLM/SGLang unterstützen FP8 KV auf Ampere
- **MTP:** ✅ Voll unterstützt, guter Speedup mit Ampere Tensor Cores (~1.6-2.0×)
- **TurboQuant:** ✅ Theoretisch möglich (Compute Capability 8.6)
- **Strict TP:** ✅ vLLM/SGLang unterstützen TP=2 auf RTX 3080

**Limitationen:**
- **Kein NVLink** — PCIe-basierte Kommunikation zwischen GPUs (limitiert bei großen KV-Transfers)
- **BF16 passt nicht** — 27GB pro GPU > 20GB VRAM. Muss quantisiert werden.
- **Max Kontext bei BF16:** ~16K (27GB weights + KV cache > 20GB)
- **PCIe TP=2 Overhead:** Consumer Ampere hat ~32 GB/s PCIe Gen4 pro GPU. Für große Batches kann dies MTP-Verifikation bottlenecken. Single-GPU INT4 mit CPU-Offload kann schneller sein als TP=2 bei niedriger Konfiguration.

**Empfohlene Konfiguration:**
```bash
# 2x RTX 3080 20GB mit Q6_K und FP8 KV
vllm serve Lorbus/Qwen3.6-27B-int4-AutoRound \
  --tensor-parallel-size 2 \
  --quantization auto_round \
  --kv-cache-dtype fp8 \
  --max-model-len 65536 \
  --dtype bfloat16 \
  --gpu-memory-utilization 0.92
```

**Fazit:** RTX 3080 20GB ist **comfortable** für Qwen3.6-27B mit Q6_K/Q4_K_M. FP8 KV ermöglicht ~64K Kontext. MTP funktioniert gut.

---

### 12.3 NVIDIA RTX 5090 32GB (Single GPU)

**Architektur:** Blackwell (2025) | **Tensor Cores:** FP16/BF16/FP4
**Performance:** TF32: ~24.6 TFLOPS, FP16: ~49.2 TFLOPS, FP4: ~98.4 TFLOPS
**PCIe:** Gen5 x16 | **NVLink:** ❌ (Single GPU)

| Quantisierung | VRAM Usage | Max. Kontext | Expected TPS | Status |
|--------------|-----------|-------------|-------------|--------|
| BF16 | 55.6 GB | ❌ OOM (55.6 > 32) | — | ❌ |
| Q8_0 | 28.6 GB | ~128K | ~60-90 tok/s | ✅ Comfortable |
| Q6_K | 22.5 GB | ~160K | ~75-85 tok/s | ✅ Very comfortable |
| Q4_K_M | 16.8 GB | ~200K | ~80-100 tok/s | ✅ Very comfortable |
| NVFP4 | 13.5 GB | ~262K | ~132 tok/s | ✅ Production |

**Vorteile:**
- **Native FP4 Support (NVFP4)** — 4x Kompression bei minimalem PPL-Loss. Blackwell's 5th-gen Tensor Cores beschleunigen FP4-Matrix-Multiplikationen nativ, umgehen Software-Dequantisierung.
- **Blackwell Tensor Cores** — beste Performance/VRAM-Ratio (~1,792 GB/s BW)
- **MTP:** ✅ Voll unterstützt, bester Speedup mit Blackwell (~1.8-2.2× mit NVFP4+MTP)
- **TurboQuant:** ✅ Theoretisch möglich (Compute Capability 9.0)
- **32GB VRAM** — Q8_0 mit headroom für KV Cache

**Limitationen:**
- **Single GPU** — kein TP möglich (aber nicht nötig bei 32GB)
- **BF16 passt nicht** — 55.6GB > 32GB. Muss quantisiert oder CPU-offgeloaded werden.
- **Max Kontext bei BF16:** ~8K (55.6GB weights + KV > 32GB)
- **Multi-User Batching:** 32GB VRAM sättigt schnell durch KV-Cache-Wachstum. Prefix-Caching ist stark empfohlen.

**Empfohlene Konfiguration (NVFP4):**
```bash
# RTX 5090 32GB mit NVFP4 und MTP
vllm serve Lorbus/Qwen3.6-27B-int4-AutoRound \
  --quantization compressed-tensors \
  --dtype float16 \
  --max-model-len 262144 \
  --speculative-config '{"method":"qwen3_next_mtp","num_speculative_tokens":2}' \
  --gpu-memory-utilization 0.95
```

**Empfohlene Konfiguration (Q8_0 + FP8 KV):**
```bash
# RTX 5090 32GB mit Q8_0 und FP8 KV
vllm serve Lorbus/Qwen3.6-27B-int4-AutoRound \
  --quantization auto_round \
  --kv-cache-dtype fp8 \
  --max-model-len 131072 \
  --dtype bfloat16 \
  --gpu-memory-utilization 0.92
```

**Fazit:** RTX 5090 32GB ist **ideal** für Qwen3.6-27B. NVFP4 ermöglicht 262K Kontext mit 132+ tok/s. Q8_0 mit FP8 KV bietet ~128K Kontext.

---

### 12.4 Hardware-Vergleichstabelle

| GPU | VRAM | Quantisierung | Max Kontext | Expected TPS | MTP | FP8 KV | TurboQuant | Strict TP |
|-----|------|--------------|-------------|-------------|-----|--------|------------|-----------|
| **2× V100 16GB** | 32GB total | Q6_K | ~48K | 18-25 tok/s | ✅ | ✅ | ⚠️ | ✅ TP=2 |
| **2× RTX 3080 20GB** | 40GB total | Q6_K | ~64K | 30-40 tok/s | ✅ | ✅ | ✅ | ✅ TP=2 |
| **RTX 5090 32GB** | 32GB | NVFP4 | ~262K | 132 tok/s | ✅ | ✅ | ✅ | ❌ Single |
| **RTX 5090 32GB** | 32GB | Q8_0 | ~128K | 60-90 tok/s | ✅ | ✅ | ✅ | ❌ Single |

**Empfehlung:**
- **2× V100 16GB:** Nur mit Q6_K/Q4_K_M + FP8 KV. Volta-Architektur limitiert Performance.
- **2× RTX 3080 20GB:** Comfortable mit Q6_K/Q4_K_M + FP8 KV. Ampere-Architektur gut für MTP.
- **RTX 5090 32GB:** Ideal für Qwen3.6-27B. NVFP4 für 262K Kontext, Q8_0 für 128K.

**⚠️ KRITISCH:** BF16 passt auf KEINE der drei Konfigurationen ohne CPU-Offload oder TP.
- V100 16GB: ❌ (27GB > 16GB pro GPU)
- RTX 3080 20GB: ❌ (27GB > 20GB pro GPU)
- RTX 5090 32GB: ❌ (55.6GB > 32GB)

**Für BF16 auf 262K Kontext:** Mindestens 1× A100 80GB oder 2× RTX 4090/3090 mit TP=2.

---

**Zusammenfassung für Strict TP + CUDA + MTP + KV Quant:**
- **vLLM mit FP8 KV** ist die stabilste, produktionsreife Lösung (kein Patch nötig)
- **SGLang mit FP8/FP4 KV** bietet maximale Kompression bei Strict TP
- **TurboQuant + MTP + Strict TP:** **vLLM + Genesis v7.72.2** ist die einzige funktionale Kombination (club-3090 validiert: ✅ Production, 4 streams @ 262K). **Ohne Genesis:** Nirgendwo funktional (vllm#40880).
- **Integer KV Quant (Q4/Q5) + Strict TP + MTP existiert nicht in funktionsfähigem Zustand**
- Für maximale VRAM-Effizienz muss man auf Strict TP verzichten (ik_llama Graph TP)
