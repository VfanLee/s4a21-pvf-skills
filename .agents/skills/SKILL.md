---
name: pvf-a21
description: >-
  Edit S4A21 DNF PVF (Script.pvf, .lst, .shp, .stk, .equ). Use for any PVF task:
  unpack/pack, NPC shop listing, item price/expiry, new items, or edits under
  npc/itemshop/stackable/equipment. Read repo AGENTS.md for structure; load
  npc-shop/ or items/ only when the task needs them.
---

# PVF A21 — how to edit

Structure and registries: repo-root [`AGENTS.md`](../../AGENTS.md). Do not load every domain file at once.

Reply to the user in Simplified Chinese.

## Load next

| Task | Read |
| --- | --- |
| Shop tabs, listing, categories, `.shp` | [npc-shop/SKILL.md](npc-shop/SKILL.md) |
| Materials, potions, price, expiry, packages, tickets, `.stk`/`.equ` | [items/SKILL.md](items/SKILL.md) |

Shop + item in one change: edit `.stk`/`.equ` first (price, expiry, effect), then write the ID into `.shp` `[item list]`.

Skills, dungeons, monsters, NUT, drops: resolve the matching `.lst` and read the source. No extra skill file; still follow the hard rules below.

## Hard rules

- Default read-only. Without explicit write permission: do not write PVF, do not edit client ImagePacks2/NPK, do not overwrite the baseline pack.
- A numeric ID is not a fact until resolved through the correct `.lst`.
- New items need a free ID, an `.lst` row, and a definition file. Renames do not allocate IDs.
- New blocks / new files: copy 2–3 same-folder, same-extension, same-purpose neighbors. Do not invent tag layout from the tag name.
- Keep existing tags, backticks, whitespace, numeric order, and paired `[/...]`. Do not paste skill examples as complete files.
- `.shp` lists items only. Price, bind, effect, expiry: item file.
- `[explain]` is not the effect. Script.pvf resource paths do not prove client assets exist.
- Packed PVF writes require a PVF-aware reader/writer: save a temporary archive, reopen it for validation, then atomically replace only the authorized target. Never edit the archive as text.
- After writes, read the files back. Behavior claims must say how to test in-game. Client and server load the **same** new PVF; restart the server to clear process-level metadata caches. Use reload only when that deployment's reload path is separately verified.
- When you learn, correct, or add a PVF fact in skills or `AGENTS.md`, update the matching Chinese page under [`docs/`](../../docs/README.md) in the same change. Do not load `docs/` as edit instructions.

## Workflow

```text
read-only close → (after permission) minimal edit → read-back → pack matching client/server PVF → in-game check
```

1. Confirm the system (NPC / item / shop / other) and whether writes are allowed.
2. Resolve name or ID through the matching `.lst`; read the target file. Do not guess from filenames.
3. List paths, fields, old values, new values, and dependents (shops, recipes, quests, packages).
4. Edit only after permission; only planned fields.
5. Read back edited files and related `.lst`.
6. Report: what changed, what did not, pack/load notes, how to verify in-game, whether old item instances refresh.

## Replies

Lead with: can it be done, which files, main risks, next step. Attach IDs, paths, and raw tags when needed.
