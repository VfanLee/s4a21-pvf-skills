---
name: items
description: >-
  Edit A21 stackable/equipment items (.stk/.equ): materials, potions, prices,
  expiry, binding, packages, vault tickets, new IDs. Use when the PVF task
  mentions materials, consumables, potions, stackable, [price], [value],
  [expiration date], packages, [cera package], or vault tickets. For shop
  listing also read npc-shop/.
---

# A21 materials, consumables, stackables

Sources: current `Script.pvf` and:

- `ServerS4A21/Server/DfoServer/Game/Inventory/ItemMetadataResolver.cs`
- `StackableExpirationPolicyResolver.cs`
- `InventoryCreateService.cs`

Do not paste examples from this file as a complete `.stk`. Listing syntax: [../npc-shop/SKILL.md](../npc-shop/SKILL.md).

## Locate

1. Stackable: `stackable/stackable.lst` → `stackable/**/*.stk` (materials, potions, quest items, packages, emblems, …)
2. Gear / titles / avatars / creatures: `equipment/equipment.lst` → `.equ`
3. To sell: put the ID in the target `.shp` `[item list]`

IDs come from `.lst`. Editing `[name]` or file contents does not allocate an ID.

`[stackable type]`: `[material]` is usually a material; potions are often `[waste]`, with other usable types. Keep the item's original type. Do not change type just to set a price (inventory tab and use rules follow type).

## Shared fields

| Goal | Tags | Notes |
| --- | --- | --- |
| Name / text | `[name]` `[explain]` `[flavor text]` | text ≠ effect |
| NPC buy gold | `[price]` | preferred for normal NPC shops |
| Recycle-related | `[value]` | not the full sell price; see formula |
| Extra gold on exchange | `[add price]` | only on a valid `[need material]` path |
| Bind | `[attach type]` `[trade limit max]` `[impossible contents]` | copy a same-kind sample |
| Stack / weight | `[stack limit]` `[weight]` | raising stack may merge existing piles |
| Level / quality | `[minimum level]` `[grade]` `[rarity]` | `grade` ≠ rarity |
| Icon | `[icon]` `[field image]` `[icon mark]` | text edits do not create art |
| Absolute expiry | `[expiration date]` | CST (UTC+8) |
| Days from create | `[usable period]` | non-negative int from **create time** |

If the tag already exists, change the value; do not add a second copy.

## A21 price (plain stackable)

- **Buy**: `[price]` first; else positive `[value]`. Neither → NPC shop buy price `0`.
- **Sell**: `floor(value / 5)` first; else positive `floor(price / 5)`. Do not store the gold the player should receive in `[value]`.
- **Material-exchange gold**: with a valid `[need material]`, `max(0, price + add price)`. **No** `[value]` fallback; missing `[price]` → gold part `0`.

Examples: `3047` 林纳斯火炉券 `price=10000` `value=10` → buy 10000, sell 2. `1006` 加速药剂 `2000` / `200` → buy 2000, sell 40. `3037` 无色小晶块 `price=100` only → buy 100, sell 20.

For the current server's NPC exchange path, `[need material]` is one effective **material ID, count** pair: it reads only the first two values. Do not add further pairs expecting them to be charged. Do not confuse the pair with the shop item's own ID. A21 samples close `[/need material]`; if a file has no closer, follow neighbors — do not mix styles.

Equipment recycle uses the server equipment rate, **not** `value ÷ 5`, with a minimum. Equipment buy still prefers `[price]`, else `[value]`; with valid `[need material]` use `price + add price`.

`[cash]` and `[medal]` are item metadata, not the current server's generic CERA or medal price setters. For CERA-shop price and page placement, resolve `etc/cerashop.etc`; for medals or another currency, trace the target handler before editing any field.

## Expiry

- `[usable period] 7`: new instance lasts 7 days from create.
- `[expiration date]`: `` `yyyy-MM-dd HH:mm:ss` ``, `yyyy-MM-dd`, parseable `yyyyMMdd`, or Unix seconds; CST.
- Both present: server prefers positive `[usable period]`.
- Expiry is stored on the **instance**. Editing PVF does not refresh old items.
- `[stat change duration]` = effect length; `[cool time]` = use cooldown (often ms). **Neither** is bag expiry.

To drop expiry on future instances: delete the whole `[expiration date]` (not empty string, not a far date) and confirm there is no `[usable period]`. Uncovered new instances get `ExpireTime=0`.

Example: `490002458` 史诗 Buff potion — remove `[expiration date]`; keep `[stat change duration]` / `[cool time]`. The file also has `[usable event]` and `[item category] event`; removing expiry does not lift event limits. Verify by **buying a new one**.

## Materials

Price-only edits: price tags only. Before changing use, search recipes, quests, shops, exchanges for that ID. Do not retag a material as a consumable to “make it listable”.

`3042` 无色大晶体: `[need material] 3037 100`, exchange gold from `[price] 2000`, recycle `400/5=80`.

## Consumables

| Topic | Common tags |
| --- | --- |
| Cooldown | `[cool time]` `[cooltime group]` `[cooltime maintenance]` |
| Effect | `[hp recovery]` `[mp recovery]` `[stat change]` `[stat change duration]` `[effect maintenance]` |
| Limits | `[usable job]` `[action usable place]` `[impossible contents]` |
| Uses / purchase cap | `[total usable count]` `[daily purchase limit]` |

Effect layouts differ per potion: copy a same-kind item; change only values you understand. Flavor like “reduces cooldown” in `[explain]` does not change mechanics (e.g. `2600021`).

`2600561` 顶级力量灵药 is `[waste]` with no `[price]`/`[value]`; listing it sells at 0. To charge, add `[price]` on the `.stk`; for recycle add `[value]` (recycle = value/5). For 7-day expiry add `[usable period] 7`.

## Other existing types

| Type | How to spot | Notes |
| --- | --- | --- |
| Quest item | `[stackable type] [quest]`, e.g. `3072` | search quest refs first |
| Expert-job material | `[material expert job]`, e.g. `2610045` | `[expert type]` and expert shops; do not retag `[material]` |
| Emblem / gem | `[avatar emblem]` / `[flag gem]` | `[enchant]`, target type, equip limits |
| Weapons / armor / jewelry | `.equ` + `[equipment type]` | durability, `[repair price]`, sets; recycle uses equipment rate |
| Title / avatar / creature | `[title name]` `[coat avatar]` `[creature]` | not ordinary weapons |

Copy a same-kind `.equ`/`.stk`. Do not paste potion field blocks. Equipment may also have `[price]`/`[value]`/expiry with the same formats, but wear / durability / recycle rules differ.

## New items

Need all of: unused ID (re-check the final PVF `.lst`), `.lst` mapping, new definition file. A new name and icon path do not create gameplay; the server must already support the use behavior. Placeholder IDs in examples (`123456789`) must be re-checked.

### Fixed package `[cera package]`

`[package data]` repeats **item ID, count**. Positive item IDs only; gold ID `0` is invalid. Do not mix with random `[booster info]`. Avatar packages are often split by job.

Minimal fragment (copy remaining fields from a neighbor; not a complete file):

```text
[stackable type]
	`[cera package]` 0
[package data]
	50 1 3037 100
[/package data]
```

Opening should consume the source item and grant the list. Icons may reuse existing art.

### Cash-shop package tables

Cash-shop placement and CERA price are controlled by `etc/cerashop.etc`, not by an item's `[cash]` field. The current client maps `[regular package]` to its daily-package page, `[package]` to its main package page, and has character-premium entries on the limited/service UI. Treat page mapping as client-specific: inspect the current client and the existing section before editing, and preserve every non-target row.

When moving a product between pages, remove its source row and add exactly one destination row. Check both sections afterwards so it cannot be sold twice. This client can display rows in the reverse of their PVF storage order, so verify the actual page order after packing rather than relying on the row order alone.

An individual timed contract needs a product row in the matching CERA section, its `.stk` definition, and a matching `etc/premiumlist_new.etc` item-to-service mapping. Register each new `.stk` ID in `stackable/stackable.lst`.

An all-service or multi-token contract is a separate path: its `[cera package]` must grant the intended tokens, and every token needs its own service mapping. The server also has a special Devil Contract catalog for all-service packages sourced from `[charac premium package]`; do not assume moving that product row to another section preserves the special path. Trace the purchase flow and verify the activated premium state in-game.

Level-up tickets (e.g. `10006124`): server grants +1 level per use; that ticket's stack limit is 10.

### Personal vault max ticket

Character vault in Seria's room, not account vault. A21 starts at 8 slots, max 200. Existing ID `10098633` (`cash/safe_upgradekit12.stk`) already expands to 200.

For a new ID that should reliably target 200 slots, use a distinct path whose basename remains `safe_upgradekit12.stk`; the server recognizes that basename as tier 12. A non-matching filename can fall back to parsing a supported capacity from the item's Chinese text, but that is less explicit and must be tested in-game. At 200 slots the ticket is not consumed. It does not change inventory, avatar slots, or account vault.

## Verify

1. Client and server load the same packed PVF; restart to clear item cache.
2. Materials: inventory tab, buy price, recycle; if exchange, deducted materials and extra gold.
3. Consumables: buy, stack, use place, cooldown, real effect, recycle; text matches effect.
4. Expiry: check a **new** instance; old items do not prove the new PVF.
5. New IDs: recognized, icon/text OK, package/ticket result matches existing server logic.
6. Quest items / emblems / gear / titles / avatars / creatures: tab, slot, durability, look, and related quest or socket behavior.
