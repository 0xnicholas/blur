# 研究：SillyTavern 角色卡生态与格式标准

> 对应 issue: [0xnicholas/blur#2](https://github.com/0xnicholas/blur/issues/2)
> 查询日期：2026-11-16（下文所有来源均在该日期访问）
> 术语遵循仓库根目录 `GLOSSARY.md`（角色卡 / 世界书 / 用户身份卡 / 语言指纹等）。
> 每条结论标注【已证实】（一手来源：官方 spec、官方文档、SillyTavern 源码）或【社区口碑】（教程、榜单、社区讨论，未经一手验证）。

---

## 结论摘要(TL;DR)

1. **以 Character Card V2 为导入基准格式**【已证实】:V2 是当前生态事实标准，字段集覆盖我们需要的全部人设信息；V3 是其后向兼容超集，新增字段全部可选，可只做无损透传。
2. **PNG 内嵌机制很简单**:PNG/APNG 文件里放一个关键字为 `chara`(V2)或 `ccv3`(V3)的 `tEXt` chunk,内容是 UTF-8 JSON 的 base64 编码，位于 `IEND` 之前；`ccv3` 存在时优先读取【已证实，SillyTavern 源码】。
3. **世界书有两种序列化形态**,是导入时最容易踩的坑：spec 形态(`character_book.entries` 为数组，字段名 `keys`/`insertion_order`/`enabled`)与 SillyTavern 原生 World Info JSON 形态(`entries` 为以 uid 为键的对象，字段名 `key`/`order`/`disable`)【已证实，SillyTavern 源码】。两者都要支持。
4. **关键词触发对中文有官方警告**:SillyTavern 世界书的 "Match whole words" 默认开启，官方文档明确说它对中文、日文等不使用空格分词的语言有害，导入中文卡时应默认关闭【已证实，官方文档】。
5. **最小字段集**:`name / description / personality / scenario / first_mes / mes_example` + `alternate_greetings` + `character_book`(可选)+ `system_prompt / post_history_instructions`(可选透传)。详见第七章。
6. **中文社区渠道**:核心在 Discord 社区（类脑 ΟΔΥΣΣΕΙΑ、旅程 ΟΡΙΖΟΝΤΑΣ),Web 端无等量级官方卡站；中文卡以 V2 PNG 为主，常附带独立预设 JSON 与独立世界书 JSON【社区口碑，详见第五章】。

---

## 一、角色卡规格:V1 / V2 / V3

一手来源:
- V1 spec: <https://github.com/malfoyslastname/character-card-spec-v2/blob/main/spec_v1.md>
- V2 spec: <https://github.com/malfoyslastname/character-card-spec-v2/blob/main/spec_v2.md>(2023-05-17 验证通过并定稿）
- V3 spec: <https://github.com/kwaroran/character-card-spec-v3/blob/main/SPEC_V3.md>

### 1.1 V1(TavernAI 遗产格式)【已证实】

扁平 JSON,6 个字段，全部强制且必须默认空字符串(不得为 null 或缺失）:

```ts
type TavernCardV1 = {
  name: string        // 角色名
  description: string // 角色描述
  personality: string // 性格摘要
  scenario: string    // 场景/情境
  first_mes: string   // 开场白(必须由角色先发)
  mes_example: string // 示例对话,<START> 分段
}
```

行为要求【已证实，V1 spec】:
- 提示词中 `{{char}}` / `<BOT>` 替换为 `name`,`{{user}}` / `<USER>` 替换为用户显示名，大小写不敏感。
- 示例对话以 `<START>` 分隔；上下文紧张时应优先裁剪示例对话为聊天历史腾地方。
- 嵌入方式：裸 JSON(spec 不鼓励）或 PNG/APNG;**WEBP 明确不在 spec 覆盖范围**("Not covered by the spec due to technical ambiguities")。
- 注意：V1 spec 原文称 JSON 存于 PNG 的 "Chara EXIF metadata field"——这一表述不精确，实际实现是 PNG `tEXt` chunk（见第二章）,V3 spec 已将其更正为 tEXt chunk 表述。

### 1.2 V2（当前事实标准）【已证实】

V2 = 包装层 + 全部 V1 字段移入 `data`:

```ts
type TavernCardV2 = {
  spec: 'chara_card_v2'       // 必须恰为该值
  spec_version: '2.0'         // 必须恰为该值
  data: {
    // —— V1 原样字段 ——
    name, description, personality, scenario, first_mes, mes_example
    // —— V2 新增 ——
    creator_notes: string            // 创作者备注,禁止进入提示词
    system_prompt: string            // 卡级系统提示,默认替换全局设置;空串则用全局
    post_history_instructions: string// 卡级"历史后指令"(ujb/jailbreak 位),同上
    alternate_greetings: string[]    // 额外开场白,前端必须以 swipe 形式提供
    character_book?: CharacterBook   // 卡内嵌世界书,可选
    tags: string[]                   // 标签,不进入提示词,可用于排序/筛选
    creator: string                  // 作者,不进入提示词
    character_version: string        // 卡版本,不进入提示词
    extensions: Record<string, any>  // 必须存在、默认 {};未知键禁止销毁
  }
}
```

关键语义【已证实，V2 spec】:
- `system_prompt` / `post_history_instructions` 必须支持 `{{original}}` 占位符（展开为前端原有的全局对应提示词）。
- `creator_notes` **MUST NOT** 进提示词；应显著展示给用户。
- `extensions` 是互操作性的核心：编辑器导入再导出时不得销毁未知键；应用自定义键应加命名空间(如 `agnai/voice`)。
- 可选字段允许编辑器不支持，但**已存在的数据不得被销毁**。

### 1.3 V3(RisuAI 主导，SillyTavern 已支持读取)【已证实】

V3 是 V2 的超集(`spec: 'chara_card_v3'`, `spec_version: '3.0'`),`data` 内新增：

| 字段 | 说明 |
|---|---|
| `assets` | 资源数组(icon/background/user_icon/emotion/…),URI 支持 HTTP(S)、base64 data URL、`embeded://`(CHARX 内嵌资源)、`ccdefault:` |
| `nickname` | 若存在,`{{char}}`/`<char>`/`<bot>` 替换为它而非 `name` |
| `creator_notes_multilingual` | 多语言创作者备注(ISO 639-1 键） |
| `source` | 来源 URL/ID 数组，只增不删 |
| `group_only_greetings` | 仅群聊使用的开场白（必填，可空数组） |
| `creation_date` / `modification_date` | Unix 秒时间戳 |
| `character_book` | 类型变为 Lorebook，条目新增 `use_regex` 等（见第三章） |

V3 还定义了两大新机制【已证实，V3 spec】:
- **Decorators**:lorebook 条目 `content` 内以 `@@` 开头的指令行，如 `@@depth`、`@@role`、`@@position`、`@@activate_only_after`、`@@scan_depth`、`@@additional_keys`、`@@exclude_keys` 等；支持 `@@@` 回退链。应用不认识的 decorator 应忽略。
- **CBS(Curly Braced Syntax)宏**:`{{char}}`、`{{user}}`、`{{random:A,B}}`、`{{pick:A,B}}`、`{{roll:N}}`、`{{// 注释}}`、`{{hidden_key:A}}`、`{{comment:}}`、`{{reverse:A}}`。

兼容性条款【已证实，V3 spec】:V3 卡在 PNG 中应同时写 `chara`(V2 形态）与 `ccv3`(V3 形态）chunk;读到双 chunk 时应用应使用 `ccv3`;只认 V2 的应用读 `chara` 时会得到一个"回填"版本（建议在 `creator_notes` 里提示用户这是降级读取）。未来版本未知字段应忽略但不得拒绝导入。

### 1.4 V1 vs V2 字段差异速查

| 维度 | V1 | V2 |
|---|---|---|
| 结构 | 扁平 6 字段 | `spec`/`spec_version` 包装 + `data` 容器 |
| 人设字段 | 6 个全部强制 | 同 V1，全部保留于 `data` |
| 卡级提示词覆盖 | 无 | `system_prompt`、`post_history_instructions`(支持 `{{original}}`) |
| 多开场白 | 无 | `alternate_greetings` |
| 内嵌世界书 | 无 | `character_book`（可选） |
| 元数据 | 无 | `creator_notes`/`tags`/`creator`/`character_version` |
| 扩展位 | 无 | `extensions`（强制存在，未知键禁销毁） |

> 实务备注【社区口碑】：大量 V2 PNG 会把 V1 的 6 个字段同时复制到 JSON 顶层作兼容 shim,`convertToV2` 一类转换器也这么干；解析时应以 `data` 为准，顶层平铺字段仅作 V1 回退。

---

## 二、分发载体与内嵌技术

### 2.1 PNG/APNG: tEXt chunk + base64【已证实】

SillyTavern 服务端源码 `src/character-card-parser.js`(<https://github.com/SillyTavern/SillyTavern/blob/release/src/character-card-parser.js>）实证：

- **写入**:UTF-8 JSON → base64 → 编码为 PNG `tEXt` chunk，关键字 `chara`，插入在 `IEND` 之前；同时尝试另写一个 `ccv3` chunk（内容同 JSON，仅把 `spec`/`spec_version` 改为 V3)。写入前先删除已存在的 `chara`/`ccv3` chunk（关键字比较大小写不敏感），避免新旧不一致。
- **读取**：扫描所有 `tEXt` chunk，先找 `ccv3`，找不到再找 `chara`,base64 解码回 UTF-8 JSON。**`ccv3` 优先于 `chara`**。

与 V3 spec 的"双 chunk + ccv3 优先"完全一致。APNG 走同一路径。

### 2.2 纯 JSON【已证实】

V1 spec 允许裸 JSON 但"不鼓励"(user-friendliness);V2/V3 spec 均允许 JSON 文件直接承载卡对象；V3 规定 JSON 文件必须就是 CharacterCardV3 对象本体。SillyTavern 导入端接受 `.json`（源码 `src/endpoints/characters.js` 的 `case 'json'` 分支）。

### 2.3 其他载体

| 载体 | 状态 | 依据 |
|---|---|---|
| WEBP | V1 spec 明确不覆盖 | spec_v1.md |
| CHARX | V3 官方容器：zip，根目录必有 `card.json`，资源经 `embeded://path` 引用（注意拼写是 `embeded://`),`.charx` 扩展名；zip 不应加密、内部路径只用 ASCII | SPEC_V3.md |
| YAML | SillyTavern 服务端导入支持（源码 import 了 `yaml` 包） | src/endpoints/characters.js |
| BYAF | SillyTavern 服务端有独立解析器 `src/byaf.js` 与导入测试 | SillyTavern/SillyTavern 仓库 |
| `chara-ext-asset_:{path}` tEXt chunk | V3 允许在 PNG 内用额外 tEXt chunk 塞二进制资源，但明确建议新应用不要实现，改用 CHARX | SPEC_V3.md |

---

## 三、世界书(Lorebook / World Info)

### 3.1 spec 中的结构【已证实，V2 spec;V3 增补】

```ts
type CharacterBook = {   // V3 改名 Lorebook,结构相同 + use_regex
  name?: string
  description?: string
  scan_depth?: number        // 只在最近 N 条消息里匹配关键词(agnai 称 Chat History Depth)
  token_budget?: number      // 世界书条目的 token 预算(agnai 称 Context Limit)
  recursive_scanning?: boolean// 条目 content 能否触发其他条目
  extensions: Record<string, any>
  entries: Array<{
    keys: string[]            // 触发关键词
    content: string           // 命中后注入的文本;为空则什么都不注入
    enabled: boolean
    insertion_order: number   // 多条同时插入时,小的排前面
    case_sensitive?: boolean
    // 以下可选;即使不支持也应保留以便无损导出
    name?: string             // 不用于提示词构造
    priority?: number         // 超预算时,值小的先丢
    id?: number               // (V3 放宽为 number|string)
    comment?: string          // 不用于提示词构造
    selective?: boolean       // true 时要求 keys 与 secondary_keys 同时命中
    secondary_keys?: string[]
    constant?: boolean        // true 则每轮必注入(预算内)
    position?: 'before_char' | 'after_char'  // 相对角色定义的位置
    use_regex: boolean        // V3 新增:keys 按正则解释;非法正则视为不命中
  }>
}
```

V3 还规定独立世界书文件的包装格式:`{ "spec": "lorebook_v3", "data": Lorebook }`【已证实，SPEC_V3.md】。

### 3.2 触发机制（SillyTavern 实现语义）【已证实，官方文档】

来源：<https://docs.sillytavern.app/usage/core-concepts/worldinfo/>

- **扫描缓冲**:ST 对最近 N 条聊天消息(`Scan Depth`，条目级可覆盖）做关键词匹配；每条消息带 `角色名:` 前缀，v1.12.6 起前缀前还有 `\x01` 分隔符（供正则精准匹配"谁说的话")。
- **关键词**:默认大小写不敏感；支持 JavaScript 风格正则（`/` 定界，任意 flag);纯文本关键词以逗号分隔，正则可含逗号。
- **Match whole words**：默认开启，整词匹配；**官方明确警告：对中文、日文等不使用空格分词的语言有不良影响，写中文条目应关闭**。
- **Selective Logic（次级关键词组合）**:AND ANY / AND ALL / NOT ANY / NOT ALL 四种。
- **预算**:`Context %` 或固定 `Budget`;超预算后不再激活新条目；常驻（constant）条目优先，其次按 insertion_order 大者（更靠近上下文末尾）优先；直接命中关键词的条目优先于被递归带出的条目。
- **递归扫描**：条目 content 里提到别的条目的关键词会链式激活（官方示例：提到 Bessie 会把 Rufus 也拉进来）；条目级有 Non-recursable / Prevent further recursion / Delay until recursion 三档控制；全局可设 Max Recursion Steps 或 Min Activations（互斥）。
- **插入位置**:Before/After Char Defs、Before/After Example Messages、Top/Bottom of AN、@D（指定聊天深度，可带 system/user/assistant 角色）、Outlet(`{{outlet::Name}}` 宏手动取用）。
- **概率触发**:Probability 0–100，可做随机事件。
- **Inclusion Group**：同组同时命中只插入一条，按 Group Weight 随机或按 Order 优先（Prioritize Inclusion);Use Group Scoring 可按命中关键词数量取胜。
- **Timed Effects**:Sticky（激活后保持 N 条消息）/ Cooldown（激活后 N 条内不再触发）/ Delay（聊天不足 N 条不触发）。
- **其他**:Vectorized（向量相似度无关键词激活，需 Vector Storage 扩展）、Character Filter、Triggers（限定生成类型）、Automation ID（联动 Quick Replies)、Additional matching sources（拿角色描述/性格/场景/用户身份卡描述等不进聊天的文本做匹配源）。

### 3.3 两种序列化形态（导入最大坑点）【已证实，SillyTavern 源码】

ST 内部并不直接使用 spec 形态。`public/scripts/world-info.js` 的 `convertCharacterBook()` 把 `character_book` 转成 ST 原生 World Info 形态：

- spec:`entries` 是**数组**;ST 原生：`entries` 是**以 uid 为键的对象**(`{ entries: { "0": {...}, "1": {...} } }`)。
- 字段改名对照（spec → ST 原生）:
  - `keys` → `key`
  - `secondary_keys` → `keysecondary`
  - `insertion_order` → `order`
  - `enabled` → `disable`（取反！)
  - `comment` → `comment`（同时置 `addMemo`)
- ST 扩展行为全部塞进条目 `extensions`:`position`(ST 的取值远多于 spec 的二选一）、`probability`、`useProbability`、`depth`、`selectiveLogic`、`outlet_name`、`group`、`group_override`、`group_weight`、`scan_depth`、`case_sensitive`、`match_whole_words`、`use_group_scoring`、`automation_id`、`role`、`vectorized`、`sticky`、`cooldown`、`delay`、`match_persona_description`、`match_character_description`、`match_character_personality`、`match_character_depth_prompt`、`match_scenario`、`match_creator_notes`、`triggers`、`ignore_budget` 等。

结论：我们的导入器要同时认两种形态——spec 数组形态（跨应用交换）与 ST 原生对象形态（ST 世界书导出文件、卡内已被转换过的数据）。

---

## 四、进阶字段

### 4.1 深度提示(Character's Note / depth prompt)【已证实】

- 官方文档：<https://docs.sillytavern.app/usage/core-concepts/characterdesign/>("Character's Note" 一节）：在聊天历史固定深度注入的文本，用于强化人设；参数为 `@ Depth`（距最新消息的消息数，0 = 最后一条之后）和 Role(User/System/Assistant)。
- 存储位置【已证实，ST 源码 `public/scripts/char-data.js` 类型定义】:

```ts
data.extensions.depth_prompt = {
  prompt: string,                          // 注入文本
  depth: number,                           // 深度
  role: 'system' | 'user' | 'assistant'
}
```

- 注意：`depth_prompt` 不是 V2 spec 字段，是 ST 放在 `extensions` 里的应用私有键；spec 正是靠 `extensions` 的"未知键禁销毁"条款保证它跨工具存活。世界书条目侧另有 `extensions.depth` / `extensions.role`(ST)与 V3 的 `@@depth`/`@@role` decorator，是同一思路在不同层级的表达。

### 4.2 卡级提示词覆盖【已证实，V2 spec + 官方文档】

- `system_prompt`：对应 ST 的 "Main Prompt" 覆盖；`post_history_instructions`：对应 "Post-History Instructions"（历史消息之后、模型注意力最近的位置，适合放行为强化）。
- 两者都支持 `{{original}}` 占位；ST 另受 "Prefer Char. Prompt / Prefer Char. Instructions" 用户开关控制是否生效。

### 4.3 开场白族

- `first_mes`：首条消息，必须由角色先发（V1 强制）【已证实】。
- `alternate_greetings`：前端必须以 swipe 提供（V2 强制）【已证实】；群聊时随机抽选（ST 文档）【已证实】。
- `group_only_greetings`:V3 新增，仅群聊使用【已证实，SPEC_V3.md】。

### 4.4 extensions 里的常见键【已证实，ST 源码 char-data.js 类型定义】

ST 自有：`talkativeness`（群聊主动发言概率，0–100，默认 50)、`fav`、`world`（角色绑定的全局世界书名）、`depth_prompt`、`regex_scripts`（卡级正则脚本）。
第三方：`pygmalion_id`、`github_repo`、`source_url`、`chub`(`{full_path}`)、`risuai`(`{source: string[]}`)、`sd_character_prompt`(`{positive, negative}`)。

### 4.5 V3 专属机制

Decorators(`@@depth`/`@@role`/`@@position`/`@@activate_only_after`/`@@activate_only_every`/`@@keep_activate_after_match`/`@@dont_activate_after_match`/`@@scan_depth`/`@@additional_keys`/`@@exclude_keys`/`@@is_greeting`/`@@ignore_on_max_context` 等，支持 `@@@` 回退链）与 CBS 宏（`{{random}}`/`{{pick}}`/`{{roll}}`/`{{comment}}`/`{{hidden_key}}`/`{{reverse}}`)。我们 MVP 不实现，但必须做到：不认识的 decorator 行按 spec 要求忽略（保留原文或安全剥离）,CBS 宏不展开时原样保留。

---

## 五、中文社区生态与格式变体

> 本章事实多为社区资料与教程，未经一手验证处均已标注【社区口碑】。

### 5.1 主要渠道

| 渠道 | 类型 | 说明 | 来源 |
|---|---|---|---|
| 类脑 ΟΔΥΣΣΕΙΑ(Discord) | 中文社区 | 中文酒馆资源核心社区：角色卡、预设、世界书、教程；有入群答题/规则；需国际网络。【口碑】约 30 万用户；2025-07 有旧卡区删卡、恢复未完成的说法（不同来源时间点不同，需以频道现状为准） | [tuniang 榜单](https://tuniang.app/guides/jue-se-ka-xia-zai-wang-zhan-tui-jian/)、[狐狐教程](https://awaa-col.github.io/Role-Play/ST/%E8%A7%92%E8%89%B2%E5%8D%A1%E8%8E%B7%E5%8F%96/)(discord.gg/TexFNafsYz) |
| 旅程 ΟΡΙΖΟΝΤΑΣ(Discord) | 中文社区 | 侧重中文新卡发布 | [tuniang 榜单](https://tuniang.app/guides/jue-se-ka-xia-zai-wang-zhan-tui-jian/) |
| 拟人(Discord) | 中文社区 | 以 Gemini 为主，无入群门槛 | [狐狐教程](https://awaa-col.github.io/Role-Play/ST/%E8%A7%92%E8%89%B2%E5%8D%A1%E8%8E%B7%E5%8F%96/) |
| Chub.ai（原 CharacterHub,characterhub.org) | 英文公开卡库 | 体量最大的公开卡库，标签/搜索完善；中文卡需自行筛 | [tuniang 榜单](https://tuniang.app/guides/jue-se-ka-xia-zai-wang-zhan-tui-jian/)、[狐狐教程](https://awaa-col.github.io/Role-Play/ST/%E8%A7%92%E8%89%B2%E5%8D%A1%E8%8E%B7%E5%8F%96/) |
| JanitorAI | 英文平台 | 站内试聊后再导出；导出规则随平台政策变化 | [tuniang 榜单](https://tuniang.app/guides/jue-se-ka-xia-zai-wang-zhan-tui-jian/) |
| Character Tavern / AI Character Cards | 英文卡库 | 更干净的浏览与补充库 | [tuniang 榜单](https://tuniang.app/guides/jue-se-ka-xia-zai-wang-zhan-tui-jian/) |
| RisuRealm | RisuAI 卡库 | V2/V3 PNG，与酒馆兼容 | [tuniang 榜单](https://tuniang.app/guides/jue-se-ka-xia-zai-wang-zhan-tui-jian/) |
| MiniTavern / AIEcho / sillytavern.one(SillyTavern Pro)/ DokiLove / 夜语AI | 中文 Web 平台 | 国产移动端/网页端产品，提供卡库浏览、ST 兼容 PNG 下载与一键导入；DokiLove 为索引站（保留来源关系） | [MiniTavern](https://www.sillytavernai.chat/)、[AIEcho](https://aiecho.xyz/landing)、[sillytavern.one](https://sillytavern.one/sillytavern-character-cards-zh)、[DokiLove](https://doki.love/cards/69323) |
| 百度贴吧 sillytavern 吧 / 网盘 | 论坛 | 零散分享，版本管理弱 | [贴吧](https://tieba.baidu.com/p/8711318791) |

### 5.2 格式变体与风险【社区口碑为主】

- 中文社区流通的卡以 **V2 PNG** 为主；完整作品常由三部分组成：角色卡 PNG + 独立**预设**(Preset，提示词/采样参数 JSON)+ 独立**世界书** JSON，只导入 PNG 会丢预设与世界书，表现明显变弱（[tuniang 榜单 FAQ](https://tuniang.app/guides/jue-se-ka-xia-zai-wang-zhan-tui-jian/))。
- 中文卡相对外网卡更依赖世界书来承载设定（[tuniang 榜单](https://tuniang.app/guides/jue-se-ka-xia-zai-wang-zhan-tui-jian/))。
- 电商（淘宝/闲鱼）"角色卡包"常见问题：被删 `personality`/`scenario`/`mes_example` 等字段、多卡合并成残缺 JSON——能导入但人设为空（[tuniang 榜单](https://tuniang.app/guides/jue-se-ka-xia-zai-wang-zhan-tui-jian/))。**建议：导入器对"V1 六字段全空但 spec 标记正常"的卡给出质量提示。**
- 中文创作者社区有"限制外传、防止倒卖"的倾向，教程资料相对封闭（[艾萝工坊](https://www.erocraft.com/docs/sillytavern/11-resources))。
- 常见导入失败原因（用户侧）：把封面图当卡下载、webm 压缩图、插图卡触发 "External media has been blocked"([狐狐教程](https://awaa-col.github.io/Role-Play/ST/%E8%A7%92%E8%89%B2%E5%8D%A1%E8%8E%B7%E5%8F%96/))。

---

## 六、兼容矩阵

### 6.1 分发载体 × 建议支持度

| 载体 | 生态流行度 | 我方支持建议 | 备注 |
|---|---|---|---|
| PNG/APNG(`chara` chunk) | ★★★★★ | **MVP 必须** | V2 卡标准载体 |
| PNG/APNG(`ccv3` chunk) | ★★ | **MVP 必须（读取）** | 读到时优先于 `chara`;V3 字段透传 |
| 纯 JSON(V1/V2/V3) | ★★★ | **MVP 必须** | 无封面图；V1 需自动升级 |
| ST 原生世界书 JSON(entries 对象形态） | ★★★（中文社区） | **MVP 必须** | 与卡分离分发的世界书都是这个形态 |
| spec 形态 lorebook(`lorebook_v3` / character_book 数组） | ★★ | **MVP 必须** | 跨应用交换形态 |
| CHARX | ★★ | 后续版本 | zip + card.json；读取不难，资源处理是工作量 |
| YAML / BYAF | ★ | 不支持 | ST 私有/小众，遇不到可报错提示 |
| WEBP 卡 | ★ | 不支持（spec 未定义） | V1 spec 明确排除 |
| `chara-ext-asset_` PNG 资源 chunk | ★ | 不支持 | V3 spec 自己也不推荐新应用实现 |

### 6.2 spec 版本 × 字段处理矩阵

| 字段（data 内） | V1 | V2 | V3 | 我方处理 |
|---|:---:|:---:|:---:|---|
| name | ✓ | ✓ | ✓ | 必填 |
| description / personality / scenario | ✓ | ✓ | ✓ | 必填（可空串） |
| first_mes | ✓ | ✓ | ✓ | 必填 |
| mes_example | ✓ | ✓ | ✓ | 必填（可空串） |
| alternate_greetings | – | ✓ | ✓ | 支持（swipe) |
| creator_notes | – | ✓ | ✓（含多语言变体） | 展示，不进提示词 |
| system_prompt / post_history_instructions | – | ✓ | ✓ | 支持，含 `{{original}}` 展开 |
| character_book | – | ✓ | ✓ | 支持（两种形态） |
| tags / creator / character_version | – | ✓ | ✓ | 存储与展示，不进提示词 |
| extensions | – | ✓ | ✓ | 原样保留，识别 `depth_prompt`/`world`/`talkativeness` |
| assets / nickname / source / group_only_greetings / creation_date / modification_date | – | – | ✓ | MVP 透传不解释；`nickname` 可用于 `{{char}}` 替换 |
| V3 decorators / CBS 宏 | – | – | ✓ | 不实现；不认识的 `@@` 行忽略，宏原样保留 |

### 6.3 来源站点 × 格式变体

| 来源 | 主要分发形态 | 备注 |
|---|---|---|
| 类脑/旅程等中文 Discord | V2 PNG + 独立预设 JSON + 独立世界书 JSON | 【口碑】 |
| Chub.ai | V2 PNG、JSON;API/Character Path 直拉 | 【口碑】 |
| JanitorAI | 站内角色，导出能力随政策 | 【口碑】 |
| RisuRealm | V2/V3 PNG、CHARX | 【口碑】 |
| 国产平台（MiniTavern/AIEcho/sillytavern.one 等） | ST 兼容 PNG，应用内卡库 | 【口碑】 |
| 电商卡包 | 残缺 JSON/PNG 混杂 | 【口碑】风险源，做质量检测 |

---

## 七、最小字段集建议（MVP)

**导入必须完整解析:**

1. V1 六字段：`name` / `description` / `personality` / `scenario` / `first_mes` / `mes_example`（缺失按 spec 默认空串，但对全空卡给质量警告）。
2. `alternate_greetings`（开场 swipe 是核心体验）。
3. `character_book`（中文卡高度依赖；支持 3.3 两种形态；触发引擎 MVP 实现：关键词包含匹配 + `scan_depth` + `token_budget` + `constant` + `enabled` + `insertion_order` + `selective`/`secondary_keys`;**中文默认关 match-whole-words**；递归扫描与概率可后置）。
4. `system_prompt` / `post_history_instructions`（含 `{{original}}`;`post_history_instructions` 与我们"防 AI 味"目标强相关）。
5. `tags` / `creator` / `creator_notes` / `character_version`（展示与筛选）。
6. `extensions` 全量透传；识别 `depth_prompt`、`world`（绑定外部世界书）、`talkativeness`（可映射"陪伴密度"雏形）。

**MVP 可不做但不得丢数据：** V3 assets/decorators/CBS、CHARX、向量触发、timed effects、inclusion group、卡级 regex_scripts（正则脚本涉及安全边界，即便后续做也应默认关闭）。

**导出：** 写 V2 PNG(`chara` chunk,IEND 前，base64 UTF-8 JSON)，可选双写 `ccv3`；顶层同时平铺 V1 六字段做兼容 shim。

---

## 八、导入映射表（角色卡 → blur 领域概念）

依 `GLOSSARY.md` 术语：

| 卡片字段 | blur 落点 | 映射说明 |
|---|---|---|
| `name` / `nickname`(V3) | 角色卡 · 角色名 | `nickname` 存在时用于 `{{char}}` 替换 |
| `description` | 角色卡 → 创建期初始化**角色档案** | 永久进提示词的核心描述 |
| `personality` | 角色卡 → 角色档案 | 性格摘要 |
| `scenario` | 角色卡 → 角色档案 | 情境设定 |
| `mes_example` | **语言指纹**样本库 | `<START>` 切段；示例对话是有行为含义的样本 |
| `first_mes` / `alternate_greetings[]` | 角色卡 · 开场白组 | 首条由角色发；多条对应 swipe |
| `character_book` | **世界书**（卡绑定） | 条目结构见 3.1；两种序列化形态见 3.3 |
| `extensions.world` | 绑定外部世界书名 | ST 里角色关联全局世界书的键 |
| `system_prompt` | 隐形管线 · 系统提示覆盖 | 支持 `{{original}}` |
| `post_history_instructions` | 隐形管线 · 历史后指令 | 与防 AI 味过滤协同的注入点 |
| `extensions.depth_prompt` | 隐形管线 · 深度提示 | `{prompt, depth, role}`；固定深度注入 |
| `tags` | 角色卡 · 标签 | 不进提示词；用于卡库筛选 |
| `creator` / `creator_notes` / `character_version` | 角色卡 · 元信息 | 展示给用户；不进提示词 |
| `extensions.talkativeness` | **陪伴密度** · 主动性参数雏形 | ST 语义是群聊发言概率（0–100)，映射时需转换语义 |
| `extensions.regex_scripts` | 不导入（MVP) | 正则改写有安全与确定性风险 |
| V3 `assets` / `source` / `creation_date` 等 | 透传存储 | 导出时回写，保证不丢 |
| （卡外）预设 JSON | 隐形管线 · 采样与提示词参数 | 中文社区"完整作品"三件套之一，建议支持一并导入 |
| （卡外）独立世界书 JSON | 世界书 | ST 原生对象形态 |

宏与占位符：`{{char}}`/`{{user}}` 在导入时保留原文，运行时由隐形管线替换（`{{user}}` → 用户身份卡显示名）。

---

## 九、已证实 vs 社区口碑清单

**已证实（spec 原文 / 官方文档 / ST 源码）:**
- V1/V2/V3 字段集、必填性与 MUST/SHOULD 语义（§1)
- PNG tEXt chunk `chara`/`ccv3` + base64 + IEND 前 + ccv3 优先（§2.1)
- WEBP 不在 spec;CHARX 结构与 `embeded://` URI;JSON 直发合法（§2)
- CharacterBook/Lorebook 结构与全部条目字段（§3.1)
- ST 世界书触发语义：扫描深度、预算、递归、整词匹配的中文警告、selective 逻辑、timed effects 等（§3.2)
- spec 数组形态 ↔ ST 原生对象形态的转换与字段改名（§3.3)
- `depth_prompt` 的存储键与结构（§4.1)
- ST 导入接受 png/json/yaml/charx/byaf（源码 import 与 case 分支，§2.3、§6.1)

**社区口碑（教程/榜单/讨论，未一手验证）:**
- 类脑约 30 万用户、2025-07 旧卡区删卡事件（§5.1)
- 中文社区以 V2 PNG + 独立预设/世界书三件套分发为主（§5.2)
- 中文卡比外网卡更依赖世界书（§5.2)
- 电商卡包字段残缺问题（§5.2)
- 各站点规模与体验排序（§5.1 表）

---

## 十、来源清单（查询日期均为 2026-11-16)

一手来源：
1. Character Card V1 spec — <https://github.com/malfoyslastname/character-card-spec-v2/blob/main/spec_v1.md>
2. Character Card V2 spec — <https://github.com/malfoyslastname/character-card-spec-v2/blob/main/spec_v2.md>
3. Character Card V3 spec — <https://github.com/kwaroran/character-card-spec-v3/blob/main/SPEC_V3.md>
4. SillyTavern 源码：PNG 卡解析 — <https://github.com/SillyTavern/SillyTavern/blob/release/src/character-card-parser.js>
5. SillyTavern 源码：卡数据结构（depth_prompt、extensions 键）— <https://github.com/SillyTavern/SillyTavern/blob/release/public/scripts/char-data.js>
6. SillyTavern 源码：世界书形态转换 convertCharacterBook — <https://github.com/SillyTavern/SillyTavern/blob/release/public/scripts/world-info.js>
7. SillyTavern 源码：导入端点（json/yaml/charx/byaf)— <https://github.com/SillyTavern/SillyTavern/blob/release/src/endpoints/characters.js>
8. SillyTavern 官方文档：World Info — <https://docs.sillytavern.app/usage/core-concepts/worldinfo/>
9. SillyTavern 官方文档：Character Design(Character's Note、Alternate Greetings 等）— <https://docs.sillytavern.app/usage/core-concepts/characterdesign/>

社区资料：
10. Tuniang《2026 角色卡下载网站推荐》— <https://tuniang.app/guides/jue-se-ka-xia-zai-wang-zhan-tui-jian/>
11. 狐狐的 AI 角色扮演教程《酒馆角色卡获取》— <https://awaa-col.github.io/Role-Play/ST/%E8%A7%92%E8%89%B2%E5%8D%A1%E8%8E%B7%E5%8F%96/>
12. 艾萝工坊《角色卡与社区资源》— <https://www.erocraft.com/docs/sillytavern/11-resources>
13. MiniTavern — <https://www.sillytavernai.chat/>
14. AIEcho — <https://aiecho.xyz/landing>
15. SillyTavern Pro（中文卡库）— <https://sillytavern.one/sillytavern-character-cards-zh>
16. DokiLove（索引站示例）— <https://doki.love/cards/69323>
