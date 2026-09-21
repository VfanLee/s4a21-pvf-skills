# 材料、消耗品与可堆叠道具

对应 [`.agents/skills/items/SKILL.md`](../../.agents/skills/items/SKILL.md)。上架语法见 [npc-shop.md](npc-shop.md)。

依据当前 `Script.pvf` 与：

- `ServerS4A21/Server/DfoServer/Game/Inventory/ItemMetadataResolver.cs`
- `StackableExpirationPolicyResolver.cs`
- `InventoryCreateService.cs`

下面示例是结构骨架。实机 `.stk` 还有未列出字段，**不要把示例粘成完整文件**。已有标签改原值，不要重复添加。

## 目录结构

```text
<解包根>/
├── stackable/
│   ├── stackable.lst
│   ├── material/cubepiece_clear.stk      # 3037 无色小晶块
│   └── cash/safe_upgradekit12.stk        # 10098633 金库升到 200 格
└── equipment/
    └── equipment.lst                     # → .equ
```

## 文件说明

| 文件 | 作用 |
| --- | --- |
| `stackable/stackable.lst` | 可堆叠 ID → `.stk`（材料、药剂、任务物、礼包、徽章等） |
| `stackable/**/*.stk` | 道具定义 |
| `equipment/equipment.lst` | 装备 / 称号 / 装扮 / 宠物 ID → `.equ` |
| `equipment/**/*.equ` | 装备类定义 |
| 目标 `itemshop/*.shp` | 要出售时，把 ID 写入 `[item list]` |

ID 由 `.lst` 映射。改 `[name]` 不会生成新 ID。

`[stackable type]`：`[material]` 通常是材料；药剂常见 `[waste]`。以目标道具原类型为准。不要为改价格而改类型（影响背包分栏和使用规则）。

## 共用字段

| 标签 | 含义 | 注意 |
| --- | --- | --- |
| `[name]` `[explain]` `[flavor text]` | 名称 / 说明 / 风味 | 说明 ≠ 效果 |
| `[price]` | NPC 金币买价 | 普通商店优先用这个 |
| `[value]` | 回收相关数值 | **不是**玩家拿到的回收价，见下方公式 |
| `[add price]` | 兑换时额外金币 | 只在有效 `[need material]` 路径生效 |
| `[need material]` … `[/need material]` | 兑换材料 | **材料 ID、数量**成对；不要和商品自身 ID 混淆 |
| `[attach type]` `[trade limit max]` `[impossible contents]` | 绑定 / 流通限制 | 仿同类样本 |
| `[stack limit]` `[weight]` | 堆叠上限 / 重量 | 提高堆叠前考虑已有堆叠会不会合并 |
| `[minimum level]` `[grade]` `[rarity]` | 等级 / 品质展示 | `grade` ≠ 稀有度 |
| `[icon]` `[field image]` `[icon mark]` | 图标 / 掉落外观 | 改文本不会生成新图 |
| `[expiration date]` | 固定到期时间 | 东八区 |
| `[usable period]` | 获得后有效天数 | 非负整数，从**创建时刻**起算 |
| `[cash]` `[medal]` | 道具元数据 | 不是当前服务端通用的点券 / 胜点定价入口；先追踪实际处理路径 |

## 买价和回收（普通可堆叠）

- **买入**：优先 `[price]`；没有则用正数 `[value]`；两者都没有 → 商店买价 `0`
- **卖出**：优先 `floor(value / 5)`；没有 `value` 则用正数 `floor(price / 5)`。不要把「想让玩家拿到的钱」填进 `[value]`
- **材料兑换金币**：有效 `[need material]` 时用 `max(0, price + add price)`。**不会**回退 `[value]`；缺 `[price]` 则金币部分为 `0`

| ID | 名称 | 字段 | 买 | 卖 |
| --- | --- | --- | --- | --- |
| `3047` | 林纳斯火炉券 | `price=10000` `value=10` | 10000 | 2 |
| `1006` | 加速药剂 | `2000` / `200` | 2000 | 40 |
| `3037` | 无色小晶块 | 只有 `price=100` | 100 | 20 |
| `3042` | 无色大晶体 | `price=2000` `value=400`，兑换 `3037 × 100` | 兑换金币 2000 | 80 |

装备回收走服务端装备费率，**不能**套 `value ÷ 5`，有最低价。装备买价同样优先 `[price]`，缺省回退 `[value]`；有兑换时用 `price + add price`。

当前服务端的 NPC 兑换只读取 `[need material]` 的前两个数值，即一个“材料 ID、数量”对；不要增加多组材料并期待服务器会扣除。A21 样本使用闭合 `[/need material]`；若某文件没有闭合，跟近邻，不要混用。

### 兑换字段示例

```text
[price]
	2000
[value]
	400
[need material]
	3037 100
[/need material]
```

## 期限

| 标签 | 含义 | 注意 |
| --- | --- | --- |
| `[usable period] 7` | 新实例从创建起 7 天 | |
| `[expiration date]` | 固定时刻 | `` `yyyy-MM-dd HH:mm:ss` ``、`yyyy-MM-dd`、可解析的 `yyyyMMdd` 或 Unix 秒；东八区 |
| `[stat change duration]` | 使用后效果持续 | 常见毫秒；**不是**背包到期 |
| `[cool time]` | 使用冷却 | 常见毫秒；**不是**背包到期 |

两字段同时存在：服务端优先正数 `[usable period]`。到期写在道具**实例**上，改 PVF 不会刷新旧道具。

去掉今后新实例的到期：删除整个 `[expiration date]`（不要改成空字符串或随意远期），并确认没有 `[usable period]`。未覆盖时新实例 `ExpireTime=0`。

例：`490002458` 史诗 Buff 药剂，去掉 `[expiration date]`；保留效果持续和冷却。该文件还有 `[usable event]`、`[item category] event`，去期限不会自动取消活动限制。验收必须**新买一件**。

## 材料

改价格只动价格标签。改用途前先搜该 ID 的配方、任务、商店、兑换。不要把材料改成消耗品类型来「方便上架」。

## 消耗品字段

| 内容 | 常见标签 |
| --- | --- |
| 冷却 | `[cool time]` `[cooltime group]` `[cooltime maintenance]` |
| 效果 | `[hp recovery]` `[mp recovery]` `[stat change]` `[stat change duration]` `[effect maintenance]` |
| 限制 | `[usable job]` `[action usable place]` `[impossible contents]` |
| 次数 / 限购 | `[total usable count]` `[daily purchase limit]` |

效果参数结构因药剂而异：复制同类道具，只改明确了解的数值。说明里写「减冷却」不代表改 `[explain]` 就会改机制（如 `2600021`）。

`2600561` 顶级力量灵药是 `[waste]`，没有 `[price]` / `[value]`，直接上架买价为 0。要收费就加 `[price]`，要回收再设 `[value]`（回收 = value/5）。要 7 天期限则加 `[usable period] 7`。

## 其他已存在类型

| 类型 | 怎么认 | 注意 |
| --- | --- | --- |
| 任务物品 | `[stackable type] [quest]`，如 `3072` | 先查任务引用 |
| 副职业材料 | `[material expert job]`，如 `2610045` | 看 `[expert type]` 与副职业商店；不要改成 `[material]` |
| 徽章 / 守护珠 | `[avatar emblem]` / `[flag gem]` | 看 `[enchant]`、目标类型、装备限制 |
| 武器防具首饰 | `.equ` + `[equipment type]` | 耐久、`[repair price]`、套装；回收走装备费率 |
| 称号 / 装扮 / 宠物 | `[title name]` `[coat avatar]` `[creature]` | 不要当普通武器改 |

改这些类型时找同类 `.equ` / `.stk`，不要复制药剂字段块。

## 新增道具

必须同时有：未占用 ID（最终 PVF 的 `.lst` 再查一次）、`.lst` 映射、新定义文件。新名字和新图标不会创造新玩法；服务端必须已支持该使用行为。示例里的占位 ID（如 `123456789`）用前必须重新查重。

### 固定内容礼包 `[cera package]`

`[package data]` 按 **道具 ID、数量** 重复。只接受正数物品 ID，不能写金币 ID `0`。不要和随机 `[booster info]` 混用。装扮礼包常按职业分款。

```text
[stackable type]
	`[cera package]` 0
[package data]
	50 1 3037 100
[/package data]
```

打开后应消耗源道具并发放列表。图标可暂借已有资源。

### 商城礼包表

商城放在哪个页面、点券卖多少钱，取决于 `etc/cerashop.etc`，不是只给道具写 `[cash]`。当前客户端把 `[regular package]` 显示为日常礼包页，`[package]` 显示为主礼包页，角色服务类商品出现在限时/服务界面。页面归属是客户端行为：每次都要对照当前客户端和已有段，不把这个映射外推到别的版本。修改前完整读取目标段，只动本次目标行，其他行保持原样。

把商品从一个页面移动到另一个页面时，要同时做两件事：从来源段删除该商品行，并在目标段只新增一行。读回时两段都要查，避免同一个商品在两页重复出售。

这个客户端的页面显示顺序可能和 PVF 中记录顺序相反。写完打包后必须打开对应商城页核对实际顺序，不能只凭文件行号判断。

新增**单项**限时契约要闭合三层关系：目标商城段的商品行、对应 `.stk`、`etc/premiumlist_new.etc` 里的“道具 → 服务类型/期限”映射。所有新增 `.stk` ID 还要登记 `stackable/stackable.lst`。

“一次开通多项服务”的契约要单独处理：顶层 `[cera package]` 必须准确发放所有服务令牌，每个令牌也都要有服务映射。服务端还对 `[charac premium package]` 的魔王全服务礼包有专门处理；不能假设把商品行移动到其他段后，这条专门路径仍然有效。必须走完整购买流程并核对实际激活的服务状态。

直升类可塞已有升级券（如 `10006124`）：每次使用只升 1 级；该券单格堆叠上限是 10。

### 个人金库满级券

塞利亚房间的**角色个人金库**，不是账号金库。A21 初始 8 格，最高 200 格。现成功能：ID `10098633`（`cash/safe_upgradekit12.stk`）。

要独立 ID 且稳定指向 200 格：用不同路径创建文件，但文件名保留 `safe_upgradekit12.stk`；服务端按这个文件名识别第 12 档。若改用其他文件名，服务端只会尝试从中文说明中解析受支持的格数，可靠性较低，必须实机验证。已满 200 格不会再消耗。不改变背包栏、装扮栏、账号金库。

## 核对

1. 客户端与服务端加载同一份打包 PVF；重启以清物品缓存。
2. 材料：背包分类、买价、回收价；有兑换则核对扣材料和附加金币。
3. 消耗品：购买、堆叠、使用地点、冷却、实际效果、回收价；说明与效果一致。
4. 期限：用**新实例**看；不要用旧道具判断新 PVF。
5. 新 ID：能识别、图标文案正常、礼包 / 券结果符合服务端已有逻辑。
6. 任务物 / 徽章 / 装备 / 称号 / 装扮 / 宠物：分类、穿戴位置、耐久、外观，以及对应任务或镶嵌功能。
