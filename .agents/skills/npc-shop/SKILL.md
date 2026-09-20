---
name: npc-shop
description: >-
  Edit A21 NPC shops in itemshop/*.shp: tabs, item lists, job categories,
  listing, new shops. Use when the PVF task touches NPC shop, itemshop, .shp,
  listing, tabs, [tab], [item list], or [use category]. Price lives on the item
  file. Read repo AGENTS.md and skills/SKILL.md first.
---

# A21 NPC shops

Tab = `[tab]`. Category = `[use category]` / `[category entry]`. Not the same layer.

Change the list: edit `.shp`. Change buy/sell/exchange: edit the item file — [../items/SKILL.md](../items/SKILL.md).

Sources: current `itemshop/*.shp`, `npc/*.npc`, and `ServerS4A21/Tool/PvfLib/Models/ItemShopFile.cs`.

## Locate

Do not mix the three numbers. Forward close is in [`AGENTS.md`](../../../AGENTS.md).

Example: 卡妮娜 NPC ID `3`, `npc/Kanna.npc` shop ID `84`, file `itemshop/84_Kanna.shp`. `[NPC] 3` ≠ shop ID `84`.

Other entries may be `[product item]` or `[secret shop]`. A static secret-shop close does not mean it appears in-game.

New shop: write `itemshop/itemshop.lst` and the NPC `[item shop]`. Existing shop: keep those ID links.

## Outer skeleton

```text
[NPC]
	<NPC ID>
[type]
	`[etc shop]`
[sell info]
	...tabs or items...
[/sell info]
[message]
	`商店对话`
```

Keep the target shop's existing `[type]` (`[etc shop]`, `[expert shop]`, …). Do not swap in an unchecked type.

## `[sell info]` (copy the target shop)

### One tab / many tabs

Each `[tab]` is a sibling with its own `[item list]`. Do not nest a second `[item list]` inside the first tab. Backtick text after `[tab]` is the tab label; `[item list]` order is display order.

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

Refs: `84_Kanna.shp` (one tab), `StackableShop2.shp` (索西雅, many tabs). Snippets above are truncated, not full shops.

### Shop-level categories mixed with tabs

`[use category]` is shop-wide. **Each tab may use either category lists or a plain list.**

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

Ref: `86_Mintai.shp`. `[category entry] [id]` is a class ID for the current `[use category]`, not a shop ID or item ID. A21 `basic job` IDs include `0,1,5,3,4,2,6,7,8,11,10,9,12,13`. Do not renumber without checking client category defs.

PVF also has `job`, `expert job`, `expert job non filter`, `pvp job`. Do not reuse `basic job` IDs on other categories.

### No tabs

- `StackableShop1.shp`: `[item list]` directly under `[sell info]`.
- `AbelroExpert.shp` (`[expert shop]`): `[category entry]` directly under `[sell info]`, plus `[use toggle]` / `[expert job level]`.

Current `ItemShopFile` **does not extract items** from either. Keep that NPC's existing shape. Do not copy expert-job toggles onto a normal shop.

### Daily rotation

`OneADayItemShop.shp`: the plain `[sell info]` list may be empty; rotation lives in `[one a day start time]` / `[one a day item]`. That is not a relocated `[item list]`. Selection rules are unconfirmed: keep the original shape and verify in-game.

## Write rules

- `[item list]`: item IDs only. Space, tab, or newline are equivalent; newlines do not group in-game. No commas.
- Do not put price or effects in `.shp`. Before listing, check ID, price, level, bind, expiry.
- Pair `[tab]`, `[category entry]`, `[item list]`, `[sell info]`. Category goods go in that `[category entry]`'s `[item list]`; plain goods go in the tab's `[item list]`.
- Negatives (`-1` `-2`) are not item IDs.
- Resolve item IDs through `stackable/stackable.lst` or `equipment/equipment.lst`. Digit count is not the type.

## Tool-index limits

`ItemShopFile.cs` only reads `[item list]` **directly inside** a `[tab]` under `[sell info]`:

- no recursion into `[category entry]`
- no tabless shops

GM/tool shop indexes cannot prove category tabs or tabless shops. Verify category UI and purchase on the target client.

## Read-only close

1. `npc.lst` → `.npc` → `[role]` shop entry → shop ID
2. `itemshop.lst` → `.shp`
3. Check `[NPC]`, `[type]`, `[message]`, `[sell info]` shape
4. Collect every positive item ID; resolve each registry; read `[name]`, `[price]`, `[value]`, `[cash]`, `[need material]`, `[medal]`, expiry
5. List secret shop, daily rotation, expert job, and log-only entries separately; they are not normal sale facts

## Write and verify

| Goal | Edit |
| --- | --- |
| List / unlist / retab | `.shp` `[item list]` / `[tab]` |
| Gold buy price | item `[price]` (plain stackables fall back to `[value]`) |
| Material exchange | item `[need material]`; resolve material IDs in stackable |
| CERA / medals | item `[cash]` / `[medal]`; do not generalize to every system |

In-game: find the NPC → open shop → check tabs / job categories / order → check price → buy or exchange → check Chinese text. If the tool index disagrees, trust the client.
