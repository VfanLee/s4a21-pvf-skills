# S4A21 PVF

This repo is an agent skill pack for **86JP S4A21** PVF. It is not the unpacked script tree.

Read this file for structure and ID relationships. For how to edit, read [`.agents/skills/SKILL.md`](.agents/skills/SKILL.md).

Reply to the user in Simplified Chinese.

## Maintainer docs

Chinese learning notes for the human maintainer live in [`docs/`](docs/README.md). Do not load them as edit instructions.

When you learn, correct, or add a PVF fact in this file or in `.agents/skills/`, update the matching `docs/` page in the **same change** so both sides share one set of facts.

## Packed vs unpacked

| Artifact | Role |
| --- | --- |
| `Script.pvf` | Packed archive. Never edit as text. |
| Unpacked script tree | Editable `.lst` / `.npc` / `.shp` / `.stk` / `.equ` / … |
| Original / baseline PVF | Anchor only. Never overwrite. |

Confirm whether the user is editing an **unpacked script directory** or only querying a packed archive. Do not invent paths if the tree is not unpacked.

Client and server must load the **same** packed PVF after changes. Server caches item metadata; restart or reload after deploy.

## `.lst` registries

A `.lst` maps `ID → relative path` under that folder. That mapping **is** the ID. Changing `[name]`, `[explain]`, or the filename does not create a new ID.

Name tables (`itemname.lst` and similar) are **not** path registries. If the user gives a name, search names then resolve the ID. If the user gives a numeric ID, resolve it through the correct `.lst`.

The same number can exist in more than one registry. Resolve in the registry that matches the current context.

## Top-level tree

| Folder | Registry | Target |
| --- | --- | --- |
| `npc/` | `npc/npc.lst` | `.npc` |
| `itemshop/` | `itemshop/itemshop.lst` | `.shp` |
| `stackable/` | `stackable/stackable.lst` | `.stk` |
| `equipment/` | `equipment/equipment.lst` | `.equ` (gear, titles, avatars, creatures) |
| `n_quest/` | `n_quest/quest.lst` | `.qst` |
| `monster/` | `monster/monster.lst` | `.mob` |
| `dungeon/` | `dungeon/dungeon.lst` | `.dgn` |
| `map/` | `map/map.lst` | `.map` |
| `skill/` | `skill/skilllist.lst` (then per-job lists) | skill scripts |
| `character/` | `character/character.lst` | character scripts |
| `town/` | `town/town.lst` | town scripts |
| `worldmap/` | `worldmap/worldmap.lst` | world map |
| `etc/` | (no single ID registry) | `.etc` / tables |

## IDs that must not be mixed

NPC ID ≠ shop ID ≠ item ID.

Forward close:

```text
npc/npc.lst  → NPC ID → npc/*.npc
.npc [role] `[item shop]`  → shop ID
itemshop/itemshop.lst  → shop ID → itemshop/*.shp
.shp [NPC]  → NPC ID (back-pointer only; not a substitute for the forward close)
```

Example: 卡妮娜 NPC ID `3`, shop ID `84`, file `itemshop/84_Kanna.shp`. `[NPC] 3` is not shop ID `84`.

## Layering

- `.shp` lists items only (`[item list]`).
- Buy price, sell/recycle, bind, effects, and expiry live on the item `.stk` / `.equ`.
- A21 shops use `[sell info]` → `[tab]` → `[item list]`, optionally `[use category]` / `[category entry]`.
- Do not apply other-version shop syntax (`[sell item]`, `[tab name]`) to this PVF.

## Runtime meaning

Interpret buy/sell price, expiry, and package grant using `ServerS4A21/` (`PvfLib`, `ItemMetadataResolver`, `InventoryCreateService`). Generic DNF tutorials and other PVF versions are fill-in only; A21 tags win on conflict.

Icon / resource paths in Script.pvf do not prove the client NPK exists.

## Next

Edit only after reading [`.agents/skills/SKILL.md`](.agents/skills/SKILL.md). Load `npc-shop/` or `items/` when the task needs them.
