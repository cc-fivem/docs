# Docs working notes

This folder is the public Mintlify site for CC Scripts. Pages here are read by **customers installing and configuring our resources on their own servers** — not maintainers and not contributors to the scripts themselves. Treat that as the lens for every editorial decision.

## Audience and intent

- The reader is a server owner. They want to get the resource working and tune it — not learn how it's built.
- Skip implementation details, internal rationale, and "why we did it this way" tangents. If a fact doesn't change something the reader has to do or decide, cut it.
- Assume Lua literacy and basic FiveM server admin. Don't explain what `ensure` does, what a resource is, or how Lua tables work.
- **But always be concrete.** Show the exact file path (`ox_inventory/data/items.lua`, `qb-core/shared/items.lua`, `server.cfg`), the exact snippet to paste, and the exact ensure block — even when "experienced admins already know." The point is copy-paste-able instructions, not a knowledge check. A reader who already knows can skim; a reader who doesn't shouldn't have to guess.
- Concretely: every install step that touches a file must name the file. Every dependency that needs ensuring must appear in a code block with the full load order, not just prose like "ensure it after cc_heistcontracts."

## Voice

- Direct, neutral, lightly conversational. Not corporate, not chummy.
- Short sentences. Verbs over nouns. Active voice.
- Fine: "Drop the resource in, ensure after `cc_heistcontracts`, you're done." Not fine: "We're excited to walk you through the installation journey!" Also not fine: "Resource installation procedure."

## Don't over-document

- Default to **fewer pages**, not more. Split only when a topic is heavy enough that mixing it in would bury the install/config flow.
- One concept per page. If a page has two unrelated H2s that each deserve a paragraph of intro, they probably belong on separate pages — or the second one isn't worth documenting at all.
- Don't add a page just because a feature exists. Document what the customer needs to **install, configure, or troubleshoot**. Internals, edge cases, and "advanced" topics need a real customer-facing reason to exist.
- If you're about to write "for completeness" or "for reference" — stop. That's a sign it's not earning its place.
- Tables earn their keep when there are 4+ rows of parallel data (fields, items, locations). For 1–2 items, a sentence is shorter and clearer.

## Page shapes

Stick to the templates customers already recognise across our docs.

**`index.mdx`** (per resource)
- One-paragraph intro: what it is, what it plugs into, what the customer gets.
- A short overview of the major surfaces (modules, NPCs, headline features) — at a glance, not exhaustively.
- Dependencies as a short list.
- A "Next steps" section linking to `installation.mdx` and (if it exists) `configuration.mdx`.
- **Keep it skimmable.** If you're stacking two or three reference tables here, that material belongs on `configuration.mdx` instead. The index is what someone reads to decide if this resource is what they want, not how to tune it.
- No marketing, no roadmap, no architecture diagrams.

**`installation.mdx`**
- `## Requirements` — bullets, link out to dependencies.
- `## Steps` — `<Steps>` component. Each step is one concrete action with the exact snippet to paste.
- A `## Verify in-game` step near the end with the smallest path that proves it works.
- A final `## Configure` step that hands off to `configuration.mdx` (if one exists), in a single sentence — don't restate tunables here.
- Trailing sections only for things the customer must do to finish install (e.g. one-time DB import, mandatory inventory paste). Configuration knobs, NPC moves, module toggles, payment sources, and rebalancing all belong on `configuration.mdx` instead.

**`configuration.mdx`**
- The home for every knob: module toggles, NPC placement, payment sources, key tunables, editable Lua files.
- Tables of fields with `Field | Default | What it does` for tunables.
- Code blocks for config snippets that are copy-pasteable as-is. No partial snippets that the reader has to assemble.
- Tell the reader where each block lives (e.g. "in `shared/config.lua` → `cfg.npc`") and that they need to restart the resource after editing.

**`exports.mdx` / `admin.mdx`**
- Reference-style, same shape as `configuration.mdx` but scoped to integrators (exports) or owners/admins (admin commands, permissions).

## Frontmatter

Every page opens with:

```
---
title: "Short Title Case"
description: "One-line, sentence case, ends with a period. Reads like a search-result snippet."
icon: 'lucide-icon-name'
---
```

- `title`: short. The sidebar label, not a marketing slogan.
- `description`: one sentence, ≤ ~140 chars, action-oriented when possible.
- `icon`: optional but used widely — match the resource family (e.g. heists tend to use `star`, `rocket`, `briefcase`, `layer-group`).

## Registering pages in [docs.json](docs.json)

A new `.mdx` file is invisible until it's listed in [docs.json](docs.json). After creating a page:

1. Open [docs.json](docs.json).
2. Find the matching `group` under `navigation.tabs[Guides].groups`.
3. Add the page path **without `.mdx`** to the `pages` array, in the order the customer should read it (usually: `index` → `installation` → `configuration` → topic pages).

Do not touch `theme`, `colors`, `appearance`, `logo`, `favicon`, `footer`, `navbar`, or `contextual` unless explicitly asked.

## Components we use

Reach for these first; don't invent alternatives:

- `<Columns cols={N}>` + `<Card>` — landing/index page resource grids.
- `<Steps>` + `<Step title="...">` — installation and any ordered procedure.
- `<Tabs>` + `<Tab title="ox_inventory">` / `<Tab title="qb-inventory">` — inventory-system variants. Always show both when items are involved.
- `<Expandable title="...">` — long code blocks the reader rarely needs open (e.g. full `gabz_entityset_mods1.lua`).
- `<Note>`, `<Tip>`, `<Warning>` — sparingly. One per page is usually enough; multiple in a row reads as noise.

## Code samples

- Snippets must match the actual code shipped in [resources/[cc]/](../resources/[cc]/). Open the source, copy the real export signature, field name, default value — don't paraphrase from memory.
- Lua only, no pseudocode.
- Paths in prose use backticks: `shared/contract.lua`, `ox_inventory/data/items.lua`.
- File references inside this doc folder use markdown links: [cc_fleecaheist/installation.mdx](cc_fleecaheist/installation.mdx).
- External dependencies link to their canonical repo on first mention: [ox_lib](https://github.com/overextended/ox_lib).

## Adding a new heist module

When a new heist resource gets a docs folder, **always update [all_heists/installation.mdx](all_heists/installation.mdx) in the same change**. The all-heists page is the one-shot setup for customers who own every heist module — it must stay in sync.

Specifically, when adding a heist:

1. Create the resource's own folder (`cc_<name>heist/`) with `index.mdx` and `installation.mdx`.
2. Add the group to [docs.json](docs.json) under the `Guides` tab.
3. In [all_heists/installation.mdx](all_heists/installation.mdx):
   - Add the new resource path to the "Place the resources" step.
   - Merge any new inventory items into the consolidated `ox_inventory` and `qb-inventory` blocks (no duplicates, alphabetical-ish grouping by purpose).
   - Add any new gabz `entitySets` entries to the consolidated `gabz_entityset_mods1.lua` block.
   - Add the new `ensure` line to the load-order example, after `cc_heistcontracts`.

If a heist gets removed or renamed, do the inverse pass over [all_heists/installation.mdx](all_heists/installation.mdx).

## Don't touch

- [changelog.mdx](changelog.mdx) — hand-curated release notes. Don't auto-edit.
- [docs.json](docs.json) theme/branding fields (see above).
- [.mintignore](.mintignore), [favicon.ico](favicon.ico), [logo/](logo/), [images/](images/) unless asked.
