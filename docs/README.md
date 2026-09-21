# 维护者文档

这份目录是**给你看的**学习笔记，不是给模型执行的操作手册。

改 PVF 时我们一起用同一套事实：模型读英文 `AGENTS.md` / `.agents/skills/`，你读这里的中文。两边说的必须一致，交流才不会跑偏。

## 怎么读

| 想了解 | 打开 |
| --- | --- |
| PVF 结构与 ID 关系 | [structure.md](structure.md) |
| 硬规则与通用修改流程 | [skills/README.md](skills/README.md) |
| NPC 商店（`.shp`） | [skills/npc-shop.md](skills/npc-shop.md) |
| 材料、药剂、价格、礼包（`.stk` / `.equ`） | [skills/items.md](skills/items.md) |

## 和 skills 的对照

| 你读（中文） | 模型读（英文） |
| --- | --- |
| [structure.md](structure.md) | [`AGENTS.md`](../AGENTS.md) |
| [skills/README.md](skills/README.md) | [`.agents/skills/SKILL.md`](../.agents/skills/SKILL.md) |
| [skills/npc-shop.md](skills/npc-shop.md) | [`.agents/skills/npc-shop/SKILL.md`](../.agents/skills/npc-shop/SKILL.md) |
| [skills/items.md](skills/items.md) | [`.agents/skills/items/SKILL.md`](../.agents/skills/items/SKILL.md) |

## 同步约定

改 PVF 过程中，只要学到、修正或补充了一条事实：

1. 先写进对应的 skill / `AGENTS.md`（模型下次按这个做）
2. **同一轮**写进这里对应的中文页（你下次按这个学、按这个对）

只改一边，之后你我说的就不是同一套。docs 不替代 skill：模型改文件仍读 skill，不把整份 docs 灌进上下文。
