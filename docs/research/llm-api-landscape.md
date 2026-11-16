# 研究：LLM API 供应商能力、价格与内容政策

- 对应 issue：#3「研究：LLM API 供应商能力、价格与内容政策」（父地图：#1）
- 研究日期：2026-11-16（除特别注明外，所有价格均于当日在官方页面或注明核价日期的二手来源确认）
- 分支：`research/llm-api-landscape`（不合并）
- 场景假设：AI 角色扮演移动应用（角色卡 + 长线叙事对话），中国大陆首发，后续可能出英文市场

**来源分级约定**：本文每条事实标注来源性质——
- 【一手】厂商官方定价页、官方文档、官方协议、监管公示，均为 2026-11-16 当日抓取；
- 【媒体】媒体报道（转述官方数据或采访）；
- 【第三方核价】价格追踪站/评测站，注明其自述核价日期；
- 【社区口碑】酒馆（SillyTavern）社区、GitHub issue、评测博客等，非受控测试，只作参考；
- 【推测】作者基于事实的推断。

**汇率口径**：国内厂商人民币价与美元价并存时分别标注；粗略换算按 1 美元 ≈ 6.9~7.1 元人民币（仅用于量级对比，非官方口径）。

---

## 0. 执行摘要

- 国内圈五家（DeepSeek、Kimi、智谱、通义、豆包）均已完成生成式 AI 备案、提供商用 API、OpenAI 兼容协议与流式输出；**内容政策是共同基线而非差异化约束**——五家的服务条款都禁止色情/未成年人性化内容，且都要求「调用方自行承担面向公众服务的内容审核责任」。blur 必须自建内容审核层，选型时审核严格度不构成差异化变量，真正的变量是**价格、上下文长度、速率限制、中文文风**。
- **小模型档位短名单（日常陪伴对话主力）**：① DeepSeek `deepseek-flash`（最低价 + 1M 上下文 + 2500 并发）；② 智谱 `GLM-5.3-Flash`（¥0.8/¥2.8 每百万 token，最便宜）；③ 火山方舟 `doubao-seed-character`（角色扮演专用模型，分段计价 ¥0.8 起）。
- **旗舰档位短名单（关键剧情/付费增值）**：① DeepSeek `deepseek-v4-pro`（峰谷价差大，可闲时调度）；② Kimi `kimi-k3.1-pro`（1M 上下文 + 中文文风社区口碑）；③ 智谱 `GLM-5.3`；备选 通义 `qwen3.8-max`、豆包 `doubao-seed-2.1-pro`。
- 国际圈（OpenAI / Anthropic / Gemini / OpenRouter）价格整体高一个数量级，且 Anthropic 明确禁止「erotic chats」、OpenAI 明确禁止未成年人性/暴力角色扮演；仅作英文市场对照与备份渠道存档。

---

## 1. 国内圈（首发渠道的硬约束）

### 1.1 合规基线：备案与「调用已备案模型」的责任划分

- 按《生成式人工智能服务管理暂行办法》，面向公众提供生成式 AI 服务须完成备案；**通过 API 调用已备案模型能力的应用/功能，走「登记」而非重新备案**，但应在显著位置公示所用模型的名称与备案号。【一手·国家网信办公告，2024-04-02】https://www.cac.gov.cn/2024-04/02/c_1713729983803145.htm
- 截至 2025-03-31，共有 346 款生成式 AI 服务在国家网信办完成备案、159 款应用/功能在地方网信办完成登记。【媒体转载网信办公告，2025-04-08】https://mp.weixin.qq.com/s/0gXrOIOoiasjQfz9HH0Ttg
- 备案信息可在「互联网信息服务算法备案系统」自查。【一手】https://beian.cac.gov.cn/
- 本研究涉及的五家模型/算法的备案证据（均为第三方页面引用的备案号，可在上述系统复核）：
  - DeepSeek 大语言模型算法：网信算备110108970550101240011号。【第三方·BeeBook 用户协议披露页，2026-11-16 查询】https://beebook.ai/terms.html
  - 通义千问：ZheJiang-TongYiQianWen-20230901。【第三方·乐播投屏隐私政策披露，2026-11-16 查询】https://activity.hpplay.cn/pro-protocol/1784282749996.html
  - 豆包（云雀）：Beijing-YunQue-20230821。【第三方·IMA GEO 合规披露页，2026-11-16 查询】https://imative.net/geo/legal/ai-disclosure
  - 智谱交互式内容生成算法：网信算备110108105858001230027号。【一手·阿里云百炼「模型备案信息公示」页，2026-11-06 更新】https://help.aliyun.com/zh/model-studio/model-filing-information-publicity
  - Kimi：官方设有「算法及模型备案信息公示」页声明「我们已完成备案的模型如下」。【一手，2026-11-16 查询】https://www.kimi.com/user/agreement/algorithm-and-model
- 重要边界：供应商协议把审核责任压给调用方。Kimi 开放平台协议明确：「您知悉使用服务向公众提供内容生成式人工智能产品或服务的需要开展安全评估和算法备案工作……建立包括但不限于内容审核、用户管理、数据安全、监测预警和应急处置等机制」，违规可被冻结账号。【一手·Kimi 开放平台服务协议，版本生效日期 2026-08-31，2026-11-16 查询】https://platform.moonshot.cn/docs/agreement/modeluse

### 1.2 DeepSeek（深度求索）

官方定价页（美元计价）【一手，2026-11-16 查询】https://api-docs.deepseek.com/quick_start/pricing

| 模型 | 上下文 | 最大输出 | 输入（缓存未命中） | 输入（缓存命中） | 输出 | 并发上限 |
|---|---|---|---|---|---|---|
| `deepseek-flash`（当前版本 DeepSeek-V4.1-Flash-1017） | 1M | 384K | 闲时 $0.15 / 高峰 $0.30 | 闲时 $0.003 / 高峰 $0.006 | 闲时 $0.6 / 高峰 $1.2 | 2500 |
| `deepseek-v4-pro` | 1M | 384K | 闲时 $0.66 / 高峰 $1.32 | 闲时 $0.022 / 高峰 $0.044 | 闲时 $1.98 / 高峰 $3.96 | 500 |

- 峰谷定价：高峰为 UTC 01:00–04:00、06:00–10:00 周一至周五（即北京时间工作日 09:00–12:00、14:00–18:00），其余含周末与中国法定节假日均为闲时，闲时价为高峰一半。【一手，同上】
- 旧模型名 `deepseek-chat` / `deepseek-reasoner` 已于 2026-07-24 退役，调用须用 `deepseek-flash` / `deepseek-v4-pro`，思考模式以参数切换。【第三方核价·permillion 追踪页，2026-11-16 查询】https://permillion.net/models/deepseek-reasoner/
- 人民币口径（火山方舟托管价，与官方峰谷一致）：`deepseek-v4-1-flash` 闲时 ¥1.0/¥4.0、高峰 ¥2.0/¥8.0，缓存命中闲时 ¥0.02；`deepseek-v4-pro 正式版` ¥9.0/¥27.0。【一手·火山方舟模型价格页，2026-11-16 查询】https://www.volcengine.com/docs/ark/model-pricing
- 能力：OpenAI 格式 + Anthropic 格式双端点；流式输出（SSE keep-alive）；Tool Calls / JSON Output；Flash 支持视觉输入。【一手·官方定价页，同上】
- 速率限制：按账号计并发（flash 2500 / pro 500），超限返回 HTTP 429；支持 `user_id` 参数做业务侧用户级的内容安全隔离、KVCache 隔离与调度隔离；可申请免费扩容。【一手·官方限速文档，2026-11-16 查询】https://api-docs.deepseek.com/quick_start/rate_limit
- 内容政策：服务条款 §3.4(5) 明确禁止「pornographic, obscene, or sexually explicit (e.g., sexual chatbots)」；§3.4(8) 禁止「专门面向或扮演未满 18 岁人设的聊天机器人」；§3.3 声明会用技术手段（风险过滤机制、违法内容特征库）审查使用行为。【一手·DeepSeek Terms of Use，最近更新 2026-03-27，2026-11-16 查询】https://cdn.deepseek.com/policies/en-US/deepseek-terms-of-use.html
- 输出权利：§4.2 输出权益归用户，且明确允许蒸馏等广泛用途（在合规前提下）。【一手，同上】
- 中文 RP 口碑：酒馆社区基准「AI Roleplay Bench」（2026-09，14 款模型 840 条回复）中 DeepSeek V4.1 Flash 与 Kimi K3、GLM-5.3 分差小于 2 分，基准方建议视为平手。【社区口碑·gptproto 博客，2026-11-06】https://gptproto.com/blog/best-ai-model-for-sillytavern
- 审核严格度口碑：DeepSeek 官方 API 存在输入与输出双向拦截，社区抱怨其内容筛选偏保守（尤其政治相关）。【社区口碑·GitHub DeepSeek-V2 issue #51，2024-05-30，年代较早仅供参考】https://github.com/deepseek-ai/DeepSeek-V2/issues/51

### 1.3 Kimi（月之暗面）

官方定价页（人民币计价）【一手·页面最后修改 2026-11-12，2026-11-16 查询】https://platform.moonshot.cn/docs/pricing/chat

| 模型 | 上下文 | 输入 | 输出 | 缓存命中 | 缓存写入（5min / 1h） |
|---|---|---|---|---|---|
| `kimi-k3.1-pro` | 1,048,576 | ¥13.00 | ¥68.00 | ¥1.30 | ¥13.00 / ¥26.00 |
| `kimi-k3.1-pro-highspeed` | 1,048,576 | ¥26.00 | ¥136.00 | ¥1.30 | ¥26.00 / ¥52.00 |
| `kimi-k3` | 1,048,576 | ¥20.00 | ¥100.00 | ¥2.00 | ¥20.00 / ¥40.00 |
| `kimi-k2.7-code` | 262,144 | ¥6.50 | ¥27.00 | ¥1.30 | — |
| `kimi-k2.6` | 262,144 | ¥6.50 | ¥27.00 | ¥1.10 | — |

- 单位均为每百万 token（MTok）；K3 系列支持批量推理（Batch API），各项费用为标准价 60%。【一手，同上】
- **K2 系列退役公告**：2026-10-31 起停止向新注册用户开放，2026 年 12 月底全线下线。【一手·官方定价页 Note，同上】
- 速率限制按累计充值分级（2026-10-23 更新的官方页面）：Tier0（¥0）并发 1 / RPM 3 / TPM 50 万 / TPD 150 万；Tier1（¥50）并发 15 / RPM 100 / TPM 200 万；Tier3（¥500）并发 50 / RPM 200 / TPM 300 万；Tier5（¥20,000）并发 100 / RPM 300 / TPM 500 万；联网搜索 QPS 独立计数。【一手，2026-11-16 查询】https://platform.moonshot.cn/docs/pricing/limits
- 按量计费，无订阅制 API；需实名认证后充值。【一手·平台协议，同上】
- 中文 RP 口碑：Kimi K 系在中文酒馆社区文风口碑较好；上述 AI Roleplay Bench 中 Kimi K3 与 DeepSeek V4.1 Flash、GLM-5.3 基本平手。【社区口碑，同上 gptproto】
- 审核严格度：官方文档未见独立的「API 层强制过滤开关」说明；平台协议要求开发者自建审核（见 §1.1）。模型自带安全对齐的具体拦截尺度未找到一手量化资料——**待实测**。【推测：基于文档缺失的判断】

### 1.4 智谱 GLM（BigModel / Z.ai）

> 说明：国内站 open.bigmodel.cn/pricing 为 JS 动态渲染，本环境无法直接抓取；以下以智谱国际官方站 Z.ai 定价页（美元，官方一手）+ 火山方舟官方托管价（人民币，官方一手）+ 注明日期的第三方核价交叉确认。

- Z.ai 官方定价页【一手，2026-11-16 查询】https://docs.z.ai/guides/overview/pricing ：

| 模型 | 输入 | 缓存命中 | 输出 |
|---|---|---|---|
| GLM-5.3 | $1.4 | $0.26 | $4.4 |
| GLM-5.3-Flash | $0.15 | $0.03 | $0.50 |
| GLM-5.3-FlashX | $0.37 | $0.075 | $1.25 |
| GLM-4.7 | $0.6 | $0.11 | $2.2 |
| GLM-4.7-Flash | 免费 | 免费 | 免费 |
| GLM-4.5-Flash | 免费 | 免费 | 免费 |

（缓存存储限时免费。）

- 国内站人民币刊例（第三方核价，引用 bigmodel.cn 官方定价页）：GLM-5.3 输入 ¥8 / 输出 ¥28 / 缓存命中 ¥2（2026-08-21 起）；GLM-5.3-Flash 输入 ¥0.8 / 输出 ¥2.8 / 缓存命中 ¥0.23。【第三方核价·稀土掘金引官方定价页，2026-08-27】https://juejin.cn/post/7678604384506839055 【第三方核价·ChooseAI，2026-09-22】https://www.chooseai.io/news/7340/
- 交叉确认：火山方舟官方托管 `glm-5.3-flash` 定价同为 ¥0.8/¥2.8、缓存命中 ¥0.23；`glm-5.2` ¥8.0/¥28.0。【一手·火山方舟模型价格页，2026-11-16 查询】https://www.volcengine.com/docs/ark/model-pricing
- 上下文：GLM-5.3 系列为 1M 上下文（第三方资料一致）。【第三方·DataLearner，2026-10-12】https://www.datalearner.com/ai-models/pretrained-models/glm-5-3-flashx-legal
- 免费模型 GLM-4.7-Flash（200K 上下文）可作为开发期/兜底零成本通道。【第三方核价·okfinish，2026-05-31，与 Z.ai 官方页一致】https://okfinish.com/article/aiInterface
- GLM Coding Plan 订阅 118–1078 元/月（按积分计费），为编程场景套餐，与 RP 场景关系不大。【第三方核价·ChooseAI，2026-09-22】https://www.chooseai.io/news/7340/
- 中文 RP 口碑：AI Roleplay Bench 中 GLM-5.3 与 Kimi K3、DeepSeek V4.1 Flash 基本平手。【社区口碑，同上 gptproto】
- 审核严格度：未找到 API 层强制过滤的一手说明——**待实测**。【推测：基于文档缺失的判断】

### 1.5 通义千问（阿里云百炼 / DashScope）

官方计费页【一手，2026-11-16 查询】https://help.aliyun.com/zh/model-studio/billing （华北 2·北京地域，人民币每百万 token）：

| 模型 | 输入 | 输出 | 备注 |
|---|---|---|---|
| `qwen3.8-max` / `qwen3.7-max` | ¥12 | ¥36 | 0–1M 输入统一价；Batch 半价；缓存享折扣；免费额度 100 万 token（90 天） |
| `qwen3.5-plus` | ¥0.8 | ¥4.8 | 0–128K 档 |
| `qwen-plus`（= 2025-12-01 快照） | ¥0.8 | ¥2（非思考）/ ¥8（思考） | Batch 半价 |
| `qwen-plus-character`（角色优化版） | ¥0.8 | ¥2 | 全球可用 |
| `qwen3.5-flash` | ¥0.2 | ¥2 | Batch 半价；缓存折扣 |
| `qwen-flash` | ¥0.15 | ¥1.5 | Batch 半价；缓存折扣 |

- 阶梯计费：单价按单次请求输入 token 总量分档，整单按所达档位结算。【一手，同上】
- 上下文缓存：显式缓存创建按标准输入价 125% 计费、命中按 10% 计费（缓存计费规则见官方「上下文缓存」文档，转述自计费页说明）。【一手，同上】
- 内容审核：百炼提供「AI 安全护栏」（输入检测/输出检测），业务空间维度两个开关**默认均未勾选**，可按需开启；另有面向百炼用户的安全护栏服务，说明平台存在「红线管控政策」基线。【一手·阿里云帮助中心，2026-11-10】https://help.aliyun.com/zh/model-studio/content-security 【一手·阿里云帮助中心，2026-10-10】https://help.aliyun.com/zh/document_detail/2923687.html
- 区域：北京/弗吉尼亚/新加坡/法兰克福/东京/香港等多地域部署，国际/美国站价格显著更高（如 qwen-plus 国际站 ¥2.936/¥8.807）——出海时成本会变。【一手·计费页，同上】
- 中文 RP 口碑：`qwen-plus-character` 为官方角色扮演优化 SKU，说明阿里对该场景有专门供给；RP 质量的独立社区评价本次未找到高质量来源——**待实测**。【推测：基于产品线存在的判断】

### 1.6 豆包（火山方舟 / 字节跳动）

官方模型价格页【一手，2026-11-16 查询】https://www.volcengine.com/docs/ark/model-pricing （人民币每百万 token，在线推理·常规）：

| 模型 | 输入 | 缓存命中 | 输出 | 备注 |
|---|---|---|---|---|
| `doubao-seed-2.1-pro` | ¥6.00 | ¥1.20 | ¥30.00 | 输入 0–1024K；低优通道半价（¥3/¥15） |
| `doubao-seed-2.1-turbo` | ¥3.00 | ¥0.60 | ¥15.00 | 输入 0–256K |
| `doubao-seed-2.1-lite` | ¥0.80 | ¥0.16 | ¥2.70 | 输入 0–1024K；低优通道 ¥0.4/¥1.35 |
| `doubao-seed-character`（角色模型） | ¥0.80（0–32K）/ ¥1.20（32–128K） | ¥0.16 | ¥2.00 / ¥6.00 | 分段计费 |
| `doubao-seed-2.0-lite` | ¥0.6（0–32K）起 | ¥0.12 起 | ¥3.6 起 | 分段计费 |

- 缓存存储统一 ¥0.017/M token/小时；批量推理约为常规价 50%。【一手，同上】
- 火山方舟同时托管 `deepseek-v4-1-flash`（闲时 ¥1/¥4）、`glm-5.3-flash`、`kimi-k3.1-pro` 等第三方模型——**火山方舟可作为国内圈的多模型聚合入口，一家合约覆盖多家模型**。【一手，同上】
- 内容审核：火山方舟提供「内容安全护栏」，官方描述为「在豆包模型原生安全能力之上……自动识别并过滤涉敏、涉黄等风险」，即豆包模型自带原生安全层 + 平台护栏可叠加。【一手·火山引擎文档，2026-09-15】https://www.volcengine.com/docs/ark/content-safety-guardrails
- 中文 RP 口碑：`doubao-seed-character` 的存在说明字节对角色扮演场景有专门模型供给；独立质量评价未找到——**待实测**。【推测：基于产品线存在的判断】

### 1.7 国内圈横向对比（2026-11-16 官方口径）

| 厂商 | 小模型档（输入/输出，每百万 token） | 旗舰档（输入/输出） | 长上下文 | 协议兼容 | 备注 |
|---|---|---|---|---|---|
| DeepSeek | flash：¥1–2 / ¥4–8（峰谷） | v4-pro：闲时 ¥4.6（$0.66）/ ¥13.7（$1.98）起 | 1M，不加价 | OpenAI + Anthropic | 并发 2500/500；闲时半价利于夜间调度 |
| 智谱 | GLM-5.3-Flash：¥0.8 / ¥2.8 | GLM-5.3：¥8 / ¥28 | 1M | OpenAI 兼容 | 另有免费 GLM-4.7-Flash |
| Kimi | （K2.6 ¥6.5/¥27，12 月底下线） | K3.1 Pro：¥13 / ¥68 | 1M | OpenAI 兼容 | 限速按充值分级，小账号并发极低 |
| 通义 | qwen3.5-flash：¥0.2 / ¥2 | qwen3.8-max：¥12 / ¥36 | 1M（max） | OpenAI 兼容（DashScope） | 有 qwen-plus-character 角色 SKU；护栏默认关 |
| 豆包 | doubao-seed-2.1-lite：¥0.8 / ¥2.7 | doubao-seed-2.1-pro：¥6 / ¥30 | 1024K（pro） | OpenAI 兼容 | 有 doubao-seed-character 角色模型；方舟可托管多家模型 |

（DeepSeek 人民币换算为官方美元价 ×7 的约数，仅作量级参考；其官方人民币页与火山方舟托管价一致，见 §1.2。）

---

## 2. 国际圈（对照组，为英文市场留档）

### 2.1 OpenAI

- API 定价【一手，2026-11-16 查询】https://developers.openai.com/api/docs/pricing ：

| 模型 | 输入 | 输出 | 缓存输入 |
|---|---|---|---|
| `gpt-6-astra` | $10.00 | $50.00 | $1.00 |
| `gpt-6.1-sol` | $2.00 | $10.00 | $0.10 |
| `gpt-6.1-terra` | $1.00 | $5.00 | $0.05 |
| `gpt-6-luna` | $0.10 | $0.50 | $0.01 |

（每百万 token；`gpt-6.1-sol` 促销价至 2026-11-30；2026-03-05 后发布的模型区域处理/数据驻留端点加价 10%。）

- 内容政策（Usage Policies，2026-11-20 生效版）【一手，2026-11-16 查询】https://openai.com/policies/usage-policies/ ：
  - 明确禁止「underaged sexual or violent roleplay」（未成年人性/暴力角色扮演）；
  - 禁止性暴力与非自愿亲密内容（NCII）、CSAM 一律禁止并上报 NCMEC；
  - 成年虚构色情的 API 层处理在该政策页面未被明确禁止，实际执法尺度存在不确定性——【推测】需以英文市场上线前的实测与法务确认。
- 出海合规注记：OpenAI 服务区域不含中国大陆，首发不可用；仅作英文市场对照。

### 2.2 Anthropic

- API 定价【一手，2026-11-16 查询】https://docs.anthropic.com/en/docs/about-claude/pricing ：

| 模型 | 输入 | 输出 | 缓存写（5m/1h） | 缓存读 |
|---|---|---|---|---|
| Claude Fable 5.2 / Mythos 5.2 | $8 | $40 | $10 / $16 | $0.20 |
| Claude Opus 5.5 | $4 | $20 | $5 / $8 | $0.20 |
| Claude Sonnet 5.5 | $2 | $10 | $2.50 / $4 | $0.10 |
| Claude Haiku 5.5 | $0.10（≤100K）/ $0.50（>100K） | $0.50 / $2.50 | $0.125/$0.20 等 | $0.01 起 |

（每百万 token；Batch API 半价；4.6 起 1M 上下文不加价；注意 4.7 起新分词器同文本多约 30% token。）

- 内容政策（Usage Policy）【一手，2026-11-16 查询】https://www.anthropic.com/legal/aup ：
  - 专节「Do Not Generate Sexually Explicit Content」：禁止描绘性行为、性癖好内容、乱伦/兽交、「erotic chats」——**对成人向 RP 是国际三家中最严的明文条款**；
  - 禁止将未成年人性化（含虚构设定与角色扮演）；
  - 面向消费者的聊天机器人/Agent 必须向用户披露其为 AI。
- 中文 RP 口碑：社区横评视 Claude Opus 5.5 为最强 RP 后端之一，但价格高。【社区口碑·BuildMVPFast，2026-11-11】https://www.buildmvpfast.com/articles/best-llms-2026-guide/roleplay-ai

### 2.3 Google Gemini

> 说明：本环境无法访问 ai.google.dev 与 policies.google.com（网络层失败），以下为注明核价日期的第三方来源，**上生产前必须在官方定价页复核**。

- 定价（第三方核价）：
  - `gemini-3-flash-preview`：$0.50 / $3.00，上下文 1M，缓存读 $0.05。【第三方核价·LiteLLM 模型目录，标注 2026-11-15 核】https://models.litellm.ai/models/gemini-3-flash-preview
  - `gemini-3.1-pro`：≤200K 输入 $2 / $12，>200K $4 / $18。【第三方核价·Apidog 引官方发布，2026 年】https://apidog.com/blog/gemini-3-0-api-cost/
  - `Gemini 4 Argon`（旗舰）：$2 / $10，2026-12-01 起涨至 $4 / $20。【第三方核价·licenseware，2026-11-03】https://licenseware.io/gemini-api-cost/
  - 上下文缓存价约为输入价 10%；Batch 半价。【第三方核价·whatisamilliontokens，2026-10-02】https://whatisamilliontokens.com/gemini-api-pricing
- 内容政策：Google「生成式 AI 禁止使用政策」官方页面本次无法抓取；历史版本禁止性露骨内容与未成年人性化内容。【推测：基于历史版本的延续性假设，待复核】

### 2.4 OpenRouter（聚合渠道）

- 模式：一个 Key 聚合多家模型，按各模型 listing 计费；服务条款要求用户同时遵守各模型的 Model Terms（条款透传）。【一手·OpenRouter Terms of Service，最近更新 2026-08-31，2026-11-16 查询】https://openrouter.ai/terms
- 内容与隐私：
  - OpenRouter 对输入做 CSAM 自动筛查并依法上报 NCMEC（ToS §6.7）。【一手，同上】
  - 默认不存储 prompt/响应，除非用户开启日志或内容命中 CSAM 审查；提供 Zero Data Retention 路由（只路由到不存 prompt 的供应商）。【媒体·OpenRouter 官方博客教程，2026-10-27】https://openrouter.ai/blog/tutorials/sillytavern-openrouter/
  - 提供 Guardrails API（预算、模型准入、内容审核策略）。【一手·OpenRouter 官方文档，2026-11 查询】https://openrouter.ai/docs/api/api-reference/guardrail-policies/create-a-moderation-policy
- 社区定位：SillyTavern/JanitorAI 圈最常用的第一个 API 渠道；未购额度账户的免费模型每日限 50 次请求。【社区口碑·Cryptwerk，2026-11-03】https://cryptwerk.com/post/best-apis-ai-roleplay-janitor-sillytavern-risuai-2026/
- 对 blur 的意义：英文市场上线时，OpenRouter 是快速覆盖多模型的通道；但聚合层 + 下游供应商双层政策都要过，且其 ToS 明确禁止向受限地区提供服务。

---

## 3. 对 blur 场景的适配分析

### 3.1 成本模型（单轮对话）

社区经验口径：一次 RP 对话轮约发送 6K token 历史 + 返回约 300 token。【社区口碑·ISEKAI ZERO，2026-09-10】https://www.isekaizero.ai/blog/sillytavern-setup-guide 按官方价估算单轮成本（不含缓存命中折扣）：

| 模型 | 单轮约价（6K 入 + 0.3K 出） |
|---|---|
| DeepSeek flash（闲时） | ¥0.0072 |
| GLM-5.3-Flash | ¥0.0056 |
| doubao-seed-2.1-lite | ¥0.0056 |
| qwen3.5-flash | ¥0.0018 |
| DeepSeek v4-pro（闲时） | ¥0.032 |
| kimi-k3.1-pro | ¥0.098 |
| qwen3.8-max | ¥0.083 |

【推测：基于上述社区 token 口径与官方单价的算术】长线叙事 + 世界书/记忆注入会显著抬高输入 token，角色卡与世界书前缀固定时缓存命中率是最大成本杠杆（各家缓存命中价约为输入价的 2%–20%）。

### 3.2 关键工程事实

- 五家均为 OpenAI 兼容协议 + 流式输出，服务端可用统一适配层，模型可插拔。
- DeepSeek 的峰谷价差（闲时半价）与 Kimi 的充值分级限速（Tier0 并发仅 1）直接影响架构：**小账号冷启动阶段 Kimi 几乎不可用作生产主力**（并发 1–15），DeepSeek 并发 2500 起步最友好。
- DeepSeek `user_id` 参数提供内容安全/KVCache/调度的用户级隔离，与 blur 的记忆系统（用户级隔离）天然对齐。【一手，见 §1.2】
- 火山方舟可一份合约托管 DeepSeek / GLM / Kimi / 豆包多家模型，降低多供应商接入的法务与结算成本。【一手，见 §1.6】
- 各家服务条款都要求调用方自建内容审核；blur 的「隐形管线」中必须包含输入/输出双向审核与应急处置机制，并公示所用模型备案号（见 §1.1）。

### 3.3 内容政策结论

- 国内圈：五家均明文禁止色情与未成年人性化内容（DeepSeek ToS §3.4 最直白）；**恋爱/黑暗/成熟题材的可行空间在「非露骨」区间**，具体拦截尺度各文档均未量化，必须实测。【推测：基于条款文本的判断】
- 国际圈：Anthropic 明文禁止 erotic chats（最严）；OpenAI 禁止未成年人性/暴力角色扮演、对成年虚构内容表述留白；Gemini 待复核；OpenRouter 为透传 + CSAM 筛查。
- 无论国内国际，「未成年人性化（含虚构角色扮演）」是所有供应商的共同红线——角色卡审核必须包含年龄属性校验。

---

## 4. 选型短名单建议

### 小模型档位（日常陪伴对话主力，成本敏感）

1. **DeepSeek `deepseek-flash`** —— 首选。最低价区间（闲时 ¥1/¥4）、1M 上下文不加价、2500 并发、OpenAI/Anthropic 双协议、`user_id` 隔离。风险：高峰期价格翻倍（可用闲时调度消化记忆写入等异步任务）；ToS 明禁色情 chatbot，题材尺度需实测。
2. **智谱 `GLM-5.3-Flash`** — 备选/压价筹码。¥0.8/¥2.8 为国内圈最低刊例之一，1M 上下文；另有免费 GLM-4.7-Flash 可作开发期与降级兜底。
3. **豆包 `doubao-seed-character` / `doubao-seed-2.1-lite`** — 场景特化备选。角色专用 SKU + 火山方舟护栏体系；同时方舟可作为多模型聚合入口。

### 旗舰档位（关键剧情、复杂人设、付费增值）

1. **DeepSeek `deepseek-v4-pro`** —— 首选。闲时 $0.66/$1.98 的旗舰价远低于 Kimi/通义/豆包旗舰；500 并发对付费用户量级够用。
2. **Kimi `kimi-k3.1-pro`** —— 体验上限候选。1M 上下文、中文文风社区口碑好；但 ¥13/¥68 为最贵旗舰，且限速按充值分级，需预充值到 Tier3+ 才有生产可用并发。
3. **智谱 `GLM-5.3`** —— 均衡备选。¥8/¥28，1M 上下文，Agent/工具调用能力强（利于隐形管线中的编排任务）。
4. 备选观察：通义 `qwen3.8-max`（¥12/¥36，多地域部署利于未来出海）、豆包 `doubao-seed-2.1-pro`（¥6/¥30，低优通道半价）。

### 国际圈存档（英文市场再启动）

- 质量上限：Claude Opus 5.5（$4/$20，注意其 AUP 禁止 erotic chats）；成本档：OpenAI `gpt-6-luna`（$0.10/$0.50）与 `gemini-3-flash-preview`（$0.50/$3）；聚合通道：OpenRouter（双层政策 + CSAM 筛查）。

## 5. 开放问题（需后续实测/核实）

1. 各家 API 层对恋爱/黑暗/成熟（非露骨）题材的实际拦截率——无官方量化数据，需用标准测试集实测（建议列入 #7 或单独测试任务）。
2. 智谱、Kimi 的内容过滤是否存在可配置开关；豆包「原生安全能力」是否可联系销售调整。
3. Google Gemini 官方定价与政策页本次无法访问，国际圈 Gemini 数据全部为第三方核价，上生产前必须复核。
4. Kimi K2 系列 2026 年 12 月底下线，本文 K2 相关价格仅作存档。
5. 各家价格 2026 年内波动剧烈（DeepSeek 8–9 月连续调价、Gemini 4 Argon 12 月涨价），建议在服务端把价格表做成可配置数据并每月复核。

---

## 附：主要来源清单

一手（官方，均 2026-11-16 查询）：
- DeepSeek 定价：https://api-docs.deepseek.com/quick_start/pricing
- DeepSeek 限速：https://api-docs.deepseek.com/quick_start/rate_limit
- DeepSeek 服务条款：https://cdn.deepseek.com/policies/en-US/deepseek-terms-of-use.html
- Kimi 定价：https://platform.moonshot.cn/docs/pricing/chat
- Kimi 充值与限速：https://platform.moonshot.cn/docs/pricing/limits
- Kimi 平台协议：https://platform.moonshot.cn/docs/agreement/modeluse
- Kimi 备案公示：https://www.kimi.com/user/agreement/algorithm-and-model
- Z.ai（智谱国际站）定价：https://docs.z.ai/guides/overview/pricing
- 阿里云百炼计费：https://help.aliyun.com/zh/model-studio/billing
- 阿里云百炼 AI 安全护栏：https://help.aliyun.com/zh/model-studio/content-security
- 阿里云百炼模型备案公示：https://help.aliyun.com/zh/model-studio/model-filing-information-publicity
- 火山方舟模型价格：https://www.volcengine.com/docs/ark/model-pricing
- 火山方舟内容安全护栏：https://www.volcengine.com/docs/ark/content-safety-guardrails
- OpenAI API 定价：https://developers.openai.com/api/docs/pricing
- OpenAI Usage Policies：https://openai.com/policies/usage-policies/
- Anthropic 定价：https://docs.anthropic.com/en/docs/about-claude/pricing
- Anthropic Usage Policy：https://www.anthropic.com/legal/aup
- OpenRouter ToS：https://openrouter.ai/terms
- 网信办备案公告：https://www.cac.gov.cn/2024-04/02/c_1713729983803145.htm ；备案系统：https://beian.cac.gov.cn/

第三方核价/媒体/社区（文中已逐条注明日期）：
- LiteLLM 模型目录（Gemini）：https://models.litellm.ai/models/gemini-3-flash-preview
- licenseware（Gemini 4 Argon 涨价）：https://licenseware.io/gemini-api-cost/
- 稀土掘金（GLM-5.3-Flash 国内刊例）：https://juejin.cn/post/7678604384506839055
- gptproto（AI Roleplay Bench 社区基准）：https://gptproto.com/blog/best-ai-model-for-sillytavern
- BuildMVPFast（RP 后端社区横评）：https://www.buildmvpfast.com/articles/best-llms-2026-guide/roleplay-ai
- ISEKAI ZERO（RP 单轮 token 口径）：https://www.isekaizero.ai/blog/sillytavern-setup-guide
- GitHub DeepSeek-V2 issue #51（审核拦截口碑）：https://github.com/deepseek-ai/DeepSeek-V2/issues/51
