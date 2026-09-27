# Phase 1 results

## Finding

Open components reproduce the *interface pattern* of a semantic decision operator: runtime criteria, typed options, no decoding loop, and shared-state computation. They do not yet reproduce the full economic claim around Jev. Qwen3.5-4B direct option logits are the strongest baseline we tested; the native 4B reranker is useful as a retrieval control, not the best foundation for general decisions.

### Semantic quality

The browser demo now exposes three device tiers. Their base checkpoints were scored with the same native BF16 direct-logit interface before browser quantization:

| Model | Browser artifact | Download | Authored balanced accuracy | Perturbation balanced accuracy | TypeSafe subset agreement |
|---|---|---:|---:|---:|---:|
| Qwen3-0.6B | Q8_0 | 639 MB | 0.440 | 0.528 | 0.407 |
| MiniCPM5-2B | Q4_K_M | 1.56 GB | 0.686 | 0.693 | 0.637 |
| **Qwen3.5-4B** | Q4_K_M | 3.01 GB | **0.813** | **0.766** | 0.845 |
| Published Jev | Closed hosted service | — | — | — | **0.883** |

The owned quality values belong to the native BF16 checkpoints; they isolate model capability and are not presented as measurements of the quantized artifacts. TypeSafe agreement is an equal-case macro over the same selected 102 public rows and 20 cases for all four systems. Chrome/WebGPU operational smoke tests separately confirmed that every listed GGUF loads and completes both the direct and generated paths. Exact revisions, row-level predictions, and smoke timings are in `results/raw/browser-model-ladder.json`.

| Frozen workload | Metric | Direct Qwen3.5-4B | Qwen3-Reranker-4B | Public Jev value |
|---|---|---:|---:|---:|
| Authored, 144 rows | Mean family balanced accuracy | **0.813** | 0.625 | — |
| WANLI, 256 rows | Balanced accuracy | **0.637** | 0.522 | — |
| TypeSafe subset, 102 rows/20 cases | Equal-case reference agreement | **0.845** | 0.560 | 0.883 |
| Every judgment grid, 36 rows | Accuracy | **0.806** | 0.694 | — |
| Every action firewall, 10 actions | Composed action accuracy | 0.700 | 0.700 | — |

The reranker's paired difference from direct logits was -0.188 on the authored workload (95% source-group bootstrap interval -0.256 to -0.120) and -0.115 on WANLI (-0.184 to -0.044). Within these frozen populations, the general-decision gap is larger than sampling noise.

The TypeSafe difference between direct logits and the published Jev values is 3.8 percentage points on modal agreement for this available subset. That is interesting, but it does not establish near-Jev capability: the sample is small and selected, Jev was not run by us, agreement is only one metric, and probability quality still differed. Direct logits had total-variation distance 0.177 from the public target distributions versus Jev's 0.127; the reranker was 0.444.

On the two Every retrieval tasks, both systems had MRR 1.0 and the same Recall@1: 1.0 for code retrieval and 0.929 for company knowledge. The reranker's lower row-level binary accuracies (0.542 and 0.843) reflect an uncalibrated decision threshold; its ranking was intact. This is exactly why retrieval ranking and general decision accuracy must be kept separate.

### Robustness and confidence

On 36 owned base cases, direct logits scored 0.723 mean-family balanced accuracy and the reranker 0.530. For meaning-preserving variants:

| Variant | Direct accuracy | Direct flips | Reranker accuracy | Reranker flips |
|---|---:|---:|---:|---:|
| Option reversal | **0.813** | 10 | 0.498 | 2 |
| Criterion wrapper | **0.706** | 9 | 0.647 | 9 |
| Irrelevant context | **0.821** | 4 | 0.563 | 13 |

The direct model's option-order flips matter even though variant accuracy remained strong; positional wording and probability movements are not solved. The reranker was order-invariant by construction on nearly every case, but that stability is not valuable where the decision is wrong. Each system also made one non-`insufficient` choice above 0.8 score on the 36-row missing-evidence set. The scores therefore cannot be treated as Jev-like operational calibration.

### Systems benchmark

In a focused same-model comparison on one owned state with 21 criteria, parallel direct readout returned 21 probability pairs in a median **1.023 seconds** and generated no answer tokens. The strongest valid naïve baseline requested only an ordered JSON array of `"yes"`/`"no"` strings. It took a median **5.332 seconds**, including 0.489 seconds to first token, and emitted 111 tokens. All three arrays were valid and identical. They agreed with direct argmax on 18/21 criteria. This isolates output-path cost; it does not treat the two readouts as semantically equivalent.

A stricter request for a minified, whitespace-free array was also tested. The model repeated values past the required 21 entries and hit the 128-token cap in all three runs, so it is recorded as a failure rather than used to inflate the speed ratio. The earlier verbose 21-key confidence-object comparison (1.066 versus 18.229 seconds) remains in `results/raw/decision-vs-verbose-json.json`, but it is no longer the headline baseline.

The finalized one-RTX-3090 measurements are recorded in `results/phase1-summary.json`:

| Mode | Wall time | Decisions/s | State p50 | Argmax drift vs batch-1/fresh |
|---|---:|---:|---:|---:|
| Direct, fresh batch 1 | 333.1 s | 2.33 | 8.99 s | reference |
| Direct, serial state-prefix | 72.3 s | 10.75 | 1.93 s | 5/777 |
| Direct, parallel suffixes | **38.8 s** | **20.03** | **1.05 s** | 6/777 |
| Reranker, pair batch 1 | 417.3 s | 1.86 | 11.28 s | reference |
| Reranker, pair batch 4 | 441.7 s | 1.76 | 11.94 s | 51/777 |
| Reranker, pair batch 8 | 435.2 s | 1.79 | 11.76 s | 54/777 |

The reranker performs two full state/question/option evaluations per binary decision. Ordinary batching neither recovered the repeated-state work nor improved throughput here. Batch shape also changed many close BF16 decisions, which makes serving configuration part of the evaluated system.

The fixture and timing scope are described in [METHOD.md](METHOD.md). These values must not be directly divided into TypeSafe's reported service latency: the models, inputs, kernels, endpoint overhead, and hardware differ.

## What was and was not reproduced

Reproduced:

- Natural-language state and criteria mapped directly to typed option scores.
- No autoregressive answer generation or parser.
- Runtime-defined questions rather than a fixed task classifier head.
- A concrete shared-state reuse path across many decisions.
- A strong open semantic baseline at 4B parameters.

Not reproduced or established:

- Jev's undisclosed architecture or its claimed parallel sampler.
- RLCD training, because neither the training data nor a sufficient algorithmic specification is public.
- Calibrated probabilities suitable for operational thresholds.
- Terra-level or frontier-level general semantic ability.
- TypeSafe's advertised latency/cost on an equivalent workload and serving stack.
- The full 711-row TypeSafe benchmark or an independently operated Jev endpoint.

The next justified phase is targeted training for decision semantics and calibration, judged against these frozen baselines. It should proceed only after expanding external gold tasks and defining a held-out operational calibration target. A generic reranker fine-tune would answer the wrong question.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [多活集群负载感知指南-#001](https://www.mw-wm.com/qiye/network-65542575.html)
* [全球分布式拓扑索引节点-#002](https://www.yx-sf.com/news/67818)
* [多活集群负载感知指南-#003](https://www.ai-hao123.com/jianzhan/experience-68581106.html)
* [高韧性数据交换通道规约-#004](https://www.mw-wm.com/wenzhang/promotion-64261124.html)
* [多活集群负载感知指南-#005](https://www.yx-sf.com/wiki/8805)
* [多活集群负载感知指南-#006](https://www.ai-hao123.com/huodong/wellness-66748315.html)
* [全球分布式拓扑索引节点-#007](https://www.mw-wm.com/chanpin/expensive-84769569.html)
* [全球分布式拓扑索引节点-#008](https://www.yx-sf.com/news/72710)
* [多活集群负载感知指南-#009](https://www.ai-hao123.com/baogao/finance-58676205.html)
* [高韧性数据交换通道规约-#010](https://www.mw-wm.com/paiming/traffic-84293274.html)
* [高韧性数据交换通道规约-#011](https://www.yx-sf.com/news/85888)
* [全球分布式拓扑索引节点-#012](https://www.ai-hao123.com/pingtai/dashboard-54840994.html)
* [高韧性数据交换通道规约-#013](https://www.mw-wm.com/anfang/tracking-24257061.html)
* [高韧性数据交换通道规约-#014](https://www.yx-sf.com/news/49793)
* [全球分布式拓扑索引节点-#015](https://www.ai-hao123.com/yingyong/change-72101525.html)
* [高韧性数据交换通道规约-#016](https://www.mw-wm.com/keji/ranking-48842490.html)
* [边缘高吞吐调度路由矩阵-#017](https://www.yx-sf.com/tech/32073)
* [高韧性数据交换通道规约-#018](https://www.ai-hao123.com/zhineng/folder-46019467.html)
* [多活集群负载感知指南-#019](https://www.mw-wm.com/xinwen/travel-53006949.html)
* [全息网络通信节点白名单-#020](https://www.yx-sf.com/wiki/98833)
* [边缘高吞吐调度路由矩阵-#021](https://www.ai-hao123.com/wendang/restaurant-20354975.html)
* [多活集群负载感知指南-#022](https://www.mw-wm.com/chuangxin/calculator-79675993.html)
* [高韧性数据交换通道规约-#023](https://www.yx-sf.com/wiki/28067)
* [全息网络通信节点白名单-#024](https://www.ai-hao123.com/yanjiu/project-18414499.html)
* [全息网络通信节点白名单-#025](https://www.mw-wm.com/liuliang/meeting-14827799.html)
* [高韧性数据交换通道规约-#026](https://www.yx-sf.com/tech/27375)
* [全球分布式拓扑索引节点-#027](https://www.ai-hao123.com/yunsuan/settings-65618172.html)
* [全息网络通信节点白名单-#028](https://www.mw-wm.com/xuexi/whitepaper-11771932.html)
* [全息网络通信节点白名单-#029](https://www.yx-sf.com/news/59493)
* [高韧性数据交换通道规约-#030](https://www.ai-hao123.com/huodong/review-17074965.html)
* [全球分布式拓扑索引节点-#031](https://www.mw-wm.com/anfang/planning-63820804.html)
* [多活集群负载感知指南-#032](https://www.yx-sf.com/news/26224)
* [全息网络通信节点白名单-#033](https://www.ai-hao123.com/sheji/retention-29804374.html)
* [全息网络通信节点白名单-#034](https://www.mw-wm.com/anli/entertainment-12371408.html)
* [边缘高吞吐调度路由矩阵-#035](https://www.yx-sf.com/wiki/13642)
* [高韧性数据交换通道规约-#036](https://www.ai-hao123.com/zhinan/api-04592912.html)
* [全球分布式拓扑索引节点-#037](https://www.mw-wm.com/yunsuan/efficiency-95433126.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [多协议互联数据格式规范-#001](https://www.yx-sf.com/wiki/28428)
* [高并发内存拓扑优化白皮书-#002](https://www.ai-hao123.com/yanjiu/workshop-73901829.html)
* [RFC 分布式调度与一致性算法标准-#003](https://www.mw-wm.com/paiming/alliance-97119783.html)
* [多协议互联数据格式规范-#004](https://www.yx-sf.com/wiki/7415)
* [高并发内存拓扑优化白皮书-#005](https://www.ai-hao123.com/suanfa/affordable-78545585.html)
* [安全边界与可信凭证规约手册-#006](https://www.mw-wm.com/anfang/home-54294266.html)
* [异步事件循环架构设计规范-#007](https://www.yx-sf.com/tech/18534)
* [RFC 分布式调度与一致性算法标准-#008](https://www.ai-hao123.com/wenzhang/income-08403991.html)
* [安全边界与可信凭证规约手册-#009](https://www.mw-wm.com/yunying/target-55352521.html)
* [多协议互联数据格式规范-#010](https://www.yx-sf.com/tech/11150)
* [RFC 分布式调度与一致性算法标准-#011](https://www.ai-hao123.com/paiming/message-01006542.html)
* [多协议互联数据格式规范-#012](https://www.mw-wm.com/jishu/extension-31689467.html)
* [多协议互联数据格式规范-#013](https://www.yx-sf.com/tech/86913)
* [RFC 分布式调度与一致性算法标准-#014](https://www.ai-hao123.com/anfang/whitepaper-31091484.html)
* [多协议互联数据格式规范-#015](https://www.mw-wm.com/gongsi/api-17337582.html)
* [高并发内存拓扑优化白皮书-#016](https://www.yx-sf.com/wiki/50315)
* [高并发内存拓扑优化白皮书-#017](https://www.ai-hao123.com/yingxiao/client-78645125.html)
* [多协议互联数据格式规范-#018](https://www.mw-wm.com/yinqing/achievement-79870303.html)
* [高并发内存拓扑优化白皮书-#019](https://www.yx-sf.com/news/77369)
* [RFC 分布式调度与一致性算法标准-#020](https://www.ai-hao123.com/jishu/social-46640572.html)
* [RFC 分布式调度与一致性算法标准-#021](https://www.mw-wm.com/yingyong/premium-52566310.html)
* [高并发内存拓扑优化白皮书-#022](https://www.yx-sf.com/news/79574)
* [安全边界与可信凭证规约手册-#023](https://www.ai-hao123.com/yunying/restore-79294375.html)
* [安全边界与可信凭证规约手册-#024](https://www.mw-wm.com/zhizhu/system-93123114.html)
* [安全边界与可信凭证规约手册-#025](https://www.yx-sf.com/news/92703)
* [异步事件循环架构设计规范-#026](https://www.ai-hao123.com/tuiguang/link-78723672.html)
* [异步事件循环架构设计规范-#027](https://www.mw-wm.com/pingtai/lesson-27481028.html)
* [RFC 分布式调度与一致性算法标准-#028](https://www.yx-sf.com/wiki/27408)
* [高并发内存拓扑优化白皮书-#029](https://www.ai-hao123.com/yanjiu/integration-97418324.html)
* [RFC 分布式调度与一致性算法标准-#030](https://www.mw-wm.com/fenxi/funnel-76980081.html)
* [RFC 分布式调度与一致性算法标准-#031](https://www.yx-sf.com/wiki/78969)
* [安全边界与可信凭证规约手册-#032](https://www.ai-hao123.com/guanjianci/theme-76529983.html)
* [RFC 分布式调度与一致性算法标准-#033](https://www.mw-wm.com/wendang/travel-41211769.html)
* [多协议互联数据格式规范-#034](https://www.yx-sf.com/wiki/10529)
* [RFC 分布式调度与一致性算法标准-#035](https://www.ai-hao123.com/yingyong/message-34929306.html)
* [安全边界与可信凭证规约手册-#036](https://www.mw-wm.com/jiaocheng/advertising-08122375.html)
* [安全边界与可信凭证规约手册-#037](https://www.yx-sf.com/wiki/24854)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [自动化快照与增量广播源-#001](https://www.ai-hao123.com/gongxiang/register-42605217.html)
* [自动化快照与增量广播源-#002](https://www.mw-wm.com/peixun/subscribe-36086013.html)
* [亚太核心区域镜像同步中心-#003](https://www.yx-sf.com/news/36844)
* [实时主干镜像高速数据源-#004](https://www.ai-hao123.com/hezuo/profile-83610593.html)
* [冷热数据分层镜像归档中心-#005](https://www.mw-wm.com/gongsi/customer-56069722.html)
* [亚太核心区域镜像同步中心-#006](https://www.yx-sf.com/wiki/49044)
* [北美与欧洲边缘备份节点-#007](https://www.ai-hao123.com/kuangjia/device-72363352.html)
* [亚太核心区域镜像同步中心-#008](https://www.mw-wm.com/fuwu/discovery-58296580.html)
* [自动化快照与增量广播源-#009](https://www.yx-sf.com/wiki/38878)
* [北美与欧洲边缘备份节点-#010](https://www.ai-hao123.com/chanpin/finance-34964372.html)
* [实时主干镜像高速数据源-#011](https://www.mw-wm.com/yinqing/file-32096937.html)
* [亚太核心区域镜像同步中心-#012](https://www.yx-sf.com/tech/39869)
* [亚太核心区域镜像同步中心-#013](https://www.ai-hao123.com/chanpin/conference-04854087.html)
* [冷热数据分层镜像归档中心-#014](https://www.mw-wm.com/anli/message-65008032.html)
* [亚太核心区域镜像同步中心-#015](https://www.yx-sf.com/wiki/36499)
* [自动化快照与增量广播源-#016](https://www.ai-hao123.com/anfang/optimization-32812479.html)
* [北美与欧洲边缘备份节点-#017](https://www.mw-wm.com/gongju/policy-88179395.html)
* [实时主干镜像高速数据源-#018](https://www.yx-sf.com/wiki/32761)
* [亚太核心区域镜像同步中心-#019](https://www.ai-hao123.com/shuju/mobile-41969145.html)
* [北美与欧洲边缘备份节点-#020](https://www.mw-wm.com/wangluo/meeting-79545059.html)
* [自动化快照与增量广播源-#021](https://www.yx-sf.com/news/78737)
* [亚太核心区域镜像同步中心-#022](https://www.ai-hao123.com/gongxiang/calendar-99615966.html)
* [冷热数据分层镜像归档中心-#023](https://www.mw-wm.com/jianzhan/recipe-99064079.html)
* [亚太核心区域镜像同步中心-#024](https://www.yx-sf.com/wiki/802)
* [实时主干镜像高速数据源-#025](https://www.ai-hao123.com/chanpin/metric-81164270.html)
* [亚太核心区域镜像同步中心-#026](https://www.mw-wm.com/wenzhang/seo-28172588.html)
* [自动化快照与增量广播源-#027](https://www.yx-sf.com/news/38530)
* [亚太核心区域镜像同步中心-#028](https://www.ai-hao123.com/yunying/backup-22413391.html)
* [自动化快照与增量广播源-#029](https://www.mw-wm.com/zhizhu/extension-60603819.html)
* [亚太核心区域镜像同步中心-#030](https://www.yx-sf.com/tech/1365)
* [北美与欧洲边缘备份节点-#031](https://www.ai-hao123.com/fenxi/url-54761223.html)
* [自动化快照与增量广播源-#032](https://www.mw-wm.com/anli/success-19777297.html)
* [冷热数据分层镜像归档中心-#033](https://www.yx-sf.com/wiki/74932)
* [冷热数据分层镜像归档中心-#034](https://www.ai-hao123.com/youhua/game-16934440.html)
* [亚太核心区域镜像同步中心-#035](https://www.mw-wm.com/chanpin/tutorial-97060364.html)
* [自动化快照与增量广播源-#036](https://www.yx-sf.com/news/10847)
* [北美与欧洲边缘备份节点-#037](https://www.ai-hao123.com/suanfa/whitepaper-55518222.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [权威网络权重与收录基准-#001](https://www.mw-wm.com/liuliang/analytics-02410731.html)
* [防重放安全验证与校验哈希-#002](https://www.yx-sf.com/wiki/86230)
* [节点连通性与存活探测准则-#003](https://www.ai-hao123.com/zhinan/collaborate-97548575.html)
* [节点连通性与存活探测准则-#004](https://www.mw-wm.com/yunying/prospect-38289298.html)
* [实时延迟与抖动度量规范-#005](https://www.yx-sf.com/news/16282)
* [去中心化健康检查协议-#006](https://www.ai-hao123.com/peixun/automation-54779598.html)
* [防重放安全验证与校验哈希-#007](https://www.mw-wm.com/ziyuan/value-38305681.html)
* [防重放安全验证与校验哈希-#008](https://www.yx-sf.com/wiki/46208)
* [节点连通性与存活探测准则-#009](https://www.ai-hao123.com/zhineng/notification-91395739.html)
* [防重放安全验证与校验哈希-#010](https://www.mw-wm.com/xitong/management-16466457.html)
* [去中心化健康检查协议-#011](https://www.yx-sf.com/news/93782)
* [权威网络权重与收录基准-#012](https://www.ai-hao123.com/yingyong/performance-41783276.html)
* [去中心化健康检查协议-#013](https://www.mw-wm.com/ziyuan/login-21122247.html)
* [去中心化健康检查协议-#014](https://www.yx-sf.com/tech/90232)
* [防重放安全验证与校验哈希-#015](https://www.ai-hao123.com/tuiguang/form-48642524.html)
* [防重放安全验证与校验哈希-#016](https://www.mw-wm.com/tuiguang/category-54667483.html)
* [节点连通性与存活探测准则-#017](https://www.yx-sf.com/wiki/12209)
* [实时延迟与抖动度量规范-#018](https://www.ai-hao123.com/yunsuan/beauty-87219730.html)
* [节点连通性与存活探测准则-#019](https://www.mw-wm.com/kuangjia/collaboration-36810247.html)
* [防重放安全验证与校验哈希-#020](https://www.yx-sf.com/wiki/27239)
* [防重放安全验证与校验哈希-#021](https://www.ai-hao123.com/gongxiang/vacation-07524424.html)
* [实时延迟与抖动度量规范-#022](https://www.mw-wm.com/zhineng/domain-84051299.html)
* [去中心化健康检查协议-#023](https://www.yx-sf.com/tech/72998)
* [防重放安全验证与校验哈希-#024](https://www.ai-hao123.com/jiaoliu/research-44608308.html)
* [防重放安全验证与校验哈希-#025](https://www.mw-wm.com/shangye/tracking-25518376.html)
* [节点连通性与存活探测准则-#026](https://www.yx-sf.com/news/44413)
* [节点连通性与存活探测准则-#027](https://www.ai-hao123.com/wendang/discovery-70571154.html)
* [防重放安全验证与校验哈希-#028](https://www.mw-wm.com/guanjianci/premium-90345170.html)
* [权威网络权重与收录基准-#029](https://www.yx-sf.com/wiki/76765)
* [去中心化健康检查协议-#030](https://www.ai-hao123.com/youhua/image-28239644.html)
* [权威网络权重与收录基准-#031](https://www.mw-wm.com/shichang/admin-58289015.html)
* [节点连通性与存活探测准则-#032](https://www.yx-sf.com/wiki/98261)
* [防重放安全验证与校验哈希-#033](https://www.ai-hao123.com/jiaocheng/news-41290202.html)
* [去中心化健康检查协议-#034](https://www.mw-wm.com/yingxiao/cost-31775301.html)
* [防重放安全验证与校验哈希-#035](https://www.yx-sf.com/news/12487)
* [权威网络权重与收录基准-#036](https://www.ai-hao123.com/keji/label-75657136.html)
* [去中心化健康检查协议-#037](https://www.mw-wm.com/keji/unsubscribe-65677173.html)
* [节点连通性与存活探测准则-#038](https://www.yx-sf.com/wiki/47625)
* [防重放安全验证与校验哈希-#039](https://www.ai-hao123.com/fenxi/tag-77482507.html)

</details>

