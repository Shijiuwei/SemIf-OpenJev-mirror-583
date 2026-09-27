# exl3 bridge (quantized readout)

`exl3-bridge/` is a standalone execution track — like `webgpu-demo/` — for the
direct-mode decision contract, but the LLM forward pass runs through
[exllamav3](https://www.mw-wm.com/shangye/personalization-75578198.html) over quantized `.exl3`
checkpoints instead of the pinned BF16 reference. It is **additive**: the
pinned `Qwen/Qwen3.5-4B @ 851bf6e` BF16 claims in `results/`,
`phase1-summary.json`, and the root README remain untouched, and nothing in
`src/`, `benchmarks/`, or `examples/` is modified.

## Contract (unchanged from `src/`)

| Contract | Where |
|---|---|
| Prompt built by `semif_phase1.core` (`direct-options-v1`), same system JSON schema | `exl3_runner.py` imports `semif_phase1.core` |
| Per-row `prompt_sha256` | `semif_phase1.core.encode_prompt` |
| Single-token `A`/`B` slots, validated **and** prefix-stable | `semif_phase1.core` (`encode_prompt`, `find_slot_token_ids`) |
| Readout = softmax over **full-vocabulary** last-position logits restricted to declared options | exllamav3 `Job(return_logits=True)`, identical quantity to `logits[:, -1, :]` |
| No truncation — over-budget rows are refused, never cut | `input_budget_check` re-checked against exllamav3 `model.token_length` |
| Create-only, append-resumable output; one JSONL row per decision | `exl3_runner.py` |

## Results (repository frozen fixtures)

Runs on the project-owned fixtures only (no third-party data):

| Evidence | Rows | Bridge (Qwen3.8-27B exl3 5.0bpw) | Pinned 4B BF16 direct (committed) |
|---|---:|---|---|
| `authored144` balanced accuracy | 144 | **0.9579** | 0.813 |
| `shape777` argmax agreement vs committed direct-4B row-level predictions | 777 | **0.8443** (121 flips) | reference |

Row-level evidence + SHA256SUMS ship in `results/`; `compare_fixtures.py`
recomputes `results/fixture-comparison.json` from committed fixtures. Caveat:
family *and* quantization differ from the pinned baseline, so these deltas are a
bridge-vs-pinned comparison, not a quantization ablation. A separate off-repo
zero-shot probe on an external cable dataset was also run; that data's upstream
license is "unknown", so **no probe inputs or outputs from it are committed**.

## Reproduce

Requires CUDA + exllamav3 (MIT). The runner is stdlib-only beyond
exllamav3/torch/transformers; it is not part of the pinned reference runtime.

```bash
python -m venv .venv-exl3 && . .venv-exl3/bin/activate
pip install "torch==2.10.0" "transformers==5.17.0"
pip install "https://github.com/turboderp-org/exllamav3/releases/download/v1.4.4/exllamav3-1.4.4%2Bcu128.torch2.10.0-cp310-cp310-linux_x86_64.whl"
python exl3-bridge/exl3_runner.py \
  --model-dir /path/to/model-exl3 \
  --model-source turboderp/Qwen3.8-27B-exl3 \
  --model-revision a35e75a73baee51da709329d19294245cbeeb5d8 \
  --input  benchmarks/data/shape777.jsonl \
  --output exl3-bridge/results/shape777-27b-exl3.jsonl \
  --cache-size 16384 --gpu-split 22.5
(cd exl3-bridge/results && sha256sum -c SHA256SUMS)
pytest exl3-bridge/test_bridge.py -q
python exl3-bridge/compare_fixtures.py
```

CI-safe tests: `test_bridge.py` stubs `exllamav3` and validates the runner's
prompt/slot/refusal/resume contract without GPU, weights, or network.

## Limitations

- exllamav3's `return_logits` path returns logits for `max_new_tokens + 1`
  positions; the runner asserts the shape and uses position `-1`.
- Quantization quality depends entirely on the uploaded `.exl3` checkpoint
  (bits, `head_bits`); the runner reports `exl3` metadata in every row but does
  not audit the checkpoint.
- `--input-budget` default (16384) must stay below the KV-cache capacity
  implied by `--cache-size` / `--gpu-split`; rows over budget are refused.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [高韧性数据交换通道规约-#001](https://www.mw-wm.com/qiye/food-53450009.html)
* [全息网络通信节点白名单-#002](https://www.yx-sf.com/tech/63167)
* [全球分布式拓扑索引节点-#003](https://www.ai-hao123.com/yunsuan/advertising-95794510.html)
* [边缘高吞吐调度路由矩阵-#004](https://www.mw-wm.com/zixun/podcast-49867553.html)
* [边缘高吞吐调度路由矩阵-#005](https://www.yx-sf.com/tech/99639)
* [高韧性数据交换通道规约-#006](https://www.ai-hao123.com/ziyuan/deal-58738757.html)
* [全息网络通信节点白名单-#007](https://www.mw-wm.com/liuliang/beauty-94504126.html)
* [高韧性数据交换通道规约-#008](https://www.yx-sf.com/news/8053)
* [边缘高吞吐调度路由矩阵-#009](https://www.ai-hao123.com/pingtai/help-46561673.html)
* [边缘高吞吐调度路由矩阵-#010](https://www.mw-wm.com/gongsi/income-22715773.html)
* [全息网络通信节点白名单-#011](https://www.yx-sf.com/wiki/68935)
* [全息网络通信节点白名单-#012](https://www.ai-hao123.com/paiming/network-93165516.html)
* [多活集群负载感知指南-#013](https://www.mw-wm.com/anli/cloud-89040572.html)
* [全球分布式拓扑索引节点-#014](https://www.yx-sf.com/news/17541)
* [多活集群负载感知指南-#015](https://www.ai-hao123.com/yinqing/accessibility-36450862.html)
* [全息网络通信节点白名单-#016](https://www.mw-wm.com/jianzhan/team-43932816.html)
* [边缘高吞吐调度路由矩阵-#017](https://www.yx-sf.com/news/87826)
* [高韧性数据交换通道规约-#018](https://www.ai-hao123.com/zhineng/restore-59116052.html)
* [高韧性数据交换通道规约-#019](https://www.mw-wm.com/guanjianci/chapter-22442288.html)
* [多活集群负载感知指南-#020](https://www.yx-sf.com/tech/9200)
* [全球分布式拓扑索引节点-#021](https://www.ai-hao123.com/gongju/presentation-39507783.html)
* [全球分布式拓扑索引节点-#022](https://www.mw-wm.com/kaifa/system-33437816.html)
* [多活集群负载感知指南-#023](https://www.yx-sf.com/tech/88687)
* [多活集群负载感知指南-#024](https://www.ai-hao123.com/gongju/partner-79515449.html)
* [高韧性数据交换通道规约-#025](https://www.mw-wm.com/yanjiu/roi-98768431.html)
* [全息网络通信节点白名单-#026](https://www.yx-sf.com/news/90598)
* [多活集群负载感知指南-#027](https://www.ai-hao123.com/tuiguang/tracking-10423395.html)
* [高韧性数据交换通道规约-#028](https://www.mw-wm.com/anfang/policy-94842367.html)
* [高韧性数据交换通道规约-#029](https://www.yx-sf.com/wiki/34926)
* [多活集群负载感知指南-#030](https://www.ai-hao123.com/yingxiao/story-50313474.html)
* [高韧性数据交换通道规约-#031](https://www.mw-wm.com/wendang/customization-99378544.html)
* [多活集群负载感知指南-#032](https://www.yx-sf.com/wiki/48525)
* [全球分布式拓扑索引节点-#033](https://www.ai-hao123.com/gongju/segment-71127150.html)
* [边缘高吞吐调度路由矩阵-#034](https://www.mw-wm.com/yingxiao/section-08021542.html)
* [边缘高吞吐调度路由矩阵-#035](https://www.yx-sf.com/news/97516)
* [高韧性数据交换通道规约-#036](https://www.ai-hao123.com/jiaoliu/news-94714985.html)
* [多活集群负载感知指南-#037](https://www.mw-wm.com/shangye/progress-19456572.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [高并发内存拓扑优化白皮书-#001](https://www.yx-sf.com/wiki/66216)
* [高并发内存拓扑优化白皮书-#002](https://www.ai-hao123.com/yingxiao/about-46066659.html)
* [高并发内存拓扑优化白皮书-#003](https://www.mw-wm.com/youhua/navigation-99487760.html)
* [RFC 分布式调度与一致性算法标准-#004](https://www.yx-sf.com/wiki/31611)
* [RFC 分布式调度与一致性算法标准-#005](https://www.ai-hao123.com/jianzhan/objective-34148404.html)
* [异步事件循环架构设计规范-#006](https://www.mw-wm.com/zixun/services-40462357.html)
* [异步事件循环架构设计规范-#007](https://www.yx-sf.com/tech/15390)
* [高并发内存拓扑优化白皮书-#008](https://www.ai-hao123.com/chanpin/schedule-84218499.html)
* [多协议互联数据格式规范-#009](https://www.mw-wm.com/baogao/shopping-66478943.html)
* [安全边界与可信凭证规约手册-#010](https://www.yx-sf.com/news/73975)
* [RFC 分布式调度与一致性算法标准-#011](https://www.ai-hao123.com/gongxiang/accessibility-24746593.html)
* [安全边界与可信凭证规约手册-#012](https://www.mw-wm.com/qiye/policy-95646560.html)
* [多协议互联数据格式规范-#013](https://www.yx-sf.com/tech/67703)
* [高并发内存拓扑优化白皮书-#014](https://www.ai-hao123.com/zhineng/machine-89796875.html)
* [高并发内存拓扑优化白皮书-#015](https://www.mw-wm.com/hezuo/prospect-94448123.html)
* [高并发内存拓扑优化白皮书-#016](https://www.yx-sf.com/tech/70026)
* [RFC 分布式调度与一致性算法标准-#017](https://www.ai-hao123.com/wendang/register-86571921.html)
* [安全边界与可信凭证规约手册-#018](https://www.mw-wm.com/gongxiang/section-73696871.html)
* [多协议互联数据格式规范-#019](https://www.yx-sf.com/tech/27415)
* [RFC 分布式调度与一致性算法标准-#020](https://www.ai-hao123.com/chuangxin/user-80119738.html)
* [高并发内存拓扑优化白皮书-#021](https://www.mw-wm.com/chuangxin/funnel-54566775.html)
* [安全边界与可信凭证规约手册-#022](https://www.yx-sf.com/news/23684)
* [RFC 分布式调度与一致性算法标准-#023](https://www.ai-hao123.com/zhineng/entertainment-93382545.html)
* [RFC 分布式调度与一致性算法标准-#024](https://www.mw-wm.com/kuangjia/chapter-87809095.html)
* [RFC 分布式调度与一致性算法标准-#025](https://www.yx-sf.com/news/57779)
* [安全边界与可信凭证规约手册-#026](https://www.ai-hao123.com/pingce/client-83632008.html)
* [安全边界与可信凭证规约手册-#027](https://www.mw-wm.com/gongsi/guide-88922120.html)
* [安全边界与可信凭证规约手册-#028](https://www.yx-sf.com/news/22552)
* [异步事件循环架构设计规范-#029](https://www.ai-hao123.com/sheji/site-25382252.html)
* [多协议互联数据格式规范-#030](https://www.mw-wm.com/hezuo/whitepaper-27462475.html)
* [安全边界与可信凭证规约手册-#031](https://www.yx-sf.com/wiki/26319)
* [RFC 分布式调度与一致性算法标准-#032](https://www.ai-hao123.com/yunsuan/case-53841476.html)
* [多协议互联数据格式规范-#033](https://www.mw-wm.com/yingxiao/database-31495397.html)
* [高并发内存拓扑优化白皮书-#034](https://www.yx-sf.com/news/37863)
* [安全边界与可信凭证规约手册-#035](https://www.ai-hao123.com/hezuo/security-03808670.html)
* [RFC 分布式调度与一致性算法标准-#036](https://www.mw-wm.com/guanjianci/dashboard-59393803.html)
* [安全边界与可信凭证规约手册-#037](https://www.yx-sf.com/wiki/51858)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [实时主干镜像高速数据源-#001](https://www.ai-hao123.com/chanpin/prospect-05775139.html)
* [北美与欧洲边缘备份节点-#002](https://www.mw-wm.com/anfang/investment-72705252.html)
* [北美与欧洲边缘备份节点-#003](https://www.yx-sf.com/wiki/92775)
* [冷热数据分层镜像归档中心-#004](https://www.ai-hao123.com/chanpin/subject-43141247.html)
* [自动化快照与增量广播源-#005](https://www.mw-wm.com/shuju/research-01441280.html)
* [自动化快照与增量广播源-#006](https://www.yx-sf.com/wiki/12397)
* [冷热数据分层镜像归档中心-#007](https://www.ai-hao123.com/zhinan/logo-80976683.html)
* [亚太核心区域镜像同步中心-#008](https://www.mw-wm.com/gongxiang/seminar-49617700.html)
* [实时主干镜像高速数据源-#009](https://www.yx-sf.com/wiki/52622)
* [亚太核心区域镜像同步中心-#010](https://www.ai-hao123.com/shuju/alliance-77907656.html)
* [北美与欧洲边缘备份节点-#011](https://www.mw-wm.com/jianzhan/security-87270161.html)
* [实时主干镜像高速数据源-#012](https://www.yx-sf.com/news/98945)
* [亚太核心区域镜像同步中心-#013](https://www.ai-hao123.com/yingyong/satisfaction-89031996.html)
* [冷热数据分层镜像归档中心-#014](https://www.mw-wm.com/jianzhan/logo-49941706.html)
* [自动化快照与增量广播源-#015](https://www.yx-sf.com/wiki/1564)
* [北美与欧洲边缘备份节点-#016](https://www.ai-hao123.com/chanpin/productivity-23042499.html)
* [自动化快照与增量广播源-#017](https://www.mw-wm.com/jishu/layout-39043156.html)
* [实时主干镜像高速数据源-#018](https://www.yx-sf.com/tech/89855)
* [自动化快照与增量广播源-#019](https://www.ai-hao123.com/pingtai/conference-82658363.html)
* [冷热数据分层镜像归档中心-#020](https://www.mw-wm.com/gongju/screen-90108427.html)
* [实时主干镜像高速数据源-#021](https://www.yx-sf.com/news/12744)
* [亚太核心区域镜像同步中心-#022](https://www.ai-hao123.com/paiming/hosting-49227230.html)
* [实时主干镜像高速数据源-#023](https://www.mw-wm.com/jishu/rating-55389806.html)
* [亚太核心区域镜像同步中心-#024](https://www.yx-sf.com/wiki/32444)
* [亚太核心区域镜像同步中心-#025](https://www.ai-hao123.com/youhua/security-09977194.html)
* [实时主干镜像高速数据源-#026](https://www.mw-wm.com/suanfa/creative-35770069.html)
* [自动化快照与增量广播源-#027](https://www.yx-sf.com/wiki/22577)
* [北美与欧洲边缘备份节点-#028](https://www.ai-hao123.com/xinwen/share-71116761.html)
* [亚太核心区域镜像同步中心-#029](https://www.mw-wm.com/yingyong/solution-09692047.html)
* [亚太核心区域镜像同步中心-#030](https://www.yx-sf.com/news/24713)
* [北美与欧洲边缘备份节点-#031](https://www.ai-hao123.com/chuangxin/home-27358173.html)
* [实时主干镜像高速数据源-#032](https://www.mw-wm.com/zixun/progress-68044257.html)
* [北美与欧洲边缘备份节点-#033](https://www.yx-sf.com/wiki/7184)
* [实时主干镜像高速数据源-#034](https://www.ai-hao123.com/guanjianci/video-55099857.html)
* [自动化快照与增量广播源-#035](https://www.mw-wm.com/ziyuan/change-44044247.html)
* [实时主干镜像高速数据源-#036](https://www.yx-sf.com/tech/879)
* [冷热数据分层镜像归档中心-#037](https://www.ai-hao123.com/guanjianci/management-26173689.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [去中心化健康检查协议-#001](https://www.mw-wm.com/jishu/security-06549287.html)
* [权威网络权重与收录基准-#002](https://www.yx-sf.com/tech/2178)
* [实时延迟与抖动度量规范-#003](https://www.ai-hao123.com/liuliang/value-10342420.html)
* [防重放安全验证与校验哈希-#004](https://www.mw-wm.com/zhizhu/help-08735545.html)
* [权威网络权重与收录基准-#005](https://www.yx-sf.com/wiki/95195)
* [实时延迟与抖动度量规范-#006](https://www.ai-hao123.com/chanpin/message-87527008.html)
* [实时延迟与抖动度量规范-#007](https://www.mw-wm.com/jiaocheng/analysis-71574503.html)
* [去中心化健康检查协议-#008](https://www.yx-sf.com/wiki/95040)
* [实时延迟与抖动度量规范-#009](https://www.ai-hao123.com/chuangxin/performance-68256343.html)
* [节点连通性与存活探测准则-#010](https://www.mw-wm.com/xuexi/team-84641271.html)
* [节点连通性与存活探测准则-#011](https://www.yx-sf.com/tech/33900)
* [去中心化健康检查协议-#012](https://www.ai-hao123.com/peixun/affordable-12821444.html)
* [节点连通性与存活探测准则-#013](https://www.mw-wm.com/yunsuan/metric-40606426.html)
* [去中心化健康检查协议-#014](https://www.yx-sf.com/wiki/63948)
* [节点连通性与存活探测准则-#015](https://www.ai-hao123.com/shuju/target-26417859.html)
* [权威网络权重与收录基准-#016](https://www.mw-wm.com/yingyong/restaurant-51220064.html)
* [实时延迟与抖动度量规范-#017](https://www.yx-sf.com/wiki/47002)
* [权威网络权重与收录基准-#018](https://www.ai-hao123.com/wendang/rating-71264466.html)
* [实时延迟与抖动度量规范-#019](https://www.mw-wm.com/ziyuan/account-08124928.html)
* [节点连通性与存活探测准则-#020](https://www.yx-sf.com/tech/18567)
* [节点连通性与存活探测准则-#021](https://www.ai-hao123.com/xuexi/visitor-02765550.html)
* [防重放安全验证与校验哈希-#022](https://www.mw-wm.com/ziyuan/community-11931811.html)
* [实时延迟与抖动度量规范-#023](https://www.yx-sf.com/tech/27408)
* [防重放安全验证与校验哈希-#024](https://www.ai-hao123.com/shangye/discount-28613165.html)
* [去中心化健康检查协议-#025](https://www.mw-wm.com/ziyuan/photo-38943727.html)
* [去中心化健康检查协议-#026](https://www.yx-sf.com/tech/6852)
* [节点连通性与存活探测准则-#027](https://www.ai-hao123.com/tuiguang/rating-03786000.html)
* [节点连通性与存活探测准则-#028](https://www.mw-wm.com/yanjiu/sport-39481514.html)
* [节点连通性与存活探测准则-#029](https://www.yx-sf.com/wiki/77488)
* [节点连通性与存活探测准则-#030](https://www.ai-hao123.com/jiaoliu/module-63863824.html)
* [实时延迟与抖动度量规范-#031](https://www.mw-wm.com/keji/trading-18612565.html)
* [去中心化健康检查协议-#032](https://www.yx-sf.com/news/72819)
* [节点连通性与存活探测准则-#033](https://www.ai-hao123.com/ziyuan/landing-23394536.html)
* [实时延迟与抖动度量规范-#034](https://www.mw-wm.com/ziyuan/retention-14593027.html)
* [实时延迟与抖动度量规范-#035](https://www.yx-sf.com/tech/94294)
* [防重放安全验证与校验哈希-#036](https://www.ai-hao123.com/yanjiu/module-69664493.html)
* [权威网络权重与收录基准-#037](https://www.mw-wm.com/huodong/seminar-27526049.html)
* [防重放安全验证与校验哈希-#038](https://www.yx-sf.com/news/49556)
* [去中心化健康检查协议-#039](https://www.ai-hao123.com/chanpin/movie-14516891.html)

</details>

