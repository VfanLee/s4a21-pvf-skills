# PVF 结构

对应 [`AGENTS.md`](../AGENTS.md)。本仓库是 skill 包，不是解包后的脚本树。

## 打包和解包

| 东西 | 是什么 |
| --- | --- |
| `Script.pvf` | 打包后的档案，**不能当文本改** |
| 解包脚本树 | 真正能改的 `.lst` / `.npc` / `.shp` / `.stk` / `.equ` 等 |
| 原版 / 基线 PVF | 对照锚点，**禁止覆盖** |

改完要重新打包装给客户端和服务端，**两边必须是同一份**。服务端会缓存物品元数据，部署后重启或重载。

未解包时不要臆造路径。先确认当前是在解包目录里改，还是只查询已打包的 `Script.pvf`。

## `.lst`：ID 从哪来

`.lst` 的一行是 `数字 ID → 该目录下的相对路径`。**这个映射才是 ID。**

改 `[name]`、`[explain]`、文件名，都不会生成新 ID。新道具必须同时有：空闲 ID、`.lst` 行、定义文件。

`itemname.lst` 这类名称表**不是**路径表。用户给中文名：先搜名称再解析 ID。用户给数字：走对应 `.lst`。

同一个数字可以同时出现在多个 registry 里（例如商店 ID 和物品 ID 撞号）。按当前任务选对表。

`.lst` 行示例（示意，不是完整文件）：

```text
3	`Kanna.npc`
84	`84_Kanna.shp`
3037	`material/cubepiece_clear.stk`
```

反引号里的路径相对该 `.lst` 所在目录。

## 顶层目录

解包后常见结构：

```text
<解包根>/
├── npc/           npc.lst → .npc
├── itemshop/      itemshop.lst → .shp
├── stackable/     stackable.lst → .stk
├── equipment/     equipment.lst → .equ
├── n_quest/       quest.lst → .qst
├── monster/       monster.lst → .mob
├── dungeon/       dungeon.lst → .dgn
├── map/           map.lst → .map
├── skill/         skilllist.lst → 各职业技能
├── character/     character.lst
├── town/          town.lst
├── worldmap/      worldmap.lst
└── etc/           .etc / 表（没有统一 ID 表）
```

| 目录 | 登记表 | 定义文件 | 内容 |
| --- | --- | --- | --- |
| `npc/` | `npc/npc.lst` | `.npc` | NPC |
| `itemshop/` | `itemshop/itemshop.lst` | `.shp` | NPC 商店上架 |
| `stackable/` | `stackable/stackable.lst` | `.stk` | 可堆叠：材料、药剂、礼包、徽章等 |
| `equipment/` | `equipment/equipment.lst` | `.equ` | 装备、称号、装扮、宠物 |
| `n_quest/` | `n_quest/quest.lst` | `.qst` | 任务 |
| `monster/` | `monster/monster.lst` | `.mob` | 怪物 |
| `dungeon/` | `dungeon/dungeon.lst` | `.dgn` | 副本 |
| `map/` | `map/map.lst` | `.map` | 地图 |
| `skill/` | `skill/skilllist.lst`（再进职业表） | 技能脚本 | 技能 |
| `character/` | `character/character.lst` | 角色脚本 | 职业/成长 |
| `town/` | `town/town.lst` | 城镇脚本 | 城镇 |
| `worldmap/` | `worldmap/worldmap.lst` | — | 世界地图 |
| `etc/` | 无统一 ID 表 | `.etc` / 表 | 杂项配置 |

## 三种编号不要混

NPC ID ≠ 商店 ID ≠ 物品 ID。

正向闭合（以这个为准）：

```text
npc/npc.lst          → NPC ID  → npc/*.npc
.npc 的 [role] 里 [item shop] 后面的数字 = 商店 ID
itemshop/itemshop.lst → 商店 ID → itemshop/*.shp
.shp 的 [NPC] 后面的数字 = NPC ID（回指，不能替代上面那条链）
```

例：卡妮娜 NPC ID `3`，商店 ID `84`，文件 `itemshop/84_Kanna.shp`。文件里的 `[NPC] 3` 不是商店号。

## 职责分层

- `.shp` 只负责**上架**（`[item list]` 里写物品 ID）
- 买价、回收、绑定、效果、期限在道具的 `.stk` / `.equ`
- A21 商店结构是 `[sell info]` → `[tab]` → `[item list]`，可再加 `[use category]` / `[category entry]`
- 不要把其他版本的 `[sell item]`、`[tab name]` 套到本 PVF

## 运行时以谁为准

买价、回收价、期限、礼包发放：看 `ServerS4A21/`（`PvfLib`、`ItemMetadataResolver`、`InventoryCreateService`）。网上教程和其他版本 PVF 只作参考；标签冲突时以 A21 为准。

Script.pvf 里的图标路径**不证明**客户端 NPK 里真有这张图。

接下来怎么改文件，见 [skills/README.md](skills/README.md)。
