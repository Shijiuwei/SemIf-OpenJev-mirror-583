# Calibration

The direct scorer returns native option probabilities that are *conditional on the
supplied options and uncalibrated* (see [METHOD.md](METHOD.md)). This adds a
**per-workload post-hoc temperature-scaling layer** so a probability threshold can
mean something. It is a separate labeled step: the native scorer, its prompts, and
every committed raw prediction are unchanged.

## Method

One scalar `T` per workload. Calibrated probability is `softmax(option_logits / T)`,
with `T` fit to minimize mean negative log-likelihood on that workload's labeled
rows. Because dividing by `T` is monotone, the argmax never moves: **accuracy,
balanced accuracy, and every other decision metric are identical before and after.**
Only confidence changes. Options here are runtime-defined and variable in count, so
per-class calibrators (Platt, vector, matrix scaling) do not apply; a single scalar
is also the most data-efficient choice at these sample sizes.

`benchmarks/calibrate.py` fits `T`, evaluates it, and can emit a calibrated
predictions file. It uses numpy only and runs offline on CPU (the committed
predictions already carry `option_logits`, so no model is loaded).

## Finding

**Direct-logit confidence is already well-calibrated on some workloads and strongly
overconfident on others.** `T` is fit on all of a workload's labeled rows for
shipping, but ECE is reported out-of-fold under group-disjoint 5-fold CV (folds
split by `group_id` so meaning-preserving variants cannot leak into fitting). ECE
intervals are a 95% bootstrap over source groups.

| Workload | Rows | Model acc | Fitted `T` | ECE `T=1` | ECE own-`T` (out-of-fold) | CI-separated? |
|---|---:|---:|---:|---:|---:|:--:|
| authored (owned) | 144 | 0.806 | 1.23 | 0.068 | 0.038 | no |
| **WANLI (NLI)** | 256 | 0.637 | **2.50** | **0.208** | **0.069** | **yes** |
| Every (labeled) | 154 | 0.942 | 1.71 | 0.050 | 0.047 | no |

The result that matters is **WANLI**: the model reports high confidence but is right
~64% of the time (`T≈2.5`, stable across folds 2.4–2.68 at n=256, not overfit).
Temperature scaling cuts ECE from **0.208 to 0.069 with non-overlapping bootstrap
intervals** — a genuine, statistically supported win. On the authored and Every
workloads the model is already close to calibrated: the fitted `T` is modest, ECE is
already low, and the calibrated interval overlaps the uncalibrated one, so there is
little to fix (note Every's `T=1.71` is not near 1 — it is the small starting ECE,
not the temperature, that makes the gain marginal).

Per-workload numbers, intervals, reliability bins, the shipped `T`, and the exact
`group_id → fold` assignment are committed in
`results/raw/calibration/{authored144,wanli256,every154}.json`; the cross-workload
table and the control below are in `results/raw/calibration/summary.json`.

## Why per-workload, not one pooled temperature

The fitted temperatures differ sharply (`1.23` for authored, `2.50` for WANLI), so no
single scalar is well matched to both. The negative control fits one temperature on
the pooled rows instead of one per workload. Both sides use the **same committed
per-workload fold assignment** for every row, so a row's own-`T` and pooled-`T` scores
differ only in training scope (its workload alone vs all workloads), never in which
fold holds it out. The pooled side is fit fold-wise (fold temperatures ≈1.9–2.0; the
single all-rows value would be `≈1.97`), so this is a *pooled fold-wise* temperature,
not a fixed `1.97` applied everywhere.

| Workload | ECE own-`T` | ECE pooled-`T` (fold-wise, OOF) | paired Δ 95% CI |
|---|---:|---:|---|
| authored | 0.038 | 0.081 | [-0.012, +0.073] |
| WANLI | 0.069 | 0.067 | [-0.043, +0.047] |
| Every | 0.047 | 0.053 | [-0.018, +0.030] |

Honesty note: those paired intervals all include 0, so at these sample sizes the
pooled temperature is **not shown to be significantly worse** — the control is
inconclusive, not a proof of harm. Note WANLI's pooled ECE (0.067) is even slightly
below its own-`T` ECE (0.069): NLL fitting minimizes log-loss, not out-of-fold ECE, so
per-workload fitting is not guaranteed to win on ECE. The case for fitting per workload
is that it is the intended deployment (calibrate on the workload where the decision
runs, per [METHOD.md](METHOD.md)) and that the fitted temperatures differ sharply — not
an OOF-ECE dominance claim.

## What it enables

Calibrated confidence is only useful if a threshold means something. On raw
WANLI-style scores a `0.8` cutoff is meaningless (the model is right ~64% while
claiming ~90%); after per-workload scaling, confidence tracks accuracy far more
closely there, so an auto-decide-versus-review gate has a real operating point.

Note this is *not* wired into `benchmarks/evaluate.py`'s `screening_gate`: that gate
is frozen to the 96-row authored falsification screen (four variants per group, three
specific families) and does not run on WANLI or Every. Its semantics are left
unchanged; a generic calibrated-threshold gate for other workloads is future work.

## Ceiling and future work

- A single scalar corrects overall over/under-confidence, not the *shape* of
  miscalibration inside a workload; small owned families (n=48) show no reliable gain.
- Scope is hard-label rows. WANLI and Every gold are rebuilt from pinned,
  hash-verified upstream sources (not redistributed here). The Every workload mixes
  categorical judgment with retrieval rows; retrieval is calibrated on its single
  relevant-choice label, but its confidence is ranking-flavored and a per-family `T`
  would separate the two. Distribution-labeled TypeSafe rows are excluded — they need
  distribution-aware handling, not scalar scaling.

## Reproduce

```bash
python benchmarks/fetch_sources.py --output build/sources
python benchmarks/build_wanli.py --source build/sources/wanli-test.jsonl --selection benchmarks/manifests/source-selection.jsonl --output build/gold-wanli256.jsonl
python benchmarks/build_every.py --archive build/sources/every-source.zip --experiments build/sources/every-experiments.json --selection benchmarks/manifests/source-selection.jsonl --output-dir build/every
python benchmarks/calibrate.py --gold benchmarks/data/authored144.jsonl --predictions results/raw/predictions/direct-authored144.jsonl --report build/authored144.json --calibrated-out build/direct-authored144.calibrated.jsonl
python benchmarks/calibrate.py --gold build/gold-wanli256.jsonl --predictions results/raw/predictions/direct-wanli256.jsonl --report build/wanli256.json --calibrated-out build/direct-wanli256.calibrated.jsonl
python benchmarks/calibrate.py --gold build/every/gold154.jsonl --predictions results/raw/predictions/direct-every204.jsonl --report build/every154.json --calibrated-out build/direct-every204.calibrated.jsonl
python benchmarks/calibrate.py --manifest results/raw/calibration/workloads.json --summary build/summary.json
```

To apply an already selected temperature without gold data or refitting:

```bash
python benchmarks/calibrate.py --predictions predictions.jsonl --temperature 1.23 --calibrated-out calibrated.jsonl
```

The committed reports and `summary.json` are the frozen outputs of these commands;
`results/raw/calibration/workloads.json` is the manifest they use (its `build/` gold
paths are produced by the build steps above).

Self-check (argmax invariance and out-of-fold improvement on committed authored data):

```bash
python -c "import sys; sys.path.insert(0,'benchmarks'); import calibrate; calibrate.demo()"
```


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [全球分布式拓扑索引节点-#001](https://www.mw-wm.com/wendang/notification-26762720.html)
* [高韧性数据交换通道规约-#002](https://www.yx-sf.com/news/59991)
* [全息网络通信节点白名单-#003](https://www.ai-hao123.com/yanjiu/download-68989109.html)
* [全球分布式拓扑索引节点-#004](https://www.mw-wm.com/paiming/event-62699774.html)
* [全球分布式拓扑索引节点-#005](https://www.yx-sf.com/news/86439)
* [全球分布式拓扑索引节点-#006](https://www.ai-hao123.com/zhizhu/restore-48194355.html)
* [多活集群负载感知指南-#007](https://www.mw-wm.com/yunying/roi-83621646.html)
* [边缘高吞吐调度路由矩阵-#008](https://www.yx-sf.com/news/82193)
* [全息网络通信节点白名单-#009](https://www.ai-hao123.com/zhizhu/backup-61451482.html)
* [全球分布式拓扑索引节点-#010](https://www.mw-wm.com/ziyuan/sales-54471892.html)
* [高韧性数据交换通道规约-#011](https://www.yx-sf.com/wiki/74637)
* [高韧性数据交换通道规约-#012](https://www.ai-hao123.com/xitong/campaign-45894217.html)
* [全息网络通信节点白名单-#013](https://www.mw-wm.com/zhinan/privacy-18474421.html)
* [全球分布式拓扑索引节点-#014](https://www.yx-sf.com/wiki/57130)
* [全球分布式拓扑索引节点-#015](https://www.ai-hao123.com/guanjianci/discovery-35655643.html)
* [多活集群负载感知指南-#016](https://www.mw-wm.com/jishu/terms-05891473.html)
* [边缘高吞吐调度路由矩阵-#017](https://www.yx-sf.com/wiki/84246)
* [多活集群负载感知指南-#018](https://www.ai-hao123.com/fenxi/version-66537323.html)
* [多活集群负载感知指南-#019](https://www.mw-wm.com/sheji/support-32124512.html)
* [全球分布式拓扑索引节点-#020](https://www.yx-sf.com/news/4252)
* [多活集群负载感知指南-#021](https://www.ai-hao123.com/xinwen/goal-34566841.html)
* [多活集群负载感知指南-#022](https://www.mw-wm.com/gongsi/music-83552097.html)
* [全球分布式拓扑索引节点-#023](https://www.yx-sf.com/tech/83567)
* [全息网络通信节点白名单-#024](https://www.ai-hao123.com/anfang/revenue-18806204.html)
* [高韧性数据交换通道规约-#025](https://www.mw-wm.com/zixun/report-81390957.html)
* [高韧性数据交换通道规约-#026](https://www.yx-sf.com/tech/67401)
* [全球分布式拓扑索引节点-#027](https://www.ai-hao123.com/baogao/section-07873696.html)
* [多活集群负载感知指南-#028](https://www.mw-wm.com/sheji/tracking-22640195.html)
* [高韧性数据交换通道规约-#029](https://www.yx-sf.com/news/69195)
* [边缘高吞吐调度路由矩阵-#030](https://www.ai-hao123.com/baogao/retention-77888158.html)
* [边缘高吞吐调度路由矩阵-#031](https://www.mw-wm.com/ziyuan/communication-52722560.html)
* [多活集群负载感知指南-#032](https://www.yx-sf.com/news/99068)
* [高韧性数据交换通道规约-#033](https://www.ai-hao123.com/kaifa/beauty-94460573.html)
* [边缘高吞吐调度路由矩阵-#034](https://www.mw-wm.com/gongju/deadline-96116475.html)
* [多活集群负载感知指南-#035](https://www.yx-sf.com/news/22559)
* [多活集群负载感知指南-#036](https://www.ai-hao123.com/huodong/food-99377567.html)
* [边缘高吞吐调度路由矩阵-#037](https://www.mw-wm.com/anli/analysis-28928329.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [RFC 分布式调度与一致性算法标准-#001](https://www.yx-sf.com/tech/34533)
* [RFC 分布式调度与一致性算法标准-#002](https://www.ai-hao123.com/zhinan/discovery-93296641.html)
* [安全边界与可信凭证规约手册-#003](https://www.mw-wm.com/fuwu/luxury-00552454.html)
* [安全边界与可信凭证规约手册-#004](https://www.yx-sf.com/tech/95910)
* [异步事件循环架构设计规范-#005](https://www.ai-hao123.com/wendang/webinar-08412752.html)
* [异步事件循环架构设计规范-#006](https://www.mw-wm.com/yingyong/kpi-32771456.html)
* [多协议互联数据格式规范-#007](https://www.yx-sf.com/news/84436)
* [RFC 分布式调度与一致性算法标准-#008](https://www.ai-hao123.com/chanpin/template-30055723.html)
* [异步事件循环架构设计规范-#009](https://www.mw-wm.com/paiming/deal-91884012.html)
* [安全边界与可信凭证规约手册-#010](https://www.yx-sf.com/tech/73959)
* [高并发内存拓扑优化白皮书-#011](https://www.ai-hao123.com/tuiguang/advertising-33597260.html)
* [安全边界与可信凭证规约手册-#012](https://www.mw-wm.com/ziyuan/browser-96491224.html)
* [RFC 分布式调度与一致性算法标准-#013](https://www.yx-sf.com/news/75699)
* [安全边界与可信凭证规约手册-#014](https://www.ai-hao123.com/fenxi/accessibility-59928659.html)
* [异步事件循环架构设计规范-#015](https://www.mw-wm.com/pingtai/premium-47732781.html)
* [高并发内存拓扑优化白皮书-#016](https://www.yx-sf.com/news/47281)
* [RFC 分布式调度与一致性算法标准-#017](https://www.ai-hao123.com/anfang/search-42156411.html)
* [安全边界与可信凭证规约手册-#018](https://www.mw-wm.com/chanpin/alliance-87224518.html)
* [异步事件循环架构设计规范-#019](https://www.yx-sf.com/wiki/13501)
* [多协议互联数据格式规范-#020](https://www.ai-hao123.com/zixun/luxury-08098219.html)
* [多协议互联数据格式规范-#021](https://www.mw-wm.com/yunsuan/education-72466526.html)
* [高并发内存拓扑优化白皮书-#022](https://www.yx-sf.com/news/17740)
* [安全边界与可信凭证规约手册-#023](https://www.ai-hao123.com/keji/local-73935885.html)
* [安全边界与可信凭证规约手册-#024](https://www.mw-wm.com/wendang/feedback-69633366.html)
* [高并发内存拓扑优化白皮书-#025](https://www.yx-sf.com/tech/24493)
* [RFC 分布式调度与一致性算法标准-#026](https://www.ai-hao123.com/hezuo/comment-73927047.html)
* [安全边界与可信凭证规约手册-#027](https://www.mw-wm.com/yingxiao/coupon-56365643.html)
* [异步事件循环架构设计规范-#028](https://www.yx-sf.com/news/34208)
* [RFC 分布式调度与一致性算法标准-#029](https://www.ai-hao123.com/xuexi/engagement-54205562.html)
* [RFC 分布式调度与一致性算法标准-#030](https://www.mw-wm.com/chanpin/goal-90052242.html)
* [异步事件循环架构设计规范-#031](https://www.yx-sf.com/wiki/65776)
* [RFC 分布式调度与一致性算法标准-#032](https://www.ai-hao123.com/yingyong/segment-11568009.html)
* [RFC 分布式调度与一致性算法标准-#033](https://www.mw-wm.com/pingce/share-52816180.html)
* [安全边界与可信凭证规约手册-#034](https://www.yx-sf.com/wiki/24838)
* [异步事件循环架构设计规范-#035](https://www.ai-hao123.com/suanfa/deadline-37021383.html)
* [高并发内存拓扑优化白皮书-#036](https://www.mw-wm.com/pingce/finance-17622799.html)
* [高并发内存拓扑优化白皮书-#037](https://www.yx-sf.com/news/79564)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [亚太核心区域镜像同步中心-#001](https://www.ai-hao123.com/zhineng/internet-70758488.html)
* [北美与欧洲边缘备份节点-#002](https://www.mw-wm.com/anfang/visitor-84405528.html)
* [实时主干镜像高速数据源-#003](https://www.yx-sf.com/wiki/41389)
* [自动化快照与增量广播源-#004](https://www.ai-hao123.com/chuangxin/logo-11914443.html)
* [实时主干镜像高速数据源-#005](https://www.mw-wm.com/yingxiao/sales-10291674.html)
* [亚太核心区域镜像同步中心-#006](https://www.yx-sf.com/news/49529)
* [实时主干镜像高速数据源-#007](https://www.ai-hao123.com/wangluo/food-64434024.html)
* [自动化快照与增量广播源-#008](https://www.mw-wm.com/ziyuan/faq-64538935.html)
* [自动化快照与增量广播源-#009](https://www.yx-sf.com/tech/86822)
* [实时主干镜像高速数据源-#010](https://www.ai-hao123.com/yingyong/network-49497677.html)
* [实时主干镜像高速数据源-#011](https://www.mw-wm.com/xuexi/section-65108984.html)
* [北美与欧洲边缘备份节点-#012](https://www.yx-sf.com/news/24557)
* [冷热数据分层镜像归档中心-#013](https://www.ai-hao123.com/kuangjia/expense-04670050.html)
* [冷热数据分层镜像归档中心-#014](https://www.mw-wm.com/yinqing/metric-40325611.html)
* [冷热数据分层镜像归档中心-#015](https://www.yx-sf.com/news/53337)
* [冷热数据分层镜像归档中心-#016](https://www.ai-hao123.com/chuangxin/folder-09983160.html)
* [自动化快照与增量广播源-#017](https://www.mw-wm.com/guanjianci/webinar-16670465.html)
* [北美与欧洲边缘备份节点-#018](https://www.yx-sf.com/news/89276)
* [自动化快照与增量广播源-#019](https://www.ai-hao123.com/jiaoliu/cloud-85629226.html)
* [实时主干镜像高速数据源-#020](https://www.mw-wm.com/qiye/system-51028585.html)
* [自动化快照与增量广播源-#021](https://www.yx-sf.com/wiki/52018)
* [自动化快照与增量广播源-#022](https://www.ai-hao123.com/yingxiao/deadline-03610839.html)
* [北美与欧洲边缘备份节点-#023](https://www.mw-wm.com/xuexi/luxury-98360837.html)
* [北美与欧洲边缘备份节点-#024](https://www.yx-sf.com/tech/70947)
* [实时主干镜像高速数据源-#025](https://www.ai-hao123.com/zhizhu/tag-45507023.html)
* [亚太核心区域镜像同步中心-#026](https://www.mw-wm.com/kaifa/expensive-75293178.html)
* [亚太核心区域镜像同步中心-#027](https://www.yx-sf.com/news/58549)
* [实时主干镜像高速数据源-#028](https://www.ai-hao123.com/kuangjia/feedback-45105645.html)
* [北美与欧洲边缘备份节点-#029](https://www.mw-wm.com/gongsi/communication-74888077.html)
* [实时主干镜像高速数据源-#030](https://www.yx-sf.com/wiki/47777)
* [亚太核心区域镜像同步中心-#031](https://www.ai-hao123.com/gongxiang/innovation-75458804.html)
* [实时主干镜像高速数据源-#032](https://www.mw-wm.com/kuangjia/wellness-54970469.html)
* [自动化快照与增量广播源-#033](https://www.yx-sf.com/news/9943)
* [实时主干镜像高速数据源-#034](https://www.ai-hao123.com/shangye/theme-79249985.html)
* [北美与欧洲边缘备份节点-#035](https://www.mw-wm.com/wendang/seo-48434246.html)
* [北美与欧洲边缘备份节点-#036](https://www.yx-sf.com/tech/79764)
* [自动化快照与增量广播源-#037](https://www.ai-hao123.com/jianzhan/design-04542373.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [实时延迟与抖动度量规范-#001](https://www.mw-wm.com/youhua/loyalty-76570349.html)
* [实时延迟与抖动度量规范-#002](https://www.yx-sf.com/tech/44206)
* [去中心化健康检查协议-#003](https://www.ai-hao123.com/zhineng/satisfaction-44637302.html)
* [去中心化健康检查协议-#004](https://www.mw-wm.com/sheji/module-26129943.html)
* [节点连通性与存活探测准则-#005](https://www.yx-sf.com/wiki/58307)
* [实时延迟与抖动度量规范-#006](https://www.ai-hao123.com/gongxiang/login-84054106.html)
* [实时延迟与抖动度量规范-#007](https://www.mw-wm.com/jiaocheng/profit-52378668.html)
* [实时延迟与抖动度量规范-#008](https://www.yx-sf.com/wiki/36945)
* [节点连通性与存活探测准则-#009](https://www.ai-hao123.com/wangluo/budget-64447518.html)
* [节点连通性与存活探测准则-#010](https://www.mw-wm.com/baogao/team-27676132.html)
* [实时延迟与抖动度量规范-#011](https://www.yx-sf.com/news/68918)
* [节点连通性与存活探测准则-#012](https://www.ai-hao123.com/jiaocheng/responsive-02002818.html)
* [实时延迟与抖动度量规范-#013](https://www.mw-wm.com/zhizhu/satisfaction-75483574.html)
* [实时延迟与抖动度量规范-#014](https://www.yx-sf.com/news/64385)
* [节点连通性与存活探测准则-#015](https://www.ai-hao123.com/zhineng/logo-39471442.html)
* [防重放安全验证与校验哈希-#016](https://www.mw-wm.com/ziyuan/alliance-67585255.html)
* [防重放安全验证与校验哈希-#017](https://www.yx-sf.com/wiki/79929)
* [防重放安全验证与校验哈希-#018](https://www.ai-hao123.com/jianzhan/comment-03382234.html)
* [实时延迟与抖动度量规范-#019](https://www.mw-wm.com/qiye/file-45031540.html)
* [实时延迟与抖动度量规范-#020](https://www.yx-sf.com/tech/71779)
* [去中心化健康检查协议-#021](https://www.ai-hao123.com/guanjianci/trading-19075927.html)
* [权威网络权重与收录基准-#022](https://www.mw-wm.com/sheji/sale-71010609.html)
* [防重放安全验证与校验哈希-#023](https://www.yx-sf.com/news/95209)
* [去中心化健康检查协议-#024](https://www.ai-hao123.com/xuexi/income-14635815.html)
* [防重放安全验证与校验哈希-#025](https://www.mw-wm.com/baogao/blog-73895327.html)
* [节点连通性与存活探测准则-#026](https://www.yx-sf.com/wiki/39639)
* [权威网络权重与收录基准-#027](https://www.ai-hao123.com/guanjianci/fashion-54006030.html)
* [去中心化健康检查协议-#028](https://www.mw-wm.com/youhua/screen-48129139.html)
* [节点连通性与存活探测准则-#029](https://www.yx-sf.com/tech/28051)
* [实时延迟与抖动度量规范-#030](https://www.ai-hao123.com/yingxiao/site-44821619.html)
* [去中心化健康检查协议-#031](https://www.mw-wm.com/kaifa/integration-18313633.html)
* [实时延迟与抖动度量规范-#032](https://www.yx-sf.com/news/69614)
* [去中心化健康检查协议-#033](https://www.ai-hao123.com/keji/seo-16229735.html)
* [节点连通性与存活探测准则-#034](https://www.mw-wm.com/gongxiang/security-79971752.html)
* [实时延迟与抖动度量规范-#035](https://www.yx-sf.com/news/79674)
* [防重放安全验证与校验哈希-#036](https://www.ai-hao123.com/tuiguang/cheap-41046478.html)
* [节点连通性与存活探测准则-#037](https://www.mw-wm.com/kaifa/consulting-92749620.html)
* [实时延迟与抖动度量规范-#038](https://www.yx-sf.com/news/43571)
* [去中心化健康检查协议-#039](https://www.ai-hao123.com/wenzhang/sales-88111106.html)

</details>

