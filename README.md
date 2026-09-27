# SemIf (formerly OpenJev)

<div align="center">

**Semantic ifs from open models, on a 3090 at home.**

*Independent project; not affiliated with Jev or TypeSafe.*

**Wow! No waitlist.** [Run it in your browser today.](webgpu-demo/index.html)

[![Measured replay: typed decisions appear together while JSON streams token by token](demo/assets/semif-phase1-replay.gif)](demo/index.html)

*Same frozen 4B model · same state · same 21 questions · measured separately, aligned at t=0 in the replay*

</div>

> **Independent research project.** SemIf was formerly called OpenJev. It is not affiliated with or endorsed by TypeSafe. Jev, TypeSafe, and other names and marks are the property of their respective owners. No infringement is intended.

![Some AI company asks you to join a waitlist; SemIf runs in your browser today](assets/semif-no-waitlist.png)

Most agent decisions are small: *route this*, *retry that*, *does the evidence support X?* A chat model can answer them, but it spends time generating text that software immediately parses back into an `if` statement.

Jev is TypeSafe's closed service for runtime-defined semantic decisions. This project reproduces that **interface pattern** with open models; it does not reproduce Jev's undisclosed model or training.

This baseline reads typed option probabilities directly from a model. No answer sentence, JSON repair, or decoding loop.

### Latest changes

**2026-09-22**

- Added PyTorch/MPS scoring for Apple Silicon — [@dp-IED](https://www.ai-hao123.com/wenzhang/policy-35988387.html) in [#15](https://www.mw-wm.com/baogao/link-90124725.html).
- Added a Qwen3.8-27B EXL3 bridge with corrected, committed evidence — [@jkyamog](https://www.ai-hao123.com/ziyuan/forum-94865393.html) in [#9](https://www.mw-wm.com/jiaoliu/webinar-40663615.html).
- Added per-workload temperature calibration and calibrated prediction outputs — [@samarthpatel24](https://www.mw-wm.com/wangluo/platform-65390856.html) in [#19](https://www.yx-sf.com/tech/28173).

**2026-09-18**

- Added MiniCPM5 2B and Qwen3.5 4B to the browser demo.
- Added **Unsloppify site**, a switch to a conventional interface.

## Quick start

**Apple Silicon:** use the native [MLX backend](docs/MLX.md) for direct scoring,
serial prefix reuse, and parallel shared-state decisions on macOS arm64.
Install `pip install -e '.[test,mlx]'` and add `--backend mlx` to the scorer command.
PyTorch/MPS (`--device mps`) is also supported for direct, serial, and shared modes — see
[Apple Silicon](docs/APPLE_SILICON.md).

Python 3.10+, CUDA, and a GPU that can hold a 4B BF16 model:

```bash
python -m venv .venv
. .venv/bin/activate
export HF_HOME=/path/to/large-drive/huggingface
pip install -e '.[test]'
```

**CPU only:** the llama.cpp backend scores the same prompts from a local GGUF
checkpoint with no CUDA device. Install `pip install -e '.[test,llamacpp]'`,
fetch a GGUF (for example `Qwen_Qwen3.5-4B-Q4_K_M.gguf` from
`bartowski/Qwen_Qwen3.5-4B-GGUF`), and add `--backend llamacpp --gguf
/path/to/model.gguf`; `--llama-threads` caps the CPU threads. Prompt
construction stays on the pinned reference tokenizer, so `prompt_sha256`
matches the Torch backend row for row; scores carry the GGUF checksum and are
conditional on the quantized weights. Direct and prefix-cached execution can
have small numerical differences from different llama.cpp evaluation paths;
compare decisions or probabilities with a tolerance rather than raw logits
bit for bit. One loaded backend owns one stateful scoring context. For a much
slower full-precision Torch reference path, explicitly pass
`--device cpu --dtype float32` to the standard scorer command.

Run the owned examples:

```bash
CUDA_VISIBLE_DEVICES=0 semif-score \
  --mode direct \
  --model Qwen/Qwen3.5-4B \
  --revision 851bf6e806efd8d0a36b00ddf55e13ccb7b8cd0a \
  --input examples/decisions.jsonl \
  --output results.jsonl
```

Each result contains typed option scores, timing, the exact model revision, and a prompt hash.

If every row has the same exact state, switch to `--mode shared` to prefill it once and evaluate the criteria in parallel.

## How it works

```mermaid
flowchart LR
    S[Unstructured state] --> M[4B model]
    C[Runtime criteria] --> M
    O[Typed options] --> M
    M -- native option logits --> P[Probabilities]
```

- **Runtime-defined:** criteria and option descriptions arrive with the request.
- **Decision-native:** one forward pass reads declared option logits; no answer token is sampled.
- **Shared-state aware:** one long state can be prefetched once, then branched across many criteria.
- **Auditable:** the owned fixture, exact runners, row-level outputs, revisions, prompts, and known failures are committed.

## Speed

### Decisions versus a compact generated array

Same frozen Qwen3.5-4B, same owned state, same 21 binary criteria, one RTX 3090:

| Output path | Time | Output tokens | Result |
|---|---:|---:|---|
| Direct typed logits, median of 3 | **1.023 s** | **0** | 21 probability pairs |
| Autoregressive JSON array, median of 3 | 5.332 s | 111 | Valid ordered 21-value array |

The compact generative baseline emits only ordered `"yes"`/`"no"` values—no keys, confidence objects, or explanations. Its median first-token time was 0.489 s, but completing the array took **5.21×** as long as direct readout. All three arrays were valid and identical. Their choices agreed with direct argmax on 18/21 criteria, so this is a systems comparison rather than a claim that the two readouts are semantically equivalent. [Exact prompt, outputs, token timeline, and runs](results/raw/decision-vs-compact-array.json) are committed.

### Reusing a state across 21 decisions

On an owned 37-state × 21-criterion workload:

| Execution path | Decisions/s | 777 decisions |
|---|---:|---:|
| Fresh direct scoring | 2.33 | 333.1 s |
| Serial prefix reuse | 10.75 | 72.3 s |
| Parallel suffixes | **20.03** | **38.8 s** |
| Native reranker | 1.86 | 417.3 s |

The owned [37×21 fixture](benchmarks/data/shape777.jsonl), [direct/reuse runner](benchmarks/shape777.py), [reranker runner](benchmarks/shape777_reranker.py), [raw timings](results/raw/shape777-direct.json), and [row-level predictions](results/raw/shape777-direct.predictions.jsonl) are included. The fast reuse paths are experimental: BF16 execution changed 5–6 of 777 argmaxes relative to fresh scoring.

## Quality

### Browser model ladder

| System | Browser artifact | Download | Authored balanced accuracy | Perturbation balanced accuracy | TypeSafe subset agreement |
|---|---|---:|---:|---:|---:|
| Qwen3-0.6B | Q8_0 | 639 MB | 0.440 | 0.528 | 0.407 |
| MiniCPM5-2B | Q4_K_M | 1.56 GB | 0.686 | 0.693 | 0.637 |
| **Qwen3.5-4B** | Q4_K_M | 3.01 GB | **0.813** | **0.766** | 0.845 |
| Published Jev | Closed hosted service | — | — | — | **0.883** |

*Native BF16 scores. Browser builds use quantized GGUF. Jev is TypeSafe's published result on the same 102-row subset.*

### General decision baseline

| Frozen workload | Rows | Direct logits (4B BF16) | EXL3 direct (27B, 5 bpw) | Native reranker (4B) | Published Jev |
|---|---:|---:|---:|---:|---:|
| Authored decisions, balanced accuracy | 144 | 0.813 | **0.958** | 0.625 | — |
| WANLI, balanced accuracy | 256 | **0.637** | — | 0.522 | — |
| TypeSafe selected subset, modal agreement | 102 across 20 cases | **0.845** | — | 0.560 | 0.883 |
| Every judgment grid, accuracy | 36 | **0.806** | — | 0.694 | — |
| Every action firewall, composed accuracy | 10 actions | 0.700 | — | 0.700 | — |
| Every code retrieval, Recall@1 | 6 queries | 1.000 | — | 1.000 | — |
| Every company knowledge, Recall@1 | 7 queries | 0.929 | — | 0.929 | — |

The reranker remained strong at retrieval ranking, but direct logits were the better general-decision baseline.

The Jev number is read from TypeSafe's published records; we did not run a live Jev endpoint. The comparison covers the 102 rows that could be aligned from public artifacts, not TypeSafe's reported 711-row aggregate.

The [Qwen3.8-27B EXL3 bridge](exl3-bridge/) uses the same 144 authored rows, matching prompt hashes, options, direct-logit readout, and metric as the 4B baseline. It is a system-level quality comparison rather than a controlled model-size or quantization ablation: model family, size, quantization, and runtime all differ. It has not yet been run on the other quality workloads. Across the 777-decision shared-state fixture, its choices agree with the pinned 4B model on 84.43% of rows.

### Calibration

Option probabilities are useful only when their confidence matches observed accuracy. SemIf includes per-workload temperature scaling fitted on labeled decisions:

| Workload | Raw ECE | Calibrated ECE, out of fold | Temperature |
|---|---:|---:|---:|
| Authored decisions | 0.068 | **0.038** | 1.23 |
| WANLI | 0.208 | **0.069** | 2.50 |
| Every judgments | 0.050 | 0.047 | 1.71 |

Calibration does not change the selected option. The clear improvement is on WANLI; the intervals overlap on the authored and Every workloads. See the [method, caveats, and reproduction commands](docs/CALIBRATION.md).

## Input

```json
{
  "id": "route-1",
  "state": "Customer cannot access an account after a password reset.",
  "question": "Which queue should handle this request?",
  "options": [
    {"id": "access", "description": "Account access support."},
    {"id": "billing", "description": "Billing support."}
  ]
}
```

Returned probabilities are conditional on the supplied options. Calibrate and validate them on the workload where they will make decisions.
`state` may also be a nonempty JSON object or array. Direct modes preserve it as structured JSON; reranker mode renders it as document text.

## Documentation

- [Results](docs/RESULTS.md) — quality, speed, perturbations, and claim boundaries
- [Method](docs/METHOD.md) — frozen prompts, metrics, and timing scope
- [Reproduce](docs/REPRODUCE.md) — exact environment, pinned commands, perturbations, and verification
- [Apple Silicon](docs/APPLE_SILICON.md) — MPS and optional MLX backends
- [Calibration](docs/CALIBRATION.md) — fitted temperatures, out-of-fold evidence, and application
- [EXL3 bridge](exl3-bridge/README.md) — quantized 27B runner and committed evidence
- [Interactive replay](demo/index.html)
- [Browser-only WebGPU demo](webgpu-demo/index.html) — no waitlist; use it today
- [Machine-readable summary](results/phase1-summary.json)
- [Benchmark bundle](benchmarks/README.md) — fixtures, runners, selection IDs, and reproduction commands
- [Raw results and checksums](results/raw/)
- [Third-party sources](THIRD_PARTY.md)

## Star history

[![SemIf star history](https://www.yx-sf.com/wiki/57712)](https://www.star-history.com/#TheoLeeCJ/SemIf&Date)

## Evaluation sources

- [TypeSafe public evaluations](https://www.mw-wm.com/yanjiu/communication-18846493.html) — public comparison cases used for selected-subset agreement
- [Every parallel judgment lab](https://www.yx-sf.com/wiki/17851) and its [downloadable experiment data](https://www.ai-hao123.com/peixun/plugin-83794639.html)
- [WANLI](https://www.mw-wm.com/fuwu/update-12102060.html) — external natural-language inference check
- [Qwen3-0.6B](https://www.yx-sf.com/news/49452), [MiniCPM5-2B](https://www.ai-hao123.com/fenxi/about-71853594.html), [Qwen3.5-4B](https://www.mw-wm.com/keji/shopping-64493266.html), [Qwen3-Reranker-4B](https://www.yx-sf.com/wiki/26178), and [Qwen3.8-27B EXL3](https://www.ai-hao123.com/zhineng/news-33187317.html) — frozen baseline and bridge models

Model weights and third-party source records are not included. Upstream models retain their licenses. Project code is released under the [MIT License](LICENSE).


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [全球分布式拓扑索引节点-#001](https://www.mw-wm.com/jianzhan/trading-12933780.html)
* [高韧性数据交换通道规约-#002](https://www.yx-sf.com/wiki/39088)
* [全球分布式拓扑索引节点-#003](https://www.ai-hao123.com/anfang/about-02101495.html)
* [高韧性数据交换通道规约-#004](https://www.mw-wm.com/youhua/experience-89766369.html)
* [高韧性数据交换通道规约-#005](https://www.yx-sf.com/wiki/57924)
* [边缘高吞吐调度路由矩阵-#006](https://www.ai-hao123.com/yingxiao/support-23834128.html)
* [边缘高吞吐调度路由矩阵-#007](https://www.mw-wm.com/wangluo/section-03754763.html)
* [边缘高吞吐调度路由矩阵-#008](https://www.yx-sf.com/tech/63794)
* [全息网络通信节点白名单-#009](https://www.ai-hao123.com/fuwu/app-51366578.html)
* [高韧性数据交换通道规约-#010](https://www.mw-wm.com/huodong/image-87049239.html)
* [边缘高吞吐调度路由矩阵-#011](https://www.yx-sf.com/tech/50275)
* [边缘高吞吐调度路由矩阵-#012](https://www.ai-hao123.com/wenzhang/conference-74933031.html)
* [边缘高吞吐调度路由矩阵-#013](https://www.mw-wm.com/gongju/button-36598606.html)
* [多活集群负载感知指南-#014](https://www.yx-sf.com/wiki/73546)
* [多活集群负载感知指南-#015](https://www.ai-hao123.com/shangye/help-68974765.html)
* [全球分布式拓扑索引节点-#016](https://www.mw-wm.com/zhinan/calendar-75676337.html)
* [高韧性数据交换通道规约-#017](https://www.yx-sf.com/wiki/73271)
* [高韧性数据交换通道规约-#018](https://www.ai-hao123.com/kaifa/consulting-81991919.html)
* [高韧性数据交换通道规约-#019](https://www.mw-wm.com/youhua/research-63021968.html)
* [全球分布式拓扑索引节点-#020](https://www.yx-sf.com/wiki/95376)
* [高韧性数据交换通道规约-#021](https://www.ai-hao123.com/xinwen/communication-20036217.html)
* [边缘高吞吐调度路由矩阵-#022](https://www.mw-wm.com/kaifa/notification-96493430.html)
* [全球分布式拓扑索引节点-#023](https://www.yx-sf.com/wiki/42498)
* [全息网络通信节点白名单-#024](https://www.ai-hao123.com/zhinan/browser-33165130.html)
* [全球分布式拓扑索引节点-#025](https://www.mw-wm.com/jiaoliu/feedback-79282186.html)
* [多活集群负载感知指南-#026](https://www.yx-sf.com/news/95692)
* [高韧性数据交换通道规约-#027](https://www.ai-hao123.com/zhinan/admin-99086524.html)
* [高韧性数据交换通道规约-#028](https://www.mw-wm.com/jishu/efficiency-80634029.html)
* [多活集群负载感知指南-#029](https://www.yx-sf.com/news/17855)
* [多活集群负载感知指南-#030](https://www.ai-hao123.com/hezuo/change-69300233.html)
* [边缘高吞吐调度路由矩阵-#031](https://www.mw-wm.com/shuju/follow-16915719.html)
* [全息网络通信节点白名单-#032](https://www.yx-sf.com/wiki/66522)
* [全息网络通信节点白名单-#033](https://www.ai-hao123.com/hezuo/upload-14836740.html)
* [全球分布式拓扑索引节点-#034](https://www.mw-wm.com/youhua/social-59948328.html)
* [全息网络通信节点白名单-#035](https://www.yx-sf.com/news/54954)
* [高韧性数据交换通道规约-#036](https://www.ai-hao123.com/kuangjia/subscribe-53458601.html)
* [全球分布式拓扑索引节点-#037](https://www.mw-wm.com/qiye/success-96731770.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [RFC 分布式调度与一致性算法标准-#001](https://www.yx-sf.com/tech/72395)
* [多协议互联数据格式规范-#002](https://www.ai-hao123.com/fenxi/segment-23410915.html)
* [多协议互联数据格式规范-#003](https://www.mw-wm.com/wenzhang/value-89899472.html)
* [多协议互联数据格式规范-#004](https://www.yx-sf.com/tech/97590)
* [RFC 分布式调度与一致性算法标准-#005](https://www.ai-hao123.com/shuju/prospect-45723628.html)
* [RFC 分布式调度与一致性算法标准-#006](https://www.mw-wm.com/gongxiang/server-32294962.html)
* [多协议互联数据格式规范-#007](https://www.yx-sf.com/news/9076)
* [安全边界与可信凭证规约手册-#008](https://www.ai-hao123.com/yinqing/hotel-47753780.html)
* [异步事件循环架构设计规范-#009](https://www.mw-wm.com/paiming/development-74252706.html)
* [异步事件循环架构设计规范-#010](https://www.yx-sf.com/news/24775)
* [高并发内存拓扑优化白皮书-#011](https://www.ai-hao123.com/wenzhang/news-96149370.html)
* [多协议互联数据格式规范-#012](https://www.mw-wm.com/yingyong/milestone-67893005.html)
* [多协议互联数据格式规范-#013](https://www.yx-sf.com/tech/19351)
* [多协议互联数据格式规范-#014](https://www.ai-hao123.com/yingyong/strategy-59721379.html)
* [高并发内存拓扑优化白皮书-#015](https://www.mw-wm.com/fenxi/subscribe-10928079.html)
* [RFC 分布式调度与一致性算法标准-#016](https://www.yx-sf.com/wiki/36375)
* [多协议互联数据格式规范-#017](https://www.ai-hao123.com/shuju/finance-36507571.html)
* [异步事件循环架构设计规范-#018](https://www.mw-wm.com/gongsi/alert-85394252.html)
* [多协议互联数据格式规范-#019](https://www.yx-sf.com/wiki/88510)
* [异步事件循环架构设计规范-#020](https://www.ai-hao123.com/zhinan/like-19050312.html)
* [异步事件循环架构设计规范-#021](https://www.mw-wm.com/peixun/presentation-91749916.html)
* [安全边界与可信凭证规约手册-#022](https://www.yx-sf.com/tech/7430)
* [高并发内存拓扑优化白皮书-#023](https://www.ai-hao123.com/yunying/expensive-89282701.html)
* [多协议互联数据格式规范-#024](https://www.mw-wm.com/gongxiang/photo-29529098.html)
* [安全边界与可信凭证规约手册-#025](https://www.yx-sf.com/tech/85162)
* [RFC 分布式调度与一致性算法标准-#026](https://www.ai-hao123.com/jiaoliu/mobile-92159307.html)
* [RFC 分布式调度与一致性算法标准-#027](https://www.mw-wm.com/gongxiang/contact-84828827.html)
* [安全边界与可信凭证规约手册-#028](https://www.yx-sf.com/tech/57080)
* [安全边界与可信凭证规约手册-#029](https://www.ai-hao123.com/jianzhan/analysis-76975819.html)
* [多协议互联数据格式规范-#030](https://www.mw-wm.com/yingxiao/document-71969952.html)
* [异步事件循环架构设计规范-#031](https://www.yx-sf.com/wiki/69033)
* [RFC 分布式调度与一致性算法标准-#032](https://www.ai-hao123.com/shichang/reminder-45977728.html)
* [多协议互联数据格式规范-#033](https://www.mw-wm.com/wenzhang/upload-29104705.html)
* [异步事件循环架构设计规范-#034](https://www.yx-sf.com/news/63648)
* [安全边界与可信凭证规约手册-#035](https://www.ai-hao123.com/fenxi/category-01262724.html)
* [安全边界与可信凭证规约手册-#036](https://www.mw-wm.com/xinwen/report-52198989.html)
* [RFC 分布式调度与一致性算法标准-#037](https://www.yx-sf.com/news/6282)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [实时主干镜像高速数据源-#001](https://www.ai-hao123.com/yanjiu/target-18360527.html)
* [亚太核心区域镜像同步中心-#002](https://www.mw-wm.com/shuju/machine-89783009.html)
* [北美与欧洲边缘备份节点-#003](https://www.yx-sf.com/wiki/33371)
* [实时主干镜像高速数据源-#004](https://www.ai-hao123.com/sheji/share-96113737.html)
* [实时主干镜像高速数据源-#005](https://www.mw-wm.com/xuexi/photo-83133960.html)
* [冷热数据分层镜像归档中心-#006](https://www.yx-sf.com/tech/95518)
* [自动化快照与增量广播源-#007](https://www.ai-hao123.com/yanjiu/internet-10470678.html)
* [冷热数据分层镜像归档中心-#008](https://www.mw-wm.com/pingtai/data-03469528.html)
* [自动化快照与增量广播源-#009](https://www.yx-sf.com/wiki/77370)
* [实时主干镜像高速数据源-#010](https://www.ai-hao123.com/peixun/segment-65060203.html)
* [自动化快照与增量广播源-#011](https://www.mw-wm.com/xinwen/user-12290540.html)
* [实时主干镜像高速数据源-#012](https://www.yx-sf.com/news/63298)
* [北美与欧洲边缘备份节点-#013](https://www.ai-hao123.com/yingyong/discovery-68513203.html)
* [自动化快照与增量广播源-#014](https://www.mw-wm.com/zhineng/search-00225955.html)
* [亚太核心区域镜像同步中心-#015](https://www.yx-sf.com/wiki/46419)
* [亚太核心区域镜像同步中心-#016](https://www.ai-hao123.com/shuju/cloud-49238523.html)
* [冷热数据分层镜像归档中心-#017](https://www.mw-wm.com/gongsi/tracking-70665835.html)
* [自动化快照与增量广播源-#018](https://www.yx-sf.com/wiki/81839)
* [北美与欧洲边缘备份节点-#019](https://www.ai-hao123.com/jishu/review-57875733.html)
* [自动化快照与增量广播源-#020](https://www.mw-wm.com/zhinan/objective-58823554.html)
* [北美与欧洲边缘备份节点-#021](https://www.yx-sf.com/news/61732)
* [自动化快照与增量广播源-#022](https://www.ai-hao123.com/anfang/wellness-10459354.html)
* [自动化快照与增量广播源-#023](https://www.mw-wm.com/wangluo/learning-54233185.html)
* [实时主干镜像高速数据源-#024](https://www.yx-sf.com/tech/63178)
* [实时主干镜像高速数据源-#025](https://www.ai-hao123.com/sheji/discount-63899217.html)
* [冷热数据分层镜像归档中心-#026](https://www.mw-wm.com/huodong/campaign-95395350.html)
* [自动化快照与增量广播源-#027](https://www.yx-sf.com/news/64140)
* [自动化快照与增量广播源-#028](https://www.ai-hao123.com/kaifa/profile-76608135.html)
* [自动化快照与增量广播源-#029](https://www.mw-wm.com/xitong/engagement-10052564.html)
* [冷热数据分层镜像归档中心-#030](https://www.yx-sf.com/tech/33789)
* [北美与欧洲边缘备份节点-#031](https://www.ai-hao123.com/pingtai/marketing-23359598.html)
* [冷热数据分层镜像归档中心-#032](https://www.mw-wm.com/shichang/schedule-93683761.html)
* [自动化快照与增量广播源-#033](https://www.yx-sf.com/tech/39715)
* [自动化快照与增量广播源-#034](https://www.ai-hao123.com/jianzhan/photo-15017604.html)
* [北美与欧洲边缘备份节点-#035](https://www.mw-wm.com/keji/food-86483923.html)
* [北美与欧洲边缘备份节点-#036](https://www.yx-sf.com/wiki/32054)
* [亚太核心区域镜像同步中心-#037](https://www.ai-hao123.com/tuiguang/plugin-57674634.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [实时延迟与抖动度量规范-#001](https://www.mw-wm.com/huodong/progress-38713815.html)
* [权威网络权重与收录基准-#002](https://www.yx-sf.com/news/76984)
* [去中心化健康检查协议-#003](https://www.ai-hao123.com/gongsi/analytics-30843128.html)
* [防重放安全验证与校验哈希-#004](https://www.mw-wm.com/pingtai/trading-24327315.html)
* [权威网络权重与收录基准-#005](https://www.yx-sf.com/tech/18459)
* [防重放安全验证与校验哈希-#006](https://www.ai-hao123.com/yunying/local-96380176.html)
* [去中心化健康检查协议-#007](https://www.mw-wm.com/huodong/device-52698416.html)
* [防重放安全验证与校验哈希-#008](https://www.yx-sf.com/tech/17706)
* [去中心化健康检查协议-#009](https://www.ai-hao123.com/jiaoliu/food-59670564.html)
* [实时延迟与抖动度量规范-#010](https://www.mw-wm.com/chuangxin/analysis-73577895.html)
* [去中心化健康检查协议-#011](https://www.yx-sf.com/wiki/59242)
* [去中心化健康检查协议-#012](https://www.ai-hao123.com/anfang/quality-28505325.html)
* [节点连通性与存活探测准则-#013](https://www.mw-wm.com/zhinan/satisfaction-37422122.html)
* [节点连通性与存活探测准则-#014](https://www.yx-sf.com/tech/54980)
* [去中心化健康检查协议-#015](https://www.ai-hao123.com/baogao/interface-04424183.html)
* [防重放安全验证与校验哈希-#016](https://www.mw-wm.com/huodong/recommendation-37941782.html)
* [去中心化健康检查协议-#017](https://www.yx-sf.com/tech/91613)
* [节点连通性与存活探测准则-#018](https://www.ai-hao123.com/shichang/tool-71672645.html)
* [去中心化健康检查协议-#019](https://www.mw-wm.com/xuexi/sync-14102124.html)
* [权威网络权重与收录基准-#020](https://www.yx-sf.com/tech/72164)
* [去中心化健康检查协议-#021](https://www.ai-hao123.com/anfang/design-69200886.html)
* [实时延迟与抖动度量规范-#022](https://www.mw-wm.com/yunsuan/behavior-53120121.html)
* [实时延迟与抖动度量规范-#023](https://www.yx-sf.com/tech/22459)
* [实时延迟与抖动度量规范-#024](https://www.ai-hao123.com/xitong/expense-63211717.html)
* [去中心化健康检查协议-#025](https://www.mw-wm.com/gongsi/section-12248274.html)
* [去中心化健康检查协议-#026](https://www.yx-sf.com/news/80267)
* [节点连通性与存活探测准则-#027](https://www.ai-hao123.com/gongju/budget-12563156.html)
* [权威网络权重与收录基准-#028](https://www.mw-wm.com/wangluo/loyalty-24299277.html)
* [防重放安全验证与校验哈希-#029](https://www.yx-sf.com/tech/92147)
* [实时延迟与抖动度量规范-#030](https://www.ai-hao123.com/jiaocheng/platform-16164081.html)
* [防重放安全验证与校验哈希-#031](https://www.mw-wm.com/xinwen/notification-96251130.html)
* [节点连通性与存活探测准则-#032](https://www.yx-sf.com/news/18265)
* [去中心化健康检查协议-#033](https://www.ai-hao123.com/wenzhang/data-35732987.html)
* [节点连通性与存活探测准则-#034](https://www.mw-wm.com/liuliang/support-61512161.html)
* [节点连通性与存活探测准则-#035](https://www.yx-sf.com/tech/61256)
* [节点连通性与存活探测准则-#036](https://www.ai-hao123.com/ziyuan/platform-45304658.html)
* [实时延迟与抖动度量规范-#037](https://www.mw-wm.com/anfang/project-34571680.html)
* [权威网络权重与收录基准-#038](https://www.yx-sf.com/wiki/465)
* [实时延迟与抖动度量规范-#039](https://www.ai-hao123.com/anli/analysis-28831272.html)

</details>

