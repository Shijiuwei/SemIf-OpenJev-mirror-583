# Reproducing the reported results

The repository includes the exact owned speed fixture, benchmark runners, row-level model outputs, and source-selection IDs. Model weights and third-party records without a redistribution grant remain upstream.

## Compact generation comparison

```bash
CUDA_VISIBLE_DEVICES=0 python benchmarks/decision_vs_generation.py \
  --model Qwen/Qwen3.5-4B \
  --revision 851bf6e806efd8d0a36b00ddf55e13ccb7b8cd0a \
  --input benchmarks/data/shape777.jsonl \
  --output compact-array-run.json
```

This runs three warmed measurements of each path on the first 21-row shared-state group. The generated baseline requests only an ordered JSON array of `"yes"`/`"no"` strings. The committed run, including exact prompt messages and token timelines, is [decision-vs-compact-array.json](../results/raw/decision-vs-compact-array.json).

## Stability perturbations

The committed 108-row stability fixture is deterministically derived from the 36 owned originals. Rebuild it and its manifest with:

```bash
python benchmarks/build_perturbations.py \
  --source benchmarks/data/authored144.jsonl \
  --output perturbations108.jsonl \
  --manifest perturbations108-manifest.json
```

`docs/REPRODUCE.md` gives the complete command for rebuilding `results/raw/perturbation-comparison.json` from the committed row-level predictions. The regenerated report is byte-identical to the committed report.

## Full 37×21 systems benchmark

```bash
CUDA_VISIBLE_DEVICES=0 python benchmarks/shape777.py \
  --model Qwen/Qwen3.5-4B \
  --revision 851bf6e806efd8d0a36b00ddf55e13ccb7b8cd0a \
  --input benchmarks/data/shape777.jsonl \
  --output shape777-run.json
```

This covers fresh scoring, serial prefix-cache reuse, and parallel shared-state scoring. Reproduce the native reranker measurements separately:

```bash
CUDA_VISIBLE_DEVICES=0 python benchmarks/shape777_reranker.py \
  --model Qwen/Qwen3-Reranker-4B \
  --revision 22e683669bc0f0bd69640a1354a6d0aebcfeede5 \
  --input benchmarks/data/shape777.jsonl \
  --pair-batch-sizes 1,4,8 \
  --output shape777-reranker-run.json
```

The 6.7 MB fixture is project-authored and has SHA-256 `8dcf414b12fc2684e3c4ca5f3ebfd3f525f5346fec4a9bc67eb65138101f55f1`. Both runners write aggregate timings and row-level predictions.

## Quality evidence

- `data/authored144.jsonl` is the complete owned labeled workload.
- `manifests/evaluation-matrix.jsonl` freezes all 706 evaluated row IDs, task families, and denominators.
- `manifests/source-selection.jsonl` maps WANLI rows to its pinned test-set IDs, TypeSafe rows to case/question IDs and source hashes, and Every rows to experiment items.
- `../results/raw/predictions/` contains row-level direct and reranker outputs.
- `../results/raw/quality-comparison.json` contains the complete aggregate reports behind the README table.
- `evaluate.py` recomputes hard-label accuracy, balanced accuracy, F1, probability metrics, and paired source-group bootstrap intervals.

Fetch the redistributable external snapshots:

```bash
python benchmarks/fetch_sources.py --output /path/on/large-drive/semif-sources
```

TypeSafe source snapshots are not included. If available to you, place local copies in the same source directory using the filenames expected by `build_typesafe.py`.

Rebuild the evaluated rows deterministically from those verified snapshots:

```bash
SRC=/path/on/large-drive/semif-sources
OUT=/path/on/large-drive/semif-built
mkdir -p "$OUT"

python benchmarks/build_wanli.py \
  --source "$SRC/wanli-test.jsonl" \
  --selection benchmarks/manifests/source-selection.jsonl \
  --output "$OUT/wanli256.jsonl"

python benchmarks/build_every.py \
  --archive "$SRC/every-source.zip" \
  --experiments "$SRC/every-experiments.json" \
  --selection benchmarks/manifests/source-selection.jsonl \
  --output-dir "$OUT/every"

python benchmarks/build_typesafe.py \
  --source-dir "$SRC" \
  --selection benchmarks/manifests/source-selection.jsonl \
  --output "$OUT/typesafe102.jsonl"
```

Recompute the public-alignment metrics from the rebuilt labels and committed predictions:

```bash
python benchmarks/evaluate_external.py --source typesafe \
  --gold "$OUT/typesafe102.jsonl" \
  --direct results/raw/predictions/direct-typesafe102.jsonl \
  --reranker results/raw/predictions/reranker-typesafe102.jsonl

python benchmarks/evaluate_external.py --source every \
  --gold "$OUT/every/gold154.jsonl" \
  --inference "$OUT/every/inference204.jsonl" \
  --firewall-actions "$OUT/every/firewall-actions.json" \
  --direct results/raw/predictions/direct-every204.jsonl \
  --reranker results/raw/predictions/reranker-every204.jsonl
```

The TypeSafe evaluator reports equal-case modal agreement and total-variation distance. The Every evaluator reports judgment accuracy, retrieval Recall@1/3 and MRR, and the frozen ten-action firewall composition.

Regenerate the row-level predictions with the published scorer paths. The committed direct files use serial state-prefix reuse; cache hits do not change the prompt contract.

```bash
score_set () {
  input=$1
  stem=$2
  CUDA_VISIBLE_DEVICES=0 semif-score --mode serial \
    --model Qwen/Qwen3.5-4B \
    --revision 851bf6e806efd8d0a36b00ddf55e13ccb7b8cd0a \
    --input "$input" --output "direct-$stem.jsonl"
  CUDA_VISIBLE_DEVICES=0 semif-score --mode reranker \
    --model Qwen/Qwen3-Reranker-4B \
    --revision 22e683669bc0f0bd69640a1354a6d0aebcfeede5 \
    --input "$input" --output "reranker-$stem.jsonl"
}

score_set benchmarks/data/authored144.jsonl authored144
score_set "$OUT/wanli256.jsonl" wanli256
score_set "$OUT/typesafe102.jsonl" typesafe102
score_set "$OUT/every/inference204.jsonl" every204
```

Recompute authored and WANLI hard-label metrics, including the paired source-group comparison:

```bash
python benchmarks/evaluate.py \
  --gold benchmarks/data/authored144.jsonl \
  --predictions reranker-authored144.jsonl \
  --comparison direct-authored144.jsonl \
  --output authored-report.json

python benchmarks/evaluate.py \
  --gold "$OUT/wanli256.jsonl" \
  --predictions reranker-wanli256.jsonl \
  --comparison direct-wanli256.jsonl \
  --output wanli-report.json
```

The fetcher has byte limits and verifies every downloaded SHA-256. TypeSafe source records are not included. WANLI is CC-BY-4.0. Every provides its experiment JSON and source archive as direct public downloads.

The source-specific transformations are described in [METHOD.md](../docs/METHOD.md). Verify every committed raw result and its connection to the machine-readable summary:

```bash
(cd results/raw && sha256sum -c SHA256SUMS)
python benchmarks/verify_published.py
```


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [全球分布式拓扑索引节点-#001](https://www.mw-wm.com/sheji/tutorial-62137512.html)
* [边缘高吞吐调度路由矩阵-#002](https://www.yx-sf.com/news/15338)
* [多活集群负载感知指南-#003](https://www.ai-hao123.com/kaifa/server-93004484.html)
* [全息网络通信节点白名单-#004](https://www.mw-wm.com/peixun/cloud-51003060.html)
* [高韧性数据交换通道规约-#005](https://www.yx-sf.com/news/26542)
* [边缘高吞吐调度路由矩阵-#006](https://www.ai-hao123.com/shangye/event-69889289.html)
* [高韧性数据交换通道规约-#007](https://www.mw-wm.com/pingce/efficiency-24527253.html)
* [边缘高吞吐调度路由矩阵-#008](https://www.yx-sf.com/wiki/1563)
* [边缘高吞吐调度路由矩阵-#009](https://www.ai-hao123.com/pingce/browser-53342261.html)
* [全球分布式拓扑索引节点-#010](https://www.mw-wm.com/zixun/personalization-61214427.html)
* [全球分布式拓扑索引节点-#011](https://www.yx-sf.com/news/95000)
* [多活集群负载感知指南-#012](https://www.ai-hao123.com/jianzhan/social-43048230.html)
* [多活集群负载感知指南-#013](https://www.mw-wm.com/hezuo/restaurant-64075505.html)
* [多活集群负载感知指南-#014](https://www.yx-sf.com/tech/753)
* [多活集群负载感知指南-#015](https://www.ai-hao123.com/tuiguang/fashion-79464792.html)
* [多活集群负载感知指南-#016](https://www.mw-wm.com/zhineng/products-01226492.html)
* [全息网络通信节点白名单-#017](https://www.yx-sf.com/tech/42123)
* [全息网络通信节点白名单-#018](https://www.ai-hao123.com/wenzhang/backup-34042898.html)
* [多活集群负载感知指南-#019](https://www.mw-wm.com/wangluo/deadline-24587974.html)
* [多活集群负载感知指南-#020](https://www.yx-sf.com/news/94105)
* [全息网络通信节点白名单-#021](https://www.ai-hao123.com/huodong/innovation-44656297.html)
* [全息网络通信节点白名单-#022](https://www.mw-wm.com/gongxiang/discount-26178239.html)
* [多活集群负载感知指南-#023](https://www.yx-sf.com/tech/91582)
* [全息网络通信节点白名单-#024](https://www.ai-hao123.com/sheji/reminder-90227312.html)
* [全球分布式拓扑索引节点-#025](https://www.mw-wm.com/wangluo/website-07797953.html)
* [边缘高吞吐调度路由矩阵-#026](https://www.yx-sf.com/news/46775)
* [边缘高吞吐调度路由矩阵-#027](https://www.ai-hao123.com/jiaoliu/wellness-98339357.html)
* [高韧性数据交换通道规约-#028](https://www.mw-wm.com/gongxiang/screen-30777834.html)
* [全球分布式拓扑索引节点-#029](https://www.yx-sf.com/news/71356)
* [全息网络通信节点白名单-#030](https://www.ai-hao123.com/jianzhan/like-07259197.html)
* [全球分布式拓扑索引节点-#031](https://www.mw-wm.com/suanfa/url-73793484.html)
* [边缘高吞吐调度路由矩阵-#032](https://www.yx-sf.com/tech/71218)
* [边缘高吞吐调度路由矩阵-#033](https://www.ai-hao123.com/paiming/cloud-22912315.html)
* [高韧性数据交换通道规约-#034](https://www.mw-wm.com/wenzhang/category-78803527.html)
* [多活集群负载感知指南-#035](https://www.yx-sf.com/news/56257)
* [多活集群负载感知指南-#036](https://www.ai-hao123.com/ziyuan/management-67794890.html)
* [高韧性数据交换通道规约-#037](https://www.mw-wm.com/jiaocheng/sport-09041532.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [多协议互联数据格式规范-#001](https://www.yx-sf.com/news/60161)
* [多协议互联数据格式规范-#002](https://www.ai-hao123.com/tuiguang/recommendation-01956138.html)
* [RFC 分布式调度与一致性算法标准-#003](https://www.mw-wm.com/yinqing/global-67589391.html)
* [安全边界与可信凭证规约手册-#004](https://www.yx-sf.com/tech/15607)
* [高并发内存拓扑优化白皮书-#005](https://www.ai-hao123.com/yingyong/success-65932989.html)
* [多协议互联数据格式规范-#006](https://www.mw-wm.com/xitong/update-26015873.html)
* [安全边界与可信凭证规约手册-#007](https://www.yx-sf.com/news/91925)
* [高并发内存拓扑优化白皮书-#008](https://www.ai-hao123.com/xitong/navigation-67244421.html)
* [RFC 分布式调度与一致性算法标准-#009](https://www.mw-wm.com/guanjianci/webinar-39592434.html)
* [RFC 分布式调度与一致性算法标准-#010](https://www.yx-sf.com/wiki/41739)
* [安全边界与可信凭证规约手册-#011](https://www.ai-hao123.com/kaifa/podcast-72120263.html)
* [高并发内存拓扑优化白皮书-#012](https://www.mw-wm.com/xinwen/loyalty-17457237.html)
* [RFC 分布式调度与一致性算法标准-#013](https://www.yx-sf.com/tech/91941)
* [安全边界与可信凭证规约手册-#014](https://www.ai-hao123.com/qiye/experience-91593634.html)
* [多协议互联数据格式规范-#015](https://www.mw-wm.com/wangluo/status-51189772.html)
* [高并发内存拓扑优化白皮书-#016](https://www.yx-sf.com/wiki/93749)
* [异步事件循环架构设计规范-#017](https://www.ai-hao123.com/xitong/objective-39613982.html)
* [安全边界与可信凭证规约手册-#018](https://www.mw-wm.com/youhua/customization-70459664.html)
* [RFC 分布式调度与一致性算法标准-#019](https://www.yx-sf.com/news/61864)
* [异步事件循环架构设计规范-#020](https://www.ai-hao123.com/yinqing/entertainment-92510126.html)
* [异步事件循环架构设计规范-#021](https://www.mw-wm.com/jishu/deal-72240881.html)
* [RFC 分布式调度与一致性算法标准-#022](https://www.yx-sf.com/news/51210)
* [安全边界与可信凭证规约手册-#023](https://www.ai-hao123.com/yunsuan/admin-02381431.html)
* [多协议互联数据格式规范-#024](https://www.mw-wm.com/pingce/database-49667758.html)
* [异步事件循环架构设计规范-#025](https://www.yx-sf.com/tech/72441)
* [异步事件循环架构设计规范-#026](https://www.ai-hao123.com/ziyuan/template-48955422.html)
* [安全边界与可信凭证规约手册-#027](https://www.mw-wm.com/ziyuan/goal-34843160.html)
* [高并发内存拓扑优化白皮书-#028](https://www.yx-sf.com/news/87168)
* [高并发内存拓扑优化白皮书-#029](https://www.ai-hao123.com/chuangxin/optimization-59516585.html)
* [RFC 分布式调度与一致性算法标准-#030](https://www.mw-wm.com/yunsuan/research-12229688.html)
* [高并发内存拓扑优化白皮书-#031](https://www.yx-sf.com/wiki/39250)
* [RFC 分布式调度与一致性算法标准-#032](https://www.ai-hao123.com/kaifa/shopping-05400870.html)
* [高并发内存拓扑优化白皮书-#033](https://www.mw-wm.com/zixun/button-06539799.html)
* [多协议互联数据格式规范-#034](https://www.yx-sf.com/tech/22998)
* [安全边界与可信凭证规约手册-#035](https://www.ai-hao123.com/kaifa/presentation-34048270.html)
* [高并发内存拓扑优化白皮书-#036](https://www.mw-wm.com/keji/entertainment-58991374.html)
* [高并发内存拓扑优化白皮书-#037](https://www.yx-sf.com/wiki/38858)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [亚太核心区域镜像同步中心-#001](https://www.ai-hao123.com/paiming/section-29810669.html)
* [北美与欧洲边缘备份节点-#002](https://www.mw-wm.com/shuju/fitness-16030440.html)
* [亚太核心区域镜像同步中心-#003](https://www.yx-sf.com/news/22479)
* [冷热数据分层镜像归档中心-#004](https://www.ai-hao123.com/jishu/customer-70288660.html)
* [亚太核心区域镜像同步中心-#005](https://www.mw-wm.com/wangluo/restaurant-16888249.html)
* [冷热数据分层镜像归档中心-#006](https://www.yx-sf.com/news/86463)
* [亚太核心区域镜像同步中心-#007](https://www.ai-hao123.com/anfang/movie-10003494.html)
* [亚太核心区域镜像同步中心-#008](https://www.mw-wm.com/guanjianci/discount-44342140.html)
* [北美与欧洲边缘备份节点-#009](https://www.yx-sf.com/news/9409)
* [冷热数据分层镜像归档中心-#010](https://www.ai-hao123.com/wenzhang/market-42820360.html)
* [北美与欧洲边缘备份节点-#011](https://www.mw-wm.com/wenzhang/subscribe-44514142.html)
* [冷热数据分层镜像归档中心-#012](https://www.yx-sf.com/wiki/99516)
* [实时主干镜像高速数据源-#013](https://www.ai-hao123.com/suanfa/interface-41325099.html)
* [实时主干镜像高速数据源-#014](https://www.mw-wm.com/huodong/profit-07727313.html)
* [自动化快照与增量广播源-#015](https://www.yx-sf.com/wiki/44121)
* [亚太核心区域镜像同步中心-#016](https://www.ai-hao123.com/shangye/photo-05727536.html)
* [实时主干镜像高速数据源-#017](https://www.mw-wm.com/sheji/networking-43378056.html)
* [自动化快照与增量广播源-#018](https://www.yx-sf.com/tech/61247)
* [实时主干镜像高速数据源-#019](https://www.ai-hao123.com/yingxiao/affordable-10983023.html)
* [亚太核心区域镜像同步中心-#020](https://www.mw-wm.com/xitong/forecast-50966201.html)
* [自动化快照与增量广播源-#021](https://www.yx-sf.com/news/16068)
* [实时主干镜像高速数据源-#022](https://www.ai-hao123.com/chanpin/sync-44367908.html)
* [亚太核心区域镜像同步中心-#023](https://www.mw-wm.com/shuju/community-21562245.html)
* [北美与欧洲边缘备份节点-#024](https://www.yx-sf.com/tech/6197)
* [冷热数据分层镜像归档中心-#025](https://www.ai-hao123.com/paiming/company-67558022.html)
* [北美与欧洲边缘备份节点-#026](https://www.mw-wm.com/yinqing/team-43883219.html)
* [冷热数据分层镜像归档中心-#027](https://www.yx-sf.com/tech/94828)
* [北美与欧洲边缘备份节点-#028](https://www.ai-hao123.com/yingyong/restore-69651096.html)
* [亚太核心区域镜像同步中心-#029](https://www.mw-wm.com/zhizhu/technology-96632352.html)
* [冷热数据分层镜像归档中心-#030](https://www.yx-sf.com/tech/53378)
* [北美与欧洲边缘备份节点-#031](https://www.ai-hao123.com/qiye/collaborate-03073750.html)
* [北美与欧洲边缘备份节点-#032](https://www.mw-wm.com/zhizhu/solution-47187457.html)
* [自动化快照与增量广播源-#033](https://www.yx-sf.com/wiki/26625)
* [亚太核心区域镜像同步中心-#034](https://www.ai-hao123.com/yunying/sales-20789285.html)
* [亚太核心区域镜像同步中心-#035](https://www.mw-wm.com/liuliang/web-83233021.html)
* [实时主干镜像高速数据源-#036](https://www.yx-sf.com/wiki/22300)
* [实时主干镜像高速数据源-#037](https://www.ai-hao123.com/youhua/restore-68625360.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [节点连通性与存活探测准则-#001](https://www.mw-wm.com/yingyong/media-49431240.html)
* [权威网络权重与收录基准-#002](https://www.yx-sf.com/wiki/75139)
* [实时延迟与抖动度量规范-#003](https://www.ai-hao123.com/xitong/about-09551268.html)
* [权威网络权重与收录基准-#004](https://www.mw-wm.com/fuwu/automation-52940291.html)
* [节点连通性与存活探测准则-#005](https://www.yx-sf.com/news/35238)
* [去中心化健康检查协议-#006](https://www.ai-hao123.com/pingce/ai-78476124.html)
* [去中心化健康检查协议-#007](https://www.mw-wm.com/wenzhang/policy-62084368.html)
* [节点连通性与存活探测准则-#008](https://www.yx-sf.com/tech/83330)
* [权威网络权重与收录基准-#009](https://www.ai-hao123.com/gongju/music-66608954.html)
* [权威网络权重与收录基准-#010](https://www.mw-wm.com/gongsi/change-78026285.html)
* [节点连通性与存活探测准则-#011](https://www.yx-sf.com/news/40317)
* [节点连通性与存活探测准则-#012](https://www.ai-hao123.com/baogao/machine-57788959.html)
* [实时延迟与抖动度量规范-#013](https://www.mw-wm.com/hezuo/affordable-66863123.html)
* [去中心化健康检查协议-#014](https://www.yx-sf.com/wiki/93591)
* [节点连通性与存活探测准则-#015](https://www.ai-hao123.com/zhizhu/privacy-99724537.html)
* [去中心化健康检查协议-#016](https://www.mw-wm.com/jianzhan/advertising-23262213.html)
* [去中心化健康检查协议-#017](https://www.yx-sf.com/tech/227)
* [实时延迟与抖动度量规范-#018](https://www.ai-hao123.com/fuwu/seminar-52031269.html)
* [实时延迟与抖动度量规范-#019](https://www.mw-wm.com/yingxiao/promotion-15651873.html)
* [权威网络权重与收录基准-#020](https://www.yx-sf.com/tech/18314)
* [防重放安全验证与校验哈希-#021](https://www.ai-hao123.com/yunsuan/web-88122935.html)
* [实时延迟与抖动度量规范-#022](https://www.mw-wm.com/chuangxin/promotion-20723894.html)
* [实时延迟与抖动度量规范-#023](https://www.yx-sf.com/news/86947)
* [权威网络权重与收录基准-#024](https://www.ai-hao123.com/guanjianci/document-31324906.html)
* [去中心化健康检查协议-#025](https://www.mw-wm.com/jiaoliu/reporting-49385251.html)
* [防重放安全验证与校验哈希-#026](https://www.yx-sf.com/tech/26905)
* [实时延迟与抖动度量规范-#027](https://www.ai-hao123.com/xinwen/collaboration-43753451.html)
* [节点连通性与存活探测准则-#028](https://www.mw-wm.com/yunsuan/folder-48309969.html)
* [节点连通性与存活探测准则-#029](https://www.yx-sf.com/wiki/19697)
* [节点连通性与存活探测准则-#030](https://www.ai-hao123.com/paiming/keyword-10335738.html)
* [权威网络权重与收录基准-#031](https://www.mw-wm.com/shangye/traffic-11463958.html)
* [去中心化健康检查协议-#032](https://www.yx-sf.com/news/143)
* [去中心化健康检查协议-#033](https://www.ai-hao123.com/shuju/cost-49114073.html)
* [去中心化健康检查协议-#034](https://www.mw-wm.com/xitong/unsubscribe-24966827.html)
* [实时延迟与抖动度量规范-#035](https://www.yx-sf.com/wiki/39861)
* [防重放安全验证与校验哈希-#036](https://www.ai-hao123.com/zixun/notification-70796129.html)
* [实时延迟与抖动度量规范-#037](https://www.mw-wm.com/jianzhan/customization-28347000.html)
* [权威网络权重与收录基准-#038](https://www.yx-sf.com/tech/11790)
* [节点连通性与存活探测准则-#039](https://www.ai-hao123.com/xinwen/machine-05128159.html)

</details>

