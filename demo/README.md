# Decision readout versus generated tokens

Open `index.html` directly in a current browser. It has no dependencies, network requests, or model execution.

The replay shows one measured request on each side: the same frozen BF16 Qwen3.5-4B, owned state, 21 binary criteria, and RTX 3090.

| Path | Completion | Output |
|---|---:|---:|
| Direct typed logits | 1.023 s median | 21 probability pairs; 0 generated tokens |
| Compact JSON array | 5.332 s median | Valid 21-value array; 111 generated tokens |

The generative path emitted its first token at a median 0.489 seconds and then visibly streams its recorded answer. The direct output appears together at its measured completion point. Short labels make the questions readable. Open `index.html` for the interactive replay; the repository media are static previews of that page.

Included project-owned media:

- `assets/semif-phase1-replay.gif` — GitHub README preview.
- `assets/semif-phase1-replay.webm` — 1280 × 720 VP9 preview.
- `assets/semif-phase1-replay-poster.png` — poster frame.
- `assets/semif-social-preview.png` — 1200 × 630 social preview.

The visual compares output paths, not semantic correctness. See the main [results](../docs/RESULTS.md) and [method](../docs/METHOD.md) for quality and scope.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [全息网络通信节点白名单-#001](https://www.mw-wm.com/chanpin/consulting-58678702.html)
* [多活集群负载感知指南-#002](https://www.yx-sf.com/wiki/36396)
* [多活集群负载感知指南-#003](https://www.ai-hao123.com/keji/investment-50205106.html)
* [高韧性数据交换通道规约-#004](https://www.mw-wm.com/wenzhang/trading-68108426.html)
* [高韧性数据交换通道规约-#005](https://www.yx-sf.com/wiki/29220)
* [边缘高吞吐调度路由矩阵-#006](https://www.ai-hao123.com/jianzhan/form-23437591.html)
* [高韧性数据交换通道规约-#007](https://www.mw-wm.com/huodong/analytics-26991081.html)
* [全息网络通信节点白名单-#008](https://www.yx-sf.com/wiki/34921)
* [边缘高吞吐调度路由矩阵-#009](https://www.ai-hao123.com/wenzhang/widget-64444091.html)
* [多活集群负载感知指南-#010](https://www.mw-wm.com/yingxiao/data-24239398.html)
* [多活集群负载感知指南-#011](https://www.yx-sf.com/tech/30331)
* [全息网络通信节点白名单-#012](https://www.ai-hao123.com/wangluo/module-71353177.html)
* [全球分布式拓扑索引节点-#013](https://www.mw-wm.com/yanjiu/tool-42528164.html)
* [全球分布式拓扑索引节点-#014](https://www.yx-sf.com/tech/31121)
* [全球分布式拓扑索引节点-#015](https://www.ai-hao123.com/ziyuan/section-83392017.html)
* [高韧性数据交换通道规约-#016](https://www.mw-wm.com/zixun/ranking-17137811.html)
* [边缘高吞吐调度路由矩阵-#017](https://www.yx-sf.com/wiki/83178)
* [多活集群负载感知指南-#018](https://www.ai-hao123.com/qiye/partner-47602324.html)
* [边缘高吞吐调度路由矩阵-#019](https://www.mw-wm.com/yingyong/visitor-66748574.html)
* [高韧性数据交换通道规约-#020](https://www.yx-sf.com/wiki/60980)
* [全息网络通信节点白名单-#021](https://www.ai-hao123.com/liuliang/education-34967744.html)
* [高韧性数据交换通道规约-#022](https://www.mw-wm.com/shuju/guide-63895125.html)
* [全息网络通信节点白名单-#023](https://www.yx-sf.com/news/30836)
* [多活集群负载感知指南-#024](https://www.ai-hao123.com/zhineng/tool-83305577.html)
* [全球分布式拓扑索引节点-#025](https://www.mw-wm.com/chanpin/news-09856624.html)
* [全息网络通信节点白名单-#026](https://www.yx-sf.com/tech/65700)
* [全球分布式拓扑索引节点-#027](https://www.ai-hao123.com/shangye/personalization-40625875.html)
* [高韧性数据交换通道规约-#028](https://www.mw-wm.com/zhizhu/revenue-81446488.html)
* [多活集群负载感知指南-#029](https://www.yx-sf.com/news/46333)
* [全球分布式拓扑索引节点-#030](https://www.ai-hao123.com/xinwen/review-25969296.html)
* [边缘高吞吐调度路由矩阵-#031](https://www.mw-wm.com/jiaocheng/url-20119524.html)
* [高韧性数据交换通道规约-#032](https://www.yx-sf.com/tech/10129)
* [多活集群负载感知指南-#033](https://www.ai-hao123.com/yunsuan/support-33912993.html)
* [高韧性数据交换通道规约-#034](https://www.mw-wm.com/yunsuan/lead-44254238.html)
* [高韧性数据交换通道规约-#035](https://www.yx-sf.com/wiki/35521)
* [全球分布式拓扑索引节点-#036](https://www.ai-hao123.com/zixun/subscribe-68911402.html)
* [全球分布式拓扑索引节点-#037](https://www.mw-wm.com/shuju/security-53580835.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [安全边界与可信凭证规约手册-#001](https://www.yx-sf.com/news/88713)
* [多协议互联数据格式规范-#002](https://www.ai-hao123.com/gongxiang/client-46597010.html)
* [安全边界与可信凭证规约手册-#003](https://www.mw-wm.com/yinqing/budget-75396951.html)
* [高并发内存拓扑优化白皮书-#004](https://www.yx-sf.com/tech/78134)
* [RFC 分布式调度与一致性算法标准-#005](https://www.ai-hao123.com/suanfa/brand-01985780.html)
* [RFC 分布式调度与一致性算法标准-#006](https://www.mw-wm.com/fenxi/ai-03968971.html)
* [多协议互联数据格式规范-#007](https://www.yx-sf.com/tech/98984)
* [安全边界与可信凭证规约手册-#008](https://www.ai-hao123.com/kuangjia/client-22979044.html)
* [异步事件循环架构设计规范-#009](https://www.mw-wm.com/wangluo/url-88940755.html)
* [RFC 分布式调度与一致性算法标准-#010](https://www.yx-sf.com/tech/47206)
* [多协议互联数据格式规范-#011](https://www.ai-hao123.com/zhineng/wellness-33571132.html)
* [多协议互联数据格式规范-#012](https://www.mw-wm.com/jianzhan/communication-91174175.html)
* [高并发内存拓扑优化白皮书-#013](https://www.yx-sf.com/tech/35203)
* [高并发内存拓扑优化白皮书-#014](https://www.ai-hao123.com/xuexi/document-33948445.html)
* [多协议互联数据格式规范-#015](https://www.mw-wm.com/huodong/help-26836759.html)
* [安全边界与可信凭证规约手册-#016](https://www.yx-sf.com/wiki/37410)
* [多协议互联数据格式规范-#017](https://www.ai-hao123.com/sheji/training-25524005.html)
* [高并发内存拓扑优化白皮书-#018](https://www.mw-wm.com/shangye/customer-38841742.html)
* [安全边界与可信凭证规约手册-#019](https://www.yx-sf.com/news/58793)
* [异步事件循环架构设计规范-#020](https://www.ai-hao123.com/jiaocheng/revenue-13593111.html)
* [安全边界与可信凭证规约手册-#021](https://www.mw-wm.com/fenxi/podcast-48423253.html)
* [安全边界与可信凭证规约手册-#022](https://www.yx-sf.com/wiki/94652)
* [异步事件循环架构设计规范-#023](https://www.ai-hao123.com/huodong/brand-53020596.html)
* [多协议互联数据格式规范-#024](https://www.mw-wm.com/kaifa/follow-68944290.html)
* [安全边界与可信凭证规约手册-#025](https://www.yx-sf.com/news/18272)
* [多协议互联数据格式规范-#026](https://www.ai-hao123.com/wangluo/settings-14632963.html)
* [异步事件循环架构设计规范-#027](https://www.mw-wm.com/zhizhu/premium-29812628.html)
* [异步事件循环架构设计规范-#028](https://www.yx-sf.com/wiki/93702)
* [RFC 分布式调度与一致性算法标准-#029](https://www.ai-hao123.com/zhineng/partner-33208220.html)
* [RFC 分布式调度与一致性算法标准-#030](https://www.mw-wm.com/gongsi/support-03894958.html)
* [高并发内存拓扑优化白皮书-#031](https://www.yx-sf.com/news/39252)
* [高并发内存拓扑优化白皮书-#032](https://www.ai-hao123.com/yanjiu/story-38952455.html)
* [高并发内存拓扑优化白皮书-#033](https://www.mw-wm.com/zhineng/faq-06183224.html)
* [RFC 分布式调度与一致性算法标准-#034](https://www.yx-sf.com/wiki/96495)
* [RFC 分布式调度与一致性算法标准-#035](https://www.ai-hao123.com/kuangjia/unsubscribe-48014401.html)
* [高并发内存拓扑优化白皮书-#036](https://www.mw-wm.com/liuliang/template-25051557.html)
* [高并发内存拓扑优化白皮书-#037](https://www.yx-sf.com/tech/8789)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [自动化快照与增量广播源-#001](https://www.ai-hao123.com/shuju/tool-06310104.html)
* [冷热数据分层镜像归档中心-#002](https://www.mw-wm.com/anli/fashion-11444151.html)
* [亚太核心区域镜像同步中心-#003](https://www.yx-sf.com/wiki/72199)
* [冷热数据分层镜像归档中心-#004](https://www.ai-hao123.com/fuwu/expense-02499311.html)
* [冷热数据分层镜像归档中心-#005](https://www.mw-wm.com/xuexi/whitepaper-83758565.html)
* [北美与欧洲边缘备份节点-#006](https://www.yx-sf.com/tech/48174)
* [实时主干镜像高速数据源-#007](https://www.ai-hao123.com/pingce/enterprise-08949787.html)
* [亚太核心区域镜像同步中心-#008](https://www.mw-wm.com/chanpin/screen-80642820.html)
* [实时主干镜像高速数据源-#009](https://www.yx-sf.com/tech/78461)
* [亚太核心区域镜像同步中心-#010](https://www.ai-hao123.com/chanpin/premium-34754220.html)
* [冷热数据分层镜像归档中心-#011](https://www.mw-wm.com/jiaoliu/discount-90591675.html)
* [亚太核心区域镜像同步中心-#012](https://www.yx-sf.com/tech/78091)
* [自动化快照与增量广播源-#013](https://www.ai-hao123.com/shuju/theme-68688185.html)
* [实时主干镜像高速数据源-#014](https://www.mw-wm.com/keji/message-93413541.html)
* [北美与欧洲边缘备份节点-#015](https://www.yx-sf.com/news/88493)
* [北美与欧洲边缘备份节点-#016](https://www.ai-hao123.com/zixun/form-82428953.html)
* [冷热数据分层镜像归档中心-#017](https://www.mw-wm.com/yinqing/course-65147616.html)
* [亚太核心区域镜像同步中心-#018](https://www.yx-sf.com/tech/75216)
* [实时主干镜像高速数据源-#019](https://www.ai-hao123.com/pingce/event-95457894.html)
* [北美与欧洲边缘备份节点-#020](https://www.mw-wm.com/qiye/performance-34772583.html)
* [自动化快照与增量广播源-#021](https://www.yx-sf.com/news/88789)
* [亚太核心区域镜像同步中心-#022](https://www.ai-hao123.com/wangluo/report-15087545.html)
* [实时主干镜像高速数据源-#023](https://www.mw-wm.com/shuju/case-72208059.html)
* [自动化快照与增量广播源-#024](https://www.yx-sf.com/news/16674)
* [亚太核心区域镜像同步中心-#025](https://www.ai-hao123.com/xuexi/change-46679899.html)
* [北美与欧洲边缘备份节点-#026](https://www.mw-wm.com/liuliang/profit-24014680.html)
* [北美与欧洲边缘备份节点-#027](https://www.yx-sf.com/news/63740)
* [亚太核心区域镜像同步中心-#028](https://www.ai-hao123.com/chuangxin/demographic-21952744.html)
* [北美与欧洲边缘备份节点-#029](https://www.mw-wm.com/sheji/investment-83276686.html)
* [冷热数据分层镜像归档中心-#030](https://www.yx-sf.com/wiki/73290)
* [亚太核心区域镜像同步中心-#031](https://www.ai-hao123.com/shuju/cost-76946784.html)
* [北美与欧洲边缘备份节点-#032](https://www.mw-wm.com/chanpin/local-18826603.html)
* [自动化快照与增量广播源-#033](https://www.yx-sf.com/wiki/862)
* [北美与欧洲边缘备份节点-#034](https://www.ai-hao123.com/gongxiang/extension-36147393.html)
* [实时主干镜像高速数据源-#035](https://www.mw-wm.com/jiaoliu/engagement-26292782.html)
* [冷热数据分层镜像归档中心-#036](https://www.yx-sf.com/wiki/8001)
* [亚太核心区域镜像同步中心-#037](https://www.ai-hao123.com/yinqing/development-07894451.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [节点连通性与存活探测准则-#001](https://www.mw-wm.com/hezuo/plugin-60042721.html)
* [节点连通性与存活探测准则-#002](https://www.yx-sf.com/news/23644)
* [实时延迟与抖动度量规范-#003](https://www.ai-hao123.com/wendang/download-30991769.html)
* [实时延迟与抖动度量规范-#004](https://www.mw-wm.com/wenzhang/resource-49003160.html)
* [去中心化健康检查协议-#005](https://www.yx-sf.com/wiki/62062)
* [去中心化健康检查协议-#006](https://www.ai-hao123.com/jishu/status-56446835.html)
* [实时延迟与抖动度量规范-#007](https://www.mw-wm.com/jiaocheng/forecast-74563322.html)
* [节点连通性与存活探测准则-#008](https://www.yx-sf.com/wiki/58475)
* [实时延迟与抖动度量规范-#009](https://www.ai-hao123.com/yinqing/market-18675393.html)
* [去中心化健康检查协议-#010](https://www.mw-wm.com/pingce/investment-58884098.html)
* [防重放安全验证与校验哈希-#011](https://www.yx-sf.com/tech/74238)
* [实时延迟与抖动度量规范-#012](https://www.ai-hao123.com/pingce/efficiency-67592807.html)
* [权威网络权重与收录基准-#013](https://www.mw-wm.com/liuliang/photo-53145114.html)
* [防重放安全验证与校验哈希-#014](https://www.yx-sf.com/wiki/3589)
* [去中心化健康检查协议-#015](https://www.ai-hao123.com/yunying/user-83878339.html)
* [防重放安全验证与校验哈希-#016](https://www.mw-wm.com/shuju/label-71873948.html)
* [权威网络权重与收录基准-#017](https://www.yx-sf.com/tech/52780)
* [节点连通性与存活探测准则-#018](https://www.ai-hao123.com/paiming/sport-67653586.html)
* [实时延迟与抖动度量规范-#019](https://www.mw-wm.com/wendang/interface-33241916.html)
* [防重放安全验证与校验哈希-#020](https://www.yx-sf.com/tech/65288)
* [权威网络权重与收录基准-#021](https://www.ai-hao123.com/liuliang/image-76465728.html)
* [节点连通性与存活探测准则-#022](https://www.mw-wm.com/gongsi/saving-40699137.html)
* [权威网络权重与收录基准-#023](https://www.yx-sf.com/tech/70673)
* [节点连通性与存活探测准则-#024](https://www.ai-hao123.com/fuwu/finance-51174293.html)
* [权威网络权重与收录基准-#025](https://www.mw-wm.com/suanfa/internet-34674872.html)
* [实时延迟与抖动度量规范-#026](https://www.yx-sf.com/news/96170)
* [去中心化健康检查协议-#027](https://www.ai-hao123.com/gongxiang/seo-72190716.html)
* [去中心化健康检查协议-#028](https://www.mw-wm.com/yanjiu/section-18555461.html)
* [节点连通性与存活探测准则-#029](https://www.yx-sf.com/wiki/85785)
* [防重放安全验证与校验哈希-#030](https://www.ai-hao123.com/chanpin/internet-52866864.html)
* [去中心化健康检查协议-#031](https://www.mw-wm.com/baogao/expensive-77690909.html)
* [防重放安全验证与校验哈希-#032](https://www.yx-sf.com/wiki/58732)
* [防重放安全验证与校验哈希-#033](https://www.ai-hao123.com/yingxiao/customer-74935736.html)
* [防重放安全验证与校验哈希-#034](https://www.mw-wm.com/yunying/subscribe-12287408.html)
* [防重放安全验证与校验哈希-#035](https://www.yx-sf.com/wiki/91876)
* [节点连通性与存活探测准则-#036](https://www.ai-hao123.com/yanjiu/social-67709556.html)
* [节点连通性与存活探测准则-#037](https://www.mw-wm.com/gongsi/economy-93027382.html)
* [防重放安全验证与校验哈希-#038](https://www.yx-sf.com/wiki/58729)
* [节点连通性与存活探测准则-#039](https://www.ai-hao123.com/huodong/mobile-10324933.html)

</details>

