# Wisteria —— AI 恋人 Kaoru 与日链管家（个人 iOS App 实施计划）

> 定稿日期：2026-08-20（已获用户批准）
> 配套文档：docs/调研整合.md（设计依据）、docs/开发日志.md（过程记录）、docs/界面与风格定稿.md（UI/视觉规格——风格已定稿 2026-08-20：打印店·灰粉 B 方案，界面一律照该文档做）
> 实施节奏：按 Bunny&Elliott 路径——需求先行、逐步确认；前端风格与用户共同深思熟虑，其余能借鉴开源就借鉴；用户核心诉求明确、不大量 review 代码，力求事半功倍。

## Context（为什么做这个）

用户是在校学生（iPhone；开发机 Windows 11；2026-08-20 暑假中，约 2 周后开学）。与用户逐问确认后的核心困境：**起床时间不固定（暑假常 12 点后）→ 全天失去"锚点"→ 结构散掉 → 彻底忘记吃药、忘记记录**。吃药一日一次、起床时吃，经常"猜自己没吃就吃"，可能漏吃或重复吃；分药盒不够，因为问题不是分药，而是"记不住吃没吃"。

**坚持不下来的本质**（用户已认同的分析，也是全部设计的依据）：不是意志力问题，而是——
1. **完全想不起来** → 需要外部触发（通知主动找到人，不靠自觉打开 App）
2. **记录动作太重** → 摩擦趋近于零（扫一眼 + 点几下）
3. **反馈滞后** → 月历/连续天数让积累看得见
4. 加上两条：**灵活顺延绝不惩罚**（起晚了安排自动后移，不是失败）、**Kaoru 的温柔口吻**（提醒=牵挂，永不制造愧疚）

**产品定位**：只给用户自己一个人用的私人 App（无账号、不上架、数据全在手机本地）。"AI 恋人 + 生活管家"融合：同一个 AI 人格 **Kaoru**（温柔照顾型恋人）既管理日程/吃药/睡前记录，也随时可聊。调研确认"被关心的伴侣 + 被管理的日程"是市场空白。

**名字与氛围**：App 名 **Wisteria**（紫藤）。视觉 = 手账小票风 + 玻璃质感，风格定稿为"打印店·灰粉"（纸 #F4F2F0 + 灰玫瑰 #D4A5A5 + 石板蓝 #9BAAB8，冷调、零手写、花体 logo）——完整规格见 docs/界面与风格定稿.md。主屏幕显示名 Wisteria。

## 已确认需求（全部）

1. **日链**：起床 → 吃药（一日一次，起床时，一般≥12点）→ 三餐 → 学习/阅读 → 运动/出门 → 睡觉 → 睡前记录。**目标时间+顺延**：各事项=相对起床的偏移；点"起床"晚于目标时全链自动顺延并重排通知；任一事项完成晚了，其后事项顺延；开学后加"固定锚点"（上课时间不顺延）——预留设计。**界面形态**：日链 = 主页草稿纸上的待办行（固定待办 + 带时间戳的定时事项），完成即勾掉（见界面与风格定稿.md §五）。
2. **吃药状态机**（核心痛点）：当天状态一眼可见（未到/待确认/已吃/没吃）；到点通知带"吃了/没吃"按钮；未确认温柔再提醒（+30/+60/+120min 逐档变软）；已吃后处处可见防重复吃；睡前记录兜底强制确认。
3. **睡前记录**（每晚，<1 分钟）：事实清单自动生成（计划 vs 实际，扫一眼确认，一键"都对"）+ 自由文字（可空）+ 明天小安排（目标起床时间 + 1~3 件事）；**不含心情打分**。
4. **月历**：记录打点 + 药点（主脏粉=吃了 / 灰=没吃 / 空=没记录）+ 连续记录天数 + 陪伴天数（安装日起算）+ 共同纪念日（首次使用日/连续 100 天等里程碑）；**补记计入连续天数**（保护成就感）。
5. **AI 层**：随时聊（流式）+ 早安简报 + 夜间回应 + 每周日晚周分析 + 月末回顾信；主动消息 4 档位 + 静默时段（默认 23:00–8:00，吃药提醒豁免）；记忆三层（持久/近期摘要/可检索卡片），可视化可编辑；接口国内模型默认（DeepSeek/通义 OpenAI 兼容）+ 可切换（Anthropic 兼容预留）；key 存 Keychain；**先只做文字**（语音以后加）。
6. **仪式感**：简单陪伴天数/连续天数，不做等级/付费/剧情；断签给鼓励不给施压。

## 技术路线

| 项 | 选择 |
|---|---|
| 框架 | Flutter stable（Windows 开发/预览，Codemagic 云 Mac 构建 iOS） |
| 数据库 | drift（SQLite；类型安全 + watch 流驱动 Riverpod 实时联动） |
| 状态管理 | Riverpod 2.x（Notifier） |
| 通知 | flutter_local_notifications + timezone（iOS 真本地通知，杀 App 也弹——"外部触发"的生命线） |
| AI 客户端 | dio + SSE 流式（OpenAI 兼容；自封装 provider 抽象，不引小众 SDK） |
| Key 存储 | flutter_secure_storage（iOS Keychain） |
| 月历 | M1 自绘（精细控制格点编码）；备选 table_calendar |

## 目录结构（feature-first，关键文件）

```
lib/
├── main.dart / app.dart            # 初始化 + 主题（打印店·灰粉：纸底/灰玫瑰/石板蓝，见界面与风格定稿.md）
├── core/theme, utils/date_time_utils.dart   # 起床日身份、分钟换算、连续段计算
├── data/db/                        # app_database.dart, tables.dart, dao/
├── data/repos/                     # 聚合查询 + watch 流
├── data/local/secure_store.dart    # Keychain 封装（AI key）
├── features/home/                  # 主页：草稿纸待办（固定待办+定时事项）+ 月历框 + 睡前入口灯（原今日页，按界面定稿更名）
├── features/chain/
│   ├── domain/schedule_engine.dart # 顺延引擎（纯函数，核心单测对象）
│   └── application/chain_service.dart   # replan 唯一入口（幂等）
├── features/medication/
│   ├── domain/medication_state_machine.dart
│   └── application/medication_service.dart
├── features/sleep_record/          # 结账页（睡前记录=打印仪式：小票+章+归档账本）+ service（事实清单/事务写入/夜间回应）
├── features/diary/                 # diary 两分区：账本（每晚打印票归档）/ 日记（自由写，稿纸打印风）
├── features/calendar/              # 月历 + streak_service（连续/陪伴天数）
├── features/chat/                  # 聊天页（流式）
├── features/proactive/             # 主动消息档位/静默时段/早安简报
├── features/weekly/, monthly/      # 周分析 / 月末回顾信
├── features/settings/              # 设置 + 记忆可视化编辑页
├── ai/                             # ai_provider 抽象 + openai_compat + anthropic_compat(预留)
│                                   # persona/kaorus_persona.dart + context_builder + memory_service
└── notifications/                  # notification_service.dart（唯一出入口）+ notification_ids.dart
test/                               # schedule_engine / medication_state_machine / streak / summary_builder 单测
codemagic.yaml                      # iOS 构建签名流水线
```

## 核心设计

### 数据模型（drift）
- `day_records`（date PK = 起床日）：wakeTargetMinutes / wakeActual / wakeConfirmed / bedActual / medicationStatus（not_due|pending|taken|skipped）/ sleepNote / recordConfirmedAt（**连续天数判据**）/ factSnapshot JSON / tomorrowWakeTarget / tomorrowWishes JSON
- `chain_items`：dayDate + type + sortOrder + offsetMinutes + isFixedAnchor + anchorMinutes + plannedAt + actualAt + status（pending|done|skipped|missed）
- `medication_events`：dueAt / status / takenAt / remindCount / lastRemindAt / confirmSource
- `chat_messages`：role / kind（chat|morning_briefing|night_reply|weekly_report|monthly_letter|proactive）/ content / dayDate
- 记忆三层：`memory_entries`（kind=core 持久全量注入；kind=moment 可检索卡片）+ `day_summaries`（本地规则模板生成，零 token，AI 只带最近 7 条）
- `settings`（key-value：链偏移/默认起床目标/人设/主动档位/静默时段/AI base_url+model）+ `meta`（install_date 陪伴天数基准/各类去重时间戳）
- 连续天数/陪伴天数**实时计算**不落库（查近 90 天内存数连续段）

### 日链顺延引擎（纯函数，单测覆盖）
- "日"的身份 = **起床日**（凌晨 2 点睡不换日，DayRecord 永远跟着起床事件走）
- `replanDay`：anchor = 实际起床（未确认则用目标时间）；shift = 实际 − 目标；遍历链——已完成项锚住游标、固定锚点不顺延（错过标 missed 不阻塞后续）、其余 = anchor + offset + shift、plannedAt 落后于游标则压到"游标 + 最小间隔（默认 30min）"
- 触发时机：点"起床" / 任一项完成或跳过 / App 回前台 reconcile（迟到项 → 补记卡；系统时间跳变 → 全量重排）
- 通知重排：**固定 slot ID**（1000 明日起床、2000+sortOrder 链项、3000+count 吃药再提醒、4000 周分析、5000 主动消息、6000 月末信）——同 ID 重复调度即覆盖，天然幂等；总 pending ≈14 个 ≪ iOS 64 上限；每次重排后 `pendingNotificationRequests()` 自检；**不预排多天**

### 吃药状态机
- NOT_DUE →（到点）→ PENDING → TAKEN / SKIPPED；SKIPPED 可"现在补吃"（taken_late）
- 提醒节奏：到点 → +30min → +60min → +120min，仅排"下一档"通知；文案逐档变软（"药放在老地方了"→"帮你记着呢"→"不着急，我陪着你"）
- 通知 action（吃了/没吃）锁屏直操作；App 被杀时冷启动用 `getNotificationAppLaunchDetails()` 恢复并落库
- 防重复吃三重保障：主页草稿纸吃药行常驻 + 链上完成态 + 月历药点；睡前记录兜底强制确认（不确认不能提交）

### 睡前记录（<1 分钟单页）
事实清单（默认全对勾，一键"都对"）→ 自由文字（可空）→ 明天安排 → [都记好啦]
事务写入：更新当天 + 修正 actualAt + 生成明天 day_record/链项 + 生成本地摘要 + 排明日起床提醒 + 异步 Kaoru 夜间回应（AI 失败用本地模板兜底，存 kind=night_reply）

### AI 层（Kaoru）
- **单一对话**：与 Kaoru 只有一个持续会话，不做多会话/多窗口管理（小克Cat 已验证：真实陪伴不该让用户手动管理窗口；旧消息由摘要自动吸收，"上次聊到哪"由线头机制在周分析中体现）
- **人格模板**：温柔照顾型恋人；短句昵称；先接住情绪再给下一步；只用 `<context>` 真实数据、**宁可承认不知道绝不编造**（小克Cat 同款铁律）；不做医疗建议；行为源于内在状态而非定时器、内心独白第一人称、疲惫即安静（desire 系统三铁律）；**注入以 Kaoru 自己的记忆/回想口吻而非系统指令**（Eventide `_sys_tool` 思想）；日常 1~3 句，周分析/月末信 100~300 字像写信
- **亲密度与状态感知**：亲密度随真实互动自然增长（认识越久越懂你，影响称呼/语气/建议倾向，纯 prompt 注入，不显示为数值）；Kaoru 从客观数据感知你的状态（睡眠时长/链完成度/顺延次数 → "你最近累不累"），注入周分析；**不向用户收集心情打分**（维持已确认需求）
- **上下文注入（场景差异化，Eventide 思路）**：闲聊全量 ≈2k token（今日状态 ~150 + 近 7 日摘要 ~200 + 检索记忆 top-k ~150 + 持久记忆全量 ~100 + 最近 20 条消息）；吃药提醒/睡前确认只带药单+最近状态 ≈1KB；周分析/月末信 ≈800 token——**不同唤醒理由用不同上下文深度**
- **三层记忆**（Kindroid 模型 + 吸收 Pando/InternalBeyond/小克Cat/Paramecium）：持久（固定事实，全量注入）/ 近期摘要（本地生成零成本）/ 可检索卡片（**存原文为真源**，SQLite FTS5 全文检索，防摘要污染）；**单向沉淀代谢**：碎片→夜间整合→日摘要→候选卡片→（淡出但永不删除）；卡片带权重——被提及回温、久不提自然淡出仍可唤起（会遗忘、会回忆）；睡前记录与周分析自动沉淀候选卡（Auto Memory），检索仅在相关时注入（不提不占 token）；全部在设置→记忆页可视化、可编辑、可删除
- **主动消息**：档位 关闭/轻/标准(默认)/频繁 + 静默时段（=Kaoru 睡了：UI 同步变安静、主动停发、吃药提醒豁免）；**触发=日链事件+记忆命中**（"被记住才被关心"，AIRI），非定时随机；早安简报（起床确认后：今日日程+吃药提醒+一句关心，以回想口吻带出昨晚整合；同日去重）；周分析（周日 21:00 通知，App 前台生成，输入=7 条摘要+统计 ≈800 token）；月末信（每月最后一天前台生成；**先汇总当月再动笔**的读-写分离仪式）；**主动生成只在前台**（iOS 后台不可靠，不依赖 BGTask）
- **provider 抽象**：`OpenAiCompatProvider`（DeepSeek `https://api.deepseek.com` / 通义 dashscope compatible-mode）默认；`AnthropicCompatProvider` 预留（未来 Claude 中转）；30s 超时 + 重试 1 次 + 本地模板兜底（绝不空白）

## 调研吸收的设计决策（详见 docs/调研整合.md）

- 记忆可视化 + 可编辑（Kindroid）＝信任来源；"连续性本身就是产品价值"（独响）；永不回撤已上线的关系功能（Replika 2023 教训）
- 提醒引用上下文、防复读、频率硬上限（溪语）；"结构化事件 ≠ 用户话语"独立标签低优先级注入（pwa-sense-bridge）；"知道什么时候不说话"（dwell-on-something）
- 月历 = 共同记忆叙事；断签给鼓励、绝不施压（Replika 愧疚式设计反例）；陪伴天数只随真实互动自然增长（筑梦岛"讨好 AI"反例）
- 数据导出 = 信任底线（Soulmate 关停教训）→ M3+ 加 JSON 导出
- 陪伴锚定在真实生活数据上（国内新规精神 + 研究共识），不做成瘾机制
- 开源生态趋势：已从"聊天壳"走向"生活嵌入"；对标 Aura（重合 ~70%，License Private 仅借鉴设计）、BaiShou（Flutter+SQLite 日→周→月总结已验证）、ZeroChat（Flutter 同栈工程参考，MIT）；"温柔恋人+生活管家+单用户本地 iOS"组合仍是空白
- 今日页 = 家的入口（loveland 门户隐喻）：问候语+光线排版承载"到家感"，不是功能图标网格

**未来功能队列**（M3 之后按需）：表情包（sticker 规范）、Kaoru 的内心（desire 系统简化版 + Eventide 数值阈值驱动主动行为）、语音 TTS、Duet 式睡前对照日记、24h 一日曲线面板（数值+吃药/事件标注）、HealthKit 健康数据注入、共读/文字小游戏、本地小模型离线模式、2D 场景背景（昼夜呼应静默时段）

## 里程碑（2 周倒排，开学 ≈ 9/3）

- **D0–D1 环境 + 流水线**：flutter.cn 镜像装 SDK + 镜像环境变量（PUB_HOSTED_URL / FLUTTER_STORAGE_BASE_URL）；`flutter create --project-name ai_love`；git 私有仓 + Codemagic 接入 + 上传 p12；**当天用户下单 p12**（发货需数天，越早越好）；空壳 IPA 装进 iPhone（流水线冒烟——最大不确定性最早消除）
- **M1（D2–7）核心闭环（无 AI）**：表结构 + 设置 + 顺延引擎（单测）→ 主页（草稿纸待办+定时事项）+ 吃药状态机 + 通知调度 → 结账打印 + 月历 + 连续天数 + diary 账本 → **真机构建 #1** 验收
- **M2（D8–12）AI 层**：provider 抽象 + key 设置 + 聊天流式 + 上下文注入 → Kaoru 人格 + 三层记忆 + 主动档位 + 早安简报/夜间回应/周分析（含手动触发调试按钮）→ **真机构建 #2**
- **M3（D13–14+）打磨**：全 App 文案过 Kaoru 基调、动效（勾选划线/对勾/月历点亮/打印盖章）、通知权限引导、开屏页（Wisteria + 自动进入）、固定锚点入口灰态 → **真机构建 #3**；开学后开放固定锚点/上课静默，每 2~3 天小迭代
- 构建节奏：只对 release 分支构建 iOS（省 Codemagic 免费 500 分钟/月）；主分支只跑 flutter test；UI 细节在 Windows 上打磨，真机只做验收清单

## 验证方式

- **Windows 本地**：`flutter test`——顺延引擎（模拟 12:00 起床全链顺延且时间单调递增）、吃药状态机四态流转、连续天数（含补记）、摘要生成；`flutter run -d windows` 交互验证（通知走桩实现）
- **真机验收清单**（每次构建逐项勾）：通知权限申请时机、起床→全链通知重排、吃药四态 + 锁屏 action + 杀 App 后通知 + 冷启动 action 落库、睡前记录全流程、跨午夜（凌晨 2 点睡）、64 pending 自检日志、Keychain 存活、流式聊天
- **AI**：设置页手动触发按钮验证周分析/月末信/简报/夜间回应；断网验证本地模板兜底
- **流水线**：D0 空壳冒烟装进 iPhone；此后每里程碑构建一次真机包

## 风险与对策

- **p12 吊销/过期**（约 1 年）：选信誉商家、备份 p12 + 描述文件、到期前 1 个月续费重签重装
- **Codemagic 免费额度**：分支过滤只对 release 构建 iOS；不月末集中打包；实施时确认当前额度政策
- **国内网络**：本机 pub 走 flutter-io.cn 镜像；**Codemagic 在境外构建，不能给它配中国镜像**；App 运行时 AI 调国内模型直连（这正是不默认用境外模型的原因）
- **iOS 通知权限**：绝不在冷启动弹窗，在"设目标起床时间"引导后请求；被拒后 App 内温柔引导
- **drift 迁移**：M1 上线后表结构尽量冻结，改动走 schemaVersion 迁移；数据全本地；M3+ 加 JSON 导出 + 可选 FaceID 锁

## 实施前置（需用户办理，实施第一天）

1. **下单 p12 UDID 签名**（约百元/年；需要手机 UDID，爱思助手可查）——建议今天下单，发货需数天
2. **注册 AI 接口 key**（DeepSeek 或通义，充值几元即可用很久；key 只在设置页粘贴一次，存 Keychain）
3. Git 私有仓（GitHub/Gitee）供 Codemagic 拉代码
4. `参考资料/` 文件夹若还有文件放进来，实施前再吸收
