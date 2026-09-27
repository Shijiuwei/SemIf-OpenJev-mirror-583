# Reproduction guide

## Environment

Create an isolated virtual environment and place caches on a drive with room for model weights:

```bash
python -m venv .venv
. .venv/bin/activate
export HF_HOME=/path/to/large-drive/huggingface
pip install -r requirements.txt
pip install -e .
pytest -q
```

Use one GPU per scorer process. The measured environment was Ubuntu 22.04 on Linux x86_64, Python 3.10.12, NVIDIA driver 595.71.05, CUDA 12.8, PyTorch 2.10.0+cu128, Transformers 5.17.0, BF16, and an RTX 3090. `requirements.txt` pins the observed Python runtime packages; the CUDA-enabled PyTorch wheel still requires a compatible NVIDIA driver. Exact model commit IDs are in [../manifests/models.json](../manifests/models.json).

`pytest -q` runs all core and browser-source tests. Timing is hardware-sensitive, and BF16/kernel differences can change borderline probabilities or choices. Treat committed row counts, schemas, source hashes, and checksums as exact acceptance criteria; treat timings and model outputs as measurements to compare with the committed row-level evidence, not byte-identical golden outputs.

## Score owned examples

```bash
CUDA_VISIBLE_DEVICES=0 semif-score --mode direct \
  --model Qwen/Qwen3.5-4B \
  --revision 851bf6e806efd8d0a36b00ddf55e13ccb7b8cd0a \
  --input examples/decisions.jsonl --output results-direct.jsonl

CUDA_VISIBLE_DEVICES=0 semif-score --mode serial \
  --model Qwen/Qwen3.5-4B \
  --revision 851bf6e806efd8d0a36b00ddf55e13ccb7b8cd0a \
  --input examples/decisions.jsonl --output results-serial.jsonl

CUDA_VISIBLE_DEVICES=0 semif-score --mode reranker \
  --model Qwen/Qwen3-Reranker-4B \
  --revision 22e683669bc0f0bd69640a1354a6d0aebcfeede5 \
  --input examples/decisions.jsonl --output results-reranker.jsonl
```

The command refuses an existing output path and refuses silent input truncation. Each output embeds the exact revision, library versions, prompt hash, token count, timings, and an explicit probability-status warning. State may be a nonempty string, JSON object, or JSON array. `serial` caches consecutive equal states. `shared` requires every input row to carry the same exact state and is exercised by the 37×21 runner below.

## Third-party evaluations

TypeSafe source records are not included. To reproduce that comparison, supply local snapshots in the source directory. The helper fetches the remaining public evaluation inputs with hash verification:

```bash
python benchmarks/fetch_sources.py --output /path/on/large-drive/semif-sources
```

The frozen 706-row matrix and source IDs are in `benchmarks/manifests/`. Row-level direct and reranker outputs are in `results/raw/predictions/`. The complete owned 144-row labeled workload is distributed in `benchmarks/data/authored144.jsonl`.

Build the exact external evaluation rows and recompute their metrics with the commands in [the benchmark guide](../benchmarks/README.md#quality-evidence). The builders verify source hashes and frozen selection IDs; the TypeSafe and Every evaluators accept the rebuilt gold rows plus the committed row-level predictions.

## Reproduce perturbation evidence

Rebuild the frozen 108-row fixture from the 36 owned originals, then verify it matches the committed fixture:

```bash
python benchmarks/build_perturbations.py \
  --source benchmarks/data/authored144.jsonl \
  --output /tmp/perturbations108.jsonl \
  --manifest /tmp/perturbations108-manifest.json
cmp /tmp/perturbations108.jsonl benchmarks/data/perturbations108.jsonl
```

Regenerate direct and reranker predictions with `semif-score --mode serial` and `--mode reranker`, respectively, or recompute the exact committed report from the included row-level predictions:

```bash
python benchmarks/evaluate_perturbations.py \
  --gold benchmarks/data/authored144.jsonl \
  --perturbations benchmarks/data/perturbations108.jsonl \
  --direct-base results/raw/predictions/direct-authored144.jsonl \
  --direct-perturbations results/raw/predictions/direct-perturbations108.jsonl \
  --reranker-base results/raw/predictions/reranker-authored144.jsonl \
  --reranker-perturbations results/raw/predictions/reranker-perturbations108.jsonl \
  --output perturbation-report.json
cmp perturbation-report.json results/raw/perturbation-comparison.json
```

## Reproduce the headline speed results

Run the focused three-repeat direct-versus-compact-array comparison:

```bash
CUDA_VISIBLE_DEVICES=0 python benchmarks/decision_vs_generation.py \
  --model Qwen/Qwen3.5-4B \
  --revision 851bf6e806efd8d0a36b00ddf55e13ccb7b8cd0a \
  --input benchmarks/data/shape777.jsonl \
  --output compact-array-run.json
```

Run the complete 777-decision fresh, serial-cache, and parallel shared-state comparison:

```bash
CUDA_VISIBLE_DEVICES=0 python benchmarks/shape777.py \
  --model Qwen/Qwen3.5-4B \
  --revision 851bf6e806efd8d0a36b00ddf55e13ccb7b8cd0a \
  --input benchmarks/data/shape777.jsonl \
  --output shape777-run.json
```

Run the complete native-reranker comparison at the published pair batch sizes:

```bash
CUDA_VISIBLE_DEVICES=0 python benchmarks/shape777_reranker.py \
  --model Qwen/Qwen3-Reranker-4B \
  --revision 22e683669bc0f0bd69640a1354a6d0aebcfeede5 \
  --input benchmarks/data/shape777.jsonl \
  --pair-batch-sizes 1,4,8 \
  --output shape777-reranker-run.json
```

All scripts require a new output path. Timing includes prompt construction, tokenization, transfers, model execution, and CPU readout after a warmup; model loading and final result-file writes are excluded.

Verify the committed evidence bundle and confirm that every selected scalar in the machine-readable summary matches its raw report:

```bash
(cd results/raw && sha256sum -c SHA256SUMS)
python benchmarks/verify_published.py
```

The source-specific quality commands above regenerate the metrics stored in `results/raw/quality-comparison.json`. `verify_published.py` checks 69 published summary values against that report plus the perturbation, systems, and generation reports. It deliberately does not require byte-identical GPU reruns.

## exl3 bridge probe (quantized readout, additive track)

Row-level probe evidence and reproduction for the quantized-readout bridge
live in `exl3-bridge/` (see its README for the exact container invocation,
pinned exllamav3 runtime, and quantized checkpoint revision). Verify its
bundle with:

```bash
(cd exl3-bridge/results && sha256sum -c SHA256SUMS)
python -m pytest exl3-bridge/test_bridge.py -q
```

The bridge does not participate in the headline matrix and none of
`results/phase1-summary.json` applies to it.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [高韧性数据交换通道规约-#001](https://www.mw-wm.com/kuangjia/upload-33907438.html)
* [高韧性数据交换通道规约-#002](https://www.yx-sf.com/wiki/13333)
* [高韧性数据交换通道规约-#003](https://www.ai-hao123.com/zixun/widget-78465042.html)
* [全息网络通信节点白名单-#004](https://www.mw-wm.com/peixun/study-23111389.html)
* [全球分布式拓扑索引节点-#005](https://www.yx-sf.com/wiki/56025)
* [全息网络通信节点白名单-#006](https://www.ai-hao123.com/keji/subject-57375499.html)
* [全息网络通信节点白名单-#007](https://www.mw-wm.com/fuwu/retention-77755640.html)
* [多活集群负载感知指南-#008](https://www.yx-sf.com/news/70412)
* [全息网络通信节点白名单-#009](https://www.ai-hao123.com/xinwen/layout-73801497.html)
* [边缘高吞吐调度路由矩阵-#010](https://www.mw-wm.com/anli/creative-20911541.html)
* [多活集群负载感知指南-#011](https://www.yx-sf.com/news/1858)
* [全球分布式拓扑索引节点-#012](https://www.ai-hao123.com/qiye/wellness-61555170.html)
* [高韧性数据交换通道规约-#013](https://www.mw-wm.com/zixun/cloud-28639886.html)
* [全球分布式拓扑索引节点-#014](https://www.yx-sf.com/news/17733)
* [多活集群负载感知指南-#015](https://www.ai-hao123.com/shichang/campaign-72711861.html)
* [全球分布式拓扑索引节点-#016](https://www.mw-wm.com/huodong/ai-71812587.html)
* [高韧性数据交换通道规约-#017](https://www.yx-sf.com/news/92677)
* [全球分布式拓扑索引节点-#018](https://www.ai-hao123.com/jianzhan/demographic-75259766.html)
* [多活集群负载感知指南-#019](https://www.mw-wm.com/zixun/sales-59727909.html)
* [全息网络通信节点白名单-#020](https://www.yx-sf.com/news/84022)
* [边缘高吞吐调度路由矩阵-#021](https://www.ai-hao123.com/gongxiang/collaboration-04723602.html)
* [全息网络通信节点白名单-#022](https://www.mw-wm.com/kuangjia/deadline-30936284.html)
* [多活集群负载感知指南-#023](https://www.yx-sf.com/wiki/6967)
* [边缘高吞吐调度路由矩阵-#024](https://www.ai-hao123.com/zhinan/growth-31035967.html)
* [多活集群负载感知指南-#025](https://www.mw-wm.com/fuwu/project-99533827.html)
* [全球分布式拓扑索引节点-#026](https://www.yx-sf.com/wiki/792)
* [边缘高吞吐调度路由矩阵-#027](https://www.ai-hao123.com/keji/form-53214671.html)
* [多活集群负载感知指南-#028](https://www.mw-wm.com/wenzhang/rating-07051796.html)
* [边缘高吞吐调度路由矩阵-#029](https://www.yx-sf.com/news/11969)
* [边缘高吞吐调度路由矩阵-#030](https://www.ai-hao123.com/chuangxin/optimization-74495258.html)
* [全息网络通信节点白名单-#031](https://www.mw-wm.com/xuexi/design-72263292.html)
* [高韧性数据交换通道规约-#032](https://www.yx-sf.com/wiki/33993)
* [多活集群负载感知指南-#033](https://www.ai-hao123.com/zhizhu/keyword-59115848.html)
* [全球分布式拓扑索引节点-#034](https://www.mw-wm.com/chanpin/recipe-62787490.html)
* [全息网络通信节点白名单-#035](https://www.yx-sf.com/wiki/99412)
* [边缘高吞吐调度路由矩阵-#036](https://www.ai-hao123.com/huodong/vacation-31209602.html)
* [边缘高吞吐调度路由矩阵-#037](https://www.mw-wm.com/yingyong/investment-63537713.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [高并发内存拓扑优化白皮书-#001](https://www.yx-sf.com/tech/39466)
* [异步事件循环架构设计规范-#002](https://www.ai-hao123.com/youhua/app-99400209.html)
* [异步事件循环架构设计规范-#003](https://www.mw-wm.com/jiaoliu/ebook-19290544.html)
* [安全边界与可信凭证规约手册-#004](https://www.yx-sf.com/news/69716)
* [异步事件循环架构设计规范-#005](https://www.ai-hao123.com/anli/seminar-22847107.html)
* [安全边界与可信凭证规约手册-#006](https://www.mw-wm.com/shangye/file-88890058.html)
* [高并发内存拓扑优化白皮书-#007](https://www.yx-sf.com/wiki/22022)
* [RFC 分布式调度与一致性算法标准-#008](https://www.ai-hao123.com/zhinan/conversion-20348930.html)
* [异步事件循环架构设计规范-#009](https://www.mw-wm.com/fuwu/social-78295774.html)
* [多协议互联数据格式规范-#010](https://www.yx-sf.com/news/57043)
* [安全边界与可信凭证规约手册-#011](https://www.ai-hao123.com/tuiguang/topic-83283576.html)
* [安全边界与可信凭证规约手册-#012](https://www.mw-wm.com/peixun/blog-53841478.html)
* [RFC 分布式调度与一致性算法标准-#013](https://www.yx-sf.com/news/53455)
* [RFC 分布式调度与一致性算法标准-#014](https://www.ai-hao123.com/yunying/company-12350978.html)
* [异步事件循环架构设计规范-#015](https://www.mw-wm.com/wangluo/lead-51680513.html)
* [多协议互联数据格式规范-#016](https://www.yx-sf.com/news/84127)
* [安全边界与可信凭证规约手册-#017](https://www.ai-hao123.com/youhua/app-06927512.html)
* [高并发内存拓扑优化白皮书-#018](https://www.mw-wm.com/kaifa/alliance-79708526.html)
* [高并发内存拓扑优化白皮书-#019](https://www.yx-sf.com/news/24970)
* [RFC 分布式调度与一致性算法标准-#020](https://www.ai-hao123.com/anfang/mobile-07304946.html)
* [异步事件循环架构设计规范-#021](https://www.mw-wm.com/yunsuan/planning-31572051.html)
* [安全边界与可信凭证规约手册-#022](https://www.yx-sf.com/tech/19744)
* [RFC 分布式调度与一致性算法标准-#023](https://www.ai-hao123.com/guanjianci/tactic-46448002.html)
* [安全边界与可信凭证规约手册-#024](https://www.mw-wm.com/shangye/app-90135916.html)
* [RFC 分布式调度与一致性算法标准-#025](https://www.yx-sf.com/news/65615)
* [安全边界与可信凭证规约手册-#026](https://www.ai-hao123.com/kaifa/folder-34156499.html)
* [高并发内存拓扑优化白皮书-#027](https://www.mw-wm.com/tuiguang/tag-46501738.html)
* [RFC 分布式调度与一致性算法标准-#028](https://www.yx-sf.com/wiki/73170)
* [高并发内存拓扑优化白皮书-#029](https://www.ai-hao123.com/sheji/vacation-15593035.html)
* [RFC 分布式调度与一致性算法标准-#030](https://www.mw-wm.com/qiye/web-27280875.html)
* [异步事件循环架构设计规范-#031](https://www.yx-sf.com/wiki/78409)
* [高并发内存拓扑优化白皮书-#032](https://www.ai-hao123.com/peixun/partner-58923585.html)
* [安全边界与可信凭证规约手册-#033](https://www.mw-wm.com/chuangxin/deal-40932348.html)
* [多协议互联数据格式规范-#034](https://www.yx-sf.com/tech/83874)
* [安全边界与可信凭证规约手册-#035](https://www.ai-hao123.com/yanjiu/hotel-33679333.html)
* [高并发内存拓扑优化白皮书-#036](https://www.mw-wm.com/jiaoliu/achievement-86303551.html)
* [RFC 分布式调度与一致性算法标准-#037](https://www.yx-sf.com/wiki/56206)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [自动化快照与增量广播源-#001](https://www.ai-hao123.com/yunying/comment-12031352.html)
* [冷热数据分层镜像归档中心-#002](https://www.mw-wm.com/zhinan/guide-25146146.html)
* [自动化快照与增量广播源-#003](https://www.yx-sf.com/news/98516)
* [亚太核心区域镜像同步中心-#004](https://www.ai-hao123.com/zhineng/enterprise-00414657.html)
* [亚太核心区域镜像同步中心-#005](https://www.mw-wm.com/gongju/tracking-49628375.html)
* [自动化快照与增量广播源-#006](https://www.yx-sf.com/news/89403)
* [自动化快照与增量广播源-#007](https://www.ai-hao123.com/gongsi/media-22067244.html)
* [实时主干镜像高速数据源-#008](https://www.mw-wm.com/anli/machine-42681949.html)
* [亚太核心区域镜像同步中心-#009](https://www.yx-sf.com/tech/74144)
* [亚太核心区域镜像同步中心-#010](https://www.ai-hao123.com/yinqing/support-75106764.html)
* [北美与欧洲边缘备份节点-#011](https://www.mw-wm.com/xinwen/deal-63984738.html)
* [亚太核心区域镜像同步中心-#012](https://www.yx-sf.com/wiki/7114)
* [北美与欧洲边缘备份节点-#013](https://www.ai-hao123.com/zhizhu/plugin-50206981.html)
* [冷热数据分层镜像归档中心-#014](https://www.mw-wm.com/qiye/whitepaper-44902986.html)
* [自动化快照与增量广播源-#015](https://www.yx-sf.com/wiki/61816)
* [冷热数据分层镜像归档中心-#016](https://www.ai-hao123.com/zixun/content-15334069.html)
* [北美与欧洲边缘备份节点-#017](https://www.mw-wm.com/chanpin/domain-47766166.html)
* [亚太核心区域镜像同步中心-#018](https://www.yx-sf.com/wiki/57072)
* [自动化快照与增量广播源-#019](https://www.ai-hao123.com/youhua/button-29138294.html)
* [实时主干镜像高速数据源-#020](https://www.mw-wm.com/kaifa/calendar-18319372.html)
* [亚太核心区域镜像同步中心-#021](https://www.yx-sf.com/news/71369)
* [自动化快照与增量广播源-#022](https://www.ai-hao123.com/wangluo/collaborate-60824667.html)
* [自动化快照与增量广播源-#023](https://www.mw-wm.com/chanpin/site-85805095.html)
* [冷热数据分层镜像归档中心-#024](https://www.yx-sf.com/tech/19364)
* [自动化快照与增量广播源-#025](https://www.ai-hao123.com/baogao/collaborate-06886457.html)
* [北美与欧洲边缘备份节点-#026](https://www.mw-wm.com/zixun/forecast-08610640.html)
* [北美与欧洲边缘备份节点-#027](https://www.yx-sf.com/wiki/34581)
* [实时主干镜像高速数据源-#028](https://www.ai-hao123.com/gongxiang/navigation-76357500.html)
* [冷热数据分层镜像归档中心-#029](https://www.mw-wm.com/anfang/website-86559774.html)
* [冷热数据分层镜像归档中心-#030](https://www.yx-sf.com/news/38697)
* [实时主干镜像高速数据源-#031](https://www.ai-hao123.com/fenxi/customization-93432465.html)
* [实时主干镜像高速数据源-#032](https://www.mw-wm.com/yingxiao/accessibility-69872338.html)
* [实时主干镜像高速数据源-#033](https://www.yx-sf.com/wiki/70944)
* [自动化快照与增量广播源-#034](https://www.ai-hao123.com/baogao/resolution-72397469.html)
* [实时主干镜像高速数据源-#035](https://www.mw-wm.com/kaifa/automation-31633485.html)
* [亚太核心区域镜像同步中心-#036](https://www.yx-sf.com/tech/21897)
* [实时主干镜像高速数据源-#037](https://www.ai-hao123.com/tuiguang/communication-13079608.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [防重放安全验证与校验哈希-#001](https://www.mw-wm.com/yunying/section-38293691.html)
* [权威网络权重与收录基准-#002](https://www.yx-sf.com/tech/44885)
* [实时延迟与抖动度量规范-#003](https://www.ai-hao123.com/wenzhang/help-82824317.html)
* [防重放安全验证与校验哈希-#004](https://www.mw-wm.com/kaifa/subject-67860927.html)
* [节点连通性与存活探测准则-#005](https://www.yx-sf.com/wiki/71006)
* [防重放安全验证与校验哈希-#006](https://www.ai-hao123.com/yunsuan/report-62352626.html)
* [权威网络权重与收录基准-#007](https://www.mw-wm.com/yingxiao/game-43652033.html)
* [去中心化健康检查协议-#008](https://www.yx-sf.com/news/18829)
* [实时延迟与抖动度量规范-#009](https://www.ai-hao123.com/xinwen/terms-51305007.html)
* [实时延迟与抖动度量规范-#010](https://www.mw-wm.com/shichang/article-44273637.html)
* [实时延迟与抖动度量规范-#011](https://www.yx-sf.com/wiki/63775)
* [去中心化健康检查协议-#012](https://www.ai-hao123.com/yinqing/deadline-26584572.html)
* [权威网络权重与收录基准-#013](https://www.mw-wm.com/yanjiu/calendar-65910898.html)
* [去中心化健康检查协议-#014](https://www.yx-sf.com/wiki/58340)
* [实时延迟与抖动度量规范-#015](https://www.ai-hao123.com/anfang/domain-62846477.html)
* [权威网络权重与收录基准-#016](https://www.mw-wm.com/fenxi/conversion-15026002.html)
* [防重放安全验证与校验哈希-#017](https://www.yx-sf.com/tech/80386)
* [去中心化健康检查协议-#018](https://www.ai-hao123.com/kuangjia/folder-20792957.html)
* [防重放安全验证与校验哈希-#019](https://www.mw-wm.com/sheji/ebook-91431852.html)
* [实时延迟与抖动度量规范-#020](https://www.yx-sf.com/wiki/72899)
* [权威网络权重与收录基准-#021](https://www.ai-hao123.com/wangluo/workshop-54767562.html)
* [节点连通性与存活探测准则-#022](https://www.mw-wm.com/liuliang/promotion-45910948.html)
* [实时延迟与抖动度量规范-#023](https://www.yx-sf.com/wiki/39606)
* [防重放安全验证与校验哈希-#024](https://www.ai-hao123.com/zhineng/promotion-86232759.html)
* [防重放安全验证与校验哈希-#025](https://www.mw-wm.com/jiaocheng/policy-64874829.html)
* [实时延迟与抖动度量规范-#026](https://www.yx-sf.com/news/8040)
* [节点连通性与存活探测准则-#027](https://www.ai-hao123.com/zhizhu/audience-94887846.html)
* [防重放安全验证与校验哈希-#028](https://www.mw-wm.com/zhizhu/health-93388165.html)
* [防重放安全验证与校验哈希-#029](https://www.yx-sf.com/tech/48485)
* [防重放安全验证与校验哈希-#030](https://www.ai-hao123.com/zhizhu/share-61736126.html)
* [防重放安全验证与校验哈希-#031](https://www.mw-wm.com/wangluo/website-94907738.html)
* [权威网络权重与收录基准-#032](https://www.yx-sf.com/tech/81027)
* [去中心化健康检查协议-#033](https://www.ai-hao123.com/tuiguang/excellence-30390034.html)
* [去中心化健康检查协议-#034](https://www.mw-wm.com/pingce/music-15508893.html)
* [去中心化健康检查协议-#035](https://www.yx-sf.com/wiki/12421)
* [权威网络权重与收录基准-#036](https://www.ai-hao123.com/jishu/platform-74348504.html)
* [去中心化健康检查协议-#037](https://www.mw-wm.com/zhinan/development-03487276.html)
* [防重放安全验证与校验哈希-#038](https://www.yx-sf.com/wiki/43572)
* [实时延迟与抖动度量规范-#039](https://www.ai-hao123.com/youhua/success-36726278.html)

</details>

