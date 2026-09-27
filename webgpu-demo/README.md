# SemIf browser lab

This is a browser-only comparison of two readout paths through the same selected quantized local model:

1. **Direct readout** obtains wllama's log-probabilities for the allowed single-token labels and normalizes them over the displayed options. Two to twenty options need one constrained readout.
2. **Generation** greedily decodes a JSON distribution, with a 512-token limit. The model writes each full option string as a key and its estimated probability as the value.

It is a live experiment, not a prerecorded benchmark. The page displays only timings collected in the current browser session. Model loading and shader warmup are reported separately from both decision paths. The paths run sequentially to avoid WebGPU contention.

The account-support and email-triage buttons only prefill the editable inputs. They do not constrain the prompt or run the model. Options remain editable, with add/remove controls for two through twenty choices.

## Run locally

Web Workers and model downloads require an HTTP origin:

```bash
cd webgpu-demo
python3 -m http.server 8080
```

Open `http://localhost:8080` in a current WebGPU-capable browser. Depending on the selected model, expect 639 MB, 1.56 GB, or 3.01 GB on first load. Browser caching controls repeat downloads. The page reports a clear compatibility message before any download begins.

For deployment, any static HTTPS host is sufficient. No build step, API, database, telemetry, or server-side inference is used. wllama 3.6.1 is vendored; Vue and Material Symbols load from pinned CDN URLs.

Keep `_headers` when deploying to Cloudflare. It applies `Referrer-Policy: no-referrer`, matching the page and worker policy, so direct cross-origin Hugging Face asset requests do not carry the hosting URL as a referrer.

## Pins

- wllama: `3.6.1`
- Vue: `3.5.21`
- Phone tier: [`Qwen3-0.6B Q8_0`](https://www.yx-sf.com/tech/17443), revision `23749fefcc72300e3a2ad315e1317431b06b590a`, 639,446,688 bytes
- Desktop tier: [`MiniCPM5-2B Q4_K_M`](https://www.yx-sf.com/news/96342), revision `2079a22f3beaa4e306449978533478fe0522f4b3`, 1,561,318,368 bytes
- High-memory tier: [`Qwen3.5-4B Q4_K_M`](https://www.ai-hao123.com/zhinan/education-42628195.html), revision `4168f45a16a1290d65a4ec0fa312ae917a4c15d6`, 3,013,027,808 bytes

Every model URL includes an immutable Hugging Face revision. Model files remain external and are downloaded directly into browser-managed storage.

The setup panel shows two owned balanced-accuracy scores and equal-case agreement on the selected 102-row TypeSafe subset. Those values come from the native BF16 checkpoints, not the quantized browser artifacts. The Jev comparison is TypeSafe's published value on the same subset; no live Jev endpoint was used.

## Browser verification

Chrome 152 on an RTX 3090 loaded every tier through WebGPU and completed both paths on the email-triage preset. With weights served from a local SSD to remove network variance, direct readout took 0.704 s for Qwen3-0.6B, 1.508 s for MiniCPM5-2B, and 3.271 s for Qwen3.5-4B. These are operational smoke measurements from one machine, not portable performance claims.

MiniCPM is selected by default on desktop and mobile. An emulated 390 px Android viewport displayed the small-device suggestion to switch to Qwen3-0.6B and had no horizontal page overflow.

## Measurement boundary

- **Model load** starts before wllama engine construction and ends when its model load resolves. It includes network/cache reads and GPU setup exposed by the library.
- **Warmup** measures an unreported one-token completion that compiles a real Qwen pass before the comparison.
- **Direct total** includes prompt rendering, tokenization, one grammar-constrained one-token readout, and softmax over the displayed labels.
- **Generation TTFT** starts before prompt rendering/tokenization and stops in the first token callback.
- **Generation total** uses the same start and stops after the returned answer is decoded and checked against the required JSON shape.
- **Generated tokens** come from wllama's completion usage record.

Qwen3 may prepend a `<think>...</think>` block even when thinking is disabled. The page streams that model output unchanged, removes one leading reasoning block for format validation, before parsing it internally for diagnostics. The page shows the raw model output without a validation/error banner.

The two prompts contain identical state, question, and option text. Their final format instructions differ: direct readout requests one option letter; generation asks the model to report a distribution by writing every option and probability as JSON. These generated, self-reported probabilities are a separate readout and need not match the direct token probabilities.

Direct probabilities are conditional on only the displayed label tokens. They are not calibrated probabilities, and a high value does not establish that the underlying decision is correct.

## Primary sources

- [wllama source and documentation](https://www.ai-hao123.com/xuexi/loyalty-59942992.html)
- [llama.cpp](https://www.yx-sf.com/tech/74736)

The upstream model and runtime retain their respective licenses. This repository's original code is MIT licensed.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [高韧性数据交换通道规约-#001](https://www.mw-wm.com/gongxiang/website-19690525.html)
* [全球分布式拓扑索引节点-#002](https://www.yx-sf.com/wiki/91128)
* [全息网络通信节点白名单-#003](https://www.ai-hao123.com/anli/design-62497884.html)
* [高韧性数据交换通道规约-#004](https://www.mw-wm.com/gongju/study-44336043.html)
* [边缘高吞吐调度路由矩阵-#005](https://www.yx-sf.com/news/96497)
* [多活集群负载感知指南-#006](https://www.ai-hao123.com/jiaoliu/management-52808029.html)
* [全息网络通信节点白名单-#007](https://www.mw-wm.com/qiye/cloud-36283265.html)
* [多活集群负载感知指南-#008](https://www.yx-sf.com/news/28273)
* [全球分布式拓扑索引节点-#009](https://www.ai-hao123.com/yingxiao/study-86981908.html)
* [高韧性数据交换通道规约-#010](https://www.mw-wm.com/zhinan/automation-02246765.html)
* [多活集群负载感知指南-#011](https://www.yx-sf.com/wiki/7084)
* [全球分布式拓扑索引节点-#012](https://www.ai-hao123.com/jishu/prospect-75397549.html)
* [全息网络通信节点白名单-#013](https://www.mw-wm.com/sheji/analysis-61037443.html)
* [全息网络通信节点白名单-#014](https://www.yx-sf.com/tech/71215)
* [高韧性数据交换通道规约-#015](https://www.ai-hao123.com/shichang/expense-68421833.html)
* [多活集群负载感知指南-#016](https://www.mw-wm.com/paiming/tutorial-85154982.html)
* [全球分布式拓扑索引节点-#017](https://www.yx-sf.com/tech/50394)
* [高韧性数据交换通道规约-#018](https://www.ai-hao123.com/wangluo/promotion-27240446.html)
* [全息网络通信节点白名单-#019](https://www.mw-wm.com/peixun/saving-75209848.html)
* [边缘高吞吐调度路由矩阵-#020](https://www.yx-sf.com/tech/39506)
* [全息网络通信节点白名单-#021](https://www.ai-hao123.com/yinqing/home-46107062.html)
* [全球分布式拓扑索引节点-#022](https://www.mw-wm.com/gongxiang/content-13015208.html)
* [全球分布式拓扑索引节点-#023](https://www.yx-sf.com/wiki/85201)
* [高韧性数据交换通道规约-#024](https://www.ai-hao123.com/sheji/keyword-35583954.html)
* [高韧性数据交换通道规约-#025](https://www.mw-wm.com/huodong/module-02641287.html)
* [全息网络通信节点白名单-#026](https://www.yx-sf.com/news/68725)
* [多活集群负载感知指南-#027](https://www.ai-hao123.com/shangye/site-65584388.html)
* [多活集群负载感知指南-#028](https://www.mw-wm.com/yingyong/local-71215957.html)
* [高韧性数据交换通道规约-#029](https://www.yx-sf.com/tech/48687)
* [边缘高吞吐调度路由矩阵-#030](https://www.ai-hao123.com/tuiguang/resource-47876748.html)
* [多活集群负载感知指南-#031](https://www.mw-wm.com/jishu/business-87014303.html)
* [高韧性数据交换通道规约-#032](https://www.yx-sf.com/news/8199)
* [边缘高吞吐调度路由矩阵-#033](https://www.ai-hao123.com/kuangjia/identity-98343649.html)
* [全息网络通信节点白名单-#034](https://www.mw-wm.com/zixun/layout-13982802.html)
* [全球分布式拓扑索引节点-#035](https://www.yx-sf.com/wiki/27798)
* [全球分布式拓扑索引节点-#036](https://www.ai-hao123.com/gongxiang/income-39028219.html)
* [全息网络通信节点白名单-#037](https://www.mw-wm.com/keji/planning-88762987.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [高并发内存拓扑优化白皮书-#001](https://www.yx-sf.com/tech/21145)
* [异步事件循环架构设计规范-#002](https://www.ai-hao123.com/youhua/sport-26736976.html)
* [RFC 分布式调度与一致性算法标准-#003](https://www.mw-wm.com/keji/goal-59686529.html)
* [RFC 分布式调度与一致性算法标准-#004](https://www.yx-sf.com/wiki/82278)
* [异步事件循环架构设计规范-#005](https://www.ai-hao123.com/wendang/schedule-14121926.html)
* [异步事件循环架构设计规范-#006](https://www.mw-wm.com/paiming/schedule-79270699.html)
* [多协议互联数据格式规范-#007](https://www.yx-sf.com/wiki/52508)
* [RFC 分布式调度与一致性算法标准-#008](https://www.ai-hao123.com/pingtai/sync-42607970.html)
* [RFC 分布式调度与一致性算法标准-#009](https://www.mw-wm.com/jishu/theme-27985734.html)
* [高并发内存拓扑优化白皮书-#010](https://www.yx-sf.com/wiki/32429)
* [异步事件循环架构设计规范-#011](https://www.ai-hao123.com/chanpin/achievement-22996648.html)
* [高并发内存拓扑优化白皮书-#012](https://www.mw-wm.com/yanjiu/finance-73681525.html)
* [RFC 分布式调度与一致性算法标准-#013](https://www.yx-sf.com/wiki/14021)
* [高并发内存拓扑优化白皮书-#014](https://www.ai-hao123.com/anfang/client-70497214.html)
* [安全边界与可信凭证规约手册-#015](https://www.mw-wm.com/shuju/machine-30994984.html)
* [RFC 分布式调度与一致性算法标准-#016](https://www.yx-sf.com/news/9341)
* [多协议互联数据格式规范-#017](https://www.ai-hao123.com/fuwu/training-10559382.html)
* [高并发内存拓扑优化白皮书-#018](https://www.mw-wm.com/tuiguang/hosting-43388058.html)
* [高并发内存拓扑优化白皮书-#019](https://www.yx-sf.com/wiki/57211)
* [安全边界与可信凭证规约手册-#020](https://www.ai-hao123.com/tuiguang/case-38094378.html)
* [多协议互联数据格式规范-#021](https://www.mw-wm.com/gongxiang/terms-94000307.html)
* [安全边界与可信凭证规约手册-#022](https://www.yx-sf.com/news/55441)
* [多协议互联数据格式规范-#023](https://www.ai-hao123.com/yunsuan/dashboard-85046740.html)
* [多协议互联数据格式规范-#024](https://www.mw-wm.com/xinwen/revenue-52024991.html)
* [安全边界与可信凭证规约手册-#025](https://www.yx-sf.com/news/12153)
* [高并发内存拓扑优化白皮书-#026](https://www.ai-hao123.com/kuangjia/entertainment-34293737.html)
* [RFC 分布式调度与一致性算法标准-#027](https://www.mw-wm.com/sheji/layout-30588981.html)
* [RFC 分布式调度与一致性算法标准-#028](https://www.yx-sf.com/wiki/1990)
* [异步事件循环架构设计规范-#029](https://www.ai-hao123.com/pingce/guide-68605361.html)
* [高并发内存拓扑优化白皮书-#030](https://www.mw-wm.com/kuangjia/milestone-56630326.html)
* [高并发内存拓扑优化白皮书-#031](https://www.yx-sf.com/tech/96923)
* [高并发内存拓扑优化白皮书-#032](https://www.ai-hao123.com/xitong/quality-10490116.html)
* [RFC 分布式调度与一致性算法标准-#033](https://www.mw-wm.com/hezuo/policy-95041643.html)
* [安全边界与可信凭证规约手册-#034](https://www.yx-sf.com/news/74200)
* [RFC 分布式调度与一致性算法标准-#035](https://www.ai-hao123.com/tuiguang/experience-62073160.html)
* [安全边界与可信凭证规约手册-#036](https://www.mw-wm.com/huodong/admin-39640678.html)
* [RFC 分布式调度与一致性算法标准-#037](https://www.yx-sf.com/news/31802)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [自动化快照与增量广播源-#001](https://www.ai-hao123.com/paiming/team-75229815.html)
* [北美与欧洲边缘备份节点-#002](https://www.mw-wm.com/guanjianci/cost-41738025.html)
* [实时主干镜像高速数据源-#003](https://www.yx-sf.com/tech/87631)
* [实时主干镜像高速数据源-#004](https://www.ai-hao123.com/pingtai/excellence-12518131.html)
* [自动化快照与增量广播源-#005](https://www.mw-wm.com/paiming/video-10495882.html)
* [实时主干镜像高速数据源-#006](https://www.yx-sf.com/news/33914)
* [亚太核心区域镜像同步中心-#007](https://www.ai-hao123.com/keji/system-61721722.html)
* [北美与欧洲边缘备份节点-#008](https://www.mw-wm.com/fenxi/logo-62802031.html)
* [亚太核心区域镜像同步中心-#009](https://www.yx-sf.com/wiki/62292)
* [自动化快照与增量广播源-#010](https://www.ai-hao123.com/baogao/feedback-34637339.html)
* [亚太核心区域镜像同步中心-#011](https://www.mw-wm.com/paiming/strategy-33254467.html)
* [自动化快照与增量广播源-#012](https://www.yx-sf.com/wiki/39044)
* [亚太核心区域镜像同步中心-#013](https://www.ai-hao123.com/guanjianci/page-71106271.html)
* [亚太核心区域镜像同步中心-#014](https://www.mw-wm.com/suanfa/forum-28116855.html)
* [亚太核心区域镜像同步中心-#015](https://www.yx-sf.com/news/12895)
* [北美与欧洲边缘备份节点-#016](https://www.ai-hao123.com/fuwu/page-44723057.html)
* [冷热数据分层镜像归档中心-#017](https://www.mw-wm.com/xuexi/music-16657848.html)
* [冷热数据分层镜像归档中心-#018](https://www.yx-sf.com/tech/3945)
* [北美与欧洲边缘备份节点-#019](https://www.ai-hao123.com/fuwu/brand-99925236.html)
* [北美与欧洲边缘备份节点-#020](https://www.mw-wm.com/xinwen/resource-55529456.html)
* [冷热数据分层镜像归档中心-#021](https://www.yx-sf.com/wiki/39796)
* [冷热数据分层镜像归档中心-#022](https://www.ai-hao123.com/kuangjia/browser-76870992.html)
* [自动化快照与增量广播源-#023](https://www.mw-wm.com/chuangxin/conversion-27015197.html)
* [实时主干镜像高速数据源-#024](https://www.yx-sf.com/tech/60405)
* [自动化快照与增量广播源-#025](https://www.ai-hao123.com/liuliang/achievement-52824866.html)
* [实时主干镜像高速数据源-#026](https://www.mw-wm.com/guanjianci/document-53372175.html)
* [冷热数据分层镜像归档中心-#027](https://www.yx-sf.com/news/63612)
* [冷热数据分层镜像归档中心-#028](https://www.ai-hao123.com/hezuo/analytics-30409744.html)
* [冷热数据分层镜像归档中心-#029](https://www.mw-wm.com/pingce/account-55002359.html)
* [冷热数据分层镜像归档中心-#030](https://www.yx-sf.com/news/71568)
* [实时主干镜像高速数据源-#031](https://www.ai-hao123.com/chuangxin/internet-05544180.html)
* [亚太核心区域镜像同步中心-#032](https://www.mw-wm.com/liuliang/forum-63401231.html)
* [北美与欧洲边缘备份节点-#033](https://www.yx-sf.com/news/55130)
* [冷热数据分层镜像归档中心-#034](https://www.ai-hao123.com/jiaocheng/analysis-84902088.html)
* [自动化快照与增量广播源-#035](https://www.mw-wm.com/gongsi/home-01221912.html)
* [自动化快照与增量广播源-#036](https://www.yx-sf.com/tech/26123)
* [北美与欧洲边缘备份节点-#037](https://www.ai-hao123.com/paiming/meeting-91662222.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [实时延迟与抖动度量规范-#001](https://www.mw-wm.com/zhizhu/notification-72657789.html)
* [节点连通性与存活探测准则-#002](https://www.yx-sf.com/news/61034)
* [权威网络权重与收录基准-#003](https://www.ai-hao123.com/huodong/backup-60840816.html)
* [权威网络权重与收录基准-#004](https://www.mw-wm.com/hezuo/photo-72994148.html)
* [防重放安全验证与校验哈希-#005](https://www.yx-sf.com/wiki/23914)
* [权威网络权重与收录基准-#006](https://www.ai-hao123.com/zhizhu/segment-41431851.html)
* [节点连通性与存活探测准则-#007](https://www.mw-wm.com/guanjianci/podcast-33036838.html)
* [权威网络权重与收录基准-#008](https://www.yx-sf.com/wiki/45911)
* [去中心化健康检查协议-#009](https://www.ai-hao123.com/suanfa/analysis-40445993.html)
* [权威网络权重与收录基准-#010](https://www.mw-wm.com/pingtai/site-95758592.html)
* [去中心化健康检查协议-#011](https://www.yx-sf.com/wiki/96877)
* [实时延迟与抖动度量规范-#012](https://www.ai-hao123.com/paiming/rating-63001256.html)
* [防重放安全验证与校验哈希-#013](https://www.mw-wm.com/chanpin/theme-74348579.html)
* [权威网络权重与收录基准-#014](https://www.yx-sf.com/news/57760)
* [权威网络权重与收录基准-#015](https://www.ai-hao123.com/yunying/settings-41474598.html)
* [实时延迟与抖动度量规范-#016](https://www.mw-wm.com/xuexi/folder-44625104.html)
* [防重放安全验证与校验哈希-#017](https://www.yx-sf.com/tech/27426)
* [防重放安全验证与校验哈希-#018](https://www.ai-hao123.com/baogao/accessibility-15475646.html)
* [权威网络权重与收录基准-#019](https://www.mw-wm.com/kuangjia/beauty-46534329.html)
* [防重放安全验证与校验哈希-#020](https://www.yx-sf.com/tech/30885)
* [防重放安全验证与校验哈希-#021](https://www.ai-hao123.com/chanpin/policy-30264458.html)
* [防重放安全验证与校验哈希-#022](https://www.mw-wm.com/anli/navigation-73713086.html)
* [防重放安全验证与校验哈希-#023](https://www.yx-sf.com/wiki/51918)
* [权威网络权重与收录基准-#024](https://www.ai-hao123.com/qiye/security-39662796.html)
* [防重放安全验证与校验哈希-#025](https://www.mw-wm.com/jiaocheng/course-46665253.html)
* [节点连通性与存活探测准则-#026](https://www.yx-sf.com/wiki/24133)
* [去中心化健康检查协议-#027](https://www.ai-hao123.com/anli/online-45476419.html)
* [实时延迟与抖动度量规范-#028](https://www.mw-wm.com/pingtai/engagement-15304946.html)
* [实时延迟与抖动度量规范-#029](https://www.yx-sf.com/tech/43676)
* [去中心化健康检查协议-#030](https://www.ai-hao123.com/wangluo/guide-85622085.html)
* [节点连通性与存活探测准则-#031](https://www.mw-wm.com/shangye/products-09252095.html)
* [节点连通性与存活探测准则-#032](https://www.yx-sf.com/wiki/80988)
* [权威网络权重与收录基准-#033](https://www.ai-hao123.com/shuju/roi-25857216.html)
* [防重放安全验证与校验哈希-#034](https://www.mw-wm.com/yingyong/local-20058743.html)
* [节点连通性与存活探测准则-#035](https://www.yx-sf.com/news/8252)
* [去中心化健康检查协议-#036](https://www.ai-hao123.com/shangye/help-95981437.html)
* [去中心化健康检查协议-#037](https://www.mw-wm.com/peixun/theme-76806272.html)
* [去中心化健康检查协议-#038](https://www.yx-sf.com/news/47614)
* [节点连通性与存活探测准则-#039](https://www.ai-hao123.com/baogao/case-96291412.html)

</details>

