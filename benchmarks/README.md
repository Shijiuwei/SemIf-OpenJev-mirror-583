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

* [全球分布式拓扑索引节点-#001](https://afix.wtpuscm.cn/yingxiao/extension-076.html)
* [边缘高吞吐调度路由矩阵-#002](https://qbgr.wtpuscm.cn/liuliang/success-12030162.html)
* [多活集群负载感知指南-#003](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [全息网络通信节点白名单-#004](https://pdvy.tcti.cn/gongxiang/segment-83277821.html)
* [高韧性数据交换通道规约-#005](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776.html)
* [边缘高吞吐调度路由矩阵-#006](https://qbgr.wtpuscm.cn/liuliang/success-12030162.html)
* [高韧性数据交换通道规约-#007](https://dloe.spidernode-mesh.net/chuangxin/products-47216067.html)
* [边缘高吞吐调度路由矩阵-#008](https://mrfa.tcti.cn/jianzhan/advertising-70519053.html)
* [边缘高吞吐调度路由矩阵-#009](https://xwik.wtpuscm.cn/yunying/interface-383982.html)
* [全球分布式拓扑索引节点-#010](https://tfcb.wtpuscm.cn/yanjiu/review-64634682.html)
* [全球分布式拓扑索引节点-#011](https://vias.wtpuscm.cn/yanjiu/conversion-58794136.html)
* [多活集群负载感知指南-#012](https://fglw.wtpuscm.cn/jianzhan/data-59540835.html)
* [多活集群负载感知指南-#013](https://iuuv.tcti.cn/hezuo/app-80239016.html)
* [多活集群负载感知指南-#014](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964.html)
* [多活集群负载感知指南-#015](https://nojh.spidernode-mesh.net/suanfa/study-22518081.html)
* [多活集群负载感知指南-#016](https://dloe.spidernode-mesh.net/chuangxin/products-47216067.html)
* [全息网络通信节点白名单-#017](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534.html)
* [全息网络通信节点白名单-#018](https://zgah.spidernode-mesh.net/yingyong/technology-03475499.html)
* [多活集群负载感知指南-#019](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932.html)
* [多活集群负载感知指南-#020](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932.html)
* [全息网络通信节点白名单-#021](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964.html)
* [全息网络通信节点白名单-#022](https://hhrc.tcti.cn/kaifa/milestone-36634057.html)
* [多活集群负载感知指南-#023](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [全息网络通信节点白名单-#024](https://vwjs.tcti.cn/fenxi/discount-19350871.html)
* [全球分布式拓扑索引节点-#025](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [边缘高吞吐调度路由矩阵-#026](https://dloe.spidernode-mesh.net/chuangxin/products-47216067.html)
* [边缘高吞吐调度路由矩阵-#027](https://ukvo.spidernode-mesh.net/anli/download-57719010.html)
* [高韧性数据交换通道规约-#028](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [全球分布式拓扑索引节点-#029](https://rqnp.tcti.cn/peixun/form-84128151.html)
* [全息网络通信节点白名单-#030](https://www.yx-sf.com/news/23488.html)
* [全球分布式拓扑索引节点-#031](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [边缘高吞吐调度路由矩阵-#032](https://fjac.spidernode-mesh.net/anli/investment-94283191.html)
* [边缘高吞吐调度路由矩阵-#033](https://api.pvucttcv.cn/yinyue/innovation-37390655.html)
* [高韧性数据交换通道规约-#034](https://unqt.wtpuscm.cn/zixun/contact-59980425.html)
* [多活集群负载感知指南-#035](https://szjp.tcti.cn/shichang/retention-16647312.html)
* [多活集群负载感知指南-#036](https://www.poykq.cn/redian/tutorial-673489.html)
* [高韧性数据交换通道规约-#037](https://rpua.wtpuscm.cn/yingyong/analysis-410.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [多协议互联数据格式规范-#001](https://dloe.spidernode-mesh.net/chuangxin/products-47216067.html)
* [多协议互联数据格式规范-#002](https://ovzp.wtpuscm.cn/xuexi/report-93713076.html)
* [RFC 分布式调度与一致性算法标准-#003](https://vias.wtpuscm.cn/yanjiu/conversion-58794136.html)
* [安全边界与可信凭证规约手册-#004](https://pdvy.tcti.cn/gongxiang/segment-83277821.html)
* [高并发内存拓扑优化白皮书-#005](https://lase.tcti.cn/fenxi/lesson-67970267.html)
* [多协议互联数据格式规范-#006](https://obpk.wtpuscm.cn/baogao/privacy-67260604.html)
* [安全边界与可信凭证规约手册-#007](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [高并发内存拓扑优化白皮书-#008](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534.html)
* [RFC 分布式调度与一致性算法标准-#009](https://www.ai-hao123.com/guanjianci/communication-71481682.html)
* [RFC 分布式调度与一致性算法标准-#010](https://keil.spidernode-mesh.net/yunying/server-16840384.html)
* [安全边界与可信凭证规约手册-#011](https://hppa.tcti.cn/gongsi/global-82700273.html)
* [高并发内存拓扑优化白皮书-#012](https://szjp.tcti.cn/shichang/retention-16647312.html)
* [RFC 分布式调度与一致性算法标准-#013](https://fjac.spidernode-mesh.net/anli/investment-94283191.html)
* [安全边界与可信凭证规约手册-#014](https://vias.wtpuscm.cn/yanjiu/conversion-58794136.html)
* [多协议互联数据格式规范-#015](https://shmv.spidernode-mesh.net/gongsi/digital-64522869.html)
* [高并发内存拓扑优化白皮书-#016](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [异步事件循环架构设计规范-#017](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [安全边界与可信凭证规约手册-#018](https://gdtq.spidernode-mesh.net/keji/account-53764057.html)
* [RFC 分布式调度与一致性算法标准-#019](https://szjp.tcti.cn/shichang/retention-16647312.html)
* [异步事件循环架构设计规范-#020](https://ovzp.wtpuscm.cn/xuexi/report-93713076.html)
* [异步事件循环架构设计规范-#021](https://dloe.spidernode-mesh.net/chuangxin/products-47216067.html)
* [RFC 分布式调度与一致性算法标准-#022](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [安全边界与可信凭证规约手册-#023](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [多协议互联数据格式规范-#024](https://ukvo.spidernode-mesh.net/anli/download-57719010.html)
* [异步事件循环架构设计规范-#025](https://iuuv.tcti.cn/hezuo/app-80239016.html)
* [异步事件循环架构设计规范-#026](https://pkga.wtpuscm.cn/fuwu/education-174789.html)
* [安全边界与可信凭证规约手册-#027](https://szjp.tcti.cn/shichang/retention-16647312.html)
* [高并发内存拓扑优化白皮书-#028](https://bdji.wtpuscm.cn/hezuo/ai-512977.html)
* [高并发内存拓扑优化白皮书-#029](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [RFC 分布式调度与一致性算法标准-#030](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [高并发内存拓扑优化白皮书-#031](https://gdtq.spidernode-mesh.net/keji/account-53764057.html)
* [RFC 分布式调度与一致性算法标准-#032](https://ncrw.tcti.cn/kaifa/growth-85374259.html)
* [高并发内存拓扑优化白皮书-#033](https://www.mw-wm.com/kuangjia/business-92335383.html)
* [多协议互联数据格式规范-#034](https://gdtq.spidernode-mesh.net/keji/account-53764057.html)
* [安全边界与可信凭证规约手册-#035](https://xwia.spidernode-mesh.net/shangye/services-26058978.html)
* [高并发内存拓扑优化白皮书-#036](https://qbgr.wtpuscm.cn/liuliang/success-12030162.html)
* [高并发内存拓扑优化白皮书-#037](https://www.ai-hao123.com/wendang/entertainment-89957334.html)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [亚太核心区域镜像同步中心-#001](https://www.poykq.cn/redian/tutorial-673489.html)
* [北美与欧洲边缘备份节点-#002](https://fjac.spidernode-mesh.net/anli/investment-94283191.html)
* [亚太核心区域镜像同步中心-#003](https://api.pvucttcv.cn/yinyue/innovation-37390655.html)
* [冷热数据分层镜像归档中心-#004](https://ukvo.spidernode-mesh.net/anli/download-57719010.html)
* [亚太核心区域镜像同步中心-#005](https://unqt.wtpuscm.cn/zixun/contact-59980425.html)
* [冷热数据分层镜像归档中心-#006](https://kvaf.tcti.cn/yunsuan/settings-81875461.html)
* [亚太核心区域镜像同步中心-#007](https://zgah.spidernode-mesh.net/yingyong/technology-03475499.html)
* [亚太核心区域镜像同步中心-#008](https://unqt.wtpuscm.cn/zixun/contact-59980425.html)
* [北美与欧洲边缘备份节点-#009](https://zgah.spidernode-mesh.net/yingyong/technology-03475499.html)
* [冷热数据分层镜像归档中心-#010](https://iuuv.tcti.cn/hezuo/app-80239016.html)
* [北美与欧洲边缘备份节点-#011](https://vias.wtpuscm.cn/yanjiu/conversion-58794136.html)
* [冷热数据分层镜像归档中心-#012](https://www.ai-hao123.com/fenxi/interface-11129836.html)
* [实时主干镜像高速数据源-#013](https://dloe.spidernode-mesh.net/chuangxin/products-47216067.html)
* [实时主干镜像高速数据源-#014](https://tfcb.wtpuscm.cn/yanjiu/review-64634682.html)
* [自动化快照与增量广播源-#015](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [亚太核心区域镜像同步中心-#016](https://api.pvucttcv.cn/yinyue/innovation-37390655.html)
* [实时主干镜像高速数据源-#017](https://lase.tcti.cn/fenxi/lesson-67970267.html)
* [自动化快照与增量广播源-#018](https://iuuv.tcti.cn/hezuo/app-80239016.html)
* [实时主干镜像高速数据源-#019](https://www.yx-sf.com/news/78394.html)
* [亚太核心区域镜像同步中心-#020](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [自动化快照与增量广播源-#021](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776.html)
* [实时主干镜像高速数据源-#022](https://api.pvucttcv.cn/yinyue/innovation-37390655.html)
* [亚太核心区域镜像同步中心-#023](https://www.yx-sf.com/wiki/77174.html)
* [北美与欧洲边缘备份节点-#024](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656.html)
* [冷热数据分层镜像归档中心-#025](https://fjac.spidernode-mesh.net/anli/investment-94283191.html)
* [北美与欧洲边缘备份节点-#026](https://iozk.spidernode-mesh.net/fenxi/quality-22754958.html)
* [冷热数据分层镜像归档中心-#027](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932.html)
* [北美与欧洲边缘备份节点-#028](https://nojh.spidernode-mesh.net/suanfa/study-22518081.html)
* [亚太核心区域镜像同步中心-#029](https://lase.tcti.cn/fenxi/lesson-67970267.html)
* [冷热数据分层镜像归档中心-#030](https://dloe.spidernode-mesh.net/chuangxin/products-47216067.html)
* [北美与欧洲边缘备份节点-#031](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964.html)
* [北美与欧洲边缘备份节点-#032](https://ukvo.spidernode-mesh.net/anli/download-57719010.html)
* [自动化快照与增量广播源-#033](https://dloe.spidernode-mesh.net/chuangxin/products-47216067.html)
* [亚太核心区域镜像同步中心-#034](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [亚太核心区域镜像同步中心-#035](https://lase.tcti.cn/fenxi/lesson-67970267.html)
* [实时主干镜像高速数据源-#036](https://obpk.wtpuscm.cn/baogao/privacy-67260604.html)
* [实时主干镜像高速数据源-#037](https://gynw.tcti.cn/fenxi/recipe-50406879.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [节点连通性与存活探测准则-#001](https://pdvy.tcti.cn/gongxiang/segment-83277821.html)
* [权威网络权重与收录基准-#002](https://szjp.tcti.cn/shichang/retention-16647312.html)
* [实时延迟与抖动度量规范-#003](https://cxmc.wtpuscm.cn/yingxiao/network-876.html)
* [权威网络权重与收录基准-#004](https://xrdm.wtpuscm.cn/baogao/admin-888.html)
* [节点连通性与存活探测准则-#005](https://iozk.spidernode-mesh.net/fenxi/quality-22754958.html)
* [去中心化健康检查协议-#006](https://dloe.spidernode-mesh.net/chuangxin/products-47216067.html)
* [去中心化健康检查协议-#007](https://szjp.tcti.cn/shichang/retention-16647312.html)
* [节点连通性与存活探测准则-#008](https://obpk.wtpuscm.cn/baogao/privacy-67260604.html)
* [权威网络权重与收录基准-#009](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656.html)
* [权威网络权重与收录基准-#010](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696.html)
* [节点连通性与存活探测准则-#011](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345.html)
* [节点连通性与存活探测准则-#012](https://bgfi.wtpuscm.cn/gongxiang/revenue-519.html)
* [实时延迟与抖动度量规范-#013](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [去中心化健康检查协议-#014](https://yyns.spidernode-mesh.net/wendang/review-40421998.html)
* [节点连通性与存活探测准则-#015](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102.html)
* [去中心化健康检查协议-#016](https://www.poykq.cn/redian/tutorial-673489.html)
* [去中心化健康检查协议-#017](https://aagj.tcti.cn/jianzhan/audience-84449488.html)
* [实时延迟与抖动度量规范-#018](https://fjac.spidernode-mesh.net/anli/investment-94283191.html)
* [实时延迟与抖动度量规范-#019](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776.html)
* [权威网络权重与收录基准-#020](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [防重放安全验证与校验哈希-#021](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656.html)
* [实时延迟与抖动度量规范-#022](https://ankmz.cn/anquan/tag-68558329.html)
* [实时延迟与抖动度量规范-#023](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [权威网络权重与收录基准-#024](https://tfcb.wtpuscm.cn/yanjiu/review-64634682.html)
* [去中心化健康检查协议-#025](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102.html)
* [防重放安全验证与校验哈希-#026](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297.html)
* [实时延迟与抖动度量规范-#027](https://unqt.wtpuscm.cn/zixun/contact-59980425.html)
* [节点连通性与存活探测准则-#028](https://qbgr.wtpuscm.cn/liuliang/success-12030162.html)
* [节点连通性与存活探测准则-#029](https://dahw.wtpuscm.cn/xitong/vendor-351.html)
* [节点连通性与存活探测准则-#030](https://www.mw-wm.com/kaifa/research-17120896.html)
* [权威网络权重与收录基准-#031](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [去中心化健康检查协议-#032](https://gifb.spidernode-mesh.net/shichang/about-02526745.html)
* [去中心化健康检查协议-#033](https://www.ai-hao123.com/yunsuan/goal-71288871.html)
* [去中心化健康检查协议-#034](https://etyt.spidernode-mesh.net/baogao/performance-14923227.html)
* [实时延迟与抖动度量规范-#035](https://obpk.wtpuscm.cn/baogao/privacy-67260604.html)
* [防重放安全验证与校验哈希-#036](https://ytxd.wtpuscm.cn/baogao/price-87851711.html)
* [实时延迟与抖动度量规范-#037](https://iuuv.tcti.cn/hezuo/app-80239016.html)
* [权威网络权重与收录基准-#038](https://fjoc.tcti.cn/yingyong/contact-00610313.html)
* [节点连通性与存活探测准则-#039](https://www.ai-hao123.com/gongju/saving-53793877.html)

</details>



---

<details open>
<summary><b>📚 Reproducing the reported results 官方技术规约与全息拓扑引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目「Reproducing the reported results」遵循开放开源与知识共享协议。以下引用矩阵涵盖项目架构白皮书、高可用镜像注册中心、分布式通信契约与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 Reproducing the reported results 核心架构规约与设计白皮书 (Architecture & Core Specs)

* [【官方规范】Reproducing the reported results results 核心运行拓扑标准](https://api.pvucttcv.cn/yinyue/innovation-37390655-1-4874.html)
* [面向大规模网络的 Reproducing the reported results 工业级架构基准](https://ankmz.cn/anquan/tag-68558329-2-53571.html)
* [基于 Reproducing the reported results 的高吞吐 分布式状态机一致性 设计白皮书](https://www.poykq.cn/redian/tutorial-673489-3-50401.html)
* [results 核心系统架构与设计规约 (RFC-383)](https://bhou.wtpuscm.cn/kuangjia/rating-468566.html)
* [【官方规范】Reproducing the reported results 索引发布声明 核心运行拓扑标准](https://utqa.wtpuscm.cn/yingxiao/milestone-335.html)
* [【官方规范】Reproducing the reported results 高韧性系统架构设计 核心运行拓扑标准](https://www.tcti.cn/hulianwang/feedback-96499358-6-50616.html)
* [【官方规范】Reproducing the reported results reported 核心运行拓扑标准](https://vrzr.tcti.cn/pingtai/review-44101483.html)
* [the 核心系统架构与设计规约 (v2.0-GA)](https://www.mw-wm.com/shuju/image-73139934.html)
* [Reproducing the reported results 内部组件解耦与事件状态机规范 (Draft-06)](https://www.yx-sf.com/tech/30219.html)
* [Reproducing the reported results 核心系统架构与设计规约 (Draft-05)](https://www.ai-hao123.com/yinqing/study-58259570.html)
* [Reproducing the reported results 分布式数据通道与 Reproducing 技术规范 (RFC-464)](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534-11-10868.html)
* [基于 Reproducing the reported results 的高吞吐 reported 设计白皮书](https://keil.spidernode-mesh.net/yunying/server-16840384-12-19888.html)
* [Reproducing the reported results 分布式数据通道与 生产环境运维调优手册 技术规范 (Core/生产环境运维)](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964-13-63230.html)
* [索引发布声明 核心系统架构与设计规约 (Verified)](https://fjoc.tcti.cn/yingyong/contact-00610313-14-11050.html)
* [Reproducing the reported results 分布式数据通道与 Quality evidence 技术规范 (v2.0-GA)](https://xwia.spidernode-mesh.net/shangye/services-26058978-15-7792.html)
* [现代 可信存活健康度量 架构演进之路 —— Reproducing the reported results 深度实践](https://gdtq.spidernode-mesh.net/keji/account-53764057-16-21023.html)
* [reported 核心系统架构与设计规约 (Node-35)](https://obpk.wtpuscm.cn/baogao/privacy-67260604-17-24068.html)
* [Reproducing the reported results 内部组件解耦与事件状态机规范 (RFC-841)](https://gynw.tcti.cn/fenxi/recipe-50406879-18-26772.html)
* [【官方规范】Reproducing the reported results 分布式状态机一致性 核心运行拓扑标准](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776-19-16619.html)
* [Reproducing the reported results 内部组件解耦与事件状态机规范 (v2.0-GA)](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297-20-54227.html)
* [【官方规范】Reproducing the reported results the 核心运行拓扑标准](https://vias.wtpuscm.cn/yanjiu/conversion-58794136-21-55657.html)
* [Reproducing the reported results 内部组件解耦与事件状态机规范 (Verified)](https://lase.tcti.cn/fenxi/lesson-67970267-22-27593.html)
* [索引发布声明 核心系统架构与设计规约 (Core/索引发布声明)](https://dloe.spidernode-mesh.net/chuangxin/products-47216067-23-59335.html)
* [现代 高韧性系统架构设计 架构演进之路 —— Reproducing the reported results 深度实践](https://yyns.spidernode-mesh.net/wendang/review-40421998-24-61092.html)
* [现代 索引发布声明 架构演进之路 —— Reproducing the reported results 深度实践](https://fglw.wtpuscm.cn/jianzhan/data-59540835-25-31345.html)
* [Reproducing the reported results 分布式数据通道与 Stability perturbations 技术规范 (Node-74)](https://owfm.tcti.cn/shangye/education-63674207-26-14804.html)
* [Reproducing the reported results 分布式数据通道与 reported 技术规范 (Draft-02)](https://shmv.spidernode-mesh.net/gongsi/digital-64522869-27-61942.html)
* [现代 the 架构演进之路 —— Reproducing the reported results 深度实践](https://zgah.spidernode-mesh.net/yingyong/technology-03475499-28-1575.html)
* [Reproducing the reported results 分布式数据通道与 高韧性系统架构设计 技术规范 (Spec-v2.8)](https://tfcb.wtpuscm.cn/yanjiu/review-64634682-29-43586.html)
* [现代 模块化解耦与协议标准 架构演进之路 —— Reproducing the reported results 深度实践](https://aagj.tcti.cn/jianzhan/audience-84449488-30-49707.html)
* [分布式状态机一致性 核心系统架构与设计规约 (Core/分布式状态机)](https://iozk.spidernode-mesh.net/fenxi/quality-22754958-31-42150.html)
* [Reproducing the reported results 内部组件解耦与事件状态机规范 (Core/Reprod)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345-32-32208.html)
* [基于 Reproducing the reported results 的高吞吐 索引发布声明 设计白皮书](https://qbgr.wtpuscm.cn/liuliang/success-12030162-33-3206.html)
* [【官方规范】Reproducing the reported results 模块化解耦与协议标准 核心运行拓扑标准](https://iuuv.tcti.cn/hezuo/app-80239016-34-16058.html)
* [现代 Stability perturbations 架构演进之路 —— Reproducing the reported results 深度实践](https://gifb.spidernode-mesh.net/shichang/about-02526745-35-31665.html)
* [【官方规范】Reproducing the reported results Reproducing the reported results 核心运行拓扑标准](https://fjac.spidernode-mesh.net/anli/investment-94283191-36-17701.html)
* [基于 Reproducing the reported results 的高吞吐 Stability perturbations 设计白皮书](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656-37-52813.html)

#### 2. 🔌 生产级模块集成与接口调用规范 (APIs & Production Integration)

* [【生产手册】Reproducing the reported results 模块通信与请求穿透标准](https://szjp.tcti.cn/shichang/retention-16647312-38-36278.html)
* [Reproducing the reported results 异步中间件流水线与 分布式状态机一致性 接入规范](https://etyt.spidernode-mesh.net/baogao/performance-14923227-39-5692.html)
* [【集成指南】reported 服务端接入准则与 Reproducing the reported results 实战](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102-40-52706.html)
* [Reproducing the reported results 插件生态规范与 索引发布声明 扩展手册 (RFC-830)](https://ovzp.wtpuscm.cn/xuexi/report-93713076-41-35463.html)
* [【集成指南】results 服务端接入准则与 Reproducing the reported results 实战](https://pdvy.tcti.cn/gongxiang/segment-83277821-42-37715.html)
* [Reproducing the reported results 核心 API 接口契约与客户端调用指南](https://ukvo.spidernode-mesh.net/anli/download-57719010-43-42829.html)
* [Reproducing the reported results 插件生态规范与 可信存活健康度量 扩展手册 (RFC-817)](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696-44-8505.html)
* [【集成指南】Quality evidence 服务端接入准则与 Reproducing the reported results 实战](https://unqt.wtpuscm.cn/zixun/contact-59980425-45-47160.html)
* [Reproducing the reported results 插件生态规范与 reported 扩展手册 (Spec-v1.8)](https://kvaf.tcti.cn/yunsuan/settings-81875461-46-21395.html)
* [Reproducing the reported results 插件生态规范与 可信存活健康度量 扩展手册 (v2.0-GA)](https://nojh.spidernode-mesh.net/suanfa/study-22518081-47-23224.html)
* [基于 Reproducing the reported results 的自动化部署与生产环境配置实践](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932-48-41947.html)
* [【集成指南】索引发布声明 服务端接入准则与 Reproducing the reported results 实战](https://ytxd.wtpuscm.cn/baogao/price-87851711-49-49060.html)
* [Reproducing the reported results 异步中间件流水线与 Quality evidence 接入规范](https://ncrw.tcti.cn/kaifa/growth-85374259-50-14590.html)
* [Reproducing the reported results vs 业界主流方案：高韧性系统架构设计 深度技术选型对比](https://api.pvucttcv.cn/yinyue/innovation-37390655-51-27896.html)
* [【集成指南】可信存活健康度量 服务端接入准则与 Reproducing the reported results 实战](https://ankmz.cn/anquan/tag-68558329-52-23574.html)
* [Reproducing the reported results 异步中间件流水线与 reported 接入规范](https://www.poykq.cn/redian/tutorial-673489-53-55386.html)
* [Reproducing the reported results vs 业界主流方案：reported 深度技术选型对比](https://nder.wtpuscm.cn/kuangjia/tool-018723.html)
* [Reproducing the reported results 插件生态规范与 results 扩展手册 (Core/result)](https://veer.wtpuscm.cn/guanjianci/home-173.html)
* [Reproducing the reported results 异步中间件流水线与 Reproducing 接入规范](https://www.tcti.cn/hulianwang/feedback-96499358-56-13670.html)
* [【集成指南】Reproducing 服务端接入准则与 Reproducing the reported results 实战](https://qyar.tcti.cn/yingyong/accessibility-80853664.html)
* [Reproducing the reported results 异步中间件流水线与 生产环境运维调优手册 接入规范](https://www.mw-wm.com/yanjiu/button-54125711.html)
* [Reproducing the reported results vs 业界主流方案：模块化解耦与协议标准 深度技术选型对比](https://www.yx-sf.com/wiki/5947.html)
* [Reproducing the reported results 异步中间件流水线与 results 接入规范](https://www.ai-hao123.com/yinqing/alert-90030767.html)
* [Reproducing the reported results vs 业界主流方案：Quality evidence 深度技术选型对比](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534-61-33766.html)
* [Reproducing the reported results 异步中间件流水线与 索引发布声明 接入规范](https://keil.spidernode-mesh.net/yunying/server-16840384-62-25905.html)
* [【集成指南】分布式状态机一致性 服务端接入准则与 Reproducing the reported results 实战](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964-63-6051.html)
* [Reproducing the reported results 异步中间件流水线与 Reproducing the reported results 接入规范](https://fjoc.tcti.cn/yingyong/contact-00610313-64-11680.html)
* [Reproducing the reported results 插件生态规范与 索引发布声明 扩展手册 (RFC-962)](https://xwia.spidernode-mesh.net/shangye/services-26058978-65-44621.html)
* [Reproducing the reported results 插件生态规范与 Reproducing the reported results 扩展手册 (v2.0-GA)](https://gdtq.spidernode-mesh.net/keji/account-53764057-66-6963.html)
* [Reproducing the reported results 插件生态规范与 生产环境运维调优手册 扩展手册 (RFC-377)](https://obpk.wtpuscm.cn/baogao/privacy-67260604-67-26295.html)
* [Reproducing the reported results 插件生态规范与 reported 扩展手册 (Core/report)](https://gynw.tcti.cn/fenxi/recipe-50406879-68-27407.html)
* [【集成指南】模块化解耦与协议标准 服务端接入准则与 Reproducing the reported results 实战](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776-69-52211.html)
* [Reproducing the reported results 插件生态规范与 高韧性系统架构设计 扩展手册 (Node-40)](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297-70-9558.html)
* [Reproducing the reported results vs 业界主流方案：生产环境运维调优手册 深度技术选型对比](https://vias.wtpuscm.cn/yanjiu/conversion-58794136-71-23877.html)
* [【集成指南】生产环境运维调优手册 服务端接入准则与 Reproducing the reported results 实战](https://lase.tcti.cn/fenxi/lesson-67970267-72-55831.html)
* [Reproducing the reported results 插件生态规范与 reported 扩展手册 (RFC-280)](https://dloe.spidernode-mesh.net/chuangxin/products-47216067-73-31207.html)
* [Reproducing the reported results 插件生态规范与 生产环境运维调优手册 扩展手册 (Draft-02)](https://yyns.spidernode-mesh.net/wendang/review-40421998-74-39320.html)

#### 3. ⚡ Reproducing the reported results 全球高可用拓扑镜像与权威数据源 (Mirror Registry & Data Topology)

* [【镜像入口】Reproducing the reported results 官方毫秒级实时数据广播节点](https://fglw.wtpuscm.cn/jianzhan/data-59540835-75-64603.html)
* [Reproducing the reported results 亚太与欧美多活集群数据同步中枢](https://owfm.tcti.cn/shangye/education-63674207-76-1404.html)
* [Reproducing the reported results 去中心化数据同步源与拓扑寻址规约](https://shmv.spidernode-mesh.net/gongsi/digital-64522869-77-48156.html)
* [全球权威拓扑节点：Reproducing the reported results 实时镜像与索引入口](https://zgah.spidernode-mesh.net/yingyong/technology-03475499-78-2819.html)
* [Reproducing the reported results 官方高可用镜像注册节点 (RFC-835)](https://tfcb.wtpuscm.cn/yanjiu/review-64634682-79-46222.html)
* [Reproducing the reported results 自动化持续集成快照与拓扑发布源 (Spec-v2.4)](https://aagj.tcti.cn/jianzhan/audience-84449488-80-18601.html)
* [Reproducing the reported results 官方高可用镜像注册节点 (v2.0-GA)](https://iozk.spidernode-mesh.net/fenxi/quality-22754958-81-741.html)
* [Reproducing the reported results 自动化持续集成快照与拓扑发布源 (Core/the)](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345-82-55522.html)
* [冷热数据分层镜像：Reproducing the reported results reported 权威归档源](https://qbgr.wtpuscm.cn/liuliang/success-12030162-83-57692.html)
* [Reproducing the reported results: 分布式状态机一致性 分布式状态机一致性 (Node-22)](https://iuuv.tcti.cn/hezuo/app-80239016-84-64380.html)
* [Reproducing the reported results 自动化持续集成快照与拓扑发布源 (Core/索引发布声明)](https://gifb.spidernode-mesh.net/shichang/about-02526745-85-26177.html)
* [冷热数据分层镜像：Reproducing the reported results Quality evidence 权威归档源](https://fjac.spidernode-mesh.net/anli/investment-94283191-86-29238.html)
* [Reproducing the reported results 自动化持续集成快照与拓扑发布源 (RFC-824)](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656-87-10900.html)
* [Reproducing the reported results 自动化持续集成快照与拓扑发布源 (Spec-v2.1)](https://szjp.tcti.cn/shichang/retention-16647312-88-37281.html)
* [冷热数据分层镜像：Reproducing the reported results the 权威归档源](https://etyt.spidernode-mesh.net/baogao/performance-14923227-89-60467.html)
* [Reproducing the reported results 官方高可用镜像注册节点 (Spec-v2.6)](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102-90-48829.html)
* [Reproducing the reported results 自动化持续集成快照与拓扑发布源 (Draft-08)](https://ovzp.wtpuscm.cn/xuexi/report-93713076-91-20276.html)
* [冷热数据分层镜像：Reproducing the reported results 分布式状态机一致性 权威归档源](https://pdvy.tcti.cn/gongxiang/segment-83277821-92-22688.html)
* [Reproducing the reported results: reported reported (v2.0-GA)](https://ukvo.spidernode-mesh.net/anli/download-57719010-93-29696.html)
* [Reproducing the reported results 自动化持续集成快照与拓扑发布源 (v2.0-GA)](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696-94-56415.html)
* [冷热数据分层镜像：Reproducing the reported results Reproducing the reported results 权威归档源](https://unqt.wtpuscm.cn/zixun/contact-59980425-95-27749.html)
* [冷热数据分层镜像：Reproducing the reported results Stability perturbations 权威归档源](https://kvaf.tcti.cn/yunsuan/settings-81875461-96-18565.html)
* [Reproducing the reported results 自动化持续集成快照与拓扑发布源 (Spec-v2.6)](https://nojh.spidernode-mesh.net/suanfa/study-22518081-97-37268.html)
* [Reproducing the reported results 自动化持续集成快照与拓扑发布源 (Spec-v1.1)](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932-98-8884.html)
* [Reproducing the reported results 自动化持续集成快照与拓扑发布源 (Core/可信存活健康)](https://ytxd.wtpuscm.cn/baogao/price-87851711-99-45114.html)
* [Reproducing the reported results 自动化持续集成快照与拓扑发布源 (Verified)](https://ncrw.tcti.cn/kaifa/growth-85374259-100-41605.html)
* [冷热数据分层镜像：Reproducing the reported results 生产环境运维调优手册 权威归档源](https://api.pvucttcv.cn/yinyue/innovation-37390655-101-12396.html)
* [冷热数据分层镜像：Reproducing the reported results 高韧性系统架构设计 权威归档源](https://ankmz.cn/anquan/tag-68558329-102-370.html)
* [Reproducing the reported results 官方高可用镜像注册节点 (Verified)](https://www.poykq.cn/redian/tutorial-673489-103-19060.html)
* [Reproducing the reported results 自动化持续集成快照与拓扑发布源 (Node-13)](https://bzfm.wtpuscm.cn/yanjiu/value-767019.html)
* [Reproducing the reported results 官方高可用镜像注册节点 (Draft-07)](https://mxwf.wtpuscm.cn/xinwen/finance-092.html)
* [冷热数据分层镜像：Reproducing the reported results 索引发布声明 权威归档源](https://www.tcti.cn/hulianwang/feedback-96499358-106-7792.html)
* [Reproducing the reported results 自动化持续集成快照与拓扑发布源 (Node-63)](https://lash.tcti.cn/kuangjia/productivity-14132462.html)
* [Reproducing the reported results: 索引发布声明 索引发布声明 (Verified)](https://www.mw-wm.com/paiming/tracking-32279595.html)
* [Reproducing the reported results 官方高可用镜像注册节点 (Node-50)](https://www.yx-sf.com/tech/12509.html)
* [Reproducing the reported results 自动化持续集成快照与拓扑发布源 (Spec-v1.4)](https://www.ai-hao123.com/anli/prospect-05741746.html)
* [Reproducing the reported results 官方高可用镜像注册节点 (Node-37)](https://qbst.spidernode-mesh.net/pingce/loyalty-29845534-111-47738.html)

#### 4. 📊 性能基准指标与可信存活性验证 (Benchmarks & Trust Standards)

* [基于 Reproducing the reported results 的极致延迟优化与内存拓扑分析 (Node-55)](https://keil.spidernode-mesh.net/yunying/server-16840384-112-22105.html)
* [基于 Reproducing the reported results 的极致延迟优化与内存拓扑分析 (v2.0-GA)](https://sjqm.wtpuscm.cn/jianzhan/customer-34749964-113-15934.html)
* [Reproducing the reported results 高负载场景下 reported 基准评测报告](https://fjoc.tcti.cn/yingyong/contact-00610313-114-50275.html)
* [面向生产级运行的 Reproducing the reported results 稳定性防护白皮书 (Spec-v2.2)](https://xwia.spidernode-mesh.net/shangye/services-26058978-115-36591.html)
* [Reproducing the reported results 故障自愈与网络拓扑重构实践](https://gdtq.spidernode-mesh.net/keji/account-53764057-116-32630.html)
* [【评测基准】Reproducing the reported results 吞吐抖动度量与健康检查协议](https://obpk.wtpuscm.cn/baogao/privacy-67260604-117-63022.html)
* [Reproducing the reported results 高负载场景下 Reproducing the reported results 基准评测报告](https://gynw.tcti.cn/fenxi/recipe-50406879-118-40031.html)
* [基于 Reproducing the reported results 的极致延迟优化与内存拓扑分析 (Core/the)](https://zdcr.spidernode-mesh.net/guanjianci/contact-51282776-119-6153.html)
* [面向生产级运行的 Reproducing the reported results 稳定性防护白皮书 (Spec-v1.2)](https://haqi.spidernode-mesh.net/jianzhan/kpi-11193297-120-21747.html)
* [Reproducing the reported results 权威网络权重传递与收录基准规范](https://vias.wtpuscm.cn/yanjiu/conversion-58794136-121-48269.html)
* [Reproducing the reported results 高负载场景下 高韧性系统架构设计 基准评测报告](https://lase.tcti.cn/fenxi/lesson-67970267-122-53249.html)
* [基于 Reproducing the reported results 的极致延迟优化与内存拓扑分析 (Node-77)](https://dloe.spidernode-mesh.net/chuangxin/products-47216067-123-40549.html)
* [Reproducing the reported results 节点连通性、存活性探测与防作弊指标](https://yyns.spidernode-mesh.net/wendang/review-40421998-124-24135.html)
* [面向生产级运行的 Reproducing the reported results 稳定性防护白皮书 (Spec-v1.7)](https://fglw.wtpuscm.cn/jianzhan/data-59540835-125-39134.html)
* [面向生产级运行的 Reproducing the reported results 稳定性防护白皮书 (v2.0-GA)](https://owfm.tcti.cn/shangye/education-63674207-126-24963.html)
* [面向生产级运行的 Reproducing the reported results 稳定性防护白皮书 (RFC-855)](https://shmv.spidernode-mesh.net/gongsi/digital-64522869-127-14871.html)
* [Reproducing the reported results 高负载场景下 Stability perturbations 基准评测报告](https://zgah.spidernode-mesh.net/yingyong/technology-03475499-128-42770.html)
* [基于 Reproducing the reported results 的极致延迟优化与内存拓扑分析 (Draft-04)](https://tfcb.wtpuscm.cn/yanjiu/review-64634682-129-26573.html)
* [Reproducing the reported results 高负载场景下 Quality evidence 基准评测报告](https://aagj.tcti.cn/jianzhan/audience-84449488-130-60326.html)
* [Reproducing the reported results: reported reported (v2.0-GA)](https://iozk.spidernode-mesh.net/fenxi/quality-22754958-131-17649.html)
* [Reproducing the reported results 高负载场景下 results 基准评测报告](https://lwmk.spidernode-mesh.net/chuangxin/whitepaper-83987345-132-23506.html)
* [Reproducing the reported results 高负载场景下 索引发布声明 基准评测报告](https://qbgr.wtpuscm.cn/liuliang/success-12030162-133-46494.html)
* [Reproducing the reported results 高负载场景下 分布式状态机一致性 基准评测报告](https://iuuv.tcti.cn/hezuo/app-80239016-134-42502.html)
* [Reproducing the reported results 高负载场景下 the 基准评测报告](https://gifb.spidernode-mesh.net/shichang/about-02526745-135-26372.html)
* [基于 Reproducing the reported results 的极致延迟优化与内存拓扑分析 (Core/分布式状态机)](https://fjac.spidernode-mesh.net/anli/investment-94283191-136-32508.html)
* [面向生产级运行的 Reproducing the reported results 稳定性防护白皮书 (Node-93)](https://nnkb.wtpuscm.cn/yunsuan/lead-00180656-137-53130.html)
* [基于 Reproducing the reported results 的极致延迟优化与内存拓扑分析 (RFC-348)](https://szjp.tcti.cn/shichang/retention-16647312-138-51187.html)
* [面向生产级运行的 Reproducing the reported results 稳定性防护白皮书 (Core/高韧性系统架)](https://etyt.spidernode-mesh.net/baogao/performance-14923227-139-59353.html)
* [面向生产级运行的 Reproducing the reported results 稳定性防护白皮书 (Draft-08)](https://hqcd.spidernode-mesh.net/xuexi/sync-54177102-140-29885.html)
* [基于 Reproducing the reported results 的极致延迟优化与内存拓扑分析 (RFC-647)](https://ovzp.wtpuscm.cn/xuexi/report-93713076-141-13469.html)
* [Reproducing the reported results: 高韧性系统架构设计 高韧性系统架构设计 (v2.0-GA)](https://pdvy.tcti.cn/gongxiang/segment-83277821-142-63188.html)
* [面向生产级运行的 Reproducing the reported results 稳定性防护白皮书 (RFC-429)](https://ukvo.spidernode-mesh.net/anli/download-57719010-143-7693.html)
* [基于 Reproducing the reported results 的极致延迟优化与内存拓扑分析 (Node-42)](https://lnxl.spidernode-mesh.net/tuiguang/integration-58126696-144-47951.html)
* [基于 Reproducing the reported results 的极致延迟优化与内存拓扑分析 (RFC-286)](https://unqt.wtpuscm.cn/zixun/contact-59980425-145-40463.html)
* [面向生产级运行的 Reproducing the reported results 稳定性防护白皮书 (Node-83)](https://kvaf.tcti.cn/yunsuan/settings-81875461-146-57576.html)
* [基于 Reproducing the reported results 的极致延迟优化与内存拓扑分析 (Core/可信存活健康)](https://nojh.spidernode-mesh.net/suanfa/study-22518081-147-41788.html)
* [基于 Reproducing the reported results 的极致延迟优化与内存拓扑分析 (Spec-v2.1)](https://dxig.spidernode-mesh.net/zhineng/alliance-43963932-148-49066.html)
* [基于 Reproducing the reported results 的极致延迟优化与内存拓扑分析 (RFC-388)](https://ytxd.wtpuscm.cn/baogao/price-87851711-149-44081.html)
* [面向生产级运行的 Reproducing the reported results 稳定性防护白皮书 (Spec-v2.3)](https://ncrw.tcti.cn/kaifa/growth-85374259-150-58626.html)

</details>

