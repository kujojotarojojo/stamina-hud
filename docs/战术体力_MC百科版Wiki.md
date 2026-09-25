# 战术体力

**战术体力**（Stamina）是一款面向搜打撤（撤离射击）玩法的独立体力系统模组，运行于 NeoForge 平台。疾跑、跳跃、游泳、近战攻击、破坏方块均会消耗体力，体力耗尽后进入透支状态，禁止疾跑与跳跃，需要回气到阈值才能恢复行动能力，还原撤离射击游戏中"跑不动、喘不上气"的压迫感。默认数值按三角洲行动、暗区突围的手感调校，全部可在配置中修改。

模组由 kujojotarojojo 开发，适用于 Minecraft 1.21.1，需要前置 LDLib2（HUD 渲染层）。模组只负责体力系统本身，不含战局、播报、枪械等内容，可作为通用体力模组单独用于生存整合包；体力数值由服务端权威计算，客户端只负责 HUD 显示。

## 模组信息

| 项目 | 内容 |
| --- | --- |
| 中文名 | 战术体力 |
| 英文名 | Stamina |
| 模组 ID | herra_stamina |
| 作者 | kujojotarojojo |
| 运行环境 | 客户端、服务端双端安装 |
| Minecraft 版本 | 1.21.1 |
| 加载器 | NeoForge 21.1.216 及以上 |
| 前置模组 | LDLib2 2.2.1 及以上 |
| 许可证 | MIT |
| 源码 | [GitHub - kujojotarojojo/stamina-hud](https://github.com/kujojotarojojo/stamina-hud) |

## 体力机制

以下均为默认数值，服务端可整体调整（见"配置文件"一节）。

### 消耗动作

| 动作 | 消耗 | 说明 |
| --- | --- | --- |
| 疾跑 | 12/秒 | 满体力约可持续疾跑 8.3 秒 |
| 跳跃 | 10/次 | 疾跑中跳跃 = 疾跑消耗 + 跳跃消耗叠加 |
| 游泳 | 5/秒 | 原地踩水不消耗；主动上浮/下潜计入消耗 |
| 疾速游泳 | 9/秒 | 水中疾跑状态 |
| 近战攻击 | 2/次 | 命中实体时结算，设为 0 关闭 |
| 破坏方块 | 1/块 | 每破坏一个方块结算，设为 0 关闭 |

### 恢复

停止消耗 1.2 秒后，体力以 25/秒的速度开始恢复，约 4 秒回满。

### 状态一览

| 状态 | 触发条件 | 效果 | 解除条件 |
| --- | --- | --- | --- |
| 正常 | 体力高于 15 | 无限制 | — |
| 呼吸急促 | 体力降至 15 及以下（未透支） | 立即禁止疾跑 | 体力恢复到 15 以上 |
| 体力透支 | 体力归零 | 禁止疾跑与跳跃，只能正常行走；恢复延迟延长至 3 秒，恢复速度降为 15/秒 | 恢复到 30 才解除，恢复疾跑能力 |

透支机制的"必须缓过来才能再跑"是本模组的核心手感：透支瞬间客户端会零延迟收到禁跑禁跳状态，按住疾跑键会立即停下且无画面抖动；透支期间还可配置行走减速（默认关闭，最高 60%）。

### 细节规则

- **赌命跳**：透支前的最后一次跳跃仍然允许——跳跃照常执行，体力直接扣空并进入透支，代价是接下来喘不上气。
- **低体力禁跳**：默认保留（低体力但未透支时仍可跳跃，留一跳逃生空间），可在服务端配置中开启。
- 创造、旁观模式不消耗体力，且创造模式下 HUD 自动隐藏（可配置为创造也消耗）。
- 死亡重生后以满体力复活；体力为运行时数据，重进服务器不保留上次的损耗。
- 骑乘、鞘翅滑翔不计入疾跑消耗，不会误扣。
- 水中游泳状态下按跳跃键上浮不触发跳跃消耗，只有从水底蹬地起跳才结算一次跳跃。

## HUD

体力条默认位于物品栏上方正中（护甲条之上、氧气条上方），像素风贴图与原版 GUI 质感统一，不遮挡血量、护甲、氧气与经验条。支持 0.5~2.0 倍整体缩放，位置可拖拽调整。

### 动画表现

- 数值平滑插值；大额扣减时显示格斗游戏式的"幽灵拖尾"——被扣除的部分以白→黄→红逐列渐变消失，伴随白闪与像素火花。
- 低体力时体力条红色呼吸脉动，归零时硬闪烁。
- 回满瞬间扫光，2 秒后自动隐藏，再次消耗时快速淡入。
- 体力条左侧有闪电图标，下方预留头、胸、臂、腿身体部位占位图标（供后续身体系统扩展），低体力时替换为「呼吸急促」「体力透支」状态文字。

### 样式预设

| 样式 | 边框风格 | 配色（充足→临界） |
| --- | --- | --- |
| 经典 classic | 原版金蓝像素边框 | 米白 → 金黄 → 橙 → 红（颜色可自定义） |
| 战术 tactical | 装甲块边框 + 硬分段刻度 | 军绿荧光 → 卡其 → 橙 → 警示红 |
| 极简 minimal | 1 像素细线边框 | 冷白 → 银灰 → 琥珀 → 红 |
| 生电 energy | 青色辉光切角 + 斜纹高光 | 青 → 天蓝 → 紫 → 品红 |

### 耗尽提示音

体力归零瞬间播放合成的低频心跳闷响（0.92 秒双响），提示透支开始；可在设置界面中开关、调节音量（0~100）并试听。

### /sta 设置界面

输入 `/sta` 打开搜打撤风格的暗色战术设置面板（全部调整即时生效、关闭时自动保存，且只对自己生效）：

- 拖拽小地图直接摆放体力条位置（带物品栏参考框、中心参考线与悬停十字线），或用水平 X、高度 Y、缩放滑条微调，一键居中复位；
- 四种样式预设点选切换，带动态大预览；
- 耗尽提示音开关、音量滑条与试听按钮；
- 闪电图标、状态文字、身体图标、自动隐藏四个开关，以及动画速度滑条。

## 指令

### 玩家指令

| 指令 | 说明 |
| --- | --- |
| `/sta` | 打开设置界面 |
| `/sta ui` | 同上（别名） |
| `/sta help` | 显示指令帮助 |
| `/stamina` | 查看自己的体力值与状态 |

### 管理指令（需要 OP 权限等级 2）

| 指令 | 说明 |
| --- | --- |
| `/sta info <玩家>` | 查看指定玩家的体力（查看自己无需 OP） |
| `/sta set <玩家> <数值>` | 设置体力 |
| `/sta add <玩家> <数值>` | 增减体力（可为负数） |
| `/sta exhaust <玩家>` | 清空体力并进入透支锁定（测试惩罚用） |
| `/sta reset <玩家>` | 体力回满 |
| `/sta modifier list [玩家]` | 查看生效中的体力修改器 |
| `/sta modifier clear <玩家>` | 清空玩家的修改器 |
| `/sta modifier give <玩家> <id> <秒数> [消耗倍率] [上限加算] [恢复/秒]` | 挂载测试增益（id 含冒号需加引号） |
| `/sta config show` | 列出全部服务器数值 |
| `/sta config <参数> <值>` | 修改数值并立即写入配置文件 |

`config` 可用参数：`max`（最大体力）、`sprint-drain`、`jump-cost`、`swim-drain`、`swim-sprint-drain`、`attack-cost`、`break-cost`、`regen`、`regen-exhausted`、`sprint-stop`、`release`、`walk-slowdown`（以上为数值）；`block-jump`、`block-sprint`、`winded-block-jump`（开关）；`delay`、`delay-exhausted`（恢复延迟，单位 tick）。

```
/sta config sprint-drain 8          # 疾跑消耗降为 8/秒，立即生效并写盘
/sta config walk-slowdown 0.15      # 透支期间移速 ×0.85
/sta modifier give Steve "herra_med:test" 60 0.5 30 5
                                     # 给 Steve 挂 60 秒增益：消耗减半、上限 +30、恢复 +5/秒
```

所有数值修改立即写入 `herra_stamina-server.toml`，重启服务器后仍生效。

## 配置文件

### 服务端配置 herra_stamina-server.toml

存放于世界目录的 `serverconfig/` 下（单机存档为 `saves/<世界名>/serverconfig/`），对整个服务器统一生效。文件被监听，手动编辑保存即热重载，无需重启。

| 配置项 | 默认值 | 说明 |
| --- | --- | --- |
| general.max_stamina | 100 | 最大体力值（1 ~ 1000000） |
| general.apply_in_creative | false | 创造模式是否消耗体力（默认关闭，创造下 HUD 自动隐藏） |
| drain.sprint_per_second | 12 | 疾跑每秒消耗 |
| drain.jump_cost | 10 | 每次跳跃一次性消耗 |
| drain.swim_per_second | 5 | 游泳每秒消耗 |
| drain.swim_sprint_per_second | 9 | 疾速游泳每秒消耗 |
| drain.attack_cost | 2 | 近战攻击命中的一次性消耗（0 = 关闭） |
| drain.break_block_cost | 1 | 每破坏一个方块的一次性消耗（0 = 关闭） |
| recovery.delay_ticks | 24 | 停止消耗后多少 tick 开始恢复（20 tick = 1 秒） |
| recovery.per_second | 25 | 正常恢复速度（每秒） |
| recovery.exhausted_delay_ticks | 60 | 透支后的额外恢复延迟（3 秒） |
| recovery.exhausted_per_second | 15 | 透支状态下的恢复速度（每秒） |
| penalty.sprint_stop_threshold | 15 | 体力低于该值时禁止疾跑 |
| penalty.exhausted_release_threshold | 30 | 透支后需恢复到该值才解除禁跑 |
| penalty.exhausted_block_jump | true | 透支期间是否禁止跳跃 |
| penalty.exhausted_block_sprint | true | 透支期间是否禁止疾跑（严格模式核心开关） |
| penalty.exhausted_walk_slowdown | 0.0 | 透支行走减速比例（0 = 正常行走，0.15 = 移速 ×0.85，上限 0.6） |
| penalty.winded_block_jump | false | 低体力（未透支）是否也禁止跳跃 |
| network.sync_interval_ticks | 2 | 数值同步最小间隔（一般无需修改） |
| network.sync_delta | 2.0 | 触发同步的最小数值变化量 |

### 客户端配置 herra_stamina-client.toml

存放于客户端 `config/` 目录，**每位玩家各自独立生效**。可在游戏内 `/sta` 界面可视化调整，也可手动编辑，保存即时生效。

| 配置项 | 默认值 | 说明 |
| --- | --- | --- |
| hud.enabled | true | 是否显示体力条 |
| hud.offset_x | 0 | 水平偏移（0 = 屏幕正中，对齐物品栏中心） |
| hud.offset_y | 84 | 体力条底边距屏幕底部的像素数（越大越靠上，避开血条与护甲条） |
| hud.scale | 1.0 | 整体缩放（0.5 ~ 2.0） |
| hud.style | classic | 样式预设：classic / tactical / minimal / energy |
| hud.auto_hide | true | 体力回满后自动隐藏 |
| hud.hide_delay_ticks | 40 | 回满后延迟多少 tick 再隐藏（默认 2 秒） |
| hud.show_icon | true | 体力条左侧显示闪电图标 |
| hud.show_body_status | true | 显示身体状态占位图标 |
| hud.show_status_text | true | 低体力/透支时显示状态文字 |
| hud.animation_speed | 1.0 | 动画速度倍率（0.1 ~ 10.0） |
| sound.exhausted_enabled | true | 体力耗尽瞬间播放提示音 |
| sound.exhausted_volume | 60 | 提示音音量（0 ~ 100） |
| colors.full | #EDF3E2 | 经典样式充足色（米白） |
| colors.mid | #F2C94C | 经典样式中等色（金黄） |
| colors.low | #E8873B | 经典样式低体力色（橙） |
| colors.crit | #D64541 | 经典样式临界色（红） |

> 四段配色仅作用于「经典」样式，其余样式预设自带配色。

## 安装与兼容性

### 安装

1. 安装 NeoForge 21.1.216 及以上版本（1.21.1）。
2. 将本模组与前置 LDLib2 放入 `mods` 文件夹。
3. **联机服务器**：客户端与服务端都需要安装本模组与 LDLib2；服务端不装则体力系统整体不存在，未装模组的客户端无法连接装有本模组的服务器。
4. **单人游戏**：仅需客户端安装一份。

### 兼容性说明

- 装有本模组的客户端可以正常进入原版服务器（体力系统不生效，不会报错）；反之，原版客户端无法进入装有本模组的服务器。
- 体力数值全部由服务端权威计算与校验，客户端仅渲染 HUD，修改客户端配置无法获得任何数值优势。
- 模组自带中文（zh_cn）与英文（en_us）双语支持。

## 二次开发

模组提供 `com.herra.stamina.api` 包供其他模组联动（服务端权威，静态门面 `StaminaAPI`），典型场景为医药类模组的兴奋剂、能量饮料，或负重系统加大疾跑消耗：

- **读取**：`getStamina` / `getMaxStamina` / `getRatio` / `isExhausted` / `canSprint`
- **写入**：`tryConsume`（原子扣除，不足则不扣）、`drain`（部分扣除）、`addStamina` / `setStamina` / `reset`
- **定时修改器 `StaminaModifier`**：一次调用同时影响体力上限、全部消耗与恢复速度，服务端自动倒计时、到期自动失效；聚合公式为 `最终上限 = (基础 + Σ加算) × Π乘算`（消耗、恢复同理）
- **消耗规则修改器 `StaminaDrainModifier`**：按动作类型改写消耗（如重甲疾跑 ×1.35）
- **事件**：`StaminaEvent.Consume`（一次性消耗前，可改量可取消）、`Changed`（数值变化）、`Exhausted`（进入透支）、`Recovered`（透支解除）

```java
// 能量饮料：90 秒内消耗减半、上限 +30、恢复提速 50%
StaminaAPI.applyModifier(player, StaminaModifier.builder("med:energy_drink")
        .maxStaminaBonus(30.0f)
        .drainMultiplier(0.5f)
        .regenMultiplier(1.5f)
        .durationSeconds(90)
        .build());
```

## 版本记录

| 版本 | 更新内容 |
| --- | --- |
| 1.0.3 | 对外名称去品牌化，模组对外统一称为「Stamina / 战术体力」；内部模组 ID 保持 herra_stamina 不变，配置文件与存档完全兼容 |
| 1.0.2 | 新增 `/sta` 游戏内设置界面（拖拽定位、缩放滑条）；4 种样式预设；体力耗尽心跳提示音；透支惩罚收紧（零延迟禁跑同步、行走减速、低体力禁跳三个新配置）；指令权限收紧（数值修改统一需要 OP 2）；HUD 默认位置上移至 84 像素 |
| 1.0.1 | 修复透支后仍可疾跑（客户端 Mixin 注入原版禁跑判定点）；新增透支禁跳（跳跃属性修饰符）；新增近战攻击与破坏方块消耗；修复游泳垂直位移不消耗、鞘翅与骑乘误判 |
| 1.0.0 | 首发版本：疾跑/跳跃/游泳消耗、延迟恢复、透支锁定、低体力禁跑、像素风 HUD（幽灵拖尾/火花/扫光/自动隐藏）、双端配置热重载、全套指令与生态 API |

## 外部链接

- GitHub 源码与下载：https://github.com/kujojotarojojo/stamina-hud
