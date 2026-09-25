# 战术体力（Stamina）专业维基 · Professional Wiki

> **适用版本 / Applies to**：Stamina（modId `herra_stamina`）v1.0.3 · Minecraft 1.21.1 · NeoForge 21.1.216+ · LDLib2 2.2.1+
> **文档约定 / Conventions**：每个章节先中文后英文，同一章节内两语言内容等价；代码示例为两种语言共用，注释双语。
> Each section provides Chinese first, then an equivalent English version. Code samples are shared, with bilingual comments.

---

## 1. 概述 · Overview

**【中文】**

战术体力（Stamina）是一款为搜打撤（撤离射击）整合包打造的独立战术体力系统模组，运行于 Minecraft 1.21.1 / NeoForge 平台。它将撤离射击游戏的体力手感引入 Minecraft：疾跑、跳跃、游泳、近战攻击、破坏方块都会消耗体力；体力耗尽后进入"透支锁定"——禁止疾跑与跳跃、只能正常行走，必须回气到阈值才能恢复行动能力。

设计目标有三：

- **单一职责**：只做体力系统本身，不含战局、播报、枪械等内容，与其他模组零硬耦合，可独立启用或禁用；
- **服务端权威**：全部数值与规则在服务端 tick 循环内计算，客户端只负责渲染与表现，无法通过修改客户端获得任何优势；
- **生态友好**：通过 `com.herra.stamina.api` 提供静态门面 API、定时修改器、消耗规则修改器与四个体力事件，供医药、负重、战局类模组单向集成。

模组自带像素风战术 HUD（4 种样式预设、拖拽定位、动画反馈、耗尽心跳提示音）与游戏内 `/sta` 设置界面，玩家侧表现完全可个性化且不影响他人。

**【English】**

Stamina (Chinese: 战术体力, "Tactical Stamina") is a standalone tactical stamina system mod for extraction-shooter ("search-and-raid") modpacks on Minecraft 1.21.1 / NeoForge. It brings extraction-shooter stamina feel into Minecraft: sprinting, jumping, swimming, melee attacks and block breaking all drain stamina; when stamina hits zero the player enters an **exhaustion lock** — sprint and jump disabled, walking only — and must recover above a threshold before full mobility returns.

Design goals:

- **Single responsibility** — the mod implements the stamina system and nothing else. No raids, kill-feed or gunplay; zero hard coupling with other mods, each may be enabled or disabled independently.
- **Server-authoritative** — all values and rules are computed in the server tick loop; the client only renders. No client-side advantage is possible.
- **Ecosystem-friendly** — a static facade API, timed modifiers, drain-rule modifiers and four events under `com.herra.stamina.api` allow medical / encumbrance / raid mods to integrate one-way.

The mod ships a pixel-art tactical HUD (4 style presets, drag-to-position, animated feedback, exhaustion heartbeat sound) and an in-game `/sta` settings screen; every visual aspect is per-player and never affects others.

---

## 2. 运行环境与安装 · Requirements & Installation

**【中文】**

| 项目 | 要求版本 | 开发验证版本 |
| --- | --- | --- |
| Minecraft | 1.21.1（声明区间 1.21.1 ~ 1.21.x） | 1.21.1 |
| NeoForge | ≥ 21.1.216 | 21.1.251 |
| LDLib2（前置） | ≥ 2.2.1，双端 | 2.2.41 |
| Java | 21 | OpenJDK 21.0.2 |

安装步骤：

1. 客户端与服务端（联机时）均安装 NeoForge 21.1.216+；
2. 将 `herra_stamina-<版本>.jar` 与 LDLib2 放入 `mods/`；
3. 单人游戏仅需客户端一份（内置服务端自动生效）。

**网络拓扑与连接矩阵**：

| 场景 | 是否可用 | 说明 |
| --- | --- | --- |
| 客户端 + 服务端均安装（联机） | ✅ 完整功能 | 服务端权威计算 + 客户端 HUD 与 Mixin |
| 仅客户端安装 → 进入原版服务器 | ✅ 可连接 | 体力系统休眠，HUD 无数据不显示，无报错 |
| 原版客户端 → 连接装有本模组的服务器 | ❌ 被拒绝 | 服务端注册了必选 S2C 通信通道，握手阶段断开 |
| 单人游戏（仅客户端安装） | ✅ 完整功能 | 内置服务端即模组服务端 |

> 原理：模组声明 `displayTest = "IGNORE_SERVER_VERSION"` 且**没有任何客户端→服务端的自定义包**，因此装了模组的客户端不会因版本检测被原版服务器拒绝；反向则不成立——NeoForge 会把本模组的两个 `playToClient` 通道标记为必选，原版客户端在协商阶段即被断开，而非"能进但没 HUD"。

**【English】**

| Component | Required | Dev-tested |
| --- | --- | --- |
| Minecraft | 1.21.1 (declared range 1.21.1 – 1.21.x) | 1.21.1 |
| NeoForge | ≥ 21.1.216 | 21.1.251 |
| LDLib2 (dependency) | ≥ 2.2.1, both sides | 2.2.41 |
| Java | 21 | OpenJDK 21.0.2 |

Installation:

1. Install NeoForge 21.1.216+ on the client and — for multiplayer — on the server;
2. Drop `herra_stamina-<version>.jar` plus LDLib2 into `mods/`;
3. Singleplayer needs the client copy only (the integrated server takes the server role).

**Network topology matrix**:

| Scenario | Works? | Notes |
| --- | --- | --- |
| Both sides installed (multiplayer) | ✅ Full feature set | Server-authoritative values + client HUD/mixin |
| Modded client → vanilla server | ✅ Connects | Stamina system dormant; HUD shows nothing; no errors |
| Vanilla client → modded server | ❌ Rejected | Server registers mandatory S2C channels; disconnect during negotiation |
| Singleplayer (client install only) | ✅ Full feature set | Integrated server acts as the mod's server side |

> Rationale: the mod declares `displayTest = "IGNORE_SERVER_VERSION"` and registers **no client-to-server custom payloads**, so a modded client is never rejected by a vanilla server. The reverse does not hold — NeoForge marks the mod's two `playToClient` channels as required, so a vanilla client is dropped during channel negotiation ("cannot join", not "joins without HUD").

---

## 3. 玩法机制 · Gameplay Mechanics

**【中文】**

### 3.1 消耗动作

| 动作 | 结算方式 | 默认消耗 | 配置键 |
| --- | --- | --- | --- |
| 疾跑 | 持续，每 tick | 12/秒（满体力约 8.3 秒） | `drain.sprint_per_second` |
| 跳跃 | 一次性 | 10/次 | `drain.jump_cost` |
| 游泳 | 持续，每 tick | 5/秒 | `drain.swim_per_second` |
| 疾速游泳 | 持续，每 tick | 9/秒 | `drain.swim_sprint_per_second` |
| 近战攻击（命中实体） | 一次性 | 2/次（0 = 关闭） | `drain.attack_cost` |
| 破坏方块 | 一次性 | 1/块（0 = 关闭） | `drain.break_block_cost` |
| 第三方模组自定义 | 原子或部分扣除 | 由调用方指定 | API `tryConsume` / `drain` |

疾跑中跳跃 = 该 tick 的疾跑消耗 **加上** 跳跃一次性消耗，两者独立结算。

### 3.2 数值公式

所有公式在服务端每 tick 结算（`/20` 表示按秒速率换算到单 tick）：

```
单次消耗      cost = 基础消耗 × Π(修改器 drainMultiplier)      （再经 StaminaDrainModifier 链与 Consume 事件处理）
持续消耗/tick cost = 每秒速率 / 20 × Π(drainMultiplier)
恢复速度/秒   regen = 基础恢复 × Π(regenMultiplier) + Σ(regenBonus)    （正常与透支恢复均生效）
最终体力上限  max  = (配置基础值 + Σ(maxStaminaBonus)) × Π(maxStaminaMultiplier)，下限 1
```

### 3.3 状态机

```
                 体力 ≤ sprint_stop_threshold (15)          体力归零
  ┌─────────┐  ────────────────────────────────────►  ┌──────────┐  ──────►  ┌────────────┐
  │  正常    │                                        │ 呼吸急促  │           │  体力透支   │
  │ NORMAL  │  ◄───────────────────────────────────   │ WINDED   │  ◄──────  │ EXHAUSTED  │
  └─────────┘    体力恢复 > 15                        └──────────┘   恢复到 30  └────────────┘
                                                                     (release_threshold) 解锁
```

| 状态 | 判定 | 效果 |
| --- | --- | --- |
| 正常 NORMAL | 体力 > 15 | 无限制 |
| 呼吸急促 WINDED | 体力 ≤ 15 且未透支 | 立即禁止疾跑（可配置额外禁跳，默认保留一跳逃生） |
| 体力透支 EXHAUSTED | 体力归零 | 禁疾跑 + 禁跳跃；恢复延迟 3 秒（额外 60 tick）、恢复速度降为 15/秒；可选行走减速；恢复到 30 解除 |

### 3.4 惩罚执行细节

- **禁跑**：客户端 Mixin 在原版饥饿禁跑的同一判定点（`hasEnoughFoodToStartSprinting`）压制，覆盖按住疾跑、切换式疾跑、双击 W 三种启动方式，且无疾跑 FOV 抖动；服务端保留 `setSprinting(false)` 兜底。透支进入/解除瞬间零延迟同步，不存在 1~2 tick 的可趁之窗。
- **禁跳**：通过同步的 `JUMP_STRENGTH` 属性修饰符（`ADD_VALUE -1024`，属性钳到 0）实现，双端判定一致、无橡皮筋；透支期间跳跃药水仍有 0.1/级的微弱残余弹跳（原版公式加算，可忽略）。
- **赌命跳**：透支前的最后一跳允许——跳跃照常执行、体力扣到空并直接进入透支。这是有意保留的战术空间。
- **行走减速**：`MOVEMENT_SPEED` 乘算修饰符（`ADD_MULTIPLIED_TOTAL`），仅影响地面移速，不影响击退与坠落；运行中改配置即时重挂。

### 3.5 边界与豁免

- 创造 / 旁观模式：不消耗（`general.apply_in_creative` 可改为创造也消耗，创造模式下 HUD 默认自动隐藏）；
- 死亡重生：满体力复活；
- 骑乘、鞘翅滑翔：不计入疾跑消耗（防误判）；
- 游泳：原地踩水不扣；主动上浮/下潜按垂直位移计入（阈值 0.05 格/tick，排除被动下沉的 0.02~0.03 格/tick）；游泳状态按跳跃上浮不触发跳跃消耗，从水底蹬地起跳正常结算；
- 存续性：体力为运行时数据（Attachment，不落盘），重进服务器满体力开局；修改器同样为运行时数据，退出即清。

**【English】**

### 3.1 Drain Actions

| Action | Settlement | Default cost | Config key |
| --- | --- | --- | --- |
| Sprint | continuous, per tick | 12/s (~8.3 s from full) | `drain.sprint_per_second` |
| Jump | one-shot | 10 | `drain.jump_cost` |
| Swim | continuous, per tick | 5/s | `drain.swim_per_second` |
| Sprint-swim | continuous, per tick | 9/s | `drain.swim_sprint_per_second` |
| Melee attack (hit) | one-shot | 2 (0 = off) | `drain.attack_cost` |
| Block break | one-shot | 1 (0 = off) | `drain.break_block_cost` |
| Third-party custom | atomic or partial | caller-defined | API `tryConsume` / `drain` |

Sprinting while jumping costs the sprint drain for that tick **plus** the one-shot jump cost — settled independently.

### 3.2 Formulas

All formulas settle server-side every tick (`/20` converts a per-second rate to per-tick):

```
One-shot cost     cost = base × Π(modifier drainMultiplier)      (then the StaminaDrainModifier chain and Consume event)
Continuous/tick   cost = perSecond / 20 × Π(drainMultiplier)
Regen per second  regen = base regen × Π(regenMultiplier) + Σ(regenBonus)    (applies to both normal and exhausted regen)
Final max stamina max  = (config base + Σ(maxStaminaBonus)) × Π(maxStaminaMultiplier), floored at 1
```

### 3.3 State Machine

```
                 stamina ≤ sprint_stop_threshold (15)           stamina reaches 0
  ┌─────────┐  ────────────────────────────────────►  ┌──────────┐  ──────►  ┌────────────┐
  │ NORMAL  │                                        │ WINDED   │           │ EXHAUSTED  │
  └─────────┘  ◄───────────────────────────────────   └──────────┘  ◄──────  └────────────┘
                  stamina recovers > 15                    unlocks at 30 (release_threshold)
```

| State | Condition | Effect |
| --- | --- | --- |
| NORMAL | stamina > 15 | unrestricted |
| WINDED | stamina ≤ 15, not exhausted | sprint blocked immediately (optional jump block, off by default — the last jump is kept as an escape tool) |
| EXHAUSTED | stamina = 0 | sprint + jump blocked; regen delay 3 s (extra 60 ticks) and regen rate 15/s; optional walk slowdown; releases at 30 |

### 3.4 Penalty Enforcement Details

- **Sprint block** — a client-side mixin suppresses sprinting at the exact vanilla hunger-check point (`hasEnoughFoodToStartSprinting`), covering hold-sprint, toggle-sprint and double-tap-W with no FOV jitter; the server keeps a `setSprinting(false)` backstop. Lock enter/leave transitions sync with zero delay — there is no 1–2 tick exploit window.
- **Jump block** — implemented via a synced `JUMP_STRENGTH` attribute modifier (`ADD_VALUE -1024`, clamping the attribute to 0); consistent on both ends, no rubber-banding. Jump boost potions retain a negligible 0.1/level residual bounce (vanilla additive formula).
- **The gamble jump** — the last jump before exhaustion is intentionally allowed: the jump executes, stamina drains to zero and the exhaustion lock engages immediately.
- **Walk slowdown** — a `MOVEMENT_SPEED` multiplier modifier (`ADD_MULTIPLIED_TOTAL`); affects ground speed only, never knockback or falling; hot-reapplied when the config changes at runtime.

### 3.5 Edge Cases & Exemptions

- Creative / spectator: no drain (`general.apply_in_creative` can enable creative drain; the HUD auto-hides in creative by default);
- Respawn: full stamina;
- Riding / elytra gliding: never miscounted as sprinting;
- Swimming: treading water in place is free; deliberate ascent/descent counts via vertical displacement (threshold 0.05 blocks/tick, excluding passive sinking of ~0.02–0.03); pressing jump to surface while swimming does **not** trigger the jump cost — only pushing off the pool floor (grounded) does;
- Persistence: stamina is runtime data (Attachment, not saved) — every session starts at full; modifiers are runtime data too and clear on logout.

---

## 4. HUD 与设置界面 · HUD & Settings UI

**【中文】**

### 4.1 渲染层

HUD 宿主为 LDLib2 的 `ModularHudLayer`（注册于 `RegisterGuiLayersEvent.registerAboveAll`），自绘 `UIElement.drawContents` 内直接使用原生 `GuiGraphics` 做像素级绘制；着色全部走顶点色 / SpriteTexture，规避 `RenderSystem.setShaderColor` 在批量渲染下不可靠的问题。整体缩放通过 pose 矩阵（translate + scale）实现，内部几何零改动。

### 4.2 位置与几何

- 默认位于屏幕底部正中、物品栏上方：体力条**底边**距屏幕底部 84 像素（`hud.offset_y`，位于护甲条之上、氧气条上方，避开常见自定义血条）；水平偏移 `hud.offset_x`（0 = 对齐物品栏中心）；
- 条体逻辑尺寸 100×12 像素（边框贴图），填充区 96×8；整体缩放 0.5 ~ 2.0；
- 闪电图标 9×9，身体状态图标（头/胸/臂/腿）各 7×7，为后续身体部位系统预留。

### 4.3 动画与反馈

| 事件 | 表现 |
| --- | --- |
| 数值变化 | 平滑插值（速度倍率 `hud.animation_speed`，0.1~10.0） |
| 大额扣减 | 格斗游戏式「幽灵拖尾」：被扣部分逐列白→黄→红渐变消失 + 白闪 + 像素火花迸溅 |
| 低体力 | 红色呼吸脉动；条下状态文字「呼吸急促」 |
| 归零透支 | 硬闪烁；状态文字「体力透支」；低频心跳提示音 |
| 回满 | 扫光 → 延迟 `hud.hide_delay_ticks`（默认 40 tick = 2 秒）自动隐藏，再次消耗快速淡入 |

### 4.4 样式预设

| 样式 id | 名称 | 边框 | 配色（充足→中等→低→临界，hex） |
| --- | --- | --- | --- |
| classic | 经典 | 原版金蓝像素边框 + 分段刻痕 | `#EDF3E2` → `#F2C94C` → `#E8873B` → `#D64541`（走客户端配置，可自定义） |
| tactical | 战术 | 装甲块边框 + 硬分段刻度 | `#B8E62E` → `#E8C43D` → `#E8873B` → `#E04A3A` |
| minimal | 极简 | 1px 细线边框 | `#E8EEF2` → `#BFC9D4` → `#E8A44C` → `#E05A50` |
| energy | 生电 | 青色辉光切角 + 斜纹高光 | `#52F0E0` → `#4CA0F0` → `#9A70F0` → `#F05090` |

### 4.5 耗尽提示音

`herra_stamina:exhausted`——合成音效（0.92 秒低频心跳双响：55 Hz 衰减正弦 + 泛音 + 低鸣，峰值 -2.5 dBFS，Ogg Vorbis q5），带双语字幕。体力归零瞬间播放（初始化首包不触发）；音量 0~100 线性映射到 0~1.25 增益，UI 通道播放，可在 `/sta` 界面试听。

### 4.6 /sta 设置界面

玩家级指令 `/sta` 通过 S2C 包打开全代码绘制的战术设置面板（暗色军规底 + 琥珀强调 + 扫描线 + 角落括号）：

- **摆位**：拖拽小地图直接放置体力条（物品栏参考框 / 中心虚线 / 悬停十字线，与 HUD 定位公式互逆 + 抓取偏移补偿），或水平 X / 高度 Y / 缩放三个自绘滑条，一键居中复位；
- **样式**：四预设按钮（带四段配色带）+ 动态大预览（呼吸填充 + ghost 残影）；
- **提示音**：开关 + 音量滑条 + 试听按钮；
- **显示项**：闪电图标 / 状态文字 / 身体图标 / 自动隐藏四个 LED 开关 + 动画速度滑条；
- **持久化**：修改即时生效（`ConfigValue.set`），关闭界面时 `SPEC.save()` 写回 `config/herra_stamina-client.toml`，仅对自己生效。

**【English】**

### 4.1 Rendering Layer

The HUD is hosted by LDLib2's `ModularHudLayer` (registered via `RegisterGuiLayersEvent.registerAboveAll`); a custom `UIElement.drawContents` override draws pixel-perfect content with the native `GuiGraphics`. All tinting goes through vertex colors / SpriteTexture, avoiding `RenderSystem.setShaderColor` being unreliable under batched rendering. Global scaling applies a pose-matrix transform (translate + scale) without touching internal geometry.

### 4.2 Position & Geometry

- Defaults to bottom-center above the hotbar: the bar's **bottom edge** sits 84 px above the screen bottom (`hud.offset_y` — above the armor row and oxygen bar, clear of common custom health bars); horizontal offset via `hud.offset_x` (0 = hotbar-centered);
- Logical bar size 100×12 px (frame texture) with a 96×8 fill area; global scale 0.5–2.0;
- Bolt icon 9×9; body-status icons (head/chest/arms/legs) 7×7 each — reserved for a future body-part system.

### 4.3 Animations & Feedback

| Event | Presentation |
| --- | --- |
| Value change | smooth interpolation (speed via `hud.animation_speed`, 0.1–10.0) |
| Large one-shot drain | fighting-game "ghost trail": the lost portion fades column-by-column white→yellow→red, plus white flash and pixel sparks |
| Low stamina | red breathing pulse; status text "WINDED" |
| Zero / exhaustion | hard flash; status text "EXHAUSTED"; low-frequency heartbeat cue |
| Full | sweep shine → auto-hide after `hud.hide_delay_ticks` (default 40 ticks = 2 s), quick fade-in on next drain |

### 4.4 Style Presets

| Style id | Name | Frame | Palette (full→mid→low→crit, hex) |
| --- | --- | --- | --- |
| classic | Classic | vanilla gold-blue pixel frame + notches | `#EDF3E2` → `#F2C94C` → `#E8873B` → `#D64541` (driven by client config, customizable) |
| tactical | Tactical | armor-block frame + hard segments | `#B8E62E` → `#E8C43D` → `#E8873B` → `#E04A3A` |
| minimal | Minimal | 1 px thin-line frame | `#E8EEF2` → `#BFC9D4` → `#E8A44C` → `#E05A50` |
| energy | Energy | cyan glow chamfered frame + diagonal stripes | `#52F0E0` → `#4CA0F0` → `#9A70F0` → `#F05090` |

### 4.5 Exhaustion Sound

`herra_stamina:exhausted` — a synthesized 0.92 s low-frequency heartbeat double-thump (55 Hz decaying sine + harmonics + low hum, peak -2.5 dBFS, Ogg Vorbis q5) with bilingual subtitles. Plays the moment stamina hits zero (the initial sync packet is exempt); volume 0–100 maps linearly to 0–1.25 gain on the UI sound channel, test-listenable from the `/sta` screen.

### 4.6 The /sta Settings Screen

The player-level `/sta` command opens (via an S2C payload) a fully code-drawn tactical settings panel — dark military backdrop, amber accents, scanlines and corner brackets:

- **Positioning** — drag the bar on a mini-map (hotbar reference frame, center guide line, hover crosshair; mathematically inverse to the HUD positioning formula with grab-offset compensation), or use the Offset X / Height Y / Scale sliders; one-click recenter;
- **Styles** — four preset buttons with four-segment color bands + a large live preview (breathing fill + ghost);
- **Sound** — toggle, volume slider, test-listen button;
- **Display items** — four LED toggles (bolt icon / status text / body icons / auto-hide) plus an animation-speed slider;
- **Persistence** — changes apply live (`ConfigValue.set`); closing the screen saves via `SPEC.save()` to `config/herra_stamina-client.toml`, affecting only that player.

---

## 5. 配置参考 · Configuration Reference

**【中文】**

两份配置文件分工明确：**服务端配置管玩法数值**（整个服务器统一），**客户端配置管 HUD 表现**（每个玩家各自生效）。两者均被文件监听器监视，手动编辑保存即热重载；`/sta` 指令的修改会直接写入服务端 TOML 并落盘。

### 5.1 服务端 `herra_stamina-server.toml`

位置：世界目录 `serverconfig/herra_stamina-server.toml`（单机为 `saves/<世界>/serverconfig/`）。

| 键 | 类型 | 取值范围 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| general.max_stamina | double | 1 ~ 1000000 | 100 | 最大体力值 |
| general.apply_in_creative | bool | — | false | 创造模式是否消耗（关闭时创造下 HUD 自动隐藏） |
| drain.sprint_per_second | double | 0 ~ 100000 | 12 | 疾跑每秒消耗（12 = 满体力约 8.3 秒） |
| drain.jump_cost | double | 0 ~ 100000 | 10 | 跳跃一次性消耗（疾跑中跳跃为疾跑消耗 + 该值） |
| drain.swim_per_second | double | 0 ~ 100000 | 5 | 游泳每秒消耗 |
| drain.swim_sprint_per_second | double | 0 ~ 100000 | 9 | 疾速游泳每秒消耗 |
| drain.attack_cost | double | 0 ~ 100000 | 2 | 近战攻击命中消耗（0 = 关闭） |
| drain.break_block_cost | double | 0 ~ 100000 | 1 | 破坏方块消耗（0 = 关闭） |
| recovery.delay_ticks | int | 0 ~ 1200 | 24 | 停止消耗后恢复延迟（20 tick = 1 秒） |
| recovery.per_second | double | 0 ~ 100000 | 25 | 正常恢复速度/秒 |
| recovery.exhausted_delay_ticks | int | 0 ~ 1200 | 60 | 透支后额外恢复延迟（tick） |
| recovery.exhausted_per_second | double | 0 ~ 100000 | 15 | 透支恢复速度/秒 |
| penalty.sprint_stop_threshold | double | 0 ~ 1000000 | 15 | 低于该值禁止疾跑 |
| penalty.exhausted_release_threshold | double | 0 ~ 1000000 | 30 | 透支解除阈值（回气到该值才恢复疾跑） |
| penalty.exhausted_block_jump | bool | — | true | 透支期间禁止跳跃（JUMP_STRENGTH 修饰符，双端同步） |
| penalty.exhausted_block_sprint | bool | — | true | 透支期间禁止疾跑（严格模式核心开关） |
| penalty.exhausted_walk_slowdown | double | 0 ~ 0.6 | 0.0 | 透支行走减速比例（0.15 = 移速 ×0.85；仅影响地面移速） |
| penalty.winded_block_jump | bool | — | false | 低体力（未透支）也禁跳（默认保留最后一跳逃生） |
| network.sync_interval_ticks | int | 1 ~ 20 | 2 | 同步最小间隔（tick，一般无需修改） |
| network.sync_delta | double | 0.1 ~ 100 | 2.0 | 触发同步的最小数值变化量 |

### 5.2 客户端 `herra_stamina-client.toml`

位置：客户端 `config/herra_stamina-client.toml`，每位玩家独立。

| 键 | 类型 | 取值范围 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| hud.enabled | bool | — | true | 显示体力条 |
| hud.offset_x | int | -1000 ~ 1000 | 0 | 水平偏移（0 = 物品栏中心对齐） |
| hud.offset_y | int | 0 ~ 1000 | 84 | 底边距屏幕底部像素数（越大越靠上） |
| hud.scale | double | 0.5 ~ 2.0 | 1.0 | 整体缩放 |
| hud.style | enum | classic/tactical/minimal/energy | classic | 样式预设 |
| hud.auto_hide | bool | — | true | 回满后自动隐藏 |
| hud.hide_delay_ticks | int | 0 ~ 600 | 40 | 回满后隐藏延迟（tick） |
| hud.show_icon | bool | — | true | 闪电图标 |
| hud.show_body_status | bool | — | true | 身体状态占位图标 |
| hud.show_status_text | bool | — | true | 低体力/透支状态文字 |
| hud.animation_speed | double | 0.1 ~ 10.0 | 1.0 | 动画速度倍率 |
| sound.exhausted_enabled | bool | — | true | 耗尽提示音开关 |
| sound.exhausted_volume | int | 0 ~ 100 | 60 | 提示音音量 |
| colors.full | int(hex) | 0x000000 ~ 0xFFFFFF | 0xEDF3E2 | 经典样式充足色 |
| colors.mid | int(hex) | 0x000000 ~ 0xFFFFFF | 0xF2C94C | 经典样式中色 |
| colors.low | int(hex) | 0x000000 ~ 0xFFFFFF | 0xE8873B | 经典样式低色 |
| colors.crit | int(hex) | 0x000000 ~ 0xFFFFFF | 0xD64541 | 经典样式临界色 |

> 配色仅作用于「经典」样式；tactical / minimal / energy 预设自带固定配色。

**【English】**

Two files split responsibilities cleanly: the **server config owns gameplay numbers** (uniform server-wide), the **client config owns HUD presentation** (per player). Both are watched by file listeners and hot-reload on save; `/sta config` writes go straight into the server TOML and persist.

### 5.1 Server — `herra_stamina-server.toml`

Location: `<world>/serverconfig/herra_stamina-server.toml` (singleplayer: `saves/<world>/serverconfig/`).

| Key | Type | Range | Default | Description |
| --- | --- | --- | --- | --- |
| general.max_stamina | double | 1 – 1,000,000 | 100 | Maximum stamina |
| general.apply_in_creative | bool | — | false | Drain in creative (HUD auto-hides in creative while off) |
| drain.sprint_per_second | double | 0 – 100,000 | 12 | Sprint drain per second (12 ≈ 8.3 s from full) |
| drain.jump_cost | double | 0 – 100,000 | 10 | One-shot jump cost (sprint-jump adds this on top of sprint drain) |
| drain.swim_per_second | double | 0 – 100,000 | 5 | Swim drain per second |
| drain.swim_sprint_per_second | double | 0 – 100,000 | 9 | Sprint-swim drain per second |
| drain.attack_cost | double | 0 – 100,000 | 2 | Melee-hit cost (0 = off) |
| drain.break_block_cost | double | 0 – 100,000 | 1 | Block-break cost (0 = off) |
| recovery.delay_ticks | int | 0 – 1,200 | 24 | Regen delay after last drain (20 ticks = 1 s) |
| recovery.per_second | double | 0 – 100,000 | 25 | Normal regen per second |
| recovery.exhausted_delay_ticks | int | 0 – 1,200 | 60 | Extra regen delay while exhausted (ticks) |
| recovery.exhausted_per_second | double | 0 – 100,000 | 15 | Exhausted regen per second |
| penalty.sprint_stop_threshold | double | 0 – 1,000,000 | 15 | Below this, sprinting is blocked |
| penalty.exhausted_release_threshold | double | 0 – 1,000,000 | 30 | Stamina must recover to this to leave the exhaustion lock |
| penalty.exhausted_block_jump | bool | — | true | Block jumping while exhausted (JUMP_STRENGTH modifier, synced) |
| penalty.exhausted_block_sprint | bool | — | true | Block sprinting while exhausted (strict-mode master switch) |
| penalty.exhausted_walk_slowdown | double | 0 – 0.6 | 0.0 | Walk slowdown ratio while exhausted (0.15 = ×0.85 speed; ground only) |
| penalty.winded_block_jump | bool | — | false | Also block jumping when winded (default keeps a last-jump escape) |
| network.sync_interval_ticks | int | 1 – 20 | 2 | Minimum sync interval in ticks (rarely needs changing) |
| network.sync_delta | double | 0.1 – 100 | 2.0 | Minimum value delta that triggers a sync |

### 5.2 Client — `herra_stamina-client.toml`

Location: `config/herra_stamina-client.toml`, per player.

| Key | Type | Range | Default | Description |
| --- | --- | --- | --- | --- |
| hud.enabled | bool | — | true | Show the stamina bar |
| hud.offset_x | int | -1000 – 1000 | 0 | Horizontal offset (0 = hotbar-centered) |
| hud.offset_y | int | 0 – 1000 | 84 | Bottom edge above screen bottom, in px |
| hud.scale | double | 0.5 – 2.0 | 1.0 | Global scale |
| hud.style | enum | classic/tactical/minimal/energy | classic | Style preset |
| hud.auto_hide | bool | — | true | Auto-hide when full |
| hud.hide_delay_ticks | int | 0 – 600 | 40 | Hide delay after full (ticks) |
| hud.show_icon | bool | — | true | Bolt icon |
| hud.show_body_status | bool | — | true | Body-status placeholder icons |
| hud.show_status_text | bool | — | true | Low/exhausted status text |
| hud.animation_speed | double | 0.1 – 10.0 | 1.0 | Animation speed multiplier |
| sound.exhausted_enabled | bool | — | true | Exhaustion cue toggle |
| sound.exhausted_volume | int | 0 – 100 | 60 | Cue volume |
| colors.full | int(hex) | 0x000000 – 0xFFFFFF | 0xEDF3E2 | Classic palette — full |
| colors.mid | int(hex) | 0x000000 – 0xFFFFFF | 0xF2C94C | Classic palette — mid |
| colors.low | int(hex) | 0x000000 – 0xFFFFFF | 0xE8873B | Classic palette — low |
| colors.crit | int(hex) | 0x000000 – 0xFFFFFF | 0xD64541 | Classic palette — critical |

> The palette applies to the **Classic** style only; tactical / minimal / energy presets carry fixed palettes.

---

## 6. 指令参考 · Command Reference

**【中文】**

### 6.1 玩家级（无需 OP）

| 指令 | 说明 |
| --- | --- |
| `/sta` / `/sta ui` | 打开设置界面（S2C 包 → 客户端弹窗，调整仅对自己生效） |
| `/sta help` | 指令帮助 |
| `/stamina` | 查看自己的体力值与状态（全称指令仅保留自查用途） |

### 6.2 管理级（OP 权限等级 2；控制台始终允许）

| 指令 | 参数范围 | 说明 |
| --- | --- | --- |
| `/sta info <玩家>` | — | 查看体力/状态/修改器（查自己无需 OP） |
| `/sta set <玩家> <数值>` | 0 ~ 1000000 | 直接设置 |
| `/sta add <玩家> <数值>` | -1000000 ~ 1000000 | 增减 |
| `/sta exhaust <玩家>` | — | 清空 + 透支锁（测试惩罚） |
| `/sta reset <玩家>` | — | 回满 |
| `/sta modifier list [玩家]` | — | 列出生效中的修改器与剩余时间 |
| `/sta modifier clear <玩家>` | — | 清空修改器 |
| `/sta modifier give <玩家> <id> <秒> [消耗×] [上限加算] [恢复/秒]` | 秒 1~86400；消耗× 0~100；上限 ±1000000；恢复 ±100000 | 挂测试增益；id 含冒号需引号 |
| `/sta config show` | — | 列出全部服务器数值 |
| `/sta config <参数> <值>` | 见下表 | 修改并写盘 |

`config` 参数 ↔ 配置键对照：

| 参数 | 配置键 | 值类型/范围 |
| --- | --- | --- |
| max | general.max_stamina | float 1 ~ 1000000 |
| sprint-drain | drain.sprint_per_second | float 0 ~ 100000 |
| jump-cost | drain.jump_cost | float 0 ~ 100000 |
| swim-drain | drain.swim_per_second | float 0 ~ 100000 |
| swim-sprint-drain | drain.swim_sprint_per_second | float 0 ~ 100000 |
| attack-cost | drain.attack_cost | float 0 ~ 100000 |
| break-cost | drain.break_block_cost | float 0 ~ 100000 |
| regen | recovery.per_second | float 0 ~ 100000 |
| regen-exhausted | recovery.exhausted_per_second | float 0 ~ 100000 |
| delay | recovery.delay_ticks | int 0 ~ 1200 |
| delay-exhausted | recovery.exhausted_delay_ticks | int 0 ~ 1200 |
| sprint-stop | penalty.sprint_stop_threshold | float 0 ~ 1000000 |
| release | penalty.exhausted_release_threshold | float 0 ~ 1000000 |
| walk-slowdown | penalty.exhausted_walk_slowdown | float 0 ~ 0.6 |
| block-jump | penalty.exhausted_block_jump | bool |
| block-sprint | penalty.exhausted_block_sprint | bool |
| winded-block-jump | penalty.winded_block_jump | bool |

```
/sta config sprint-drain 8                        # 疾跑消耗降至 8/秒 | lower sprint drain to 8/s
/sta config walk-slowdown 0.15                    # 透支移速 ×0.85 | exhausted speed ×0.85
/sta modifier give Steve "med:test" 60 0.5 30 5   # 60 秒：消耗减半、上限+30、恢复+5/s
                                                  # 60 s: drain ×0.5, max +30, regen +5/s
```

> 所有 `config` 修改通过 `ConfigValue.set` + `SPEC.save()` 直接写入服务端 TOML，立即生效且重启不丢；手动编辑文件同样被监听热重载。

**【English】**

### 6.1 Player-level (no OP required)

| Command | Description |
| --- | --- |
| `/sta` / `/sta ui` | Open the settings screen (S2C payload → client-side screen; changes affect only that player) |
| `/sta help` | Command help |
| `/stamina` | View your own stamina and state (the long-form command is kept for self-checks only) |

### 6.2 Admin-level (permission level 2; console always allowed)

| Command | Argument range | Description |
| --- | --- | --- |
| `/sta info <player>` | — | View stamina/state/modifiers (self-view needs no OP) |
| `/sta set <player> <value>` | 0 – 1,000,000 | Set directly |
| `/sta add <player> <value>` | -1,000,000 – 1,000,000 | Add/subtract |
| `/sta exhaust <player>` | — | Empty + exhaustion lock (penalty testing) |
| `/sta reset <player>` | — | Refill to max |
| `/sta modifier list [player]` | — | List active modifiers and remaining time |
| `/sta modifier clear <player>` | — | Clear modifiers |
| `/sta modifier give <player> <id> <sec> [drain×] [maxBonus] [regen/s]` | sec 1–86,400; drain× 0–100; max ±1,000,000; regen ±100,000 | Attach a test buff; quote ids containing colons |
| `/sta config show` | — | List all server values |
| `/sta config <key> <value>` | see below | Modify and persist |

`config` literals ↔ config keys:

| Literal | Config key | Type/range |
| --- | --- | --- |
| max | general.max_stamina | float 1 – 1,000,000 |
| sprint-drain | drain.sprint_per_second | float 0 – 100,000 |
| jump-cost | drain.jump_cost | float 0 – 100,000 |
| swim-drain | drain.swim_per_second | float 0 – 100,000 |
| swim-sprint-drain | drain.swim_sprint_per_second | float 0 – 100,000 |
| attack-cost | drain.attack_cost | float 0 – 100,000 |
| break-cost | drain.break_block_cost | float 0 – 100,000 |
| regen | recovery.per_second | float 0 – 100,000 |
| regen-exhausted | recovery.exhausted_per_second | float 0 – 100,000 |
| delay | recovery.delay_ticks | int 0 – 1,200 |
| delay-exhausted | recovery.exhausted_delay_ticks | int 0 – 1,200 |
| sprint-stop | penalty.sprint_stop_threshold | float 0 – 1,000,000 |
| release | penalty.exhausted_release_threshold | float 0 – 1,000,000 |
| walk-slowdown | penalty.exhausted_walk_slowdown | float 0 – 0.6 |
| block-jump | penalty.exhausted_block_jump | bool |
| block-sprint | penalty.exhausted_block_sprint | bool |
| winded-block-jump | penalty.winded_block_jump | bool |

```
/sta config sprint-drain 8                        # lower sprint drain to 8/s
/sta config walk-slowdown 0.15                    # exhausted speed ×0.85
/sta modifier give Steve "med:test" 60 0.5 30 5   # 60 s: drain ×0.5, max +30, regen +5/s
```

> Every `config` change goes through `ConfigValue.set` + `SPEC.save()` and is written to the server TOML immediately — effective at once and persistent across restarts. Manual file edits are hot-reloaded by the file watcher as well.

---

## 7. 开发者接口 · Developer API

**【中文】**

### 7.1 设计原则

- **静态门面，类始终存在**：`com.herra.stamina.api.StaminaAPI` 是无状态静态类，不需要 Holder 判空——本模组被禁用时方法自然无人调用；
- **单向集成**：本模组从不调用生态模组，其他模组**先判空再使用**，不要在 `mods.toml` 里硬依赖；
- **服务端权威**：写入类方法仅服务端有效（客户端调用抛 `IllegalStateException`）；读取类方法双端安全，客户端传本地玩家时经显示桥返回 HUD 插值显示值。

### 7.2 StaminaAPI 方法一览

| 方法 | 端 | 说明 |
| --- | --- | --- |
| `getStamina(player)` | 双端 | 当前体力（客户端本地玩家 = 插值显示值） |
| `getMaxStamina(player)` | 双端 | 最终上限（含修改器聚合） |
| `getRatio(player)` | 双端 | 体力比例 [0, 1]，HUD 类模组直取 |
| `isExhausted(player)` | 双端 | 是否透支锁定中 |
| `canSprint(player)` | 双端 | 是否允许疾跑（含低体力判定） |
| `tryConsume(player, amount, action)` | 服务端 | 原子扣除：不足则完全不扣并返回 false（触发 Consume 事件） |
| `drain(player, amount, action)` | 服务端 | 部分扣除：扣多少算多少，返回实际扣除量 |
| `addStamina(player, amount)` | 服务端 | 直接增加（钳制到上限） |
| `setStamina(player, amount)` | 服务端 | 直接设置（钳制 + 事件 + 立即同步） |
| `reset(player)` | 服务端 | 重置为满体力（战局开局调用） |
| `applyModifier(player, modifier)` | 服务端 | 挂载/刷新定时修改器（同 id 覆盖）并立即同步 |
| `clearModifier(player, id)` | 服务端 | 移除指定 id 修改器（解药等场景） |
| `clearModifiers(player)` | 服务端 | 清空全部（死亡/新战局/洗胃） |
| `getActiveModifiers(player)` | 双端 | 生效中修改器只读快照 |
| `getActiveModifierEntries(player)` | 双端 | 修改器 + 剩余 tick（-1 = 永久），UI/指令展示用 |
| `registerDrainModifier(fn)` / `unregisterDrainModifier(fn)` | 双端 | 注册/注销消耗规则修改器 |

### 7.3 消耗来源 `StaminaAction`

`SPRINT`（疾跑，持续）· `JUMP`（跳跃，一次性）· `SWIM`（游泳，持续）· `SWIM_SPRINT`（疾速游泳，持续）· `ATTACK`（近战命中，一次性）· `BREAK_BLOCK`（破坏方块，一次性）· `CUSTOM`（第三方模组自定义，一次性）

### 7.4 消耗规则修改器 `StaminaDrainModifier`

函数式接口 `(player, action, cost) -> cost`，在每次消耗结算前调用（含疾跑/游泳等持续消耗），按注册顺序链式应用，结果下限 0。适合按装备/状态改写消耗：

```java
// 重甲加大疾跑消耗，外骨骼减免跳跃消耗
// Heavy armor raises sprint cost; exoskeleton discounts jumps
StaminaAPI.registerDrainModifier((player, action, cost) -> {
    if (action == StaminaAction.SPRINT && isHeavilyLoaded(player)) return cost * 1.35f;
    if (action == StaminaAction.JUMP && hasExoskeleton(player)) return cost * 0.7f;
    return cost;
});
// 卸载时注销 | unregister on unload:
StaminaAPI.unregisterDrainModifier(fn);
```

### 7.5 定时修改器 `StaminaModifier`（医药/增益类推荐入口）

Builder 模式，一次调用同时影响**上限、全部消耗、恢复速度**；服务端自动倒计时、到期自动失效并重同步 HUD，无需自己计时。同 id 重复 apply = 刷新。

| Builder 方法 | 默认 | 说明 |
| --- | --- | --- |
| `maxStaminaBonus(float)` | 0 | 上限加算 |
| `maxStaminaMultiplier(float)` | 1.0 | 上限乘算 |
| `drainMultiplier(float)` | 1.0 | 全部消耗乘算（疾跑/跳跃/游泳/攻击/挖块/自定义全部生效） |
| `regenBonus(float)` | 0 | 恢复速度加算（每秒，正常与透支均生效） |
| `regenMultiplier(float)` | 1.0 | 恢复速度乘算 |
| `durationSeconds(int)` / `durationMinutes(int)` / `durationTicks(int)` | -1 | 持续时间（-1 = 永久，直到 clear 或退出） |

聚合公式（多修改器叠加）：`最终上限 = (基础 + Σ加算) × Π乘算`（下限 1）；消耗与恢复同理。修改器为运行时数据（不落盘，退出即清），药品自身管理药效周期即可。

```java
// 能量饮料：90 秒内消耗减半、上限 +30、恢复提速 50%
// Energy drink: 90 s of halved drain, +30 max, 50% faster regen
StaminaAPI.applyModifier(player, StaminaModifier.builder("med:energy_drink")
        .maxStaminaBonus(30.0f)       // 上限 +30，体力条变长 | +30 max, longer bar
        .drainMultiplier(0.5f)        // 全部消耗减半 | all costs halved
        .regenMultiplier(1.5f)        // 恢复提速 | faster regen
        .durationSeconds(90)
        .build());

// 重病 debuff：上限乘算压低 + 消耗加重 + 恢复变慢
// Fever debuff: lower max, heavier drain, slower regen
StaminaAPI.applyModifier(player, StaminaModifier.builder("med:fever")
        .maxStaminaMultiplier(0.7f)
        .drainMultiplier(1.4f)
        .regenMultiplier(0.6f)
        .durationMinutes(10)
        .build());

// 解药打断 | antidote interrupts:
StaminaAPI.clearModifier(player, "med:energy_drink");
```

### 7.6 事件（NeoForge 游戏总线，仅服务端）

| 事件类 | 时机 | 能力 |
| --- | --- | --- |
| `StaminaEvent.Consume` | 一次性消耗结算前（跳跃、API 自定义） | `setAmount` 改量、`setCanceled` 取消（取消 = 拒绝这次消耗） |
| `StaminaEvent.Changed` | 每次数值变化 | `getOldValue` / `getNewValue`（高频事件，监听保持轻量） |
| `StaminaEvent.Exhausted` | 归零进入透支锁瞬间（一次） | 叠加惩罚（枪械抖动、呼吸音效…） |
| `StaminaEvent.Recovered` | 透支解除（回气到阈值） | 撤销惩罚 |

> 持续消耗（疾跑/游泳）**不触发** Consume 事件——请用 `StaminaDrainModifier` 改写规则。取消标志为手动实现（非 `ICancellableEvent`），跨版本语义稳定。

```java
@SubscribeEvent
static void onConsume(StaminaEvent.Consume e) {
    // 负重联动：重载时跳跃消耗 ×1.5
    // Encumbrance hook: jump costs ×1.5 when heavily loaded
    if (e.getAction() == StaminaAction.JUMP && isHeavilyLoaded(e.getPlayer())) {
        e.setAmount(e.getAmount() * 1.5f);
    }
}
```

### 7.7 客户端视觉读取（HUD 类模组）

两种方式：

- **跨端安全**：`StaminaAPI.getRatio(localPlayer)` —— 内部经 `ClientDisplayBridge` 显示桥取插值值，公共代码无需判端、无需引用客户端类；
- **直接取**：`com.herra.stamina.client.ClientStaminaData` 静态类（`getRatio` / `isExhausted` / `isWinded` / `getGhostRatio` 等），全部为同步后的插值状态，仅在客户端调用。

### 7.8 编译依赖

```groovy
repositories { maven { url = "https://maven.firstdark.dev/snapshots" } }
dependencies {
    compileOnly "com.lowdragmc.ldlib2:ldlib2-neoforge-1.21.1:2.2.41:all"
    compileOnly files("libs/herra_stamina-1.0.3.jar")   // 或发布到内部 maven
}
```

**【English】**

### 7.1 Design Principles

- **Static facade, class always present** — `com.herra.stamina.api.StaminaAPI` is a stateless static class; no Holder null-checks needed. If the mod is disabled, its methods are simply never called.
- **One-way integration** — this mod never calls into ecosystem mods. Integrators should **null-check first** and avoid hard `mods.toml` dependencies.
- **Server-authoritative** — write methods are server-only (client calls throw `IllegalStateException`); read methods are side-safe and, when passed the local player on the client, return interpolated HUD display values via the display bridge.

### 7.2 StaminaAPI Method Reference

| Method | Side | Description |
| --- | --- | --- |
| `getStamina(player)` | both | Current stamina (local player on client = interpolated display value) |
| `getMaxStamina(player)` | both | Final max (with modifier aggregation) |
| `getRatio(player)` | both | Ratio [0, 1] — ready for HUD mods |
| `isExhausted(player)` | both | Inside the exhaustion lock? |
| `canSprint(player)` | both | Sprinting currently allowed (incl. low-stamina rule) |
| `tryConsume(player, amount, action)` | server | Atomic consume: refuses entirely and returns false when insufficient (fires Consume) |
| `drain(player, amount, action)` | server | Partial consume: drains what's available, returns the amount drained |
| `addStamina(player, amount)` | server | Add (clamped to max) |
| `setStamina(player, amount)` | server | Set (clamped + event + immediate sync) |
| `reset(player)` | server | Reset to full (call at raid start) |
| `applyModifier(player, modifier)` | server | Attach/refresh a timed modifier (same id overwrites) + immediate sync |
| `clearModifier(player, id)` | server | Remove one modifier by id (antidote scenarios) |
| `clearModifiers(player)` | server | Remove all (death / new raid / stomach pump) |
| `getActiveModifiers(player)` | both | Read-only snapshot of active modifiers |
| `getActiveModifierEntries(player)` | both | Modifiers + remaining ticks (-1 = permanent), for UI/commands |
| `registerDrainModifier(fn)` / `unregisterDrainModifier(fn)` | both | Register/unregister a drain-rule modifier |

### 7.3 `StaminaAction` Sources

`SPRINT` (continuous) · `JUMP` (one-shot) · `SWIM` (continuous) · `SWIM_SPRINT` (continuous) · `ATTACK` (one-shot, on hit) · `BREAK_BLOCK` (one-shot) · `CUSTOM` (third-party one-shot)

### 7.4 Drain-Rule Modifier — `StaminaDrainModifier`

A functional interface `(player, action, cost) -> cost` invoked before **every** drain settlement (including continuous sprint/swim drains), chained in registration order, floored at 0. Ideal for gear/state-driven cost rewrites:

```java
// Heavy armor raises sprint cost; exoskeleton discounts jumps
StaminaAPI.registerDrainModifier((player, action, cost) -> {
    if (action == StaminaAction.SPRINT && isHeavilyLoaded(player)) return cost * 1.35f;
    if (action == StaminaAction.JUMP && hasExoskeleton(player)) return cost * 0.7f;
    return cost;
});
// unregister on unload:
StaminaAPI.unregisterDrainModifier(fn);
```

### 7.5 Timed Modifier — `StaminaModifier` (recommended for medical/buff mods)

Builder pattern; a single call affects **max stamina, all drains and regen** at once. The server auto-counts-down, auto-expires and re-syncs the HUD — no timers needed on your side. Re-applying the same id refreshes it.

| Builder method | Default | Description |
| --- | --- | --- |
| `maxStaminaBonus(float)` | 0 | Additive max stamina |
| `maxStaminaMultiplier(float)` | 1.0 | Multiplicative max stamina |
| `drainMultiplier(float)` | 1.0 | Multiplier on **all** drains (sprint/jump/swim/attack/break/custom) |
| `regenBonus(float)` | 0 | Additive regen per second (both normal and exhausted) |
| `regenMultiplier(float)` | 1.0 | Multiplicative regen |
| `durationSeconds(int)` / `durationMinutes(int)` / `durationTicks(int)` | -1 | Duration (-1 = permanent until cleared or logout) |

Aggregation across stacked modifiers: `final max = (base + Σbonus) × Πmultiplier` (floored at 1); drains and regen follow the same multiplicative pattern. Modifiers are runtime data (not persisted, cleared on logout) — your item manages its own effect lifecycle.

```java
// Energy drink: 90 s of halved drain, +30 max, 50% faster regen
StaminaAPI.applyModifier(player, StaminaModifier.builder("med:energy_drink")
        .maxStaminaBonus(30.0f)
        .drainMultiplier(0.5f)
        .regenMultiplier(1.5f)
        .durationSeconds(90)
        .build());

// Fever debuff: lower max, heavier drain, slower regen
StaminaAPI.applyModifier(player, StaminaModifier.builder("med:fever")
        .maxStaminaMultiplier(0.7f)
        .drainMultiplier(1.4f)
        .regenMultiplier(0.6f)
        .durationMinutes(10)
        .build());

// Antidote interrupts:
StaminaAPI.clearModifier(player, "med:energy_drink");
```

### 7.6 Events (NeoForge game bus, server-side only)

| Event class | Fires | Capabilities |
| --- | --- | --- |
| `StaminaEvent.Consume` | Before one-shot drains settle (jump, API custom) | `setAmount` to rewrite, `setCanceled` to refuse the drain |
| `StaminaEvent.Changed` | On every value change | `getOldValue` / `getNewValue` (high-frequency — keep listeners light) |
| `StaminaEvent.Exhausted` | The instant the exhaustion lock engages (once) | Stack extra penalties (weapon sway, breathing SFX…) |
| `StaminaEvent.Recovered` | Lock released (recovered to threshold) | Undo those penalties |

> Continuous drains (sprint/swim) do **not** fire Consume — use `StaminaDrainModifier` for those. The cancel flag is hand-rolled (not `ICancellableEvent`) for stable cross-version semantics.

```java
@SubscribeEvent
static void onConsume(StaminaEvent.Consume e) {
    // Encumbrance hook: jump costs ×1.5 when heavily loaded
    if (e.getAction() == StaminaAction.JUMP && isHeavilyLoaded(e.getPlayer())) {
        e.setAmount(e.getAmount() * 1.5f);
    }
}
```

### 7.7 Client-Side Visual Reads (HUD Mods)

Two options:

- **Side-safe**: `StaminaAPI.getRatio(localPlayer)` — routed through the `ClientDisplayBridge` display bridge; common code needs no side checks and no client class references;
- **Direct**: the static `com.herra.stamina.client.ClientStaminaData` (`getRatio` / `isExhausted` / `isWinded` / `getGhostRatio`, …), all interpolated post-sync states — client-side calls only.

### 7.8 Compile Dependencies

```groovy
repositories { maven { url = "https://maven.firstdark.dev/snapshots" } }
dependencies {
    compileOnly "com.lowdragmc.ldlib2:ldlib2-neoforge-1.21.1:2.2.41:all"
    compileOnly files("libs/herra_stamina-1.0.3.jar")   // or publish to an internal maven
}
```

---

## 8. 架构与技术注记 · Architecture & Technical Notes

**【中文】**

本节面向模组开发者和深度整合包作者，记录实现选型与踩坑结论（均经反编译原版/NeoForge 源码核实）。

### 8.1 服务端权威模型

体力数值与全部规则在服务端每 tick 的 `PlayerTickEvent` 中结算，存储于 NeoForge **Attachment**（运行时数据，不落盘——新战局永远满体力开局；另提供 NBT 快照方法供未来的战局系统持久化）。登录、重生、跨维度事件均触发强制重同步。客户端只持有"显示副本"并做插值，不参与任何规则判定。

### 8.2 网络协议

- 通道版本 `"1"`，全部为 **S2C**（服务端→客户端），无任何 C2S 包：
  - `herra_stamina:stamina_sync` —— 4 字段：`stamina`（float）、`maxStamina`（float）、`exhaustedLock`（bool）、`sprintBlocked`（bool）；
  - `herra_stamina:open_settings` —— 指令触发客户端打开设置界面。
- **同步节流**：数值变化 ≥ `sync_delta`（默认 2.0）或状态位翻转，且距上次发包 ≥ `sync_interval_ticks`（默认 2 tick）才发送；透支进入/解除瞬间**零延迟直发**（消除 1~2 tick 的客户端疾跑窗口）。
- **客户端插值**：HUD 数值逐帧向目标值收敛，20 tick 的节流间隔下视觉无阶梯感。

### 8.3 禁跑实现（客户端 Mixin）

透支禁跑不能靠"事后杀疾跑"：客户端 `LocalPlayer.aiStep` 每 tick 会重启疾跑（`canStartSprinting` 不含体力检查），tick 末尾 `setSprinting(false)` 等于没杀；且 `keySprint` 是 `ToggleKeyMapping`，切换式疾跑下 `setDown(false)` 无效。本模组 Mixin 注入原版饥饿禁跑的**同一判定点** `hasEnoughFoodToStartSprinting()`（HEAD），`sprintBlocked` 时返回 false——同时门控疾跑的启动 flag 与维持 flag，按住/切换/双击 W 三种方式全覆盖，FOV 平滑回落无抖动。服务端保留 `setSprinting(false)` 兜底。

### 8.4 禁跳实现（同步属性修饰符）

`LivingJumpEvent` 在 NeoForge 21.1 不可取消。本模组改用 `JUMP_STRENGTH` 属性（`LivingEntity` 基础属性、范围 0~32、`setSyncable(true)`）：透支期间挂 `ADD_VALUE -1024` 修饰符，属性钳到 0 并同步客户端，两端 `jumpFromGround()` 的 `f <= 1.0E-5` 守卫直接跳过——无橡皮筋、无客户端预测回弹。修饰符每幂等 tick 同步一次，创造/旁观分支外处理防止泄漏。

### 8.5 服务端移动检测

服务端玩家移动走 `handleMovePlayer -> Entity.move()`，只写坐标不写 `deltaMovement`（仅撞墙/击退时清分量），输入移动时 `getDeltaMovement()` 恒为 0——v1.0.0 曾因此出现"疾跑不消耗"的 bug。现改用 `StaminaData.updateMovement()` 记录每 tick **位置差**。游泳垂直位移单独记录，阈值 0.05 格/tick（排除水中被动下沉 0.02~0.03 格/tick）；滑翔 `isFallFlying` 与骑乘 `isPassenger` 直接豁免。

### 8.6 跳跃消耗策略

跳跃消耗挂在 `LivingEvent.LivingJumpEvent`（在 `jumpFromGround()` 内触发，含跳跃药水/马匹等变体，精确无误判）。由于不可取消，策略为**允许跳跃、一次性扣费**；体力不足则扣到空并进入透支锁（战术上保留"赌命跳"）。

### 8.7 HUD 渲染与 dist 安全

- HUD 宿主为 LDLib2 `ModularHudLayer`（官方 HUD Overlays 方案），自绘 `UIElement.drawContents` 像素绘制；着色全部走顶点色/SpriteTexture，规避 `RenderSystem.setShaderColor` 批量渲染不可靠问题；
- 专用服务器上 `ClientPacketHandlers` 方法体若直接引用 `Screen`/`Minecraft` 会在 `RegisterPayloadHandlersEvent` 时触发 `BootstrapMethodError`——已抽取 `ClientGuiHandlers` 中转（间接引用），与 `ClientStaminaData` 同构。

### 8.8 配置热重载

服务端/客户端 TOML 均被 NeoForge 文件监听器监视，存盘即热重载；指令修改走 `ConfigValue.set` + `ModConfigSpec.save()` 直接落盘。数值全部实时读取（无快照缓存），改动下个 tick 生效。

**【English】**

This section documents implementation choices and hard-won lessons (all verified against decompiled vanilla/NeoForge sources) — aimed at mod developers and deep integrators.

### 8.1 Server-Authoritative Model

Stamina values and every rule settle in the server's `PlayerTickEvent`; state lives in a NeoForge **Attachment** (runtime-only, not persisted — every session starts full; NBT snapshot methods are provided for a future persistence layer). Login, respawn and dimension-change events force a resync. The client only keeps a "display copy" with interpolation and judges nothing.

### 8.2 Network Protocol

- Channel version `"1"`; **S2C only**, zero client-to-server payloads:
  - `herra_stamina:stamina_sync` — 4 fields: `stamina` (float), `maxStamina` (float), `exhaustedLock` (bool), `sprintBlocked` (bool);
  - `herra_stamina:open_settings` — command-triggered client screen.
- **Throttling**: a packet is sent only when the value moved ≥ `sync_delta` (default 2.0) or a state flag flipped, and at least `sync_interval_ticks` (default 2) since the last one; lock enter/leave transitions are sent with **zero delay** (closing the 1–2 tick client sprint window).
- **Interpolation**: the HUD value converges frame-by-frame, so 20-tick throttling shows no staircase.

### 8.3 Sprint Blocking (Client Mixin)

Killing sprint after the fact does not work: `LocalPlayer.aiStep` restarts sprinting every tick (`canStartSprinting` knows nothing about stamina), so an end-of-tick `setSprinting(false)` is ineffective; and `keySprint` is a `ToggleKeyMapping`, where `setDown(false)` does nothing for toggle-sprint users. The mod instead mixins into the exact vanilla hunger-check point `hasEnoughFoodToStartSprinting()` (HEAD) and returns false when `sprintBlocked` — gating both the start and maintain flags, covering hold/toggle/double-tap with smooth FOV. The server keeps a `setSprinting(false)` backstop.

### 8.4 Jump Blocking (Synced Attribute Modifier)

`LivingJumpEvent` is not cancelable on NeoForge 21.1. The mod uses the `JUMP_STRENGTH` attribute instead (a base `LivingEntity` attribute, range 0–32, `setSyncable(true)`): while exhausted, an `ADD_VALUE -1024` modifier clamps it to 0 and syncs, so the `f <= 1.0E-5` guard inside `jumpFromGround()` skips on both ends — no rubber-banding, no prediction snap-back. The modifier is re-asserted idempotently each tick and handled outside creative/spectator branches to avoid leakage.

### 8.5 Server-Side Movement Detection

Server-side player movement flows through `handleMovePlayer -> Entity.move()`, which writes coordinates but never `deltaMovement` (only collisions/knockback touch it) — input-driven movement therefore reads `getDeltaMovement()` as 0, which caused v1.0.0's "sprint drains nothing" bug. Detection now uses per-tick **position deltas** (`StaminaData.updateMovement()`). Swim vertical displacement is tracked separately with a 0.05 blocks/tick threshold (excluding passive sinking of 0.02–0.03); elytra (`isFallFlying`) and riding (`isPassenger`) are exempt.

### 8.6 Jump Cost Strategy

The jump cost hooks `LivingEvent.LivingJumpEvent` (fires inside `jumpFromGround()`, covering jump-boost/horse variants with zero false positives). Being non-cancelable, the strategy is **allow the jump, charge one-shot**; insufficient stamina drains to zero and engages the lock (the deliberate "gamble jump").

### 8.7 HUD Rendering & Dist Safety

- The HUD is hosted by LDLib2's `ModularHudLayer` (the official HUD Overlays approach) with a custom `UIElement.drawContents` pixel renderer; tinting goes through vertex colors/SpriteTexture to avoid `RenderSystem.setShaderColor` unreliability under batching.
- On dedicated servers, `ClientPacketHandlers` bodies referencing `Screen`/`Minecraft` directly would throw `BootstrapMethodError` at `RegisterPayloadHandlersEvent` — solved by the `ClientGuiHandlers` indirection (same pattern as `ClientStaminaData`).

### 8.8 Config Hot Reload

Both TOML files are watched by NeoForge's file listener and hot-reload on save; command edits go through `ConfigValue.set` + `ModConfigSpec.save()`. Values are read live (no snapshot caching), so changes apply the next tick.

---

## 9. 整合包集成指南 · Modpack Integration Guide

**【中文】**

- **服务端数值预置**：把调好的 `herra_stamina-server.toml` 放进服务器根目录 `defaultconfigs/`，新建世界时自动套用；
- **客户端表现**：可直接随包分发 `config/herra_stamina-client.toml`（全局默认样式/位置），或保留默认让玩家用 `/sta` 自行调整——推荐后者，零维护成本；
- **脚本化增益**：地图/指令方块可用 `/sta modifier give <玩家> <id> <秒> [消耗×] [上限] [恢复/秒]` 发放剧情增益（如撤离前注射肾上腺素），无需写代码；
- **数值风格建议**：

| 风格 | 关键调整 |
| --- | --- |
| 拟实搜打撤（默认） | 保持默认即可（8.3 秒疾跑 / 3 秒透支延迟 / 回气到 30%） |
| 硬核军事 | `sprint-drain 15`、`regen 18`、`release 50`、`walk-slowdown 0.25`、`winded-block-jump true` |
| 轻量生存 | `sprint-drain 8`、`jump-cost 6`、`delay 12`（1 秒内回气）、`release 20` |
| 长途探索 | `max 150`、`regen 35`、`attack-cost 0`、`break-cost 0`（关闭战斗/挖掘消耗） |

- 与其他移动类模组无已知冲突：Mixin 只注入单一判定点，疾跑/跳跃以外的移动系统不受影响。

**【English】**

- **Server value presets** — drop your tuned `herra_stamina-server.toml` into the server root's `defaultconfigs/`; new worlds pick it up automatically.
- **Client presentation** — either ship `config/herra_stamina-client.toml` with the pack (global default style/position) or leave defaults and let players personalize via `/sta` — the latter is zero-maintenance and recommended.
- **Scripted buffs** — maps/command blocks can use `/sta modifier give <player> <id> <sec> [drain×] [max] [regen/s]` for story buffs (e.g. a pre-extraction adrenaline shot) with zero code.
- **Tuning profiles**:

| Flavor | Key changes |
| --- | --- |
| Realistic extraction (default) | keep defaults (8.3 s sprint / 3 s exhausted delay / release at 30) |
| Hardcore military | `sprint-drain 15`, `regen 18`, `release 50`, `walk-slowdown 0.25`, `winded-block-jump true` |
| Light survival | `sprint-drain 8`, `jump-cost 6`, `delay 12` (sub-second regen delay), `release 20` |
| Long-distance exploration | `max 150`, `regen 35`, `attack-cost 0`, `break-cost 0` (combat/mining drains off) |

- No known conflicts with movement mods: the mixin injects a single check point and never touches movement systems beyond sprint/jump gating.

---

## 10. 兼容性 · Compatibility

**【中文】**

- **连接矩阵**：见 §2；要点——模组客户端可进原版服（休眠无报错），原版客户端不能进模组服（通道协商即断开）；
- **displayTest**：`IGNORE_SERVER_VERSION`，版本不一致不拒绝连接；
- **数据存续**：体力与修改器均为运行时数据，不写入玩家 NBT/存档——卸载模组无残留，重装无损；
- **游戏模式**：创造/旁观自动豁免（HUD 隐藏），生存/冒险全量生效；
- **本地化**：内置 zh_cn / en_us 双语（HUD 状态文字、指令反馈、设置界面、音效字幕）；
- **服务端环境**：纯专用服务器已验证加载正常（client-only 代码路径全部隔离）；开发环境 RCON 端到端冒烟 29/29 通过（v1.0.2）。

**【English】**

- **Connection matrix**: see §2 — a modded client joins vanilla servers (dormant, no errors); a vanilla client cannot join a modded server (dropped at channel negotiation).
- **displayTest**: `IGNORE_SERVER_VERSION` — version mismatch never rejects a connection.
- **Data persistence**: stamina and modifiers are runtime-only, never written to player NBT/world saves — uninstalling leaves no residue, reinstalling loses nothing.
- **Game modes**: creative/spectator auto-exempt (HUD hidden); survival/adventure fully active.
- **Localization**: bundled zh_cn / en_us (HUD status text, command feedback, settings screen, sound subtitles).
- **Server environments**: verified loading cleanly on a dedicated server (all client-only code paths isolated); dev-environment RCON end-to-end smoke tests 29/29 passing (v1.0.2).

---

## 11. 版本历史 · Changelog

**【中文】**

| 版本 | 内容 |
| --- | --- |
| **1.0.3** | 对外名称去品牌化：displayName 改为「Stamina」，语言文件与文档中的旧品牌字样移除；内部 modId 保持 `herra_stamina`，配置文件与既有存档完全兼容 |
| **1.0.2** | `/sta` 游戏内设置界面（拖拽小地图摆位 / X/Y/缩放滑条 / 居中复位，即时生效 + 自动保存）；4 种样式预设（经典/战术/极简/生电）+ 动态预览；耗尽心跳提示音（合成音效 + 音量/试听）；透支惩罚收紧（锁切换零延迟同步；新增 `exhausted_block_sprint`、`exhausted_walk_slowdown`、`winded_block_jump`）；指令权限收紧（数值管理统一 OP 2）；HUD 默认上移 offset_y 84；修复创造模式消耗一致性、运行时改 walk-slowdown 修饰符热更新、config 显示精度、专用服务器 dist 崩溃 |
| **1.0.1** | 修复透支后仍可疾跑（客户端 Mixin 注入 `hasEnoughFoodToStartSprinting`，hold/toggle/双击 W 全覆盖）；新增透支禁跳（JUMP_STRENGTH 同步属性修饰符，无橡皮筋）；新增近战攻击 2/次与破坏方块 1/块消耗；修复游泳垂直位移消耗与原地踩水误扣；鞘翅/骑乘不再误判为疾跑；新增 `attack-cost`/`break-cost`/`block-jump` 指令 |
| **1.0.0** | 首发：疾跑/跳跃/游泳/疾速游泳消耗、延迟恢复、透支锁定、低体力禁跑；LDLib2 像素 HUD（幽灵拖尾/火花/扫光/自动隐藏）；双端配置（中文注释、热重载）；`/stamina` 指令全套；生态 API（StaminaAPI / StaminaModifier / StaminaDrainModifier / 4 事件 / 客户端显示桥）；GitHub Actions CI/CD；修复服务端疾跑不消耗（deltaMovement 恒 0 → 位置差检测） |

**【English】**

| Version | Changes |
| --- | --- |
| **1.0.3** | External debranding: displayName became "Stamina"; legacy brand wording removed from lang files and docs. Internal modId stays `herra_stamina` — configs and existing worlds fully compatible |
| **1.0.2** | In-game `/sta` settings screen (drag-to-position mini-map, X/Y/scale sliders, recenter; live-apply + auto-save); 4 style presets (Classic/Tactical/Minimal/Energy) with live preview; exhaustion heartbeat sound (synthesized, volume + test-listen); stricter exhaustion (zero-delay lock sync; new `exhausted_block_sprint`, `exhausted_walk_slowdown`, `winded_block_jump`); command permissions tightened (all value management at OP 2); HUD default raised to offset_y 84; fixes for creative-mode drain consistency, runtime walk-slowdown modifier re-apply, config display precision, dedicated-server dist crash |
| **1.0.1** | Fixed post-exhaustion sprint escape (client mixin into `hasEnoughFoodToStartSprinting`, covering hold/toggle/double-tap); added exhaustion jump block (synced JUMP_STRENGTH modifier, no rubber-banding); added melee-hit (2) and block-break (1) costs; fixed swim vertical-displacement drain and idle-treading misdetection; elytra/riding no longer miscounted; new `attack-cost`/`break-cost`/`block-jump` commands |
| **1.0.0** | Initial release: sprint/jump/swim/sprint-swim drains, delayed regen, exhaustion lock, low-stamina sprint block; LDLib2 pixel HUD (ghost trail / sparks / sweep / auto-hide); dual configs (Chinese comments, hot reload); full `/stamina` command suite; ecosystem API (StaminaAPI / StaminaModifier / StaminaDrainModifier / 4 events / client display bridge); GitHub Actions CI/CD; fixed server sprint not draining (deltaMovement always 0 → position-delta detection) |
