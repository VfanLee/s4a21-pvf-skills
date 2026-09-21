# NPC 商店

对应 [`.agents/skills/npc-shop/SKILL.md`](../../.agents/skills/npc-shop/SKILL.md)。价格、效果见 [items.md](items.md)。

**页签** = `[tab]`。**分类** = `[use category]` / `[category entry]`。两者不是同一层。

改列表：动 `.shp`。改买价 / 回收 / 兑换：动道具文件。

依据：当前 `itemshop/*.shp`、`npc/*.npc`，以及 `ServerS4A21/Tool/PvfLib/Models/ItemShopFile.cs`。

下面示例是**成对闭合的结构骨架**。实机文件还有未列出字段，改时保留。

## 目录结构

```text
<解包根>/
├── npc/
│   ├── npc.lst              # NPC ID → .npc
│   └── Kanna.npc
└── itemshop/
    ├── itemshop.lst         # 商店 ID → .shp
    ├── 84_Kanna.shp
    ├── StackableShop2.shp   # 索西雅，多页签
    └── 86_Mintai.shp        # 敏泰，职业分类
```

## 文件说明

| 文件 | 作用 |
| --- | --- |
| `npc/npc.lst` | NPC ID → `.npc` 路径 |
| `npc/*.npc` | NPC 定义；`[role]` 里 `[item shop]` 后面是**商店 ID** |
| `itemshop/itemshop.lst` | 商店 ID → `.shp` 路径 |
| `itemshop/*.shp` | 商店上架列表 |

三种编号不要混，正向闭合见 [../structure.md](../structure.md)。

例：卡妮娜 NPC ID `3`，`npc/Kanna.npc` 商店 ID `84`，文件 `itemshop/84_Kanna.shp`。`[NPC] 3` ≠ 商店 ID `84`。

其他入口还可能是 `[product item]`、`[secret shop]`。秘密商店写在文件里，不等于游戏里一定能打开。

新建商店：同时写 `itemshop.lst` 和 NPC 的 `[item shop]`。改已有商店：保留这些编号关系。

## 字段说明

| 标签 | 含义 | 注意 |
| --- | --- | --- |
| `[NPC]` | 回指 NPC ID | 不能替代 `npc.lst` → `[item shop]` 那条正向链 |
| `[type]` | 商店类型 | 沿用原值，如 `[etc shop]`、`[expert shop]`；不要换成未核对类型 |
| `[sell info]` … `[/sell info]` | 售卖区 | 页签或商品都写在这里 |
| `[tab]` … `[/tab]` | 一个页签 | 反引号文本是页签名；每个页签自己带 `[item list]` |
| `[item list]` … `[/item list]` | 商品 ID 列表 | 只写 ID；空格 / Tab / 换行等效；禁止逗号；换行不会在游戏里分组 |
| `[use category]` | 商店级分类开关 | 如 `basic job`；每个页签仍可自选分类列表或普通列表 |
| `[category entry]` … `[/category entry]` | 一个职业/分类块 | 内含 `[id]`（分类编号，不是商店 ID 或物品 ID）和 `[item list]` |
| `[message]` | 商店对话 | 反引号字符串 |
| `[use toggle]` / `[expert job level]` | 副职业商店限制 | 不要抄到普通商店 |
| `[one a day start time]` / `[one a day item]` | 每日轮换 | 不是把 `[item list]` 换个位置 |

负数（`-1` `-2`）不是商品 ID。商品 ID 要按上下文走 `stackable.lst` 或 `equipment.lst`，不能从位数猜类型。

## 完整示例

### 外层骨架

```text
[NPC]
	3
[type]
	`[etc shop]`
[sell info]
	[tab]
		`消耗品`
		[item list]
			1150 1151 1153
		[/item list]
	[/tab]
[/sell info]
[message]
	`商店对话`
```

### 多页签

每个 `[tab]` 同级。不要把第二个 `[item list]` 写进第一个页签。`[item list]` 顺序即显示顺序。

参考：`84_Kanna.shp`（单页签）、`StackableShop2.shp`（索西雅）。下列为截取，不是完整商店。

```text
[sell info]
	[tab]
		`消耗品`
		[item list]
			1150 1151 1153
		[/item list]
	[/tab]
	[tab]
		`其它`
		[item list]
			10099377
		[/item list]
	[/tab]
[/sell info]
```

### 职业分类 + 页签混用

参考：`86_Mintai.shp`。A21 `basic job` 现有编号包括 `0,1,5,3,4,2,6,7,8,11,10,9,12,13`。未核对客户端分类定义时不要改号。不要把 `basic job` 的编号套到 `job` / `expert job` / `pvp job`。

```text
[sell info]
	[use category]
		`basic job`
	[tab]
		`神器`
		[category entry]
			[id]
				0
			[item list]
				101000282 101030306
			[/item list]
		[/category entry]
	[/tab]
	[tab]
		`消耗品`
		[item list]
			10088618
		[/item list]
	[/tab]
[/sell info]
```

### 无页签 / 每日轮换

- `StackableShop1.shp`：`[sell info]` 下直接 `[item list]`。
- `AbelroExpert.shp`（`[expert shop]`）：`[sell info]` 下直接 `[category entry]`，并带 `[use toggle]` / `[expert job level]`。
- `OneADayItemShop.shp`：普通 `[item list]` 可为空，轮换写在 `[one a day start time]` / `[one a day item]`。

当前 `ItemShopFile` **不会**从无页签结构提取商品，也不递归 `[category entry]`。GM 工具索引对不上时，以客户端为准。改造时沿用该 NPC 已有结构。

## 改哪里

| 目的 | 改哪个文件 |
| --- | --- |
| 上架 / 下架 / 换页签 | `.shp` 的 `[item list]` / `[tab]` |
| 金币买价 | 道具 `[price]`（普通可堆叠没有则回退 `[value]`） |
| 材料兑换 | 道具 `[need material]`；材料 ID 再走 `stackable.lst` |
| 点券商城 | `etc/cerashop.etc` 的商品行；道具 `[cash]` 不会设置当前服务端的点券价 |
| 胜点 / 其他货币 | 先追踪目标客户端和服务端的处理路径；不能只凭 `[medal]` 推断定价方式 |

实机：找到 NPC → 打开商店 → 核对页签 / 职业分类 / 顺序 → 看价格 → 试买或兑换 → 看中文是否乱码。
