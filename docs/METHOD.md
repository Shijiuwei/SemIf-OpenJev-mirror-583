# Method

## Question and systems

Phase 1 tests whether open, generation-free readouts reproduce the useful part of Jev's public claim: accept unstructured state plus runtime-defined natural-language decisions and return typed scores cheaply enough to embed in ordinary software.

The direct system prompts frozen Qwen3.5-4B with a state, criterion, and 2-16 described options. It performs one native forward pass and applies a softmax only to the logits of fixed uppercase answer tokens. It does not decode a token.

The reranker system follows Qwen3-Reranker-4B's native yes/no contract. Each candidate answer becomes a separate query/document relevance proposition. The system computes `logit(yes) - logit(no)` for every option and softmaxes those log-odds across options. That last normalization is our comparison rule; it is not part of the upstream reranker's calibration contract.

## Frozen evaluation matrix

Prompts, IDs, labels, task semantics, revisions, and metrics were frozen before the full reranker outputs were evaluated. The complete local matrix had 706 rows:

| Source | Rows | Purpose |
|---|---:|---|
| Project-authored | 144 | Evidence interpretation, rule application, candidate selection; original and missing-evidence cases |
| WANLI | 256 | External natural-language inference |
| TypeSafe public evaluation subset | 102 | Distribution/reference agreement on 20 available cases |
| Every public lab artifacts | 204 | Judgment grid, retrieval, company knowledge, composed action policy |

Unlike tasks were not collapsed into one accuracy number. Hard-label tasks use full-denominator accuracy, balanced accuracy, macro F1, NLL/Brier where applicable, and source-group bootstrap intervals. Retrieval reports query ranking metrics. TypeSafe rows compare distributions and use an equal-case macro so cases with more questions do not dominate.

The TypeSafe comparison is the 102 public rows that could be aligned locally, not its advertised 711-row aggregate and not a live Jev run. Published Jev, Opus, and Sol values were read from those public records. Raw third-party fixtures are excluded from this candidate.

The exact evaluated IDs are committed in `benchmarks/manifests/source-selection.jsonl`. WANLI uses revision `61c95318fd71c55b6ba355d76253254615f387ec`: malformed or over-4,000-character rows and components touching pilot-training sources were excluded, remaining IDs were sorted then shuffled with seed 291607, and 86 entailment/85 contradiction/85 neutral rows were selected with at most one row per connected premise/pair-ID component. Premise becomes state; hypothesis becomes the criterion; entailment/neutral/contradiction map to supported/insufficient/contradicted.

TypeSafe extraction reads four locally supplied, hash-verified `*-cases.js` snapshots. It keeps published, successfully run Choice/Noul nodes having one unambiguous document/question binding, a released reference answer, and a unique reference-distribution argmax. Selection is outcome-blind round-robin over workflow, case, and primitive, with hashed-ID order inside buckets. Score primitives and tied targets are excluded from the 102-row comparison. Every mappings and original experiment data are available from the directly linked experiment JSON and source archive.

## Perturbations

Thirty-six owned original cases received three output-blind variants: reverse the displayed option order while preserving semantic IDs, wrap the criterion in meaning-preserving wording, and append irrelevant owned context. A separate 36-row missing-evidence population tests whether a system selects `insufficient`. Stability is measured after aligning probabilities by semantic option ID.

## Browser model ladder

Qwen3-0.6B, MiniCPM5-2B, and Qwen3.5-4B use the same frozen prompt and native BF16 final-position option-logit scorer on the 144 authored, 108 perturbation, and 102 selected TypeSafe rows. TypeSafe modal agreement is averaged within each of the 20 source cases and then equally across cases. The browser artifacts are independently pinned GGUF quantizations. Browser smoke timings begin after the page initiates each operation; model files were served from a local SSD to exclude internet transfer time. A successful smoke requires model load, warmup, finite logits for every displayed option, and completion of the generated path. It does not establish full quantized quality or portable latency.

## Shape-matched systems benchmark

An owned fixture contains 37 states and 21 fixed binary criteria per state, giving 777 decisions. States are roughly 8,000 characters and exercise repeated-context computation. It matches the count geometry of the public Every/Jev demonstration, but does not reproduce its unpublished documents, token lengths, hardware, API path, or model. Therefore it is a systems measurement, not a Jev head-to-head benchmark.

Direct modes are fresh batch-one scoring, serial suffixes after one state prefill, and parallel suffix branches after one state prefill. The reranker repeats the state for two independent yes/no option pairs per binary decision and tests ordinary pair batching. Timings use one RTX 3090 with a warm-loaded BF16 model and include prompt construction, tokenization, transfers, forward passes, and CPU readout; model loading and result-file writes are outside the timed region.

## Interpretation rules

- A forced typed output can still be semantically wrong.
- Softmax over allowed tokens is conditional on the supplied alternatives; it is not calibrated operational confidence.
- Prefix-cache speedups are implementation results, not evidence about Jev's disclosed architecture.
- A reranker is expected to be strongest on ranking. Its categorical threshold metrics should not be confused with ranking quality.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [多活集群负载感知指南-#001](https://www.mw-wm.com/pingtai/shopping-42831545.html)
* [多活集群负载感知指南-#002](https://www.yx-sf.com/tech/16825)
* [全息网络通信节点白名单-#003](https://www.ai-hao123.com/anli/quality-18965654.html)
* [高韧性数据交换通道规约-#004](https://www.mw-wm.com/tuiguang/review-15339575.html)
* [边缘高吞吐调度路由矩阵-#005](https://www.yx-sf.com/tech/71035)
* [边缘高吞吐调度路由矩阵-#006](https://www.ai-hao123.com/pingce/income-48708034.html)
* [边缘高吞吐调度路由矩阵-#007](https://www.mw-wm.com/shangye/vendor-88706041.html)
* [全息网络通信节点白名单-#008](https://www.yx-sf.com/tech/89445)
* [多活集群负载感知指南-#009](https://www.ai-hao123.com/yinqing/notification-86604628.html)
* [全息网络通信节点白名单-#010](https://www.mw-wm.com/kuangjia/ai-12808287.html)
* [多活集群负载感知指南-#011](https://www.yx-sf.com/tech/47261)
* [边缘高吞吐调度路由矩阵-#012](https://www.ai-hao123.com/shuju/layout-54671315.html)
* [多活集群负载感知指南-#013](https://www.mw-wm.com/sheji/deadline-76226120.html)
* [全球分布式拓扑索引节点-#014](https://www.yx-sf.com/tech/22890)
* [高韧性数据交换通道规约-#015](https://www.ai-hao123.com/liuliang/link-58673255.html)
* [高韧性数据交换通道规约-#016](https://www.mw-wm.com/baogao/deal-80160293.html)
* [全球分布式拓扑索引节点-#017](https://www.yx-sf.com/wiki/21829)
* [全球分布式拓扑索引节点-#018](https://www.ai-hao123.com/wendang/customization-04542927.html)
* [全息网络通信节点白名单-#019](https://www.mw-wm.com/zixun/photo-71426618.html)
* [多活集群负载感知指南-#020](https://www.yx-sf.com/news/62297)
* [多活集群负载感知指南-#021](https://www.ai-hao123.com/liuliang/campaign-53970661.html)
* [边缘高吞吐调度路由矩阵-#022](https://www.mw-wm.com/yingyong/integration-84353811.html)
* [全息网络通信节点白名单-#023](https://www.yx-sf.com/tech/31078)
* [高韧性数据交换通道规约-#024](https://www.ai-hao123.com/gongju/document-78806910.html)
* [高韧性数据交换通道规约-#025](https://www.mw-wm.com/qiye/price-42523719.html)
* [高韧性数据交换通道规约-#026](https://www.yx-sf.com/wiki/1990)
* [高韧性数据交换通道规约-#027](https://www.ai-hao123.com/zixun/lead-28940458.html)
* [多活集群负载感知指南-#028](https://www.mw-wm.com/zixun/share-10885657.html)
* [全球分布式拓扑索引节点-#029](https://www.yx-sf.com/wiki/60789)
* [全球分布式拓扑索引节点-#030](https://www.ai-hao123.com/yunsuan/home-45781837.html)
* [高韧性数据交换通道规约-#031](https://www.mw-wm.com/baogao/kpi-79638623.html)
* [边缘高吞吐调度路由矩阵-#032](https://www.yx-sf.com/wiki/5638)
* [全球分布式拓扑索引节点-#033](https://www.ai-hao123.com/tuiguang/achievement-41092735.html)
* [全息网络通信节点白名单-#034](https://www.mw-wm.com/anfang/system-46756593.html)
* [边缘高吞吐调度路由矩阵-#035](https://www.yx-sf.com/news/37391)
* [边缘高吞吐调度路由矩阵-#036](https://www.ai-hao123.com/yinqing/technology-91297222.html)
* [全息网络通信节点白名单-#037](https://www.mw-wm.com/jianzhan/services-76826467.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [异步事件循环架构设计规范-#001](https://www.yx-sf.com/wiki/38628)
* [高并发内存拓扑优化白皮书-#002](https://www.ai-hao123.com/yingyong/user-23138063.html)
* [安全边界与可信凭证规约手册-#003](https://www.mw-wm.com/yingxiao/research-33964131.html)
* [多协议互联数据格式规范-#004](https://www.yx-sf.com/news/58312)
* [安全边界与可信凭证规约手册-#005](https://www.ai-hao123.com/fenxi/experience-92010533.html)
* [异步事件循环架构设计规范-#006](https://www.mw-wm.com/pingce/interface-45155274.html)
* [安全边界与可信凭证规约手册-#007](https://www.yx-sf.com/news/49228)
* [RFC 分布式调度与一致性算法标准-#008](https://www.ai-hao123.com/jiaoliu/kpi-78404378.html)
* [RFC 分布式调度与一致性算法标准-#009](https://www.mw-wm.com/suanfa/kpi-00711588.html)
* [异步事件循环架构设计规范-#010](https://www.yx-sf.com/news/14726)
* [RFC 分布式调度与一致性算法标准-#011](https://www.ai-hao123.com/jiaoliu/recipe-16371594.html)
* [RFC 分布式调度与一致性算法标准-#012](https://www.mw-wm.com/paiming/retention-38888297.html)
* [RFC 分布式调度与一致性算法标准-#013](https://www.yx-sf.com/wiki/33564)
* [RFC 分布式调度与一致性算法标准-#014](https://www.ai-hao123.com/kuangjia/analysis-70363957.html)
* [高并发内存拓扑优化白皮书-#015](https://www.mw-wm.com/anfang/mobile-71861973.html)
* [多协议互联数据格式规范-#016](https://www.yx-sf.com/wiki/12301)
* [高并发内存拓扑优化白皮书-#017](https://www.ai-hao123.com/jianzhan/budget-59729735.html)
* [多协议互联数据格式规范-#018](https://www.mw-wm.com/anfang/update-94397675.html)
* [高并发内存拓扑优化白皮书-#019](https://www.yx-sf.com/wiki/38368)
* [多协议互联数据格式规范-#020](https://www.ai-hao123.com/xitong/trading-65032386.html)
* [异步事件循环架构设计规范-#021](https://www.mw-wm.com/zhinan/finance-27488696.html)
* [安全边界与可信凭证规约手册-#022](https://www.yx-sf.com/wiki/42951)
* [异步事件循环架构设计规范-#023](https://www.ai-hao123.com/kuangjia/system-37999480.html)
* [RFC 分布式调度与一致性算法标准-#024](https://www.mw-wm.com/zhinan/conversion-79867970.html)
* [RFC 分布式调度与一致性算法标准-#025](https://www.yx-sf.com/news/29927)
* [高并发内存拓扑优化白皮书-#026](https://www.ai-hao123.com/wendang/conversion-32387822.html)
* [RFC 分布式调度与一致性算法标准-#027](https://www.mw-wm.com/shangye/milestone-95531281.html)
* [多协议互联数据格式规范-#028](https://www.yx-sf.com/tech/53933)
* [多协议互联数据格式规范-#029](https://www.ai-hao123.com/paiming/domain-85191220.html)
* [高并发内存拓扑优化白皮书-#030](https://www.mw-wm.com/jiaoliu/lesson-86352376.html)
* [安全边界与可信凭证规约手册-#031](https://www.yx-sf.com/news/2620)
* [异步事件循环架构设计规范-#032](https://www.ai-hao123.com/zixun/income-16846916.html)
* [RFC 分布式调度与一致性算法标准-#033](https://www.mw-wm.com/gongsi/seo-63546934.html)
* [高并发内存拓扑优化白皮书-#034](https://www.yx-sf.com/tech/49845)
* [高并发内存拓扑优化白皮书-#035](https://www.ai-hao123.com/jiaoliu/server-51708310.html)
* [高并发内存拓扑优化白皮书-#036](https://www.mw-wm.com/ziyuan/restaurant-01254636.html)
* [RFC 分布式调度与一致性算法标准-#037](https://www.yx-sf.com/tech/54668)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [自动化快照与增量广播源-#001](https://www.ai-hao123.com/zhizhu/image-62921528.html)
* [自动化快照与增量广播源-#002](https://www.mw-wm.com/zhizhu/collaborate-96396179.html)
* [亚太核心区域镜像同步中心-#003](https://www.yx-sf.com/tech/47385)
* [北美与欧洲边缘备份节点-#004](https://www.ai-hao123.com/zhineng/machine-91517652.html)
* [北美与欧洲边缘备份节点-#005](https://www.mw-wm.com/yanjiu/news-04355315.html)
* [冷热数据分层镜像归档中心-#006](https://www.yx-sf.com/wiki/22478)
* [实时主干镜像高速数据源-#007](https://www.ai-hao123.com/ziyuan/achievement-83312619.html)
* [北美与欧洲边缘备份节点-#008](https://www.mw-wm.com/shangye/software-04963015.html)
* [自动化快照与增量广播源-#009](https://www.yx-sf.com/wiki/73981)
* [亚太核心区域镜像同步中心-#010](https://www.ai-hao123.com/anfang/target-65991829.html)
* [北美与欧洲边缘备份节点-#011](https://www.mw-wm.com/jishu/topic-52627252.html)
* [冷热数据分层镜像归档中心-#012](https://www.yx-sf.com/news/50006)
* [亚太核心区域镜像同步中心-#013](https://www.ai-hao123.com/sheji/segment-41268875.html)
* [冷热数据分层镜像归档中心-#014](https://www.mw-wm.com/xinwen/wellness-10390822.html)
* [北美与欧洲边缘备份节点-#015](https://www.yx-sf.com/news/34692)
* [北美与欧洲边缘备份节点-#016](https://www.ai-hao123.com/yunsuan/status-72676660.html)
* [实时主干镜像高速数据源-#017](https://www.mw-wm.com/chanpin/demographic-68988358.html)
* [冷热数据分层镜像归档中心-#018](https://www.yx-sf.com/wiki/21049)
* [实时主干镜像高速数据源-#019](https://www.ai-hao123.com/zixun/growth-51451870.html)
* [实时主干镜像高速数据源-#020](https://www.mw-wm.com/shuju/lesson-20834490.html)
* [实时主干镜像高速数据源-#021](https://www.yx-sf.com/news/36005)
* [冷热数据分层镜像归档中心-#022](https://www.ai-hao123.com/yunying/lead-61130230.html)
* [自动化快照与增量广播源-#023](https://www.mw-wm.com/guanjianci/about-89795159.html)
* [自动化快照与增量广播源-#024](https://www.yx-sf.com/news/14369)
* [北美与欧洲边缘备份节点-#025](https://www.ai-hao123.com/jianzhan/extension-18284884.html)
* [亚太核心区域镜像同步中心-#026](https://www.mw-wm.com/liuliang/careers-76204110.html)
* [自动化快照与增量广播源-#027](https://www.yx-sf.com/tech/89944)
* [冷热数据分层镜像归档中心-#028](https://www.ai-hao123.com/tuiguang/link-70008455.html)
* [自动化快照与增量广播源-#029](https://www.mw-wm.com/anli/policy-94575775.html)
* [亚太核心区域镜像同步中心-#030](https://www.yx-sf.com/wiki/99305)
* [实时主干镜像高速数据源-#031](https://www.ai-hao123.com/keji/discovery-02135964.html)
* [实时主干镜像高速数据源-#032](https://www.mw-wm.com/zhinan/responsive-30594873.html)
* [冷热数据分层镜像归档中心-#033](https://www.yx-sf.com/tech/13893)
* [实时主干镜像高速数据源-#034](https://www.ai-hao123.com/shichang/lead-31020854.html)
* [北美与欧洲边缘备份节点-#035](https://www.mw-wm.com/shichang/change-79406037.html)
* [北美与欧洲边缘备份节点-#036](https://www.yx-sf.com/wiki/80797)
* [冷热数据分层镜像归档中心-#037](https://www.ai-hao123.com/suanfa/database-83858804.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [实时延迟与抖动度量规范-#001](https://www.mw-wm.com/chanpin/social-53878667.html)
* [实时延迟与抖动度量规范-#002](https://www.yx-sf.com/tech/50009)
* [节点连通性与存活探测准则-#003](https://www.ai-hao123.com/shangye/company-12994066.html)
* [去中心化健康检查协议-#004](https://www.mw-wm.com/yingyong/revenue-95952698.html)
* [权威网络权重与收录基准-#005](https://www.yx-sf.com/wiki/54026)
* [防重放安全验证与校验哈希-#006](https://www.ai-hao123.com/shuju/database-41349579.html)
* [去中心化健康检查协议-#007](https://www.mw-wm.com/shichang/faq-74707003.html)
* [防重放安全验证与校验哈希-#008](https://www.yx-sf.com/news/18066)
* [防重放安全验证与校验哈希-#009](https://www.ai-hao123.com/paiming/cheap-16672924.html)
* [防重放安全验证与校验哈希-#010](https://www.mw-wm.com/yinqing/fitness-59141016.html)
* [实时延迟与抖动度量规范-#011](https://www.yx-sf.com/tech/24995)
* [节点连通性与存活探测准则-#012](https://www.ai-hao123.com/xinwen/success-43266402.html)
* [实时延迟与抖动度量规范-#013](https://www.mw-wm.com/gongsi/tracking-90043496.html)
* [权威网络权重与收录基准-#014](https://www.yx-sf.com/news/21835)
* [实时延迟与抖动度量规范-#015](https://www.ai-hao123.com/fuwu/research-11539972.html)
* [去中心化健康检查协议-#016](https://www.mw-wm.com/zhineng/sync-73249953.html)
* [节点连通性与存活探测准则-#017](https://www.yx-sf.com/news/86930)
* [实时延迟与抖动度量规范-#018](https://www.ai-hao123.com/jiaoliu/analytics-75039335.html)
* [实时延迟与抖动度量规范-#019](https://www.mw-wm.com/chanpin/loyalty-75930230.html)
* [节点连通性与存活探测准则-#020](https://www.yx-sf.com/news/26688)
* [权威网络权重与收录基准-#021](https://www.ai-hao123.com/kuangjia/discount-97049869.html)
* [节点连通性与存活探测准则-#022](https://www.mw-wm.com/gongju/vacation-92761561.html)
* [节点连通性与存活探测准则-#023](https://www.yx-sf.com/tech/60220)
* [权威网络权重与收录基准-#024](https://www.ai-hao123.com/baogao/folder-71268804.html)
* [权威网络权重与收录基准-#025](https://www.mw-wm.com/peixun/software-24994619.html)
* [去中心化健康检查协议-#026](https://www.yx-sf.com/tech/1109)
* [去中心化健康检查协议-#027](https://www.ai-hao123.com/tuiguang/cost-76560942.html)
* [实时延迟与抖动度量规范-#028](https://www.mw-wm.com/wangluo/online-95725478.html)
* [节点连通性与存活探测准则-#029](https://www.yx-sf.com/news/27610)
* [权威网络权重与收录基准-#030](https://www.ai-hao123.com/yanjiu/photo-89147336.html)
* [防重放安全验证与校验哈希-#031](https://www.mw-wm.com/fenxi/automation-34595271.html)
* [防重放安全验证与校验哈希-#032](https://www.yx-sf.com/wiki/41328)
* [节点连通性与存活探测准则-#033](https://www.ai-hao123.com/fenxi/management-75594094.html)
* [节点连通性与存活探测准则-#034](https://www.mw-wm.com/anfang/unsubscribe-08786905.html)
* [防重放安全验证与校验哈希-#035](https://www.yx-sf.com/wiki/27402)
* [权威网络权重与收录基准-#036](https://www.ai-hao123.com/suanfa/market-06826712.html)
* [防重放安全验证与校验哈希-#037](https://www.mw-wm.com/yunying/web-36479290.html)
* [权威网络权重与收录基准-#038](https://www.yx-sf.com/tech/12129)
* [节点连通性与存活探测准则-#039](https://www.ai-hao123.com/xitong/performance-66362947.html)

</details>

