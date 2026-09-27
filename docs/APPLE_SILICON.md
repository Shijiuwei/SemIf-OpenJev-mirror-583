# Apple Silicon

SemIf runs on Apple Silicon through two backends. The PyTorch backend executes on the
MPS GPU; the native MLX backend runs the same pinned Qwen3.5-4B checkpoint on Metal.
See [MLX.md](MLX.md) for MLX install, quantization options, and benchmark evidence.
Published CUDA numbers in `results/` are unaffected: both Apple backends are additive.

## Install

```bash
python -m venv .venv
. .venv/bin/activate
pip install -e '.[test]'
# MLX backend (Apple Silicon only):
pip install -e '.[test,mlx]'
```

## Run

```bash
# PyTorch/MPS:
semif-score --mode direct --device mps \
  --model Qwen/Qwen3.5-4B --revision 851bf6e806efd8d0a36b00ddf55e13ccb7b8cd0a \
  --input examples/decisions.jsonl --output results-mps-direct.jsonl

# MLX (direct/serial/shared; reranker is CUDA-only):
semif-score --backend mlx --mode direct \
  --model Qwen/Qwen3.5-4B --revision 851bf6e806efd8d0a36b00ddf55e13ccb7b8cd0a \
  --input examples/decisions.jsonl --output results-mlx-direct.jsonl
```

## Behavior and guarantees

- Both backends preserve the prompt contracts: identical chat template, token
  boundaries, and prompt hashes; scores carry the same uncalibrated-probability
  warnings as CUDA output.
- Shared mode on both Apple backends prefills once, then forwards independent
  batch-1 suffixes rather than a parallel batch. CUDA keeps its batched path.
- Timing fields are device-synchronized on both backends. Shared timing separates
  prompt encoding, prefix prefill, cache replication/copying, and suffix forwards.
- The reranker and published CUDA benchmark runners remain CUDA-only. Use the
  `semif-score` commands above for Apple Silicon.

## Measured on an M5 (24 GB), pinned Qwen3.5-4B, BF16

Measurements are illustrative, not committed benchmarks; they were taken 2026-09-17
and are hardware- and version-sensitive.

| Path | PyTorch MPS | MLX |
|---|---:|---:|
| Warm direct decision (~140-token prompt) | ~0.6 s | ~0.26 s |
| First forward (shader compilation) | ~8 s | ~7 s |
| One group from the 37×21 fixture (one prefill + 21 suffixes) | 16.4 s | 9.1 s |
| Peak allocation during direct scoring | — | ~8.6 GB |

Known limitations: PyTorch MPS falls back to reference kernels for Qwen3.5's
hybrid attention (`causal_conv1d`, `flash-linear-attention` are CUDA-only), so MPS
should not be compared directly against the committed RTX 3090 numbers. MLX
quantization and accuracy trade-offs are documented in [MLX.md](MLX.md).


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [多活集群负载感知指南-#001](https://www.mw-wm.com/jiaocheng/conference-79966742.html)
* [多活集群负载感知指南-#002](https://www.yx-sf.com/tech/42836)
* [全息网络通信节点白名单-#003](https://www.ai-hao123.com/guanjianci/satisfaction-75194031.html)
* [多活集群负载感知指南-#004](https://www.mw-wm.com/gongxiang/share-58843783.html)
* [边缘高吞吐调度路由矩阵-#005](https://www.yx-sf.com/tech/3062)
* [多活集群负载感知指南-#006](https://www.ai-hao123.com/gongxiang/tag-51939031.html)
* [边缘高吞吐调度路由矩阵-#007](https://www.mw-wm.com/yinqing/market-79176422.html)
* [边缘高吞吐调度路由矩阵-#008](https://www.yx-sf.com/tech/64403)
* [全息网络通信节点白名单-#009](https://www.ai-hao123.com/anfang/template-49669921.html)
* [多活集群负载感知指南-#010](https://www.mw-wm.com/suanfa/design-62391495.html)
* [全球分布式拓扑索引节点-#011](https://www.yx-sf.com/wiki/86096)
* [多活集群负载感知指南-#012](https://www.ai-hao123.com/xuexi/web-87941659.html)
* [边缘高吞吐调度路由矩阵-#013](https://www.mw-wm.com/keji/social-02424152.html)
* [全息网络通信节点白名单-#014](https://www.yx-sf.com/wiki/35390)
* [全球分布式拓扑索引节点-#015](https://www.ai-hao123.com/paiming/blog-04829914.html)
* [高韧性数据交换通道规约-#016](https://www.mw-wm.com/kuangjia/milestone-45987090.html)
* [多活集群负载感知指南-#017](https://www.yx-sf.com/tech/82629)
* [多活集群负载感知指南-#018](https://www.ai-hao123.com/sheji/policy-96867085.html)
* [多活集群负载感知指南-#019](https://www.mw-wm.com/keji/conversion-75119204.html)
* [全球分布式拓扑索引节点-#020](https://www.yx-sf.com/tech/10970)
* [全球分布式拓扑索引节点-#021](https://www.ai-hao123.com/wangluo/feedback-35674211.html)
* [高韧性数据交换通道规约-#022](https://www.mw-wm.com/zhizhu/campaign-78159925.html)
* [边缘高吞吐调度路由矩阵-#023](https://www.yx-sf.com/tech/29038)
* [边缘高吞吐调度路由矩阵-#024](https://www.ai-hao123.com/baogao/search-11089088.html)
* [边缘高吞吐调度路由矩阵-#025](https://www.mw-wm.com/zhinan/whitepaper-44663577.html)
* [全球分布式拓扑索引节点-#026](https://www.yx-sf.com/news/15012)
* [边缘高吞吐调度路由矩阵-#027](https://www.ai-hao123.com/wangluo/like-33619298.html)
* [多活集群负载感知指南-#028](https://www.mw-wm.com/anfang/support-50265658.html)
* [多活集群负载感知指南-#029](https://www.yx-sf.com/tech/81103)
* [高韧性数据交换通道规约-#030](https://www.ai-hao123.com/yingxiao/message-48366581.html)
* [全球分布式拓扑索引节点-#031](https://www.mw-wm.com/gongxiang/accessibility-40220381.html)
* [全球分布式拓扑索引节点-#032](https://www.yx-sf.com/wiki/52813)
* [多活集群负载感知指南-#033](https://www.ai-hao123.com/zhinan/income-14624032.html)
* [高韧性数据交换通道规约-#034](https://www.mw-wm.com/peixun/luxury-42520437.html)
* [高韧性数据交换通道规约-#035](https://www.yx-sf.com/wiki/80897)
* [全息网络通信节点白名单-#036](https://www.ai-hao123.com/kaifa/terms-92357531.html)
* [全球分布式拓扑索引节点-#037](https://www.mw-wm.com/sheji/vacation-71716573.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [安全边界与可信凭证规约手册-#001](https://www.yx-sf.com/news/52982)
* [异步事件循环架构设计规范-#002](https://www.ai-hao123.com/gongxiang/presentation-25064721.html)
* [RFC 分布式调度与一致性算法标准-#003](https://www.mw-wm.com/anli/contact-10833841.html)
* [多协议互联数据格式规范-#004](https://www.yx-sf.com/news/92935)
* [RFC 分布式调度与一致性算法标准-#005](https://www.ai-hao123.com/gongsi/upload-11912145.html)
* [高并发内存拓扑优化白皮书-#006](https://www.mw-wm.com/anli/content-96892273.html)
* [高并发内存拓扑优化白皮书-#007](https://www.yx-sf.com/news/74902)
* [多协议互联数据格式规范-#008](https://www.ai-hao123.com/gongju/collaborate-05278731.html)
* [高并发内存拓扑优化白皮书-#009](https://www.mw-wm.com/pingtai/case-31975628.html)
* [异步事件循环架构设计规范-#010](https://www.yx-sf.com/news/16285)
* [高并发内存拓扑优化白皮书-#011](https://www.ai-hao123.com/zhizhu/content-04744019.html)
* [RFC 分布式调度与一致性算法标准-#012](https://www.mw-wm.com/pingtai/saving-88217525.html)
* [RFC 分布式调度与一致性算法标准-#013](https://www.yx-sf.com/tech/58628)
* [安全边界与可信凭证规约手册-#014](https://www.ai-hao123.com/yanjiu/automation-66602279.html)
* [RFC 分布式调度与一致性算法标准-#015](https://www.mw-wm.com/suanfa/api-19524942.html)
* [多协议互联数据格式规范-#016](https://www.yx-sf.com/tech/91601)
* [异步事件循环架构设计规范-#017](https://www.ai-hao123.com/pingtai/folder-05187693.html)
* [多协议互联数据格式规范-#018](https://www.mw-wm.com/pingtai/news-49275637.html)
* [高并发内存拓扑优化白皮书-#019](https://www.yx-sf.com/wiki/69753)
* [安全边界与可信凭证规约手册-#020](https://www.ai-hao123.com/tuiguang/education-58110583.html)
* [安全边界与可信凭证规约手册-#021](https://www.mw-wm.com/yingxiao/food-16959314.html)
* [多协议互联数据格式规范-#022](https://www.yx-sf.com/tech/11982)
* [多协议互联数据格式规范-#023](https://www.ai-hao123.com/pingtai/calculator-07176286.html)
* [高并发内存拓扑优化白皮书-#024](https://www.mw-wm.com/shangye/shopping-03846563.html)
* [多协议互联数据格式规范-#025](https://www.yx-sf.com/news/93951)
* [高并发内存拓扑优化白皮书-#026](https://www.ai-hao123.com/wendang/media-45072792.html)
* [RFC 分布式调度与一致性算法标准-#027](https://www.mw-wm.com/keji/case-06269176.html)
* [RFC 分布式调度与一致性算法标准-#028](https://www.yx-sf.com/wiki/68456)
* [安全边界与可信凭证规约手册-#029](https://www.ai-hao123.com/fuwu/data-78755958.html)
* [异步事件循环架构设计规范-#030](https://www.mw-wm.com/baogao/software-01628435.html)
* [多协议互联数据格式规范-#031](https://www.yx-sf.com/wiki/75332)
* [异步事件循环架构设计规范-#032](https://www.ai-hao123.com/anfang/growth-24568971.html)
* [异步事件循环架构设计规范-#033](https://www.mw-wm.com/gongsi/consulting-01789486.html)
* [高并发内存拓扑优化白皮书-#034](https://www.yx-sf.com/wiki/82183)
* [RFC 分布式调度与一致性算法标准-#035](https://www.ai-hao123.com/xuexi/button-99801466.html)
* [多协议互联数据格式规范-#036](https://www.mw-wm.com/jianzhan/economy-54782368.html)
* [异步事件循环架构设计规范-#037](https://www.yx-sf.com/wiki/7001)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [冷热数据分层镜像归档中心-#001](https://www.ai-hao123.com/pingce/subject-19426658.html)
* [北美与欧洲边缘备份节点-#002](https://www.mw-wm.com/kaifa/article-59216362.html)
* [冷热数据分层镜像归档中心-#003](https://www.yx-sf.com/wiki/32013)
* [实时主干镜像高速数据源-#004](https://www.ai-hao123.com/xinwen/analysis-76364890.html)
* [北美与欧洲边缘备份节点-#005](https://www.mw-wm.com/anli/user-13067661.html)
* [北美与欧洲边缘备份节点-#006](https://www.yx-sf.com/tech/36778)
* [亚太核心区域镜像同步中心-#007](https://www.ai-hao123.com/kuangjia/terms-14440402.html)
* [实时主干镜像高速数据源-#008](https://www.mw-wm.com/gongju/revenue-61790526.html)
* [实时主干镜像高速数据源-#009](https://www.yx-sf.com/tech/82920)
* [亚太核心区域镜像同步中心-#010](https://www.ai-hao123.com/pingtai/seo-39173240.html)
* [北美与欧洲边缘备份节点-#011](https://www.mw-wm.com/youhua/dashboard-51067133.html)
* [冷热数据分层镜像归档中心-#012](https://www.yx-sf.com/wiki/1498)
* [北美与欧洲边缘备份节点-#013](https://www.ai-hao123.com/paiming/communication-43042153.html)
* [北美与欧洲边缘备份节点-#014](https://www.mw-wm.com/hezuo/extension-24809015.html)
* [自动化快照与增量广播源-#015](https://www.yx-sf.com/tech/33530)
* [冷热数据分层镜像归档中心-#016](https://www.ai-hao123.com/jiaoliu/profile-77947087.html)
* [北美与欧洲边缘备份节点-#017](https://www.mw-wm.com/chuangxin/food-20054290.html)
* [冷热数据分层镜像归档中心-#018](https://www.yx-sf.com/wiki/21693)
* [自动化快照与增量广播源-#019](https://www.ai-hao123.com/zhinan/price-32185699.html)
* [北美与欧洲边缘备份节点-#020](https://www.mw-wm.com/chuangxin/training-79608268.html)
* [自动化快照与增量广播源-#021](https://www.yx-sf.com/news/61697)
* [实时主干镜像高速数据源-#022](https://www.ai-hao123.com/yunying/management-76524970.html)
* [实时主干镜像高速数据源-#023](https://www.mw-wm.com/jiaocheng/efficiency-63028847.html)
* [亚太核心区域镜像同步中心-#024](https://www.yx-sf.com/wiki/74860)
* [亚太核心区域镜像同步中心-#025](https://www.ai-hao123.com/yanjiu/growth-43017284.html)
* [自动化快照与增量广播源-#026](https://www.mw-wm.com/jishu/objective-11042427.html)
* [亚太核心区域镜像同步中心-#027](https://www.yx-sf.com/wiki/31645)
* [实时主干镜像高速数据源-#028](https://www.ai-hao123.com/baogao/home-11783304.html)
* [冷热数据分层镜像归档中心-#029](https://www.mw-wm.com/xinwen/upload-09444944.html)
* [实时主干镜像高速数据源-#030](https://www.yx-sf.com/news/69501)
* [实时主干镜像高速数据源-#031](https://www.ai-hao123.com/liuliang/discovery-58865364.html)
* [自动化快照与增量广播源-#032](https://www.mw-wm.com/liuliang/business-97359592.html)
* [实时主干镜像高速数据源-#033](https://www.yx-sf.com/wiki/14837)
* [亚太核心区域镜像同步中心-#034](https://www.ai-hao123.com/zhinan/notification-60127690.html)
* [亚太核心区域镜像同步中心-#035](https://www.mw-wm.com/jiaoliu/section-02015225.html)
* [自动化快照与增量广播源-#036](https://www.yx-sf.com/news/52912)
* [亚太核心区域镜像同步中心-#037](https://www.ai-hao123.com/chanpin/forecast-99916307.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [权威网络权重与收录基准-#001](https://www.mw-wm.com/zhizhu/roi-74164878.html)
* [节点连通性与存活探测准则-#002](https://www.yx-sf.com/tech/11408)
* [实时延迟与抖动度量规范-#003](https://www.ai-hao123.com/kuangjia/roi-07258638.html)
* [实时延迟与抖动度量规范-#004](https://www.mw-wm.com/xuexi/kpi-88530744.html)
* [权威网络权重与收录基准-#005](https://www.yx-sf.com/wiki/84970)
* [防重放安全验证与校验哈希-#006](https://www.ai-hao123.com/hezuo/responsive-80878234.html)
* [去中心化健康检查协议-#007](https://www.mw-wm.com/ziyuan/milestone-56612382.html)
* [防重放安全验证与校验哈希-#008](https://www.yx-sf.com/tech/80640)
* [防重放安全验证与校验哈希-#009](https://www.ai-hao123.com/sheji/cost-31018401.html)
* [权威网络权重与收录基准-#010](https://www.mw-wm.com/wendang/event-80189351.html)
* [实时延迟与抖动度量规范-#011](https://www.yx-sf.com/wiki/80961)
* [权威网络权重与收录基准-#012](https://www.ai-hao123.com/anfang/target-87718253.html)
* [实时延迟与抖动度量规范-#013](https://www.mw-wm.com/jishu/target-02698635.html)
* [节点连通性与存活探测准则-#014](https://www.yx-sf.com/tech/86035)
* [去中心化健康检查协议-#015](https://www.ai-hao123.com/tuiguang/site-26154321.html)
* [节点连通性与存活探测准则-#016](https://www.mw-wm.com/qiye/media-53912090.html)
* [防重放安全验证与校验哈希-#017](https://www.yx-sf.com/news/9762)
* [去中心化健康检查协议-#018](https://www.ai-hao123.com/zhinan/module-48603866.html)
* [防重放安全验证与校验哈希-#019](https://www.mw-wm.com/yunying/lesson-77598169.html)
* [防重放安全验证与校验哈希-#020](https://www.yx-sf.com/news/89798)
* [实时延迟与抖动度量规范-#021](https://www.ai-hao123.com/baogao/services-05364929.html)
* [节点连通性与存活探测准则-#022](https://www.mw-wm.com/jiaocheng/review-07488655.html)
* [去中心化健康检查协议-#023](https://www.yx-sf.com/wiki/22546)
* [节点连通性与存活探测准则-#024](https://www.ai-hao123.com/xuexi/business-06079938.html)
* [防重放安全验证与校验哈希-#025](https://www.mw-wm.com/zhinan/accessibility-31440384.html)
* [节点连通性与存活探测准则-#026](https://www.yx-sf.com/wiki/92144)
* [去中心化健康检查协议-#027](https://www.ai-hao123.com/ziyuan/alliance-72270474.html)
* [去中心化健康检查协议-#028](https://www.mw-wm.com/shichang/partner-34131629.html)
* [权威网络权重与收录基准-#029](https://www.yx-sf.com/news/57088)
* [防重放安全验证与校验哈希-#030](https://www.ai-hao123.com/gongju/premium-29472702.html)
* [去中心化健康检查协议-#031](https://www.mw-wm.com/wangluo/database-72208530.html)
* [节点连通性与存活探测准则-#032](https://www.yx-sf.com/tech/1937)
* [防重放安全验证与校验哈希-#033](https://www.ai-hao123.com/liuliang/restore-54632484.html)
* [权威网络权重与收录基准-#034](https://www.mw-wm.com/keji/trading-99966432.html)
* [实时延迟与抖动度量规范-#035](https://www.yx-sf.com/wiki/18464)
* [权威网络权重与收录基准-#036](https://www.ai-hao123.com/xitong/machine-42913157.html)
* [节点连通性与存活探测准则-#037](https://www.mw-wm.com/tuiguang/brand-79173618.html)
* [权威网络权重与收录基准-#038](https://www.yx-sf.com/news/3677)
* [实时延迟与抖动度量规范-#039](https://www.ai-hao123.com/shuju/recommendation-50881466.html)

</details>

