---
layout: default
title: 基于结构化推理的无向量检索框架 PageIndex 实用性深度评估报告
parent: 🤖 AI 实战笔记
nav_order: 5
---


# **基于结构化推理的无向量检索框架 PageIndex 实用性深度评估报告**

## **PageIndex 的技术起源与核心设计哲学**

检索增强生成（RAG）技术在处理大规模、异构化文档库时已成为企业级人工智能应用的核心架构。然而，传统的向量化检索方案在面对长篇幅、高密度且结构复杂的专业文档时，正暴露出明显的性能瓶颈1。针对这一痛点，由 VectifyAI 团队于2025年9月推出并开源的 PageIndex 框架，代表了一种完全背离传统向量相似度匹配的全新技术路线1。作为在 GitHub 上迅速积累了超过 23,000 颗星和近 2,000 次分叉的开源项目，PageIndex 提出了“无向量、基于推理”的检索新范式，试图解决传统检索中相似性不等于相关性的系统性矛盾1。

### **传统向量检索在长文档分析中的物理缺陷**

传统的 RAG 架构高度依赖分块、文本嵌入（Embedding）与向量数据库检索这一工具链2。在实际的工业级应用中，这种机制存在以下无法通过参数微调解决的固有缺陷7：

* **上下文碎片化**：将长文档硬性切割为固定大小的静态文本块，不可避免地会将紧密关联的上下文信息截断1。例如，一个跨越数页的复杂财务报表，其表头、注解和核心数据会被分散到不同的向量块中，导致检索结果彻底失去结构完整性3。这种现象在工程界常被称为“氛围检索（Vibe Retrieval）”，即寻找的是语义上感觉相关的碎片，而非逻辑上绝对精准的答案7。
* **语义相似性不等于逻辑相关性**：向量相似度匹配本质上是高维空间中的余弦距离计算，其寻找的是词汇语义相近的内容1。然而，专业分析人员的提问往往包含高度抽象的逻辑关系，这种逻辑关系在简单的向量空间中难以映射，从而导致检索准确率低下4。
* **跨章节引用盲区**：当文档正文中出现“详见附录 G”等显式跨章节交叉引用时，向量检索由于无法理解文档的逻辑脉络，无法顺藤摸瓜检索到附录中的具体内容，从而导致回答缺失或产生严重的模型幻觉5。

### **PageIndex 的无分块树状索引机制**

PageIndex 的核心技术灵感源自 AlphaGo 的蒙特卡洛树搜索（MCTS）机制1。AlphaGo 通过策略网络和价值网络在复杂的棋局博弈树中进行启发式搜索，而非穷举所有可能1。PageIndex 同样将一个长文档视为一个具有层级深度的决策树，模拟人类专家翻阅书籍和复杂报告时的检索行为4：

* **树状索引构建（ToC Generation）**：在文档摄入阶段，PageIndex 并不对其进行向量化或硬性切片，而是调用先进的文档解析与大语言模型（LLM）分析技术，自顶向下地重构出文档的天然层级树1。每一个树节点（Node）都封装了该章节的显式标题、高度概括的语义摘要、起始页码范围、子节点指针以及元数据1。
* **上下文内索引推理（In-Context Indexing）**：构建完成的轻量级 JSON 树状索引并不存放在外部向量数据库中，而是在生成和推理阶段直接加载到 LLM 的活动上下文窗口中1。这使得大模型能够以全局视角实时审视文档的整条逻辑骨架1。
* **主动式树状搜索检索**：当用户提交查询时，PageIndex 并不直接寻找最相似的文本段，而是引导大模型在 JSON 目录树上开展启发式推理1。大模型决定目标节点后，系统动态调取该节点覆盖的原始文本进行充分性评估1。若当前文本信息已足够解答，则立即终止搜索并生成答案；若不足，则大模型会根据上下文线索主动寻址并调取其他关联节点的信息，直至拼凑出完整的逻辑链条1。

## **实用性评估：技术集成、工具链与主流解析器对比**

在实际生产系统的部署中，PageIndex 并非孤立运行，其性能表现高度依赖于上游文档解析引擎的输出质量1。为了准确评估其技术实用性，有必要将其置于当前的文档解析与开发集成生态中进行横向审视9。

### **常见文档解析器的横向技术评估**

在目前的主流 RAG 工作流中，LlamaParse、Unstructured 和 Vectorize 是最常被提及的文档解析工具，它们在处理复杂文档排版时各有侧重10：

* **LlamaParse**：由 LlamaIndex 官方推出，专注于将复杂的 PDF、PowerPoint 及 Word 文档精准转化为结构化的 Markdown 格式11。LlamaParse 极佳的表格提取与层级保留能力，使其成为构建 PageIndex JSON 树状索引时最理想的上游数据源，能够使财务文档分析的准确率获得显著提升10。
* **Unstructured**：提供了极高的工作流灵活性，并与 LangChain 等生态深度集成，但在面对多栏排版或嵌套表格等极端复杂的版面时，容易出现 preprocessing 阶段的解析错乱，增加额外的清洗成本10。
* **Vectorize**：则专注于上下文嵌入的优化，在处理扫描件和图像干扰较多的文档时具备优势，但对文档宏观层级树的重构支持力度不及专门的结构化解析器10。

### **开发接口与代理工具的集成能力**

PageIndex 提供了极佳的开发者友好度，通过标准化的软件开发工具包（SDK）和模型上下文协议（MCP）工具，极大地降低了其融入现有 AI 生态的门槛3。

| 模式分类 | 接口方法 / 工具名称 | 主要功能与描述 | 实用性与系统集成价值 |
| :---- | :---- | :---- | :---- |
| **JS/TS SDK REST API** | submitDocument \[cite: 9\] | 上传并提交原始 PDF 或 Markdown 文档4 | 统一的入口点，自动触发底层的文档解析与树构建工作流4 |
|  | getTree \[cite: 9\] | 获取处理完毕的层级 JSON 树状结构3 | 允许开发团队在本地缓存树状索引，避免重复解析3 |
|  | chatCompletions \[cite: 9\] | 执行包含树状检索路径和源引用引用的对话生成8 | 兼容 OpenAI 风格的流式传输，支持元数据与引用的实时呈现8 |
| **MCP 代理工具集** | browseDocuments \[cite: 9\] | 基于时间或关联度对文档库进行全局统一发现9 | 允许自主 AI 代理在没有人工干预的情况下检索和筛选文档4 |
|  | getDocumentStructure \[cite: 9\] | 提取指定文档的大纲和层级结构目录9 | 帮助大模型在进行复杂推理前快速建立文档结构认知9 |
|  | getPageContent \[cite: 9\] | 读取并获取特定页码范围内的原始文本9 | 动态调取文本，最大程度减少上下文窗口的无效占用8 |
|  | getDocumentImage \[cite: 9\] | 提取文档中嵌入的图表、插图或视觉页面9 | 为多模态代理提供视觉分析支撑，适合图表密集的报告9 |

在本地部署与自动化脚本场景中，PageIndex 还配备了功能完备的命令行工具（CLI）8。开发者通过运行 run\_pageindex.py 并指定 \--pdf\_path 或 \--md\_path（支持通过 \# 标题层级解析 Markdown），即可自动调用后台的大语言模型（默认使用 gpt-4o-2024-11-20）进行智能 Table of Contents 构建，并就地生成同名的 \_pageindex.json 索引文件4。这种灵活的本地生成与云端 API 双轨制，使得企业在数据合规与开发便利性之间能够取得良好的平衡8。

## **真实应用案例深度剖析：SpaceX 2026年 S-1 招股书的跨领域穿透分析**

为了验证 PageIndex 在工业级生产环境下的真实实用性，本评估报告模拟引入了一个极具挑战性的真实商业案例：针对美国商业航天与人工智能巨头 SpaceX 于2026年5月20日向美国证券交易委员会（SEC）正式提交的 S-1 招股说明书进行跨领域穿透分析14。这份招股书披露了高达 ![][image1] 亿美元的融资意向与 ![][image2] 万亿美元的初始估值，创下了全球资本市场历史上的 IPO 纪录14。

然而，该文件的分析难度堪称行业“地狱级”：它不仅横跨商业火箭发射（Space 研发分部）、卫星互联网运营（Connectivity 联接分部，即 Starlink）以及于2026年2月刚刚并入的超大规模人工智能实体（AI 分部，即 xAI 与 X 平台）三大极度消耗资金且技术逻辑完全不同的板块，而且充斥着复杂的“共同控制实体合并”追溯会计报表、大规模的基础设施资本支出计划、繁重的政府国防合同（Starshield 星盾计划）以及错综复杂的关联交易16。

在此案例中，传统的向量 RAG 检索面临着毁灭性的失效，而 PageIndex 则通过其逻辑推理树展现出了精准的分析能力1。

### **挑战一：共同控制会计准则（Common-Control Accounting）下的报表追溯与历史转亏穿透**

在财务报表中，SpaceX 披露其 2025 年度的合并营收为 ![][image3] 亿美元，但同时录得高达 ![][image4] 亿美元的净亏损，而这一亏损趋势延续到了 2026 年一季度（单季净亏损达 ![][image5] 亿美元）16。然而，招股书中同样披露，SpaceX 在 2024 年度曾实现了 ![][image6] 亿美元的净利润16。这种财务表现的剧烈逆转背后隐藏着复杂的会计游戏：由于埃隆·马斯克同时控制着 SpaceX、xAI 和 X 平台，根据 GAAP 共同控制会计准则，S-1 招股书必须将这三家在2026年2月才正式完成合并的实体财务数据进行历史年度追溯合并16。

#### **传统向量 RAG 的失效机理**

分析人员提问：“SpaceX 是如何从 2024 年的盈利转为 2025 年的巨额亏损的？合并报表的历史追溯调整在此过程中扮演了什么角色？”16。传统向量 RAG 系统的向量数据库在匹配“亏损”、“盈利”等高频财务词汇时，会被正文表层关于“2025年录得 ![][image4] 亿美元净亏损，AI 分部录得 ![][image7] 亿美元运营损失”的显性陈述主导，优先召回这些表层数据块16。

然而，解释“共同控制追溯调整准则”的核心论据，往往隐藏在附录中极其晦涩且没有高频财务数字的会计政策声明章节中16。在向量空间中，这类条文的语义距离与具体的财务查询极远，因此极易被系统漏检索，导致生成的答案仅停留在“AI 板块亏损导致整体转亏”的肤浅层面，完全忽视了追溯合并这一决定性的会计准则调整7。

#### **PageIndex 的解析路径**

PageIndex 在摄入 S-1 文件后，构建了包含合并财务政策、分部报告、重大重组事件等层级的 JSON 推理树1。在接收到提问时，大模型在上下文内的 JSON 树状索引中进行全局扫视，敏锐地识别到解答此问题必须协同调取“Node\_S12: 业务分部业绩报告”和“Node\_F03: 合并财务报表附注 \- 共同控制合并会计政策”两个关键节点1。

通过这种跨节点的“顺藤摸瓜”式寻址，PageIndex 完美重构了以下财务真相，并以结构化形式呈现给分析人员16：

| SpaceX 核心业务分部 (2025 财年) | 分部营收规模 (亿美元) | 占比 (%) | 运营利润 / (亏损) (亿美元) | 关键财务与资本支出特征 |
| :---- | :---- | :---- | :---- | :---- |
| **Connectivity 联接分部 (Starlink)** | $113.916 | ![][image8] | $44.216 | 拥有 1030 万全球订阅用户的高利润 SaaS 模式现金牛16 |
| **Space 空间研发分部 (发射业务)** | $41.016 | ![][image9] | ($6.57)16 | 商业发射市占率达 90%，但超 ![][image10] 亿美元的 Starship 研发投入导致分部亏损16 |
| **AI 智能基建分部 (xAI & X 平台)** | $32.016 | ![][image11] | ($63.5)16 | Memphis COLOSSUS 超算中心等基建消耗了 $127 亿 capex16 |
| **综合合并调整 (GAAP 追溯后)** | **$186.7** \[cite: 16, 18\] | **100.0%** | **($49.4) (净亏损)** \[cite: 16, 18\] | 受共同控制合并影响，追溯合并导致历史报表利润被 AI 的高额赤字完全吞噬16 |

PageIndex 不仅准确列出了各分部的财务细节，更在其推理痕迹（Reasoning Trace）中清晰指明：2024 年度的 ![][image6] 亿美元盈利是属于 SpaceX 合并 xAI 之前的独立财务实体表现；在共同控制会计准则下，由于历史报表被强行并入了 xAI 处于早期的巨额研发开支，导致追溯调整后的 2024 和 2025 年度合并报表双双陷入巨额赤字16。

### **挑战二：技术与工程瓶颈的交叉印证——从 Starship 12 号飞行测试失败到 Starlink 提价决策的商业逻辑穿透**

在招股书的“风险因素”与“管理层讨论与分析”中，SpaceX 详细陈述了其新一代大型卫星 Starlink V3 的部署高度依赖于重型运载火箭 Starship 的运营 cadence 提升23。2026年5月22日，Starship 进行了备受瞩目的第十二次飞行测试（Flight 12），这也是配备了全新 Raptor 3 发动机、具有更大推进剂储箱的 Block 3 车型的首飞26。

然而，飞行测试结果极为惨烈：Booster 19 在热分离后发生异常翻转，导致 33 台 Raptor 3 引擎大面积熄火，最终在 landing burn 阶段仅有一台发动机点火成功，以 ![][image12] 公里/小时的高速撞击墨西哥湾坠毁；同时，Ship 39 在上升段同样损失了一台真空版 Raptor 3 引擎，不得不执行应急剖面调整才勉强在印度洋软着陆26。这一事件直接导致联邦航空管理局（FAA）宣布暂停后续 Flight 13 试验并强制启动事故调查，使得下一代 10,000 颗 Starlink V3 卫星的发射窗口大幅延后23。

与此工程困境相呼应的是，Starlink 的用户平均单客收入（ARPU）由 2023 年的 ![][image13] 美元急剧下滑至 2026 年一季度的 ![][image14] 美元，反映了其在价格敏感性地区（非洲、东南亚、拉丁美洲）疯狂扩张导致的单位经济效益恶化16。作为直接对冲手段，Starlink 在 2026 年 5 月和 6 月迅速祭出全球性提价大招，不仅上调了消费级计划的月租费，更是取消了新用户的硬件买断政策，强制推行每月 ![][image15] 美元的设备租赁费30。

#### **传统向量 RAG 的失效机理**

分析人员提问：“SpaceX 最新的重型火箭发射故障对 Starlink 2026 年中旬的全球价格调整及未来营收确定性产生了何种连锁反应？”23。

对于传统的向量 RAG 系统，这无异于一场灾难。因为“运载火箭发动机物理故障”与“卫星宽带资费方案调整”在人类分析师眼中是因果相扣的商业决策，但在向量空间中却是风马牛不相及的两个专业领域的文本7。系统无法通过余弦相似度将这两个高度不相似的文本块同时检索出来，最终 LLM 只能给出一个断章取义的回答，无法拼凑出“火箭故障 ![][image16] V3 satellite 部署受阻 ![][image16] 单卫网络容量瓶颈显现 ![][image16] ARPU 下滑压力加剧 ![][image16] 提价与设备租赁政策出台以转嫁研发成本”的因果链条16。

#### **PageIndex 的解析路径**

PageIndex 凭借其树搜索机制，能够通过主动式跨章节寻址完成完美的商业逻辑链穿透1：

* **寻址 Node\_S18 (Starship 开发进度与故障分析)**16：检索到 Flight 12 飞行事故的核心细节（Raptor 3 熄火与 FAA 禁飞事故调查），确认 V3 卫星由于单星重达数吨，无法使用 Falcon 9 发射，只能等待 Starship 复飞，因此面临至少半年的运力扩张停滞风险23。
* **寻址 Node\_C05 (Starlink 用户分布与单位经济分析)**16：获取 ARPU 从 ![][image13] 美元一路下滑至 ![][image14] 美元的事实，并了解到由于 V2 Mini 卫星单星带宽仅为 80 Gbps（而 V3 卫星可达 1 Tbps），在无法发射新卫星的情况下，现有网络带宽已接近饱和瓶颈，必须限制新用户加入并提升存量用户的单客收入23。
* **寻址 Node\_P02 (资费方案调整与硬件租赁政策)**30：获取2026年6月10日最新生效的资费表，确认资费调整的精确细节30。

PageIndex 能够将上述三个高度割裂的数据块无缝编织成一幅连贯的战略图谱，并以清晰的表格对照资费变化，直接向决策层揭示提价背后的技术胁迫逻辑30。

| 美国 Starlink 核心套餐方案 (2026 年中旬变更对照) | 2026年5月前月租费 (美元) | 2026年6月后新月租费 (美元) | 新增月度设备租金 (美元) | 战略目的与商业对冲逻辑 |
| :---- | :---- | :---- | :---- | :---- |
| **Residential 100 (低端家庭计划)** | $5034 | $5534 | $1030 | 针对基础设施滞后，通过租赁费降低首次加入门槛，同时保障长期现金流入30 |
| **Residential 200 (中端标准家庭)** | $8034 | $8534 | $1030 | 提升高饱和地区的单客收入（ARPU），变相进行网络流量控制23 |
| **Residential Max (高端旗舰服务)** | $12034 | $13034 | $1030 | 取消附赠 Mini 终端等免费福利，最大化榨取高价值存量用户的剩余价值31 |
| **Roam Unlimited (全球移动漫游)** | $16534 | $17534 | 暂不适用30 | 维持高净值移动房车用户的提价惯性，缓解核心网络骨干负载31 |

### **挑战三：法律、政治与合规性雷区——特权选择性法区、巨额不确定税收利益与地缘政治对冲**

除了财务与技术的纵深印证，SpaceX 招股书中还隐藏着极高的企业治理、法律合规与政治风险，这些风险在 PageIndex 的结构化穿透下同样无处遁形25：

* **Nevada 特权法区的设立避险**：为了规避特拉华州大法官法院（Delaware Chancery Court）在 *Tornetta v. Musk* 案中撤销马斯克 ![][image17] 亿美元巨额薪酬包时所展现的“全部公允性”严苛监管，SpaceX 巧妙地于 2026 年 1 月 21 日在内华达州设立了两个全新的控制实体来进行与 xAI 的合并25。内华达州法律（AB 239 修正案与 NRS § 78.138(7)）规定，除非原告股东能证明董事会存在欺诈或蓄意违法，否则董事会与控制股东享有极高的高管免责保护 presumption25。
* **$19 亿巨额不确定税收利益 (UTBs)**：招股书 S-1 极其低调地披露了 SpaceX 账面上累积了高达 ![][image18] **亿美元的不确定税收利益（Uncertain Tax Benefits）**37。这代表着该企业声称享受了免税，但其财务部门自知这些免税项目一旦面临国税局（IRS）的深度审计，极大概率会被驳回和重罚37。这一数据直接勾勒出马斯克设立政府效率部（DOGE）并猛烈推动 defund IRS（削减国税局审计经费）的底层核心利益关联37。
* **地缘政治与巨额政府卫星合同的对冲**：招股书显示，SpaceX 虽深受巨额亏损困扰，但其军工防火墙板块（Starshield 星盾计划）在 2026 年 5 月底密集斩获了美国国防部和太空军的重磅订单，包括 ![][image19] 亿美元的 SDN 太空数据网络骨干网合同及 ![][image20] 亿美元的 AMTI 空中移动目标指示星群合同19。同时，其向五角大楼开出的绕过伊朗政府黑网的 Direct-to-Cell 直连手机服务开出了首笔 ![][image21] 亿美元头款及月均 ![][image22] 亿美元的创纪录账单，充分展现了其在政治博弈中的议价资本39。
* **地缘竞争者的紧密逼近**：与此同时，亚马逊旗下的竞争网络 Amazon Leo（原 Project Kuiper）虽然因为运载火箭不可用和 prototype 重设计问题错过了 2026 年 7 月 30 日发射半数星座（1,618 颗）的 FCC 监管期限，并遭到 FCC 暂时 demote 频谱优先权的处罚，但亚马逊已于 2026 年 4 月火速收购了 Globalstar 以强化频谱资源，并在 6 月 17 日通过 Ariane 6 重型火箭大举发射了首批 upgraded P160C 固推卫星40。这场外轨道的巨头厮杀正严重威胁着 SpaceX 的通信护城河33。

通过将上述零散分布于招股书不同偏僻角落、总计长达数万字的非结构化段落进行树形逻辑链穿透，PageIndex 能够自动将上述政治、法律避险与商业合同的深层勾连进行整合生成，为企业战略决策层或二级市场投资人交付一份具有高度洞察力、杜绝泛泛而谈的可审计报告1。

## **生产环境落地的性能挑战与架构优化策略**

尽管 PageIndex 在解析极度复杂的长文档时展示了碾压传统向量检索的准确率与可解释性，但由于其高度依赖 LLM 的多轮主动寻址决策，在走向高并发、低延迟的实际生产环境时，开发团队必须直面其高延迟与高昂 API Token 费用的物理瓶颈1。

### **多级遍历延迟与成本控制**

在未经过深度调优的 PageIndex 部署中，一次对 200 页招股书的深度问答可能需要 LLM 进行多达 5 至 8 轮的 ToC JSON 树遍历、节点读取与内容 Sufficiency 判断1。这不仅会导致单次查询的端到端延迟推高至 **4 至 10 秒**，更会导致 API token 的消耗成本呈指数级上升，严重制约了其作为大规模在线服务的经济可行性1。

为了使 PageIndex 在企业级生产环境中真正具备商用其实用性，VectifyAI 团队和开源社区提出了一整套先进的性能调优与架构演进方案6：

       \[用户查询输入\]
              |
              v
     1\. \[轻量向量粗筛过滤\] \----\> 使用极其廉价的轻量级向量模型, 快速剪枝 ToC JSON 树
              |                (将 ToC 节点从 100 降至 Top-5)
              v
     2\. \[多路扁平路径合并\] \----\> 将多级子路径拼装为单行文本: Options 1: 财务 \> 附注 \> 合并
              |                (一次性喂给大模型, 变多轮串行为单轮选择)
              v
     3\. \[多线程并行遍历\] \------\> 并行执行 Top-3 候选子节点的原始文本調用与校验
              |                (耗时从多轮叠加转变为单轮最大耗时)
              v
     4\. \[主动提前截断决策\] \----\> 评估节点 Summary 是否已覆盖核心事实, 触发 Early-Stopping
              |
              v
     5\. \[多级缓存 (ConDB)\] \----\> 在 "用户 Query-寻址路径" 及解码阶段 KV-Cache 建立强缓存机制 \[cite: 4, 6\]
              |
              v
       \[高精度答案流式输出\]

通过部署这套高度优化的混合同步架构，PageIndex 能够将平均检索延迟压缩至 **1.5 至 2.5 秒** 的合理区间，同时将每次问答的整体 Token 开销降低 ![][image23] 以上，使其真正具备了进入高频业务系统的实用化资本2。

### **PageIndex 的生态扩展组件**

伴随 PageIndex 的日渐成熟，VectifyAI 进一步完善了其外围的技术生态链，为不同粒度的知识检索场景提供了精准的工具矩阵4：

* **OpenKB**：专注于企业级多源异构文档库的“百科化编译”4。它并不保留孤立的文档个体，而是通过提取 PageIndex 树的语义节点，将数千篇独立文档自动编译、互联为一个逻辑严密的全局交叉知识百科（Interlinked Wiki），极大地提升了企业跨部门知识管理的效率4。
* **ChatIndex**：专门针对超长上下文对话历史（Long Conversation Histories）进行树形层级索引4。它能够把长达数月、包含数万条的历史聊天记录自动整理为主题分明、带有时间与事件标记的树状对话索引，使大模型能在毫秒内追溯到遥远对话中的关键事实细节4。
* **ConDB**：是一款专为大模型“长上下文解码阶段”量身定制的 KV-Cache 原生上下文数据库4。ConDB 能够在物理层缓存已经加载并经过推理的 ToC 节点状态，在执行多次重复寻址时，大模型无需对已读文本重新进行计算（Prefill Phase），从而实现首 Token 响应速度（Time-to-First-Token）的断崖式削减，极大地提升了最终用户的感知体验2。

## **系统实用性评估结论与企业部署决策矩阵**

经过全方位、跨领域的深度剖析，本评估报告对 VectifyAI 的 PageIndex 开源框架给出如下终审性的实用性评估结论与部署建议1：

PageIndex **绝非** 一个能完美替代所有传统向量数据库的“万能灵丹妙药”；相反，它是一个**极具破坏性、极高特化性的深水区深度检索武器**1。其完全抛弃向量相似度、诉诸大模型主动结构推理的设计，使其在处理高价值、严密组织的长文档时展现出了无人能及的优势1。

为了帮助企业级首席信息官（CIO）和系统架构师进行科学的选型决策，以下决策矩阵明确界定了 PageIndex 在生产环境下的最佳适用边界1：

                    企业文档检索场景选型矩阵

   高 |-------------------------------------------------------|
      |                                                       |
      |   \[ 选用 PAGEINDEX / 混合架构 \]                         |   \[ 选用 传统向量 RAG \]
      |   \- SEC 申报文件、审计年报             |   \- 企业统一知识检索 (跨几万篇短文档)
      |   \- 巨型军工、国防与航天采购合同 \[cite: 19, 38\]       |   \- 海量小文本、博客文章检索
      |   \- 高度合规敏感的法律、医疗、准则条文 |   \- 大并发、低延迟的电商客服机器人
逻辑  |   \- 包含多表关联与跨章节多跳推理的任务 |   \- 寻找语义相似段落的模糊匹配任务
复杂度|                                                       |
      |-------------------------------------------------------|
   低 |                                                       |
      |-------------------------------------------------------|
     单篇文档长度 (高密度、高结构化) \-----------\> 跨文档库规模 (海量、无组织、碎片化)

1. **强监管与高价值场景下的“必选项”**：如果企业的核心业务属于金融审计、证券分析、知识产权诉讼、合规审计或军工航天等领域，且检索错误的业务成本和法律责任极高，开发团队应当**毫不犹豫地将 PageIndex 作为核心检索引擎引入**1。其提供的确定性页码定位、完整的可解释推理路径以及跨章节顺藤摸瓜检索能力，是企业建立可信 AI 系统的坚实物理保障4。
2. **海量异构碎片的“防踩坑指南”**：如果企业的数据源主要是数万篇零散的用户日常反馈邮件、缺乏格式章法的聊天记录日志，或者业务系统需要承载每秒数千次（QPS）的即时搜索响应，则开发团队应当**坚决避开纯 PageIndex 架构**1。在这种场景下，构建和反复遍历成千上万个文档树不仅在算力成本上无法承受，其推理响应延迟也会彻底摧毁用户体验1。
3. **推行“前置轻量粗筛 \+ 后置 PageIndex 逻辑穿透”的混合系统架构**：对于绝大多数复杂的大型企业内部知识库，最佳的实用化落地途径是构建**双轨 RAG 检索管线**6。即先使用轻量级的向量检索或传统全文索引（如 Elasticsearch）在庞大的企业文件库中快速定位到最相关的几篇 100 页以上的 PDF 原件1；随后，将这些长文档的主导权移交给 PageIndex 引擎，利用其树状索引与主动推理，在文档深水区完成精准的数据清洗、比对、多跳计算和分析生成1。这一混合同步架构能够在保障企业级检索精度的同时，将计算成本与系统延迟优化至最平衡的黄金比例1。

#### **引用的著作**

1. PageIndex: The RAG Framework That Threw Out Vector Databases and Still Hit 98.7% Accuracy \- Towards AI, [https://pub.towardsai.net/pageindex-the-rag-framework-that-threw-out-vector-databases-and-still-hit-98-7-accuracy-d194e0549478](https://pub.towardsai.net/pageindex-the-rag-framework-that-threw-out-vector-databases-and-still-hit-98-7-accuracy-d194e0549478)
2. Just tried PageIndex \- a vectorless RAG system that hit 98.7% on FinanceBench (no embeddings, no chunking, no vector DB) : r/WebAfterAI \- Reddit, [https://www.reddit.com/r/WebAfterAI/comments/1t4i8fb/just\_tried\_pageindex\_a\_vectorless\_rag\_system\_that/](https://www.reddit.com/r/WebAfterAI/comments/1t4i8fb/just_tried_pageindex_a_vectorless_rag_system_that/)
3. Vectorless RAG: How PageIndex Works (2026 Guide) \- Build Fast with AI, [https://www.buildfastwithai.com/blogs/vectorless-rag-pageindex-guide](https://www.buildfastwithai.com/blogs/vectorless-rag-pageindex-guide)
4. VectifyAI/PageIndex \- Vectorless, Reasoning-based RAG \- GitHub, [https://github.com/VectifyAI/PageIndex](https://github.com/VectifyAI/PageIndex)
5. PageIndex Deep Dive: The Good, The Bad, and The Ugly of Vectorless RAG \- sjramblings.io, [https://sjramblings.io/pageindex-deep-dive-vectorless-rag/](https://sjramblings.io/pageindex-deep-dive-vectorless-rag/)
6. Vectorless RAG with PageIndex: A Practical Guide for Production Systems \- Medium, [https://medium.com/@techieman/vectorless-rag-with-pageindex-a-practical-guide-for-production-systems-10cc5c8972e4](https://medium.com/@techieman/vectorless-rag-with-pageindex-a-practical-guide-for-production-systems-10cc5c8972e4)
7. Vector RAG Is Dead. PageIndex Just Proved It. \- Artificial Intelligence in Plain English, [https://ai.plainenglish.io/vector-rag-is-dead-pageindex-just-proved-it-470ea6ac446a](https://ai.plainenglish.io/vector-rag-is-dead-pageindex-just-proved-it-470ea6ac446a)
8. RAG-Tutorials/PageIndex\_Vectorless\_RAG\_CrashCourse (1).ipynb at main · krishnaik06 ... \- GitHub, [https://github.com/krishnaik06/RAG-Tutorials/blob/main/PageIndex\_Vectorless\_RAG\_CrashCourse%20(1).ipynb](https://github.com/krishnaik06/RAG-Tutorials/blob/main/PageIndex_Vectorless_RAG_CrashCourse%20\(1\).ipynb)
9. VectifyAI/pageindex-js-sdk: TypeScript SDK for PageIndex document processing. \- GitHub, [https://github.com/VectifyAI/pageindex-js-sdk](https://github.com/VectifyAI/pageindex-js-sdk)
10. Best PDF Extractor for RAG: LlamaParse vs Unstructured vs Vectorize \- Chitika, [https://www.chitika.com/best-pdf-extractor-rag-comparison/](https://www.chitika.com/best-pdf-extractor-rag-comparison/)
11. Prepare Documents RAG Ready with LlamaParse from LlamaIndex: Complete Guide, [https://www.youtube.com/watch?v=TYLUTIAn1Yg](https://www.youtube.com/watch?v=TYLUTIAn1Yg)
12. Vectorize PDFs for RAG with Imprompt.ai and LlamaIndex \- YouTube, [https://www.youtube.com/watch?v=YqtTOdicngY](https://www.youtube.com/watch?v=YqtTOdicngY)
13. Ingesting Complex PDFs with LlamaParse for RAG Workflows \- YouTube, [https://www.youtube.com/watch?v=EL9lCOgLR58](https://www.youtube.com/watch?v=EL9lCOgLR58)
14. SpaceX IPO targets June 2026 after SEC filing \- Capital.com, [https://capital.com/en-int/learn/ipo/spacex-ipo](https://capital.com/en-int/learn/ipo/spacex-ipo)
15. SpaceX IPO Guide: S-1 Breakdown, Valuation & Trading Strategy | BitMEX, [https://www.bitmex.com/blog/spacex-ipo-guide](https://www.bitmex.com/blog/spacex-ipo-guide)
16. SpaceX Guide: Everything You Need to Know About the Biggest IPO in History, [https://www.investing.com/analysis/spacex-guide-everything-you-need-to-know-about-the-biggest-ipo-in-history-200682043](https://www.investing.com/analysis/spacex-guide-everything-you-need-to-know-about-the-biggest-ipo-in-history-200682043)
17. SpaceX Stock and IPO Guide \- Investing.com, [https://www.investing.com/academy/stocks/spacex-stock-guide/](https://www.investing.com/academy/stocks/spacex-stock-guide/)
18. SpaceX IPO: Listing price, time, valuation to outlook; Key things to know about Wall Street debut of Elon Musk's company, [https://www.livemint.com/market/ipo/spacex-ipo-spacex-ipo-valuation-spacex-ipo-price-spacex-ipo-listing-date-spacex-ipo-date-and-time-spacex-ipo-size-11781248703761.html](https://www.livemint.com/market/ipo/spacex-ipo-spacex-ipo-valuation-spacex-ipo-price-spacex-ipo-listing-date-spacex-ipo-date-and-time-spacex-ipo-size-11781248703761.html)
19. American military space closed around one company in seven days \- SatNews, [https://satnews.com/2026/06/03/american-military-space-closed-around-one-company-in-seven-days/](https://satnews.com/2026/06/03/american-military-space-closed-around-one-company-in-seven-days/)
20. SpaceX wins $2.29B to speed Space Force's LEO communications 'backbone', [https://breakingdefense.com/2026/05/spacex-wins-2-29b-to-speed-space-forces-leo-communications-backbone/](https://breakingdefense.com/2026/05/spacex-wins-2-29b-to-speed-space-forces-leo-communications-backbone/)
21. 6 Charts on SpaceX's Pre-IPO Financials \- Morningstar, [https://www.morningstar.com/stocks/6-charts-spacexs-s-1-financials](https://www.morningstar.com/stocks/6-charts-spacexs-s-1-financials)
22. Inside SpaceX's IPO filing – revenue, Starlink, AI and key financials | HL, [https://www.hl.co.uk/news/inside-spacexs-ipo-filing-revenue-starlink-ai-and-key-financials](https://www.hl.co.uk/news/inside-spacexs-ipo-filing-revenue-starlink-ai-and-key-financials)
23. Starlink growth is getting harder ahead of SpaceX IPO \- TNW, [https://thenextweb.com/news/starlink-is-spacexs-cash-machine-but-the-maths-is-getting-harder](https://thenextweb.com/news/starlink-is-spacexs-cash-machine-but-the-maths-is-getting-harder)
24. SpaceX: What Investors Need to Know About Its Enormous Upcoming IPO \- Morningstar, [https://www.morningstar.com/stocks/spacex-what-investors-need-know-about-its-enormous-upcoming-ipo](https://www.morningstar.com/stocks/spacex-what-investors-need-know-about-its-enormous-upcoming-ipo)
25. The SpaceX–xAI Merger \- The D\&O Diary, [https://www.dandodiary.com/2026/03/articles/director-and-officer-liability/the-spacex-xai-merger/](https://www.dandodiary.com/2026/03/articles/director-and-officer-liability/the-spacex-xai-merger/)
26. Starship's Twelfth Flight Test \- SpaceX, [https://www.spacex.com/launches/starship-flight-12](https://www.spacex.com/launches/starship-flight-12)
27. Starship flight test 12 \- Wikipedia, [https://en.wikipedia.org/wiki/Starship\_flight\_test\_12](https://en.wikipedia.org/wiki/Starship_flight_test_12)
28. Starship Flight 12 test flight \- Science\! Astronomy & Space Exploration, and Others \- Cloudy Nights, [https://www.cloudynights.com/forums/topic/999550-starship-flight-12-test-flight/](https://www.cloudynights.com/forums/topic/999550-starship-flight-12-test-flight/)
29. FAA requires SpaceX-led mishap investigation before resumption of Starship launches, [https://spaceflightnow.com/2026/05/27/faa-requires-spacex-led-mishap-investigation-before-resumption-of-starship-launches/](https://spaceflightnow.com/2026/05/27/faa-requires-spacex-led-mishap-investigation-before-resumption-of-starship-launches/)
30. Starlink Prices June 2026 – UK & US Costs, Plans and Equipment, [https://findcheapbroadband.com/blog/starlink-prices/](https://findcheapbroadband.com/blog/starlink-prices/)
31. Starlink Raises Prices, Adding $5 to $10 to Its Monthly Plans | PCMag, [https://www.pcmag.com/news/starlink-raises-prices-adding-5-to-10-on-monthly-plans](https://www.pcmag.com/news/starlink-raises-prices-adding-5-to-10-on-monthly-plans)
32. Another Price Hike: Starlink Adds $10 'Monthly Kit Fee' for New Users, [https://www.pcmag.com/news/another-price-hike-starlink-adds-10-monthly-kit-fee-for-new-users](https://www.pcmag.com/news/another-price-hike-starlink-adds-10-monthly-kit-fee-for-new-users)
33. Analyst Projects Massive Subscription Growth for Starlink Ahead of Imminent SpaceX IPO, [https://satnews.com/2026/06/07/analyst-projects-massive-subscription-growth-for-starlink-ahead-of-imminent-spacex-ipo/](https://satnews.com/2026/06/07/analyst-projects-massive-subscription-growth-for-starlink-ahead-of-imminent-spacex-ipo/)
34. Starlink drops purchase option in favor of hardware rentals as it raises prices for some plans, [https://9to5google.com/2026/06/10/starlink-drops-purchase-option-raises-prices-for-some-plans/](https://9to5google.com/2026/06/10/starlink-drops-purchase-option-raises-prices-for-some-plans/)
35. Starlink Hikes Prices for Nearly 3 Million US Customers. Just One Plan Escaped \- CNET, [https://www.cnet.com/home/internet/starlink-hikes-prices-for-nearly-3-million-us-customers-just-one-plan-escaped/](https://www.cnet.com/home/internet/starlink-hikes-prices-for-nearly-3-million-us-customers-just-one-plan-escaped/)
36. Starlink Price Increase: Monthly Plans Rise $5-$10 Starting \- 5Gstore.com, [https://5gstore.com/blog/2026/05/16/starlink-price-increase-monthly-plans/](https://5gstore.com/blog/2026/05/16/starlink-price-increase-monthly-plans/)
37. The Final Frontier of Tax Avoidance: Elon Musk’s SpaceX Has $1.9 Billion to Gain from Defunding the IRS, [https://itep.org/elon-musk-spacex-tax-breaks-doge/](https://itep.org/elon-musk-spacex-tax-breaks-doge/)
38. Space Force awards $2.29B deal to SpaceX to accelerate 'backbone' SATCOM network, [https://defensescoop.com/2026/05/27/space-force-awards-spacex-contract-backbone-satcom-network/](https://defensescoop.com/2026/05/27/space-force-awards-spacex-contract-backbone-satcom-network/)
39. SpaceX awarded $2.2B contract for military data network with laser links and advanced encryption \- Crypto Briefing, [https://cryptobriefing.com/spacex-military-data-network-contract/](https://cryptobriefing.com/spacex-military-data-network-contract/)
40. ESA \- Date is set for bigger booster, more powerful Ariane 6 \- European Space Agency, [https://www.esa.int/Enabling\_Support/Space\_Transportation/Ariane/Date\_is\_set\_for\_bigger\_booster\_more\_powerful\_Ariane\_6](https://www.esa.int/Enabling_Support/Space_Transportation/Ariane/Date_is_set_for_bigger_booster_more_powerful_Ariane_6)
41. Amazon Leo \- Wikipedia, [https://en.wikipedia.org/wiki/Amazon\_Leo](https://en.wikipedia.org/wiki/Amazon_Leo)
42. Amazon Leo Launch Delayed Again as FCC Deployment Deadline Looms, [https://broadbandbreakfast.com/amazon-leo-launch-delayed-again-as-fcc-deployment-deadline-looms/](https://broadbandbreakfast.com/amazon-leo-launch-delayed-again-as-fcc-deployment-deadline-looms/)
43. Amazon Leo mission updates: 330+ satellites deployed following latest Atlas V launch, [https://www.aboutamazon.com/news/innovation-at-amazon/project-kuiper-satellite-rocket-launch-progress-updates](https://www.aboutamazon.com/news/innovation-at-amazon/project-kuiper-satellite-rocket-launch-progress-updates)

[image1]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAB8AAAAZCAYAAADJ9/UkAAABx0lEQVR4Xu2UMSiFURTHj1CEiEGivAEZSDLIoCTZLDKI0cBEGciglMlqFMnAwmCxSFEWZZNiMZBsUkIZxP//zv3e+77z7ocoi+9fv96759x7zr3nO/eKJEr031QBqqzRo3JrgKpBoTVCRaAFlFqHVRt4B3fgxnAGWt08zjkHa2AF7IJ9iSbIB5PgHmy73yXxbzCtPtHAPg4ke2LrI5XOF2gCvIBeN24WPdQcyAsmhTUOhhz9oA7UgEvQHpp3CpZFTz0DmkK+QBdgT7TsgbjmQfQz5GjEjLlDBrel2jBjn1iNWWMbcPbvrJdOcGyNootZkXnR009F3Wl9lvxIvmhAdv0JGLYO6Akcgm7QCLZE+yUQS/2r5GyYK1BrHRJtPoobZTM1uDED/zg5A/PUO6DA+OLEoAvuf4kb/yg5u5KTpq0DqhddzG8elg3qSz7o7JsSc93KRJuMk7hTK9reQI+xc364UhyvZt1pcTO+TWXEE11LfPIu0cXh6xc0WLg5byX3s7Gir6IxvOIT+yjxyYvBmETLxlfMPq+L4Fn0ulIp0QZel9x3IyMG5RXi1eH77BPtHWBU9DVMRbxZMRZfP1aEv97vnCjRn+oDHwRp4lXbRGcAAAAASUVORK5CYII=>

[image2]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACUAAAAZCAYAAAC2JufVAAABoUlEQVR4Xu2VvytGURjHH4WIGFgYiBgopcQgg4EiUUoWy1sWq9Vg8R8YDMpgshhY5Mc7vINBZnakhCgDi8T367mXc5/Ovb2va1Dutz713u85557v+9z7nCuSKVOmv6My0GHNBM2Dfmsa8Z7N1oQarGHFhd1gF1xEh2JVA/LgHTyCK8NCMK8WFMAOWAcb4AxMBeNe9YA38CK6wWV0OFaN4Fx0jY+BYF4Yyh17Ei1ErCpEF/aBZyk+VBc4BTkwAzpBE2gB+9/TPu+9J1qlVTApWuWiVGqoXjBuPFbnGLQ6HkPx0f1IpYayqgeHYNT4YagJsCZarcHIjASlDbUCtkC58RmKYZdFKzgL7qXIR5g21K34O4qhDoyXE/0TiS87lTbUDWi3Zoy41zVoswNWaUKNiJ5Z9pGwEovgwfjsUh4N7MREpQm1BDatKdEzqtLxudcrGHI8r5JCsbPYOdyYv63o+0LxpefLf2R8VvZE/Pf6Ess8Jpr+Lrh2xTKHp7Etefi58YWieEQUnOtqsC3+pvhVsdXrrGk0DebAMKiKDmXK9M/0AT3xWbAfOENGAAAAAElFTkSuQmCC>

[image3]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAC8AAAAZCAYAAAChBHccAAACZ0lEQVR4Xu2WT4hOURjGH6EohiQi03zKxkQWsiASKTSxQPkzytaasLMwe8pGiYUkko2FbGQmIykLG7JSSBaEDQr58zy95/3uuafvu+fMSun+6ldzz3u+e997zvueO0BLS0vLv2IaXZEORii+iG6m05NYiuIb6Pw00Ied6UApSmqY3qav6qEuHfqQPqUX6Vu6NZ4Q0L3208/0Cn1Hj4fxJi7RL/RND89G82qspr/oN/qHvq6HuzyAJeVJeIKbujMsdpJ+otvpPHojzFsTzUuZRe/Anp+q3PZWU+vMpHPoWvoV/ZMfh81zZtBbsBVzdsMedipc+z1/020+qQcLYbu6L7iKLqFL6QXYsxrJJa9VuEeXhevF9Dk9Gq430p90ks4NY6V06KFkbAj2QkXkkv8Ae4Ef9AS9Sm+i2o1jIa7d0Co/gfXPQeSbO0XVcBnVwmTJJT8Ci3stnqOzo7iaU+Mv6HW6ANYX2g31Qa5hYw7TR7CeKSKXvBI9Q7+jaiQlpVUSnryaUzUrvC/0m/VhLIeO4meo+qaIXPL3YaUyACsF3wFfVU8+rXkfPx+NNXEA9rLqoWKakl9JR5MxrbjOeyW2C1XNX0O9RLSCGp+Ixvqh53ykL2EHQjFNySumBFPWwb4PivlpM4H6kaoV90bOsQfVi8b3yNKU/HL0rsEtsPlaMTXXY9jxqXNbaAe0E0pI5SC0Y6fp3XAdE+9ScfJ6yA7Yyr0P1zG6ViPqfxVHp4n6YCzEhY42NbJOGdGBlYC+D35y+CIpyRR98KacfCmqQ5XIkfB3PwZhX0olOpUzXjumHdLXtaWl5X/hL8n0lGcCBzR7AAAAAElFTkSuQmCC>

[image4]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACUAAAAZCAYAAAC2JufVAAABqUlEQVR4Xu2VvytGURzGv0IRMZBSyjspKYv8BaQMDAwWMVgof4HpXYxGZVBioYxSBmUz2KXEwGIQG4MBz9M5x/t9v/fHue87Ge5Tn95zvvfc5z7nnnPPK1KqVKn/pzVb8GoBA55Wc60RTdlCTBVwaIvi6tfgCTyAGzCiBxRUBTzaYp7awYEkQw2De3AJeqU27g4MqXExhft+7IU8LYFXSYY6Fme0oGrT4BtUVS2m4F8oFGewB5bBnCRDfXgmVI1t1t7AqKpnSfsXCrUIjsSFazTUJ5hU9Sxp/0KhrsCYb6eFehG3VFyyIAZhID6A9+SJ+077R0O1gXXVTwu1L8k9telrsVD031X9QqHmQafqp4WyXx/7F+KWz75BK/qfqn40FAd8gWfFu6qdgK6/0U7d/jfsqVvQX7tcpyx/hsrylw4waFgRNzO2+8Sd4j1g1teCuJQ0r6qaVZY/79P+UaUt37Y4o3NxD+I+4dnFMydsYGpc3EezoWpW0eWz4ifLTX8m9a91S5zRjriZ8Qjh0vFXi5PhOP4VpSn4c0xi2ZoRTWbAqm+XKlWqWf0CsgZlQST69lAAAAAASUVORK5CYII=>

[image5]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACUAAAAZCAYAAAC2JufVAAACC0lEQVR4Xu2VT0hVQRTGP0mhkNAsAs2FC3UTiBAElRBICIG5UWhRuGlhrYOiXRDtWkS0iCikVmXRxoWBQi7dBrkRFxZuQlAIalH05/s883jnzpt373ubELwf/ODNuXPnfufMmXlAqVKl9p6uxYGgNjJAjpKW6FmROskI7H2t05T6yIsoJgPj5CvZJH/JOjnvJ+Woi6yR52SZfEMTSSmDWdSamiAfSX8YXya/yQ45XZlUR1rzNTngYmfJVTfOlT62hVpT78kCGQxjZfkYVjE9OxTiKZ0hq3GQWiLtcdBL2TyBub+EWlPzMAM3XUzzFPtOTrl4rIuweUMupiQeomALJ8lLmLmUKZX+cBSbgX1sg5zIPsroOPkEm/sI1l93Sbebk9QHcjL8TpmKpUy1bfrQPRRkTA3D5oo/sMrnqpVcd+NGTKn3tLj6oiN6lpIq/QNVYyK3n3SyfKM2YkqnTo3fiCGt/Yz0kFHYVSJTT1HnvpKBn+SLY9vFXiGbkUwsorqYqnwO1icpHYOdvPtRXNstY7ej+K4OwhrOM03ehN/+5lbGyu5OGEtHyDvS62JeWuMz0h/XJZyKJ5XaPlVGhh7AtqGSwAVY5WRO0rHXobkRxqrsCuxCjiVTer9Q+riaXqfDb9stZJvU8xa2jZKSUUzVqUh33y9kT6iqPoc6PfU/NUaukCnY/2upUvtX/wCGjmRbi+JbhQAAAABJRU5ErkJggg==>

[image6]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACUAAAAZCAYAAAC2JufVAAABr0lEQVR4Xu2VwStFQRTGj6QsiFKkrISihCwk7CQbKxbyD7BSiP/AXhaytbCzsbCzs7EWsVFIlKWipPB9nZnuvPPudN+9vbK5X/16zTnnzf1m5s65IqVKlfpftYMOG8ypZtAPWmzCqAH0gUabsBoBv+AFPBmuwHBSWiVOvg4+wSl4B9ugKSxyGhKteQDdlalqzYqaSuMctCWlFeKqd8A3WHSxCVFjW75IdFHXosY556PUYGoVLDnmQA+YAndgLKizmgRf4AK0uhiP8Qw8+yLRXesC4+BDajS1YsbcgUNJP4JQR6Ir529a3CqXKSseQacNpijLFBcXqrAp3sJLG4xoV/ThPC4eG0Ujxy5ub2JhU2vg3gYj8g8J3ym+O7dSR1O8ZdylE5uIyN4+jjfAjagpv3tehUzti062aRM1iP3KX4y6vejcfh4DJ1swuZi4M2y805J0aD9P2BK8cptiEYtjpni0B6K74JtpL3gFP6INmPK9a8+NQ+U2xRWzE8dMMea7vM+zyXJHaGxQ1Cy/ALGvwLzo+/cGRk0uVTyKGdEVZ34sA/F/A2DZ/XJcqlSpLP0BNJlo3kPFCYcAAAAASUVORK5CYII=>

[image7]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACUAAAAZCAYAAAC2JufVAAACOUlEQVR4Xu2VzYvNURjHn4mmEWVBIy+FkoU0NZEiC2koMQqTFAvTrCylkIWN5h+YhYVYWNlYjJS8xW1WsiYWpCkRkigWNMb30zmPe+65v3PvXSq/b326ned3Xr7nOc8516xWrVr/jgbE5vhbpQViUCwTfdm3kui3Kg9amKOjFosp8VPcEm/F1pYeZrvFa/FIfBUfxQHrbm6JaIhpcUVcE8/EwaRPpR6KGbEmtl/FGGbRsHgptsT2BvFOzIk9MVaSm5pPYFMdN7NQfLfmgoiBz8Xy2D4XYy/ECmtd6GrsUxJ971jIEqcxas3NFjUpLuTBTNTSDrEytjGGQUxNeKeCMMXR9Swcc0zj4ox4I36ITWmnCp2wYIixS7NvudzUfnHZQrbYYFHsfFY8FuctZGSR+CK2Jf1cq8VxC7XE5F2PwYKp++KiWCuOik/WYaybumvBjOu9eGLtWcDUmHhq4WJ0yyjC1L0sdlJcskKxu6m8poj9FiNZ3MUbgzEydiT71ou4VDw76/MPyGvqcBZvWKgZbh3iCaAm0pRzi/yK70viqcjEafE5i5MMxjFHpbh9vriL54BB1A/6ENvUkYuNuKnS5OnT0Z/EydQvsTOJtWi7tS6GSG2a3m/WboojJ5b2G7JwaU7FNm/gDfEgtl2URVXN/hUFzt/HuiRGrZy1ZiHetnDM/ph6TWEq7Xc9xqhJFy9+I2mz3k3r4W+Gp4CUclzcriqx8EZxTOy1Dle6oEMW5t9l5T/8WrX+D/0Bgo5yEVL7R90AAAAASUVORK5CYII=>

[image8]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAC8AAAAWCAYAAABQUsXJAAACtUlEQVR4Xu2WS8hNURTHl1CEPCNRHpmISKKUkVcZkEIpAzMmZso7MZCZgciAEkOvDJCBcmOilFLMGJAohVJM5PH/3b3XOevu77q+8yUM7q/+3fba+57z32uvvfcx69Onz79mgjRVGl52BEZl/VeMlJ5IN6XL0qTO7opb0sYyCMxoYf7tBS8aVgZ/A+NXSNOte2b3SNMsjTsgPZc2hX7ii6Sz0ogQt7vSfWlmbvNHYmOqEWZvpO/SF+mHNDb09YIyeCpdzW2Mn5BeVCPMlkqfQ5tnX5Hmh9joHOuAWfBHHuBg7pk0JcTIGCuyIfcP1vxuS5NeE2KYem/16nUzfyPHHVbmTGi3YQB1FEvluLQ6tCNNzDOmZclozCKJeGl1chZLn+puG2ep7ufm9hLpgdWVUYHRg2WwB03M8/K3loxi2PFJeWbHSw+tXgn23TVL4xCbeMAm9Qzck/Zbqkdq66O0PIyLNDFPqVAyvzLPsxyMv5I+SIcsHQrQNePg5u9YMu2QLTJBRkqamPexgzHfDd4fM77MUmLP03DzZdkQKzeZ08T8ehu6eVZir9XH4gJLK0N1cJS2z1WOrF15gNOy9NJ9RRyamPdTZCjmKVtKBpjIaelRbuO7DRu2NMkxicHtRRyamJ9o6YWcJJwojq/4jBBzPOMci551H9/K7QpuvVNF7HXWnCIOvcyvs7RXKBfnqPRVWhlifs533JQZMs4BwuXmeIW0QqwNG5WO2SH2zdLsyUKEettpyfysos9LgT5+fXLzLN3OF3KbZ1Kz8VJy2KDcOWuLOHcQcS+jjsTxKcCyHJMuSYetPqocXoaxUhdzP6a4BZn4kdx2VknvpG3SOUufGFtDv0PCTtrApAGnDkmAzbGDjLK5qPEtseMPwnfSDkulFb+ZIrelyWUwg0cmd116XPT1+Wv8BPIpkao7Yy92AAAAAElFTkSuQmCC>

[image9]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAC8AAAAWCAYAAABQUsXJAAAC3ElEQVR4Xu2Wy8tOURTGl1CEXCOXKKFEJCGXUkIZkEiZ+QfMlJKSMjCRJFKITN0ycCvKFyWllFJKDBhQCqWYyOX5vXuvc9a7nfP5Ui6D96mn912Xs8/aa6+91jHroYce/jXGiBPFwaUhYFjmf4Wh4mPxqnheHNdtrnBN3BQV7HSauFqcHA0FeMFscZU4qLD9Cvgvt7R+U2Z3iZMs+e0Rn4ubgx39AvGEOMSVi8QVLuT/X8WtQQfIyqz8n4Xw+SAurTyaQRk8ES9mmcAPii8qD7PF4qcgjxQviHODbnjWVSCIY+LKqBS+i08tZQLw4A1xTuWRnsPvZra3Yaf4TVwbdAT1zurTawr+StY7OJnjQe449WXy3/HZul/IURPoucrDbEvWEUTMUISvX/qw3ktxQpYXih9rs42yVPczs0x13LNU2hXY+REr6shSFghsY5bJLDK7d2BDh2/MUAQvf2Mp0HiXfFP+3GjxgdUnMV+8ZMkPcom7LmkbWICg2DkXtA2UC34HrP3ycnKcYFvwnhzAGq/E9+Jeq9/9U8b7wzLxmTijNBQgqNuWstYGP52BBN8E1o4ZX2KpSZwOugoEPJDA6TBc4P4CBxvs94PnJHZbXc7zLJ0M3YpW2gUCIZN3S0MB3+CI0tAA7yK/EzwJomSAd8WHWfYu2AE7OWl1y2O3ZywdkwMbPvj6kBkrXrb2msTOC+kkdBSHd5upQefwjNMWPevu35flCjgzQKZbcoJMsvtWtypA4IfEKcGPC3nLUpBgvaWuQbk49otfLE1lh/f52OEcZPyOpeHmINMMtb6g64BJSl2WJGMelHegJrJxgvBSQMevzw2m8mvxbJZZi9OLQ8lB6fLtsq7Q8yGG3suos3Z8YUmc/euNLJd259HsQ1DMAT4b9mXZsUZ8K24XT1kagtuC3UG5HLbm1kvXIQmg/HT54+CC77BUWm2X/bo4vlRmcM/YHHfsUWHr4a/hB/sTnt4NC1ePAAAAAElFTkSuQmCC>

[image10]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABIAAAAWCAYAAADNX8xBAAABV0lEQVR4Xu2TvyuGURTHv5Iiym96i43NRAZlNFj0irfetwyMZgOL/AdKNlKyGmzKYJBFMRmMBn+BDIrkx/f7nOfe5zwPIhPlW5/e59x7+p773nMu8K+/rTrSSUqkh9Tnt6MayCAZJm2FPfSRaRfL8JwsujXpiSzAikrdZBdmnmiK3JKhsEDtkWPSnMZN5Ip0xQzTC5kMwQR5JGNx24xOSEsaj5BD0hgSUr2SzRDoqAPZXlL9iKy7tTmYeVH3sIJRHWSWbJMHcuA3YVU/M7rxCzIqwyrrNJfILlWSybeMvNSFfbKMzGwHPzCSZpBvwAq+MNLM6G8UOzIK64gMpHHkxyFIOZol9JJrvDcqnqiVXJD2mGGSUU0fuoMN2NgHhTtSB+PUUs+k6mKZn6a/cUGTvUWWyBmskubJa5XckTVYd/WM/Pwl6oe9t3nYk9HdfSTlVWBGmvZfqjddDz+TcAmsUwAAAABJRU5ErkJggg==>

[image11]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAC8AAAAWCAYAAABQUsXJAAACiElEQVR4Xu2WzatOURTGl1CEfEZiQibylYSUgYRQpLhl5h9QBkpJycxMd0CEMveVgWvqLQNKkTJkQKIUSjGRj+dn73Xeffbd57znGmDw/urp3r32bp/nrLP22q/ZkCFD/jVzpIXS5HwiYVrUf8VU6Zl0V7ouzatPV4xJ+/MgTJJW5MHIMWl6HpwA7L1FWmzlzB6XFllYd1J6IR1I5omvlS5KUzy4RvoufZV+Sq98IoO5Jl1O1uVQBs+lm3GM8bPSy2qF2QbpSzKeKd2QViYxEkesBp+Lxb5Bk/kL0qGobRYyOCI9lZb2l43jqPRD2pHEMPXBQjahZP5OjDt8GTwUaTPPZ9qYxTD8UNqcxVMw0bNgNM0iL85zFsTxOulzf9pmWaj75XG8XnpgLUlqM8/pphOkXJJOWD97JXj4Owt7Ytjxl/LMzpYeWX+v1dItC+sQh7h4SJ028zkcag7fICgVSqbJ/L4khvHX0kfplIVyhtaMO13N85BRCwYGgTlvAoPMl+CLpBmndD9JV5PYb7qaXya9yYMN7LE/N0+SKEtvi6ssfBm6Fa20RlfzdA8MdSHdc6LmN1koGeBFzkuP45j7oEYX89QeWU/bWhtzLTyQTkJHcbzbLElijmectuhZ9/W9OB5HF/O+psn8Lgtdg3JxzkjfpK1JzPt8dVMmkPH7Fi43h0xzqfWSWAVvu9vCQ97HcQmv4ZJ5LwXm+esHms70VroWx+xNzZb24ICOSTuzOK2auJdR1Sz4JPl1nxtwvPWlF4qDKW5BfmqcjmNnu4WkHJauWPgpMpLMO5TLOSsnj65DEuBgOtEVeu9eq1/bXZkhHbFQWvxf4p40Pw9G6DK83G3pSTY35K/xC/xBjgh0GHU8AAAAAElFTkSuQmCC>

[image12]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAZCAYAAABzVH1EAAACRUlEQVR4Xu2WO2gVQRiFT1AhYoIGiRojGMUmSrQQFYOFRRQFtRFsUsRKLWzEQiSNnb0vRJAQBVNooU0gUVCwtBM7m0RQQbEwRAvFxzn5d3JnZ2eWcHNJuLAffCzznn/nsQtUVFRUNDstdEeYWSdb6DBd5+Wp/810hZcn1gRpxwZ6kHah2CaKBthJn9GpfFHdjNBp2CQcbfQVnaX36H36jv7y6jh6YGWq95q+z5VG6KN/6E/6DzZ4I/iLdCAax/mdnvDqCK2EglidpbUa12HBJVkFG2Av/YHGBNKDdCDjsLd8g55EfFtdhLX36aW3YLunlEYFcpo+gPUTC+QpbKwU2+lnFOehtppfWds5GhHIVvoCdmGUBbKf3oGtSj/yBzk1D7XVVtQqlpLqYKGspLfphSydCmQSdqEo6DP0K33o1RlAbVv6LFkgp+hj1A5oKpAJ2PZznIVN0O19TTR26SxZIC/pLi8dCySGG3dblj6OZQxEk/1EP3i661X5b2Fv/BL9Rs9ZsznUVmO6CaYOezvsE7EvyC+wmEB0WDUh34+Ze+hG5L8hOvAON+6hLN1B39CZ+RqG+pyi3UF+gbJA1tJR+oh2BmUpwq2ly2CMPqe7XSXUDrfGcFyjv7200HdE7dVPEi37MVjjL1naxx1AeSUoi6H26kf6kz4CW5VNWVoXwxPYn4WPrm9tSX2shfq7Sg/M11gkWv6bYWYdrKeD9DBtzRflOEqHsmfsD6Bu9OtwPsxsNvQWtQ3cNdmUaK/epZfDgoqKIv8B+qmL/uIaxu8AAAAASUVORK5CYII=>

[image13]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABUAAAAZCAYAAADe1WXtAAABY0lEQVR4Xu2TvyuFURjHv2IgtxApuUWSkgkxslzKYMBoMFrMlOn+A/4GmZQyKoPZwGCSsimLJBObH9/vfc57PO953+6u3k99ejvPec5zznnOvUBFRUY/naOj6YSjgw7D8jqTuQJn9J2e0Du6nJ9uMU6v6RMs74ZO+QRPH32kY2Gs02iD2ZhhKOcKli+O6QOtxwxHkx4msTd6SrvCWN8fuhkzgAb9hq3PMQK7znoSV0wLtFDM04/wzchiOsC0i7ct6k/WrugnXXBxDNBbuuuD5BlW9CCMJ5A/uVAhFVReeqjWwrSnKuCLlvV0L8RKi6avr/ELLNnfIH39S9j10xvk6KaDsJ+Uelp4gEAN9kfJenpPh3IZpIdOJjH1yv+kxBrsYTPUCt2m6WKRbRR3+6IrbixU4AJ2I6FNX+lMzHBox3PYicUi3Ye1waOiRy6uq2/9TRdZgp14A9bXMnrpKt0J34r/zC/yOEwdnhTOZQAAAABJRU5ErkJggg==>

[image14]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABUAAAAZCAYAAADe1WXtAAABjElEQVR4Xu2UvSvGURTHr1BEDBSKMBjI5qWUTAYGC8pgkMkfoLztZiWjxSCbMihJyCCjgcVLUmKSFAby8vm6v/t7znPJrp5vfXo655577rnn/O7jXE45WRVBa/L7l0qhCQrjBasSWIRX2IAbaM+K8KqGdXiCfTiGehtgtQMHUJvYF4lPhwVp8xmsQhn0wyMsm5hUBfAMbcb3CadQaXzbcA51ib3ifNym+6Vd8zAXOyPpYCUYjBd+k66na47DJFzBC7TYIFQFb9AHC3ALay7TrizVwDXswQzkQzE8QKeJU2t02CGMQB7sOt/jH4MKSbecTxZ0B0dQnthKqusvOZ9Q6na+elWs9qQKSeOeyvcBvYkdktqeyqcB30Oz8ac9jQew73yS6cRWT3X9jhDgMgUpbsD4v6Xph81B+pwUPJrYYfp2s6pTlWqBWpGlLudfk5VelGg0PiWdMLaqVvW296k0oEtoML53mHKZoUgnzr+6kGDW+bihNCKSPiU1Xtcdjtas9AfSA2NQEa3l9J/0BZZTT7ag/ybgAAAAAElFTkSuQmCC>

[image15]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABUAAAAZCAYAAADe1WXtAAABCUlEQVR4Xu2TzQoBURTHj7AQFjZKNpINJUqSspRYeAbP4Qm8gJWNpY2NN1DKE9hJjY2FslFs5ON/umM6d4y7EAs1v/rV3HPu/d/5uEPk4yMJwJy7KOB+EtZh0NV7gScX4AxaesshA5dwBUdwDctygqQIr/AM73Crtx04bAoj9ngAN6Q2eyEMY7ACT/Q+9AabYpyHBzgk9aSemEKzpOopUeMbmZNaw2s9MYVy710ov7KuqGuYQvmxvx7Ki74e2qEfhHp9qDhckDqKVVHXMIUm4BGWRI034LkWTIu6himUucCGGD/P6QSGRN2BD2+b1MK9PXazg2NSPwvTJ3UTNWfGh0RhC/bsa59/5QHLOEGkpKkc+AAAAABJRU5ErkJggg==>

[image16]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABUAAAAZCAYAAADe1WXtAAAAaklEQVR4XmNgGAWjYBQMLOAH4s1ArIkuQSkoh2KqAjEg3o8uSA1gBsQq6ILIgAeIJcnAj4A4CYg5GagEuIG4j4GKBrIA8VQgZkSXoAS4AvFqdEFKAMiVC4HYA12CEsAKxEIMVPb6KBgAAABENAjYKGAerwAAAABJRU5ErkJggg==>

[image17]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAB8AAAAZCAYAAADJ9/UkAAABgUlEQVR4Xu2VzSsGURTGj3xEFFE+SqHshIWykMW7YCVKysZGKdlaWSgbf4BSLJTExoKFslUsreUPQFhQStn5fJ7uzNuZY+bO3cr86re459x5n/feuTMjUlDw36iAlbYIWmxB0QxbbTGFTrgKm2wjpgG+wRO4A3fhNZzSkyLq4QJ8hSuml8YevIUdthHD8G8jf5w7onmBn/BD3JyQ8C8JCOeK6SacFLdCC7eZt2dfwsK7JTB8yBY9hITPwANxwbnhw3ACbotb/UhiRpK88C54BnslMPwGrom7cBY+S/rWE194FdyCS9E4KJzbpJmH6/L70BFfOJ+QI1gXjXPD0+AZuIc9tiH+8HPYp8becK5sGS6aOiczgCffkhXOax7hnTJ+dFm/gv3l2eK2/ELcC6ZG1bnydziqajFZ4XwM+Qe0D5GDsA1Wl2eLOyCHcEAXwRi8hI2mTrLC0/BuOxmH7WrMw3Is2a/XU3HhG6Zn4S19irSLS8CPyDScgyVYm+gWFPx1fgDSY1owj3vgygAAAABJRU5ErkJggg==>

[image18]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABUAAAAZCAYAAADe1WXtAAABGElEQVR4Xu2TsWoCQRCGR4wgKMRCBMFCJAhRbAy+gUgsbPQBfIE8hC/gM0iq1CkEC2sLexGskjYhpTZi9B92925277iD64T74OPYmfH3du+OKCVFkoFPblHA/QrswqzTC8DDLfgJv+yWRx1u4Dd8h1vYlAOSDrzAE7yS+lEYB7iGj3q9gHtY8yYEOViEL/BI4aEPpP5wLGp9+A9nohYgKtT0+OrW/uCzqFskDeVj64m6RVRog9RWecsGDjLPYSTqFlGhYWf6pmuJQxn36a9Izbs7sIgLNfCbUiJ/fgfL1oQgLnQIq2LNR8Fbn4lagLhQDljCvF5/wF/Y9iYc+DN9hWf4o9cuHDonv8c3MPHbySjAAZzqa8o9cwPOjkPLHoOY8AAAAABJRU5ErkJggg==>

[image19]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACUAAAAZCAYAAAC2JufVAAACGUlEQVR4Xu2Vv0uWURTHj1RQg/QTRI0CsaAlqChBBxeFBguswcE/IJqDok1obWgIgoagKTFxcShQaGtxEKdCHRRCKCgnx7Lvx3vv+9z3Ps/1carB5wsfeM+55z733HvPua9Zo0aN/r+OiPNiWHQnY7GOiUvirOhIxup0SlwXvea+U6uvYkv8ELtiwtoX5feY+C6++Zh1c5s4iM6IbfFWLIu19uGyronByOb3b3E/8t0VK6Lf2yRNDAvdCkEZnRSL4qK32eBTc+tWioCXYijxcxJfRJe3P4oP4rK3wzziGDvh/VWaMhcX64p4J44m/j3x8RfilbUH7Jj70B1vz3v7USvCjeEj9kbkj0V9blo5Kfx/xEjib4nE0qJlQrwYjdBZDO/pgbnFNswVb5X2SwrfvcSfFQkyYcbyXcJ1cW3EPbPypoJOiyUrJ0Wn43uS+LMaEKtWFGaVKHROkwKmkPcTC6dJ3fa+AyVFInUJIbqOwq9LCKXdhz1tLilKICuu6bV4b8VCnFhaK4wtWHGtNAedyztUp+NWPLrU1E9zXZgV7wZJhdZmsTfiZivCjRFDbBA1M2euRnJi3mji47vZJwGR+ay4YG4HcFV8Fn0+Jpzkc9ETxdHSnBzJIeZ9Eg+9jSbNXdW5yDdl5UTbxMvNpBS6Jiz2uGI8wIbCjvkbwcczEETb4wu3wD/AL8t37D8TJ82JjZurq0aNDrf+Aua+aETDpKgaAAAAAElFTkSuQmCC>

[image20]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACUAAAAZCAYAAAC2JufVAAABg0lEQVR4Xu2UzysFURTHj1BEWbBRZNiSFCnWtmyesiAbJdYK/4B/wIaFEitlJVvlx8rOylokG2WpWOD77d6nM+fO8G7ekzKf+vRmzsy8931z7rkiBQUFf48FWzAksM4WK6AJDnijSOCeLXrq4Tg8g63pS9+yCV/hkXckfTmfRrgrYagJ+ADf4LPEh2qDF7DLnw/BE9jyeccXzMBHCUPxtXf6z0mJC9UAt+Cwqq3Ba9ihagF8Q9twTtyP2lCa2FAb8N0WK6EE98WFq2YotodtYqgVeCOu/ef6pjxOYb8/rmYotvxWXKh1cYPSDHfgqLovgD1fUue1CsUwZfrgpbgByGRK0g/UKpStc5I51QH8Ae4dd8onVTuQcGxjQuk1peGzrHEKA8qjrp2Hh/64XcKdOyYUyZo+bgWszZp6Lj9p36C4oVlWtTH4os4JN9F72GvqmXBL4KI/lrBthNOzCK9gj7lG+Gf4BriOynC98vsSVeMmvSphF34d7uhs1zTsNtcKCv4XH6QsTr9GnXGyAAAAAElFTkSuQmCC>

[image21]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAoAAAAbCAYAAABFuB6DAAAAx0lEQVR4Xu3RL++BURjG8WPMGJuuGG9BM0EgURRFkb0IL+IXBJtggiKoKF6EqAuSoNj8+d7P497vdjb/qrm2T7nOtbOd53HulzcSQdYvScov0lhhhgGGWONoNkF0eDH2aJhNEBnKbS/z8bCOPv5QQtSOJDJcoIccWthhbEcSGc69ruPCR8mne5oiDsj7B34K2KKqhf2GcS3d/41lLWKYYKnFLXLTGRlb1lx4qyaJKU6mCyIv62Lkwv+8cQ9+oaaJNipI3B99Ua6kfiLt1VZxUgAAAABJRU5ErkJggg==>

[image22]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABoAAAAZCAYAAAAv3j5gAAABHUlEQVR4Xu2ULQvCUBSGj4hBEBQ/gt0kCIJBMJg1WCwKFps/wX9iMxgsYhGLUUyi2a5VsAgaFD/e4x3bvdf5MZBh2AMPY/ee3Xd32xmRh8c/4IMp6NcnPhCBBRjQJ3Q4IA1HcA2TyuxronBA4pou3MGyXCCTgRd4hDe4oe+CgnAMlzBhjDXhHuaNcwXebgjm4IG+DyrBK2xJY3G4ghMSN2KL0yDeCdfzdTI9Ek+moo2bOA3iundBbW3cxGkQv3hXgrjWlSD+ulwJsvsYuB/7JIKq0riC06AhiQXlBuU2mdLzDSh8CuqQeCxh47xOIqhhVlh9NCerToG3zA14hluYVacf8KJyf8TgAs7IWrQGT8bx5/CfpUhiZxzu4fGaO7iOTV/OwfqNAAAAAElFTkSuQmCC>

[image23]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACYAAAAZCAYAAABdEVzWAAACTElEQVR4Xu2VT4jNURTHv0LR0BCNRNHsZPxLaMIOZUGKUlKWY02RrGVnITJJaRaThT9ZYKW8UlIWIqJQWLDQNKXYKHy/nXO8++7c+7xENr9vferd87vvnnPPOfdeoFGjRn9d88j03JhpVm7IpUUW5MaKppEBMoyy46PkOVlETpDXZA+Z6d/1/9XkMbngtqrWkB/kA3mf8ZSs8nnLyQPyjFwkr8ha/yYthAU15uNB0iLnYAFqvY+wNV+QpT6vqm2wwErcJf0+TwFdI7N9fJq8gQUsrSdfyHEfzyFXyQofS/qvbLsTW1UjZJ+zA7aTzeQlWZfM+w7bREgOJ2AZUYlKgd10e0ilPk9mJLaqDmRjORlFuy8kleUdWZzY5LgFC0bOldmHZBy2xhC57vOkW+gxUzVtgjV4KjmuBaaS73KbNvmJbIT14UG3Sz1nqiSdTu06l0rYS2DK1H5Yo59EO+tqid82ezcdhjV0LjnuJbCSVOLbyXgDeUImSV9iryp6RCcv1078WWDK4DFYGaWVsGzuhd2DZ3xOV52FOTmSf0C5+eeS++QrLAu5IlPR8PPJI9hm4kBcQudJn6JwUtu9Fv0Mu4xDClLBviVLEnsoMhUNH/NbaAd2CmV/vxR/qgUmfSNbknHcY1dQPm330Hm69Uypf1toB3YZ1iZVKRPKSLfA9GRpoThpegt1h+l6yaUybs9serRVWlVGFZLyl2GK1IBbYfUuPc4hnSK9Dof8d0m6Gu7kRpfWVomVuRtkWefnfytltBZ0SKVUaRs1+i/6CekyeBKYI2VgAAAAAElFTkSuQmCC>



## 结论

**PageIndex 有实用价值，但更适合作为“长文档专业检索/问答”的专项组件，而不是通用 RAG 的全面替代品。**

适合：金融年报、SEC filing、法律合同、政策文件、技术手册、产品白皮书、招投标文件、复杂 PDF 知识库。
不适合：海量碎片化网页、短 FAQ、高并发低延迟搜索、频繁更新的动态知识库、对实时检索速度要求很高的客服系统。

我会给它一个分层评分：

| 使用方式           |    实用性评分 | 判断                          |
| -------------- | -------: | --------------------------- |
| 技术概念验证 / Demo  | **8/10** | 思路清晰，适合快速验证长文档问答            |
| 企业内部长文档分析      | **7/10** | 场景匹配时价值明显，但要控制成本、延迟和 OCR 质量 |
| 直接用 OSS 自建生产系统 | **5/10** | 开源仓库还偏早期，生产化能力需要补很多工程       |
| 替代全部向量数据库 RAG  |  **不建议** | 更适合做混合架构中的“精读/深搜层”          |

---

## 它到底解决什么问题

PageIndex 的核心思路是：**不把文档切成小块再做 embedding 相似度搜索，而是把文档转成类似目录的层级树，再让 LLM 沿着树结构推理检索。** 官方描述是“无向量数据库、无 chunking、基于推理的 RAG”，主要用于长专业文档。项目 README 明确说它先生成文档的 Table-of-Contents 树，再通过 tree search 做推理检索。([GitHub][1])

这个思路对“结构化长文档”确实有意义。传统向量检索经常遇到两个问题：语义相似不等于真正相关，以及 chunk 切分破坏上下文。PageIndex 的优势在于保留章节结构、页码、节点摘要和检索路径，所以更容易做可解释引用。官方也把适用文档列为金融报告、监管文件、学术教材、法律/技术手册等超过 LLM 上下文限制的文档。([GitHub][1])

---

## 实用性判断

### 1. 真实痛点成立

长文档 RAG 的痛点是真实存在的，尤其是金融、法律、政府、制造业技术资料。FinanceBench 原始论文也指出，金融问答需要领域知识、最新信息、数值推理、表格理解、多源长文本推理等能力；这些正是普通 LLM 和简单 RAG 容易失败的地方。([ar5iv][2])

Patronus AI 对 FinanceBench 的介绍也说明，金融场景下错误回答会带来下游交易和决策风险，原始测试中 GPT-4-Turbo 加检索系统错误回答或拒答了 81% 的问题。([docs.patronus.ai][3])

所以 PageIndex 面向的问题不是伪需求。

### 2. 场景匹配时效果可能明显

PageIndex 背后的 Mafin 2.5 在 FinanceBench 上声称达到 **98.7% accuracy**，项目方也开源了评测结果。其 Mafin2.5-FinanceBench 仓库说明该评测基于 FinanceBench，并称采用更接近真实金融应用的设置：所有文档存储在单个数据库中，在 FinanceBench public set 上测试。([GitHub][4])

但这里要注意：这是**项目方自报评测**，不是第三方审计。另一个细节是，Patronus 文档称 FinanceBench 全量数据集包含 10,231 个问题，而开源支持的是样本/子集，完整 benchmark 需要授权。([docs.patronus.ai][3]) 所以 PageIndex 说的“100% full benchmark”应理解为其公开评测集覆盖，而不是一定等于完整商业授权数据集。

### 3. 开源工程还不算成熟

GitHub 主仓库热度很高：约 **33.1k stars、2.9k forks、292 commits**，MIT license；但同时没有正式 release。([GitHub][1]) 开源包依赖也很轻，主要是 litellm、PyMuPDF、PyPDF2、python-dotenv、pyyaml。([GitHub][5])

这意味着它更像一个快速迭代的研究/产品前沿项目，而不是传统意义上稳定成熟的企业级开源框架。Issues 里也能看到一些生产化问题：大 PDF 因 LLM 上下文限制生成不完整树、增量索引需求、大 workspace list_documents 不可用、LLM 并发导致 429、依赖冲突等。([GitHub][6])

---

## 主要优势

| 优势               | 实际价值                          |
| ---------------- | ----------------------------- |
| 保留文档层级结构         | 对年报、合同、政策文件、手册很重要             |
| 检索路径可解释          | 比纯向量 top-k 更容易审计              |
| 不依赖向量数据库         | 降低一部分基础设施复杂度                  |
| 适合“相关性”而非“相似性”检索 | 能处理“问法和答案表述不相似”的情况            |
| 可接 MCP/API       | 更容易接入 Claude、Cursor、Agent 框架等 |

PageIndex 还有 MCP 项目，说明可以通过 MCP 接入 Claude、Cursor、OpenAI Agents SDK、LangChain 等工具；PageIndex MCP 仓库也有独立 release，最新版本为 2026 年 5 月 28 日的 v1.8.1。([GitHub][7])

---

## 主要风险

### 1. 成本和延迟

PageIndex 把“检索”从 embedding 相似度计算变成 LLM 推理树搜索。好处是精度和可解释性提升，坏处是：

* 建索引需要 LLM；
* 查询也可能多轮调用 LLM；
* 对海量文档和高并发场景，成本与延迟压力更大；
* 不像向量数据库那样天然适合毫秒级 top-k 检索。

官方快速使用方式也要求设置 LLM API key，默认模型参数是 `gpt-4o-2024-11-20`，并支持设置 max pages per node、max tokens per node 等参数。([GitHub][1])

### 2. PDF 解析/OCR 是关键瓶颈

README 明确提示：开源包使用标准 PDF parsing；复杂 PDF 场景，官方云服务有增强 OCR、树构建和检索能力。([GitHub][1])

这点很重要。很多企业真实文档是扫描件、表格、图片、盖章合同、混合版式 PDF。如果 OCR 和版面结构解析做不好，后面的树结构再先进也会出错。

### 3. 多文档能力需要谨慎验证

PageIndex 最初最适合“单个长文档内的结构化检索”。虽然官方在 2026 年 5 月发布了 PageIndex File System，称 Enterprise 可支持百万级文档，并通过虚拟节点、query-dependent tree 等机制扩展到大规模文档库，但这部分主要是企业版/云服务能力，不等于主仓库开源能力。([PageIndex][8])

GitHub issue 里也有用户反馈：把三个文档的树拼在一起后，跨文档问答只遍历第一棵树。([GitHub][9]) 这说明自建版本在多文档编排上需要额外工程设计。

### 4. “无 chunking”不要字面理解

它不是完全没有分段，而是不做传统固定长度 chunk。实际仍然会有节点、页范围、max pages per node、max tokens per node 等限制。更准确的说法是：**按文档自然结构建树，而不是粗暴切块。**

---

## 真实案例与可参考评估

### 案例 1：Mafin 2.5 / FinanceBench

这是 PageIndex 最核心的公开案例。Mafin 2.5 是基于 PageIndex 的金融文档 RAG 系统，项目方称在 FinanceBench 上达到 98.7% accuracy，且用 GPT-4o 和 DeepSeek v3 作为底座都达到同样结果。([GitHub][4])

**可参考，但不能完全等同于第三方背书。** 它证明了该方法在金融年报类结构化文档中很有潜力，但仍需要在自己的数据集上复测。

### 案例 2：Dewey 的 agentic RAG FinanceBench 评估

Dewey 在 2026 年 5 月发布了 FinanceBench 评估，用 agentic retrieval 做金融分析；其 Claude Opus 4.6 配置达到 87.3%，高于传统向量 RAG和 full-context baseline。([Dewey][10])

这不是 PageIndex 案例，但说明一个行业趋势：**金融长文档问答正在从简单向量 RAG 转向 agentic / section-summary / iterative retrieval。** PageIndex 属于这个方向。

### 案例 3：FinSage 金融 filings RAG

FinSage 是一个面向金融 filings 的多模态、多路径 RAG 系统，包含多模态预处理、稀疏+稠密检索、query expansion、metadata-aware semantic search 和专用 reranker。论文称其在专家问题上达到 92.51% recall，并已作为在线会议金融问答 agent 服务超过 1,200 人。([arXiv][11])

这说明 PageIndex 不是唯一方向。金融文档场景里，强工程化的 hybrid RAG、reranker、多模态解析同样有效。

### 案例 4：Ragie FinanceBench

Ragie 用传统/混合 RAG 工程方案处理 FinanceBench，称 360 个 PDF、50,000+ 页在 4 小时内完成 ingestion；在 shared-store retrieval 上达到 27% accuracy，高于 benchmark 的 19%。([Ragie][12])

这个案例说明：如果目标是大规模 ingest、低成本、高吞吐，传统向量/混合检索仍然有现实价值。PageIndex 的优势更多在“精确长文档分析”，不一定在“便宜快速的大规模检索”。

---

## 是否值得用

### 值得试的场景

* 企业内部长 PDF 问答；
* 金融报告自动分析；
* 合同条款定位与比较；
* 政策/法规/招投标文件问答；
* 技术手册、产品手册、SOP 检索；
* 需要“答案可追溯到章节/页码”的专业场景；
* 不想一开始就搭建向量数据库的 PoC。

### 不建议直接用的场景

* 海量网页知识库；
* 短文本 FAQ；
* 实时客服高并发；
* 频繁新增/删除/更新文档；
* 严重依赖表格、图片、扫描件 OCR 的业务，除非先验证 OCR；
* 跨上万/百万文档的企业知识库，除非使用官方企业版或自己重做文件系统层。

---

## 推荐落地方式

不要把 PageIndex 当成“替代向量数据库”的唯一方案，更合理的是：

```text
文档解析/OCR
→ 结构化清洗
→ PageIndex 建长文档树
→ BM25/向量/元数据检索做粗召回
→ PageIndex 对候选长文档做深度树搜索
→ LLM 生成答案 + 页码/章节引用
```

也就是：**向量检索负责“先找到可能相关的文档”，PageIndex 负责“在长文档里精确读懂和定位”。**

---

## 建议验证清单

做 PoC 时不要只跑官方 demo，应至少验证这些指标：

| 指标        | 建议目标                             |
| --------- | -------------------------------- |
| 文档类型      | 选 20–50 份真实 PDF，不要只用干净英文 PDF     |
| 问题集       | 人工设计 100 个真实业务问题                 |
| 对照组       | 普通 RAG、BM25、hybrid RAG、PageIndex |
| 指标        | 命中率、答案准确率、引用页码准确率、拒答率            |
| 成本        | 每份文档建索引成本、每个问题平均 LLM 调用成本        |
| 延迟        | P50 / P95 查询耗时                   |
| 更新        | 新增、删除、替换文档是否方便                   |
| 多文档       | 是否能跨文档比较、归纳、汇总                   |
| 中文/泰文/扫描件 | OCR 和目录结构是否稳定                    |

---

## 最终判断

**PageIndex 是一个值得关注和试用的长文档 RAG 项目，尤其适合金融、法律、政策、技术文档等“结构强、篇幅长、需要引用依据”的场景。**

但它目前不应被理解为“向量数据库终结者”。更现实的定位是：

> **普通 RAG 负责广覆盖召回，PageIndex 负责专业长文档深度检索与可解释问答。**

对于商业化或企业项目，建议从 **20–50 份真实业务文档 + 100 个真实问题** 开始做对比测试。若准确率、引用质量明显优于现有 RAG，再考虑接 MCP/API 或企业版；不建议直接基于开源仓库裸奔上线。

[1]: https://github.com/VectifyAI/PageIndex "GitHub - VectifyAI/PageIndex:  PageIndex: Document Index for Vectorless, Reasoning-based RAG · GitHub"
[2]: https://ar5iv.org/abs/2311.11944 "[2311.11944] FinanceBench: A New Benchmark for Financial Question Answering"
[3]: https://docs.patronus.ai/docs/research_and_differentiators/financebench "FinanceBench"
[4]: https://github.com/VectifyAI/Mafin2.5-FinanceBench "GitHub - VectifyAI/Mafin2.5-FinanceBench: FinanceBench evaluation of Mafin 2.5 (Powered by PageIndex) · GitHub"
[5]: https://github.com/VectifyAI/PageIndex/blob/main/requirements.txt "PageIndex/requirements.txt at main · VectifyAI/PageIndex · GitHub"
[6]: https://github.com/VectifyAI/PageIndex/issues "Issues · VectifyAI/PageIndex · GitHub"
[7]: https://github.com/VectifyAI/pageindex-mcp "GitHub - VectifyAI/pageindex-mcp: MCP server for PageIndex. PageIndex is a vectorless reasoning-based RAG system which uses multi-step reasoning and tree search to retrieve information like a human expert would. · GitHub"
[8]: https://pageindex.ai/blog/pageindex-filesystem "PageIndex File System:
 Massive-Scale Document Search"
[9]: https://github.com/VectifyAI/PageIndex/issues/187 "Multiple Document Chat · Issue #187 · VectifyAI/PageIndex · GitHub"
[10]: https://meetdewey.com/blog/financebench-eval "Evaluating agentic RAG for financial analysis: a FinanceBench study — Dewey"
[11]: https://arxiv.org/html/2504.14493v1 "FinSage: A Multi-aspect RAG System for Financial Filings Question Answering"
[12]: https://www.ragie.ai/blog/ragie-outperformed-financebench "How Ragie Outperformed the FinanceBench Test"
