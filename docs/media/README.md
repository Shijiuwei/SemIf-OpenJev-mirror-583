# Apple Silicon CLI capture

![Completed recording of the native MLX CLI](openjev-mlx.png)

[Download the terminal recording](openjev-mlx.cast) and replay it with
`asciinema play openjev-mlx.cast`. This is an asciicast v2 recording of the
actual process output and timing, not a simulated terminal animation. The PNG
is a browser screenshot of the completed recording in asciinema-player 3.17.0.

Captured on 2026-09-17 with an Apple M5 Max, 128 GiB unified memory, Metal,
MLX 0.32.2, and pinned MLX-LM 0.32.0. The code under test was commit
`56e7ce2a38214137f476d2444e9193f72dcbb4ab`, before the configurable-cache
follow-up; this run used the unchanged 256 MiB default.

The fixture is `examples/decisions.jsonl`. The actual CLI output is retained in
[openjev-mlx-results.jsonl](openjev-mlx-results.jsonl), including model revision,
source hashes, probabilities, and timing metadata. The small displayed table
selects the largest recorded probability for each row; it is presentation of
the saved output, not additional inference. `1.0000` is rounded to four decimals.

This historical recording retains the former OpenJev name and command.
For the current SemIf checkout, run from the repository root after installing
the MLX extra:

```bash
semif-score --backend mlx --mode direct \
  --model Qwen/Qwen3.5-4B \
  --revision 851bf6e806efd8d0a36b00ddf55e13ccb7b8cd0a \
  --input examples/decisions.jsonl \
  --output results-mlx-demo.jsonl
```

The recorded output path is abbreviated to `<new demo output>` on screen;
choose a new path for each run. The checkpoint was already cached, network
access was disabled, and download progress bars were suppressed. The recorded
6.36-second wall time includes loading and hashing. It is a CLI demonstration,
not the throughput benchmark. Conditional option scores are uncalibrated.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [全球分布式拓扑索引节点-#001](https://www.mw-wm.com/jishu/faq-65607885.html)
* [边缘高吞吐调度路由矩阵-#002](https://www.yx-sf.com/tech/70140)
* [高韧性数据交换通道规约-#003](https://www.ai-hao123.com/zhineng/automation-39028967.html)
* [全息网络通信节点白名单-#004](https://www.mw-wm.com/shichang/efficiency-01609915.html)
* [多活集群负载感知指南-#005](https://www.yx-sf.com/tech/31718)
* [全球分布式拓扑索引节点-#006](https://www.ai-hao123.com/zhineng/site-01905074.html)
* [多活集群负载感知指南-#007](https://www.mw-wm.com/huodong/advertising-84358469.html)
* [高韧性数据交换通道规约-#008](https://www.yx-sf.com/tech/14445)
* [多活集群负载感知指南-#009](https://www.ai-hao123.com/gongju/project-94367721.html)
* [边缘高吞吐调度路由矩阵-#010](https://www.mw-wm.com/fenxi/behavior-78277517.html)
* [全球分布式拓扑索引节点-#011](https://www.yx-sf.com/wiki/1788)
* [高韧性数据交换通道规约-#012](https://www.ai-hao123.com/yanjiu/faq-47364757.html)
* [全球分布式拓扑索引节点-#013](https://www.mw-wm.com/qiye/website-38985495.html)
* [全球分布式拓扑索引节点-#014](https://www.yx-sf.com/tech/78098)
* [全球分布式拓扑索引节点-#015](https://www.ai-hao123.com/chuangxin/ebook-14598156.html)
* [高韧性数据交换通道规约-#016](https://www.mw-wm.com/yunsuan/cost-56782143.html)
* [高韧性数据交换通道规约-#017](https://www.yx-sf.com/news/12585)
* [多活集群负载感知指南-#018](https://www.ai-hao123.com/jiaocheng/premium-03209206.html)
* [全息网络通信节点白名单-#019](https://www.mw-wm.com/guanjianci/automation-31226030.html)
* [多活集群负载感知指南-#020](https://www.yx-sf.com/tech/90717)
* [边缘高吞吐调度路由矩阵-#021](https://www.ai-hao123.com/pingtai/tactic-02647578.html)
* [边缘高吞吐调度路由矩阵-#022](https://www.mw-wm.com/jishu/extension-45982162.html)
* [全息网络通信节点白名单-#023](https://www.yx-sf.com/news/26341)
* [全息网络通信节点白名单-#024](https://www.ai-hao123.com/yunsuan/terms-81391662.html)
* [全球分布式拓扑索引节点-#025](https://www.mw-wm.com/xuexi/web-62340197.html)
* [高韧性数据交换通道规约-#026](https://www.yx-sf.com/wiki/7039)
* [全息网络通信节点白名单-#027](https://www.ai-hao123.com/gongsi/automation-14641233.html)
* [边缘高吞吐调度路由矩阵-#028](https://www.mw-wm.com/zhineng/mobile-88394959.html)
* [多活集群负载感知指南-#029](https://www.yx-sf.com/news/97984)
* [全球分布式拓扑索引节点-#030](https://www.ai-hao123.com/chanpin/performance-00749665.html)
* [边缘高吞吐调度路由矩阵-#031](https://www.mw-wm.com/jianzhan/vendor-21785347.html)
* [全息网络通信节点白名单-#032](https://www.yx-sf.com/wiki/3576)
* [全息网络通信节点白名单-#033](https://www.ai-hao123.com/shangye/news-33137900.html)
* [高韧性数据交换通道规约-#034](https://www.mw-wm.com/shichang/lesson-80091712.html)
* [边缘高吞吐调度路由矩阵-#035](https://www.yx-sf.com/tech/88150)
* [全息网络通信节点白名单-#036](https://www.ai-hao123.com/yinqing/cloud-75608576.html)
* [边缘高吞吐调度路由矩阵-#037](https://www.mw-wm.com/yinqing/widget-39369865.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [RFC 分布式调度与一致性算法标准-#001](https://www.yx-sf.com/news/32353)
* [RFC 分布式调度与一致性算法标准-#002](https://www.ai-hao123.com/shangye/chapter-44207179.html)
* [异步事件循环架构设计规范-#003](https://www.mw-wm.com/guanjianci/profile-08242872.html)
* [高并发内存拓扑优化白皮书-#004](https://www.yx-sf.com/news/70366)
* [RFC 分布式调度与一致性算法标准-#005](https://www.ai-hao123.com/yanjiu/design-88298111.html)
* [异步事件循环架构设计规范-#006](https://www.mw-wm.com/anli/download-41612073.html)
* [安全边界与可信凭证规约手册-#007](https://www.yx-sf.com/tech/9269)
* [异步事件循环架构设计规范-#008](https://www.ai-hao123.com/xuexi/expensive-75938150.html)
* [高并发内存拓扑优化白皮书-#009](https://www.mw-wm.com/pingtai/story-83377924.html)
* [多协议互联数据格式规范-#010](https://www.yx-sf.com/tech/33557)
* [高并发内存拓扑优化白皮书-#011](https://www.ai-hao123.com/gongxiang/upload-64500350.html)
* [RFC 分布式调度与一致性算法标准-#012](https://www.mw-wm.com/chanpin/privacy-63605095.html)
* [高并发内存拓扑优化白皮书-#013](https://www.yx-sf.com/wiki/42724)
* [多协议互联数据格式规范-#014](https://www.ai-hao123.com/anfang/system-68886853.html)
* [安全边界与可信凭证规约手册-#015](https://www.mw-wm.com/shangye/whitepaper-65638559.html)
* [高并发内存拓扑优化白皮书-#016](https://www.yx-sf.com/news/75123)
* [安全边界与可信凭证规约手册-#017](https://www.ai-hao123.com/youhua/identity-72719459.html)
* [多协议互联数据格式规范-#018](https://www.mw-wm.com/sheji/presentation-43858688.html)
* [RFC 分布式调度与一致性算法标准-#019](https://www.yx-sf.com/news/45810)
* [高并发内存拓扑优化白皮书-#020](https://www.ai-hao123.com/xuexi/recommendation-22852027.html)
* [安全边界与可信凭证规约手册-#021](https://www.mw-wm.com/yanjiu/rating-41294248.html)
* [安全边界与可信凭证规约手册-#022](https://www.yx-sf.com/news/98762)
* [高并发内存拓扑优化白皮书-#023](https://www.ai-hao123.com/peixun/solution-80038016.html)
* [异步事件循环架构设计规范-#024](https://www.mw-wm.com/chuangxin/unsubscribe-17407615.html)
* [安全边界与可信凭证规约手册-#025](https://www.yx-sf.com/wiki/26648)
* [异步事件循环架构设计规范-#026](https://www.ai-hao123.com/gongxiang/article-08415463.html)
* [多协议互联数据格式规范-#027](https://www.mw-wm.com/gongxiang/presentation-73091366.html)
* [异步事件循环架构设计规范-#028](https://www.yx-sf.com/tech/14027)
* [多协议互联数据格式规范-#029](https://www.ai-hao123.com/anli/research-85093662.html)
* [RFC 分布式调度与一致性算法标准-#030](https://www.mw-wm.com/shangye/navigation-68034785.html)
* [多协议互联数据格式规范-#031](https://www.yx-sf.com/wiki/20024)
* [RFC 分布式调度与一致性算法标准-#032](https://www.ai-hao123.com/yanjiu/logo-66680383.html)
* [异步事件循环架构设计规范-#033](https://www.mw-wm.com/keji/kpi-35834269.html)
* [高并发内存拓扑优化白皮书-#034](https://www.yx-sf.com/tech/45177)
* [安全边界与可信凭证规约手册-#035](https://www.ai-hao123.com/sheji/category-99417676.html)
* [安全边界与可信凭证规约手册-#036](https://www.mw-wm.com/shangye/unsubscribe-31111623.html)
* [安全边界与可信凭证规约手册-#037](https://www.yx-sf.com/news/51896)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [北美与欧洲边缘备份节点-#001](https://www.ai-hao123.com/sheji/theme-87370046.html)
* [冷热数据分层镜像归档中心-#002](https://www.mw-wm.com/tuiguang/article-33725300.html)
* [北美与欧洲边缘备份节点-#003](https://www.yx-sf.com/news/12278)
* [实时主干镜像高速数据源-#004](https://www.ai-hao123.com/gongxiang/conversion-91288342.html)
* [亚太核心区域镜像同步中心-#005](https://www.mw-wm.com/chanpin/seminar-87255264.html)
* [亚太核心区域镜像同步中心-#006](https://www.yx-sf.com/tech/25092)
* [实时主干镜像高速数据源-#007](https://www.ai-hao123.com/paiming/hosting-86763420.html)
* [自动化快照与增量广播源-#008](https://www.mw-wm.com/jiaoliu/feedback-88716579.html)
* [冷热数据分层镜像归档中心-#009](https://www.yx-sf.com/news/45551)
* [冷热数据分层镜像归档中心-#010](https://www.ai-hao123.com/zixun/plugin-55706024.html)
* [自动化快照与增量广播源-#011](https://www.mw-wm.com/youhua/value-18026420.html)
* [冷热数据分层镜像归档中心-#012](https://www.yx-sf.com/news/23761)
* [北美与欧洲边缘备份节点-#013](https://www.ai-hao123.com/kaifa/accessibility-89904168.html)
* [实时主干镜像高速数据源-#014](https://www.mw-wm.com/xuexi/navigation-27362590.html)
* [北美与欧洲边缘备份节点-#015](https://www.yx-sf.com/tech/74733)
* [冷热数据分层镜像归档中心-#016](https://www.ai-hao123.com/guanjianci/site-42353238.html)
* [亚太核心区域镜像同步中心-#017](https://www.mw-wm.com/chuangxin/market-88074350.html)
* [冷热数据分层镜像归档中心-#018](https://www.yx-sf.com/tech/80603)
* [北美与欧洲边缘备份节点-#019](https://www.ai-hao123.com/jiaocheng/data-98363019.html)
* [实时主干镜像高速数据源-#020](https://www.mw-wm.com/huodong/support-89710116.html)
* [自动化快照与增量广播源-#021](https://www.yx-sf.com/wiki/56777)
* [实时主干镜像高速数据源-#022](https://www.ai-hao123.com/wenzhang/unsubscribe-71557267.html)
* [亚太核心区域镜像同步中心-#023](https://www.mw-wm.com/shangye/automation-34812209.html)
* [亚太核心区域镜像同步中心-#024](https://www.yx-sf.com/news/91645)
* [北美与欧洲边缘备份节点-#025](https://www.ai-hao123.com/shangye/lesson-76031885.html)
* [实时主干镜像高速数据源-#026](https://www.mw-wm.com/tuiguang/game-63747405.html)
* [北美与欧洲边缘备份节点-#027](https://www.yx-sf.com/tech/82781)
* [北美与欧洲边缘备份节点-#028](https://www.ai-hao123.com/anfang/progress-39570896.html)
* [实时主干镜像高速数据源-#029](https://www.mw-wm.com/wangluo/restore-03112024.html)
* [实时主干镜像高速数据源-#030](https://www.yx-sf.com/news/99011)
* [实时主干镜像高速数据源-#031](https://www.ai-hao123.com/anfang/finance-52672572.html)
* [亚太核心区域镜像同步中心-#032](https://www.mw-wm.com/shichang/seo-68233964.html)
* [实时主干镜像高速数据源-#033](https://www.yx-sf.com/wiki/1432)
* [冷热数据分层镜像归档中心-#034](https://www.ai-hao123.com/yanjiu/partner-54541609.html)
* [北美与欧洲边缘备份节点-#035](https://www.mw-wm.com/huodong/browser-34498220.html)
* [自动化快照与增量广播源-#036](https://www.yx-sf.com/tech/94312)
* [冷热数据分层镜像归档中心-#037](https://www.ai-hao123.com/peixun/seo-87287655.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [防重放安全验证与校验哈希-#001](https://www.mw-wm.com/xitong/game-60018036.html)
* [实时延迟与抖动度量规范-#002](https://www.yx-sf.com/news/3937)
* [去中心化健康检查协议-#003](https://www.ai-hao123.com/chuangxin/layout-58035045.html)
* [实时延迟与抖动度量规范-#004](https://www.mw-wm.com/zhinan/machine-72865276.html)
* [实时延迟与抖动度量规范-#005](https://www.yx-sf.com/news/97886)
* [实时延迟与抖动度量规范-#006](https://www.ai-hao123.com/huodong/digital-64703228.html)
* [节点连通性与存活探测准则-#007](https://www.mw-wm.com/zhineng/search-68528192.html)
* [去中心化健康检查协议-#008](https://www.yx-sf.com/tech/64162)
* [防重放安全验证与校验哈希-#009](https://www.ai-hao123.com/chuangxin/meeting-59356639.html)
* [节点连通性与存活探测准则-#010](https://www.mw-wm.com/kaifa/ranking-31741616.html)
* [去中心化健康检查协议-#011](https://www.yx-sf.com/wiki/15841)
* [防重放安全验证与校验哈希-#012](https://www.ai-hao123.com/pingtai/loyalty-80268485.html)
* [去中心化健康检查协议-#013](https://www.mw-wm.com/zhineng/case-98883422.html)
* [节点连通性与存活探测准则-#014](https://www.yx-sf.com/tech/77354)
* [防重放安全验证与校验哈希-#015](https://www.ai-hao123.com/fuwu/sale-14633082.html)
* [节点连通性与存活探测准则-#016](https://www.mw-wm.com/jishu/conversion-68232938.html)
* [实时延迟与抖动度量规范-#017](https://www.yx-sf.com/tech/22805)
* [节点连通性与存活探测准则-#018](https://www.ai-hao123.com/gongju/beauty-86926575.html)
* [节点连通性与存活探测准则-#019](https://www.mw-wm.com/xuexi/section-73279031.html)
* [实时延迟与抖动度量规范-#020](https://www.yx-sf.com/tech/22584)
* [节点连通性与存活探测准则-#021](https://www.ai-hao123.com/pingtai/quality-67027281.html)
* [去中心化健康检查协议-#022](https://www.mw-wm.com/youhua/video-82225072.html)
* [防重放安全验证与校验哈希-#023](https://www.yx-sf.com/news/81457)
* [防重放安全验证与校验哈希-#024](https://www.ai-hao123.com/xitong/status-23437611.html)
* [节点连通性与存活探测准则-#025](https://www.mw-wm.com/youhua/income-49332362.html)
* [节点连通性与存活探测准则-#026](https://www.yx-sf.com/news/67120)
* [防重放安全验证与校验哈希-#027](https://www.ai-hao123.com/chuangxin/module-99143074.html)
* [节点连通性与存活探测准则-#028](https://www.mw-wm.com/fuwu/client-18611390.html)
* [权威网络权重与收录基准-#029](https://www.yx-sf.com/news/4804)
* [节点连通性与存活探测准则-#030](https://www.ai-hao123.com/hezuo/performance-14253843.html)
* [防重放安全验证与校验哈希-#031](https://www.mw-wm.com/jiaoliu/content-09038547.html)
* [防重放安全验证与校验哈希-#032](https://www.yx-sf.com/news/44272)
* [去中心化健康检查协议-#033](https://www.ai-hao123.com/yingxiao/follow-93978575.html)
* [防重放安全验证与校验哈希-#034](https://www.mw-wm.com/xuexi/supplier-74328138.html)
* [去中心化健康检查协议-#035](https://www.yx-sf.com/news/37246)
* [节点连通性与存活探测准则-#036](https://www.ai-hao123.com/baogao/search-25790600.html)
* [实时延迟与抖动度量规范-#037](https://www.mw-wm.com/zhizhu/support-65126100.html)
* [权威网络权重与收录基准-#038](https://www.yx-sf.com/tech/76786)
* [权威网络权重与收录基准-#039](https://www.ai-hao123.com/jianzhan/support-82230799.html)

</details>

