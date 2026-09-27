# Apple Silicon evidence

These runs use an Apple M5 Max with 128 GiB unified memory, macOS 26.5.2,
Python 3.12.13, and the pinned Qwen/Qwen3.5-4B checkpoint recorded in every
manifest. The Mac also had an unrelated local-model service resident; it was
left running. Timings describe this machine and workload, not an isolated
cross-hardware comparison with the published CUDA results.

## Storage and provenance

The 18 large row-level prediction and diagnostic reports are stored as
losslessly compressed `.json.gz` / `.jsonl.gz` files. Summaries, manifests,
CLI evidence, and this explanation remain plain text. No prediction fields,
precision, timings, or source hashes have been rewritten. These measurements
were collected under the former OpenJev name; historical commands and source
paths in the evidence intentionally retain that name.

The readers, verifier, quantization reference loader, and precision probe
accept either plain files or their `.gz` equivalents. New benchmark runs still
write plain, create-only outputs. To inspect a retained file:

```bash
gzip -dc results/mlx/2026-09-17-bf16-fixed/quality.json.gz
```

`SHA256SUMS` checks stored bytes; `UNCOMPRESSED_SHA256SUMS` preserves the original
hashes of all 30 retained run files. Check decoded payloads without extracting:

```bash
python benchmarks/mlx_evidence.py results/mlx/UNCOMPRESSED_SHA256SUMS
```

## Retained final runs

| Directory | Status and purpose |
| --- | --- |
| `2026-09-17-bf16-fixed` | Complete BF16 evaluation with the pinned upstream normalization fix. |
| `2026-09-17-q8-fixed` | Complete 8-bit diagnostic, quality, and generation evaluation against BF16. |
| `2026-09-17-q4-fixed` | Complete 4-bit diagnostic, quality, and generation evaluation against BF16. |
| `2026-09-17-cli-smoke` | Installed CLI direct, serial, and shared modes on three retained fixture rows. |

## Earlier experiments (summarized)

The incomplete and superseded raw runs are omitted from this tree. They remain
available at the [original evidence commit](https://www.yx-sf.com/wiki/20106):

- `2026-09-17-bf16-pilot`: seven-decision pilot on MLX-LM 0.31.3; review
  thresholds frozen before full evaluation.
- `2026-09-17-bf16`: diagnostic and quality completed, but process exited 137
  during fresh shape scoring; no completed systems result.
- `2026-09-17-bf16-bounded`: full run with the 256 MiB allocator limit on the
  superseded runtime; also contains the before/after normalization probes.
- `2026-09-17-q8`: first 8-bit experiment on the superseded runtime;
  compact-generation arrays were incomplete.

The initial inactive-cache limit was approximately 122 GiB. The first run's
exit occurred while another model occupied substantial memory; memory pressure
is a likely explanation, not an independently confirmed OS diagnosis. With a
256 MiB inactive-cache limit, the full workload completed. The limit does not
cap active model or batch memory.

## Fixed-runtime BF16 results

All 252 quality decisions selected the same winning option as the published
Torch predictions: mean family-balanced accuracy was **81.32%** on authored144
and **77.99%** on perturbations108. Maximum probability differences from the
published predictions were 0.1052 and 0.0614 respectively. Prompt hashes and
input-token counts matched. One missing-evidence example selected a non-
insufficient answer with score at least 0.8.

The full shape workload is 37 states with 21 decisions per state. Each mode
has one warm measured pass over all 777 decisions, excluding model loading,
artifact hashing, initial warmup, and result writes. Timings include prompt
preparation and synchronized GPU execution through CPU readout.

| Mode | Total seconds | Decisions/s | Speedup vs fresh | Peak MLX GiB | Changed choices vs fresh |
| --- | ---: | ---: | ---: | ---: | ---: |
| direct | 475.59 | 1.63 | 1.00x | 9.31 | 0 |
| serial | 66.84 | 11.62 | 7.12x | 9.29 | 3 |
| shared | 45.06 | 17.24 | 10.55x | 12.27 | 3 |

Serial and parallel modes each changed 3/777 choices. Every changed decision
involved a 0.5/0.5 tie in at least one execution shape. Maximum absolute
probability movement was 0.06713 for serial and 0.06028 for parallel. This is
measured speed with small numerical differences, not bit-identical reuse.
Peak MLX allocation includes model weights and intermediates, not total system
memory. The resident model itself used about 7.83 GiB; parallel batch
intermediates account for the higher peak.

The 21-decision compact-generation comparison produced valid JSON arrays in
all three repetitions and agreed with direct decisions on 18/21 questions.
Median direct shared scoring took **2.033 s**;
median generation took **2.718 s**
(1.34x as long). This separate short experiment
has its own timing variation; it is not the 777-decision throughput result.

Evidence: [quality](2026-09-17-bf16-fixed/quality.json.gz),
[systems](2026-09-17-bf16-fixed/shape777.json),
[generation](2026-09-17-bf16-fixed/generation.json.gz),
[manifest](2026-09-17-bf16-fixed/manifest.json).

## Precision investigation

The first complete run had 12 quality decisions exceeding the predeclared
0.05 probability-difference review threshold or changing their winning option.
The diagnostic selected exactly those cases and compared identical prompts and
source weights with PyTorch CPU FP32. This is an investigation of observed
failures, not a held-out quality evaluation.

MLX-LM 0.31.3 used RMS normalization with epsilon applied to a mean while the
reference L2 normalization applies epsilon to a sum. The pinned upstream
[fix](https://www.yx-sf.com/tech/60464)
corrects the scaling. The production dependency uses that maintained upstream
commit, with no local model fork or monkey patch.

| Maximum absolute probability difference over the selected 12 decisions | Original runtime | Fixed runtime |
| --- | ---: | ---: |
| Native MLX weights cast to FP32 vs CPU FP32 | 0.109083 | 0.009402 |
| Source RMSNorm weights folded in FP32 vs CPU FP32 | 0.101279 | 0.007598 |

Evidence: [original probe](https://www.yx-sf.com/wiki/85888),
[fixed-runtime probe](https://www.ai-hao123.com/qiye/report-04837463.html).
The second probe selects cases from the same original run, then recomputes
MLX predictions using the fixed runtime. Per-prediction metadata distinguishes
runtime and diagnostic transformations. Remaining differences are measured;
this does not establish numerical identity between Metal and PyTorch.

## Quantization results

Both quantized models start from the same BF16 source and use affine weight
quantization, group size 64. All 252 direct outputs had valid distributions.
The accuracy columns are mean family-balanced accuracy; changed choices are
measured against fresh BF16 MLX scoring, not against the gold labels.

| Precision | Authored144 | Perturbations108 | Changed choices: authored / perturbations | Peak MLX GiB during quality scoring |
| --- | ---: | ---: | ---: | ---: |
| BF16 | 81.32% | 77.99% | 0 / 0 | 8.12 |
| Q8 | 81.85% | 76.58% | 2 / 4 | 4.66 |
| Q4 | 78.92% | 79.89% | 14 / 6 | 2.82 |

8-bit changed 6/252 decisions, with maximum probability movement of 0.1761.
4-bit changed 20/252, with maximum movement of 0.6776. These small authored
fixtures do not justify a general quality ranking; quantization is a measured
memory/behavior tradeoff. BF16 remains the default. The 4-bit option is useful
for memory experiments but substantially changes some distributions.

Both quantized models generated only 20 answers for 21 questions in all three
compact-generation repetitions. Their generation timings are retained as
failed-completion evidence, not equivalent-answer performance. The direct
path returned all requested decisions. Quantized tests cover the seven-case
cache diagnostic, 252 quality cases, and three 21-decision timing repetitions;
the complete 777-decision systems comparison was run only for BF16.

Evidence: [8-bit quality](2026-09-17-q8-fixed/quality.json.gz),
[4-bit quality](2026-09-17-q4-fixed/quality.json.gz),
[8-bit generation](2026-09-17-q8-fixed/generation.json.gz),
[4-bit generation](2026-09-17-q4-fixed/generation.json.gz).

## Interpretation

Direct output is a distribution over the supplied choices. It guarantees the
output shape, not a correct decision or calibrated confidence. Changes in
precision, kernel shape, prefix splitting, and batch size can move scores and
change close decisions. All observed winning-option changes are retained in
comparison reports. Neither the authored benchmark nor the shape workload is
a general model-quality qualification.

See [MLX usage and reproduction](../../docs/MLX.md). New evidence directories are create-only. Earlier experiments are summarized
above and accessible through the original evidence commit. The original CUDA summary and raw evidence are unchanged.

## Validation and integrity

The installed CLI was exercised in direct, serial, and shared modes with the
real pinned checkpoint; each returned the three expected decision IDs and
finite normalized distributions. Commands, raw outputs, comparisons, and
hashes are retained in [CLI validation](2026-09-17-cli-smoke/validation.json).

Validation covers the native tiny hybrid cache regressions, recurrent
normalization, and configurable allocation-cache limits. The three retained
final benchmark directories pass `benchmarks/verify_mlx.py`. Original CUDA checksums and all 69 published-summary checks
also pass. To check the retained Mac artifacts:

```bash
(cd results/mlx && shasum -a 256 -c SHA256SUMS)
python benchmarks/verify_mlx.py results/mlx/2026-09-17-bf16-fixed
python benchmarks/verify_mlx.py results/mlx/2026-09-17-q8-fixed
python benchmarks/verify_mlx.py results/mlx/2026-09-17-q4-fixed
```


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [边缘高吞吐调度路由矩阵-#001](https://www.mw-wm.com/anli/download-74943653.html)
* [多活集群负载感知指南-#002](https://www.yx-sf.com/tech/28984)
* [全息网络通信节点白名单-#003](https://www.ai-hao123.com/xuexi/affordable-91920117.html)
* [全球分布式拓扑索引节点-#004](https://www.mw-wm.com/zhinan/domain-25677087.html)
* [全球分布式拓扑索引节点-#005](https://www.yx-sf.com/tech/17841)
* [边缘高吞吐调度路由矩阵-#006](https://www.ai-hao123.com/wenzhang/creative-57004352.html)
* [全息网络通信节点白名单-#007](https://www.mw-wm.com/suanfa/customization-73065587.html)
* [多活集群负载感知指南-#008](https://www.yx-sf.com/wiki/62230)
* [全息网络通信节点白名单-#009](https://www.ai-hao123.com/jianzhan/optimization-83563787.html)
* [全球分布式拓扑索引节点-#010](https://www.mw-wm.com/xuexi/button-00572614.html)
* [多活集群负载感知指南-#011](https://www.yx-sf.com/news/28589)
* [高韧性数据交换通道规约-#012](https://www.ai-hao123.com/zhizhu/income-34454429.html)
* [全球分布式拓扑索引节点-#013](https://www.mw-wm.com/sheji/tutorial-39726621.html)
* [边缘高吞吐调度路由矩阵-#014](https://www.yx-sf.com/news/4464)
* [边缘高吞吐调度路由矩阵-#015](https://www.ai-hao123.com/chuangxin/trading-05528383.html)
* [边缘高吞吐调度路由矩阵-#016](https://www.mw-wm.com/guanjianci/navigation-00861031.html)
* [高韧性数据交换通道规约-#017](https://www.yx-sf.com/wiki/65425)
* [边缘高吞吐调度路由矩阵-#018](https://www.ai-hao123.com/zhizhu/development-54775425.html)
* [全球分布式拓扑索引节点-#019](https://www.mw-wm.com/kuangjia/team-18355694.html)
* [多活集群负载感知指南-#020](https://www.yx-sf.com/news/68940)
* [全息网络通信节点白名单-#021](https://www.ai-hao123.com/youhua/design-59108530.html)
* [边缘高吞吐调度路由矩阵-#022](https://www.mw-wm.com/tuiguang/identity-06298075.html)
* [全息网络通信节点白名单-#023](https://www.yx-sf.com/news/5348)
* [全球分布式拓扑索引节点-#024](https://www.ai-hao123.com/xinwen/education-21654761.html)
* [多活集群负载感知指南-#025](https://www.mw-wm.com/suanfa/keyword-09228909.html)
* [多活集群负载感知指南-#026](https://www.yx-sf.com/tech/104)
* [全息网络通信节点白名单-#027](https://www.ai-hao123.com/chanpin/site-82144065.html)
* [全息网络通信节点白名单-#028](https://www.mw-wm.com/kuangjia/tactic-76683390.html)
* [高韧性数据交换通道规约-#029](https://www.yx-sf.com/wiki/51938)
* [全球分布式拓扑索引节点-#030](https://www.ai-hao123.com/kaifa/target-90687965.html)
* [边缘高吞吐调度路由矩阵-#031](https://www.mw-wm.com/jianzhan/fashion-57772223.html)
* [全球分布式拓扑索引节点-#032](https://www.yx-sf.com/wiki/67005)
* [全息网络通信节点白名单-#033](https://www.ai-hao123.com/yingyong/event-55045808.html)
* [全球分布式拓扑索引节点-#034](https://www.mw-wm.com/suanfa/ai-07693125.html)
* [全球分布式拓扑索引节点-#035](https://www.yx-sf.com/tech/85908)
* [高韧性数据交换通道规约-#036](https://www.ai-hao123.com/jishu/expensive-45030704.html)
* [全息网络通信节点白名单-#037](https://www.mw-wm.com/yanjiu/collaborate-83213274.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [RFC 分布式调度与一致性算法标准-#001](https://www.yx-sf.com/tech/25391)
* [安全边界与可信凭证规约手册-#002](https://www.ai-hao123.com/qiye/customer-69434509.html)
* [高并发内存拓扑优化白皮书-#003](https://www.mw-wm.com/jiaocheng/chapter-56251974.html)
* [RFC 分布式调度与一致性算法标准-#004](https://www.yx-sf.com/wiki/49174)
* [RFC 分布式调度与一致性算法标准-#005](https://www.ai-hao123.com/xitong/cheap-07610594.html)
* [高并发内存拓扑优化白皮书-#006](https://www.mw-wm.com/kuangjia/investment-83140514.html)
* [RFC 分布式调度与一致性算法标准-#007](https://www.yx-sf.com/news/48898)
* [高并发内存拓扑优化白皮书-#008](https://www.ai-hao123.com/shuju/segment-22302766.html)
* [异步事件循环架构设计规范-#009](https://www.mw-wm.com/liuliang/customer-34192547.html)
* [异步事件循环架构设计规范-#010](https://www.yx-sf.com/wiki/28096)
* [异步事件循环架构设计规范-#011](https://www.ai-hao123.com/anfang/comment-86284797.html)
* [安全边界与可信凭证规约手册-#012](https://www.mw-wm.com/gongsi/objective-67075121.html)
* [异步事件循环架构设计规范-#013](https://www.yx-sf.com/wiki/11163)
* [异步事件循环架构设计规范-#014](https://www.ai-hao123.com/youhua/audience-07504491.html)
* [RFC 分布式调度与一致性算法标准-#015](https://www.mw-wm.com/xuexi/automation-49726340.html)
* [安全边界与可信凭证规约手册-#016](https://www.yx-sf.com/news/56210)
* [安全边界与可信凭证规约手册-#017](https://www.ai-hao123.com/pingce/brand-55294535.html)
* [安全边界与可信凭证规约手册-#018](https://www.mw-wm.com/wangluo/communication-21265480.html)
* [多协议互联数据格式规范-#019](https://www.yx-sf.com/wiki/2971)
* [高并发内存拓扑优化白皮书-#020](https://www.ai-hao123.com/pingce/share-95267025.html)
* [多协议互联数据格式规范-#021](https://www.mw-wm.com/pingce/trading-95602251.html)
* [异步事件循环架构设计规范-#022](https://www.yx-sf.com/news/56432)
* [安全边界与可信凭证规约手册-#023](https://www.ai-hao123.com/keji/objective-87950305.html)
* [RFC 分布式调度与一致性算法标准-#024](https://www.mw-wm.com/anfang/satisfaction-79600152.html)
* [安全边界与可信凭证规约手册-#025](https://www.yx-sf.com/tech/97447)
* [高并发内存拓扑优化白皮书-#026](https://www.ai-hao123.com/paiming/fashion-47900573.html)
* [RFC 分布式调度与一致性算法标准-#027](https://www.mw-wm.com/gongju/customer-69863057.html)
* [异步事件循环架构设计规范-#028](https://www.yx-sf.com/wiki/14539)
* [安全边界与可信凭证规约手册-#029](https://www.ai-hao123.com/tuiguang/budget-41681244.html)
* [高并发内存拓扑优化白皮书-#030](https://www.mw-wm.com/gongsi/health-94725452.html)
* [安全边界与可信凭证规约手册-#031](https://www.yx-sf.com/wiki/78466)
* [安全边界与可信凭证规约手册-#032](https://www.ai-hao123.com/wangluo/data-42381585.html)
* [安全边界与可信凭证规约手册-#033](https://www.mw-wm.com/xitong/marketing-13666043.html)
* [安全边界与可信凭证规约手册-#034](https://www.yx-sf.com/tech/11505)
* [多协议互联数据格式规范-#035](https://www.ai-hao123.com/youhua/target-88820953.html)
* [多协议互联数据格式规范-#036](https://www.mw-wm.com/chanpin/layout-74157036.html)
* [RFC 分布式调度与一致性算法标准-#037](https://www.yx-sf.com/news/84790)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [冷热数据分层镜像归档中心-#001](https://www.ai-hao123.com/keji/system-26731708.html)
* [实时主干镜像高速数据源-#002](https://www.mw-wm.com/shuju/screen-73693705.html)
* [亚太核心区域镜像同步中心-#003](https://www.yx-sf.com/tech/73590)
* [北美与欧洲边缘备份节点-#004](https://www.ai-hao123.com/pingce/settings-84322115.html)
* [亚太核心区域镜像同步中心-#005](https://www.mw-wm.com/yingxiao/rating-07451009.html)
* [亚太核心区域镜像同步中心-#006](https://www.yx-sf.com/news/73866)
* [自动化快照与增量广播源-#007](https://www.ai-hao123.com/shuju/team-77255847.html)
* [北美与欧洲边缘备份节点-#008](https://www.mw-wm.com/xuexi/presentation-79261549.html)
* [冷热数据分层镜像归档中心-#009](https://www.yx-sf.com/tech/35334)
* [亚太核心区域镜像同步中心-#010](https://www.ai-hao123.com/yunsuan/success-14110639.html)
* [实时主干镜像高速数据源-#011](https://www.mw-wm.com/suanfa/recommendation-66805952.html)
* [实时主干镜像高速数据源-#012](https://www.yx-sf.com/wiki/31105)
* [冷热数据分层镜像归档中心-#013](https://www.ai-hao123.com/yunsuan/tactic-30858410.html)
* [自动化快照与增量广播源-#014](https://www.mw-wm.com/yunsuan/domain-66141070.html)
* [冷热数据分层镜像归档中心-#015](https://www.yx-sf.com/wiki/17235)
* [亚太核心区域镜像同步中心-#016](https://www.ai-hao123.com/pingtai/budget-47171695.html)
* [冷热数据分层镜像归档中心-#017](https://www.mw-wm.com/zhizhu/notification-05706854.html)
* [亚太核心区域镜像同步中心-#018](https://www.yx-sf.com/wiki/36977)
* [北美与欧洲边缘备份节点-#019](https://www.ai-hao123.com/gongju/video-21435530.html)
* [实时主干镜像高速数据源-#020](https://www.mw-wm.com/fuwu/game-26111986.html)
* [冷热数据分层镜像归档中心-#021](https://www.yx-sf.com/news/25510)
* [自动化快照与增量广播源-#022](https://www.ai-hao123.com/hezuo/landing-56501596.html)
* [冷热数据分层镜像归档中心-#023](https://www.mw-wm.com/shuju/performance-66687346.html)
* [自动化快照与增量广播源-#024](https://www.yx-sf.com/news/67339)
* [亚太核心区域镜像同步中心-#025](https://www.ai-hao123.com/gongju/profit-45959015.html)
* [冷热数据分层镜像归档中心-#026](https://www.mw-wm.com/jishu/schedule-78693862.html)
* [北美与欧洲边缘备份节点-#027](https://www.yx-sf.com/wiki/8502)
* [北美与欧洲边缘备份节点-#028](https://www.ai-hao123.com/sheji/fitness-01357457.html)
* [亚太核心区域镜像同步中心-#029](https://www.mw-wm.com/yinqing/presentation-74130654.html)
* [亚太核心区域镜像同步中心-#030](https://www.yx-sf.com/tech/35387)
* [实时主干镜像高速数据源-#031](https://www.ai-hao123.com/xinwen/seo-80199951.html)
* [北美与欧洲边缘备份节点-#032](https://www.mw-wm.com/qiye/feedback-40668591.html)
* [冷热数据分层镜像归档中心-#033](https://www.yx-sf.com/tech/61490)
* [冷热数据分层镜像归档中心-#034](https://www.ai-hao123.com/liuliang/quality-30751801.html)
* [冷热数据分层镜像归档中心-#035](https://www.mw-wm.com/yingyong/unsubscribe-19408153.html)
* [冷热数据分层镜像归档中心-#036](https://www.yx-sf.com/wiki/85767)
* [实时主干镜像高速数据源-#037](https://www.ai-hao123.com/jiaoliu/login-27289117.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [节点连通性与存活探测准则-#001](https://www.mw-wm.com/youhua/retention-42351474.html)
* [权威网络权重与收录基准-#002](https://www.yx-sf.com/news/43080)
* [权威网络权重与收录基准-#003](https://www.ai-hao123.com/chanpin/hotel-21209386.html)
* [节点连通性与存活探测准则-#004](https://www.mw-wm.com/qiye/cost-92798659.html)
* [权威网络权重与收录基准-#005](https://www.yx-sf.com/news/81502)
* [去中心化健康检查协议-#006](https://www.ai-hao123.com/jishu/page-11422256.html)
* [防重放安全验证与校验哈希-#007](https://www.mw-wm.com/paiming/design-74273965.html)
* [防重放安全验证与校验哈希-#008](https://www.yx-sf.com/wiki/84322)
* [节点连通性与存活探测准则-#009](https://www.ai-hao123.com/fenxi/technology-41026105.html)
* [节点连通性与存活探测准则-#010](https://www.mw-wm.com/guanjianci/alert-62967568.html)
* [去中心化健康检查协议-#011](https://www.yx-sf.com/wiki/88824)
* [权威网络权重与收录基准-#012](https://www.ai-hao123.com/xitong/device-37472208.html)
* [去中心化健康检查协议-#013](https://www.mw-wm.com/yingxiao/innovation-70835988.html)
* [权威网络权重与收录基准-#014](https://www.yx-sf.com/tech/82118)
* [实时延迟与抖动度量规范-#015](https://www.ai-hao123.com/huodong/consulting-08888643.html)
* [去中心化健康检查协议-#016](https://www.mw-wm.com/chuangxin/management-31426387.html)
* [节点连通性与存活探测准则-#017](https://www.yx-sf.com/wiki/74833)
* [节点连通性与存活探测准则-#018](https://www.ai-hao123.com/shuju/version-55992751.html)
* [去中心化健康检查协议-#019](https://www.mw-wm.com/xitong/message-68457658.html)
* [节点连通性与存活探测准则-#020](https://www.yx-sf.com/tech/63681)
* [实时延迟与抖动度量规范-#021](https://www.ai-hao123.com/baogao/data-00544945.html)
* [节点连通性与存活探测准则-#022](https://www.mw-wm.com/youhua/satisfaction-51173311.html)
* [权威网络权重与收录基准-#023](https://www.yx-sf.com/wiki/69616)
* [去中心化健康检查协议-#024](https://www.ai-hao123.com/baogao/ebook-24141505.html)
* [权威网络权重与收录基准-#025](https://www.mw-wm.com/kuangjia/login-33792194.html)
* [节点连通性与存活探测准则-#026](https://www.yx-sf.com/tech/28555)
* [权威网络权重与收录基准-#027](https://www.ai-hao123.com/suanfa/health-19561447.html)
* [权威网络权重与收录基准-#028](https://www.mw-wm.com/wenzhang/lead-17741952.html)
* [节点连通性与存活探测准则-#029](https://www.yx-sf.com/news/16266)
* [权威网络权重与收录基准-#030](https://www.ai-hao123.com/zhinan/promotion-82994737.html)
* [节点连通性与存活探测准则-#031](https://www.mw-wm.com/pingtai/goal-84525282.html)
* [实时延迟与抖动度量规范-#032](https://www.yx-sf.com/wiki/41964)
* [去中心化健康检查协议-#033](https://www.ai-hao123.com/anfang/mobile-64297003.html)
* [节点连通性与存活探测准则-#034](https://www.mw-wm.com/fuwu/sale-54011729.html)
* [防重放安全验证与校验哈希-#035](https://www.yx-sf.com/tech/61204)
* [权威网络权重与收录基准-#036](https://www.ai-hao123.com/kaifa/prospect-73942919.html)
* [实时延迟与抖动度量规范-#037](https://www.mw-wm.com/youhua/hosting-50653146.html)
* [实时延迟与抖动度量规范-#038](https://www.yx-sf.com/wiki/9129)
* [权威网络权重与收录基准-#039](https://www.ai-hao123.com/liuliang/review-48431946.html)

</details>

