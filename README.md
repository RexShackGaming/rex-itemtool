# RSG Item Tool

A single-file HTML tool for validating and cleaning up RSG-Core `shared/items.lua` item tables. Open `rex_item_tool.html` in any browser — no server or build step required.

<img width="1903" height="895" alt="Screenshot 2026-10-07 061252" src="https://github.com/user-attachments/assets/5bb26476-6ce8-4112-a534-51c72a704541" />

---

## What it does

Opening the tool shows a **start page** with two options:

- **Option 1 — Create items.lua from images:** pick a folder of item images and an item is generated for each one.
- **Option 2 — Check an existing items.lua:** upload your file and it's checked straight away for format errors, missing fields and duplicate keys.

There's also a link to open the tool empty and paste items by hand. Use **← Start page** in the header to go back.

Paste (or upload) your items table, hit **Check Items**, and the tool breaks down every item on the right side: clean items, suggestions, and errors. From there you can **Fix** issues one item at a time, **Fix All** in one click, or **Export** a clean, reformatted copy of the whole file.

| Button | What happens |
| --- | --- |
| **Upload items.lua** | Loads a `.lua`/`.txt` file into the editor. |
| **From Image Folder** | Pick a folder of `.png`/`.jpg`/`.jpeg` images and one item is created per image (key = filename, lowercased, spaces/symbols → `_`), with Fix defaults for every other field. The `image` field keeps the real filename and extension. Appends to the current table, skipping keys that already exist. |
| **Check Items** | Validates every item and lists errors / suggestions per item, with summary stats up top. |
| **Fix** *(per item)* | Repairs that one item's block in the editor (and removes any later duplicates of its key), then re-checks it automatically so it re-renders clean. |
| **Fix All Items** | Removes duplicate keys (first occurrence kept) and applies the same fixes to every item at once. |
| **Export Clean items.lua** | Downloads a reformatted, deduplicated, grouped version of the table. |
| **Load Sample** | Fills the editor with an example table so you can try the tool. |
| **Clear** | Wipes the editor and the results panel. |

The **Clean**, **Suggestions**, **Errors**, and **Duplicate keys** stat tiles above the results are also filters — click one (or several) to narrow the list to just those items, click again to unclick, or use **Clear Filters** to reset. Handy for working through a big table's errors first without scrolling past everything that's already clean.

A **Category** dropdown next to **Fix All Items** lets you narrow the list to a single category (e.g. `tools`, `medical`, `general`) — it's populated automatically from whatever categories are present in your table (declared `category` field, falling back to `perishable` for items with an active `decay` field, then `type`, then `item`). It combines with the stat-tile filters, so you can e.g. show only the **errors** within the **medical** category. **Clear Filters** resets this back to "All Categories" too.

Below that, a **Set category for all items** bar changes the category of every item in one go. It applies to whatever the list is currently showing, so combine it with the filters to bulk-recategorize just a subset (e.g. pick `general` in the Category dropdown, then move all of those to `tools`). Choose an existing category or "+ New category...", click **Apply** (or press Enter in the new-category box), and confirm.

Each item's expanded view also has a **Category** editor: a dropdown (pre-filled with every category already in your table) plus a "+ New category..." option that reveals a text box for typing a brand new one. Pick a category and click **Set** (or press Enter in the new-category box) — the editor sets (or adds) the item's `category` field in place in the source editor and re-checks, without touching any other field.

---

## What Fix does to an item

- **`name`** → set to match the item key.
- **`label`** → Title Case if empty or lowercase (e.g. `bandage` → `Bandage`).
- **`weight`** → `100` if missing, not a number, or negative.
- **`type`** → `'item'` if missing. Custom types (e.g. `stew`) are **kept** and not flagged.
- **`image`** → `key.png` if missing, invalid, or not matching the item key.
- **`unique` / `useable` / `shouldClose`** → `false` / `true` / `true` unless a valid boolean is present.
- **`description`** → filled with label-based text (e.g. `'Bandage — useful item to have around.'`) when empty or very short.
- **`combinable`** → removed (deprecated in current RSG-Core).
- **`category`** → set to `'general'` if missing; existing values are kept.
- Extra fields (e.g. `decay`, `blends`) are preserved.
- Duplicate keys are removed, keeping the first occurrence (same as Export).
- Missing commas — both between fields (`label = 'Water'  weight = 100`) and between items — are inserted.
- Invalid Lua escape sequences inside quoted strings (e.g. `\S`) are repaired.

---

## What Check looks for

**Errors** (breaks the table or won't work):
- Missing required fields: `name`, `label`, `weight`, `type`, `image`, `unique`, `useable`, `shouldClose`, `description`
- `name` not matching the item's table key
- `weight` not a number, or negative
- `image` not ending in `.png`/`.jpg`
- `unique` / `useable` / `shouldClose` not `true` or `false`
- Missing commas between items or between fields within an item
- Invalid Lua escape sequences inside quoted strings (e.g. `\S`)
- **Duplicate item keys** (the same key used for more than one entry — only one will actually load, the rest are silently shadowed)

**Suggestions** (safe to ignore, auto-fixed where possible):
- Lowercase or empty `label`
- Unusually high `weight`
- `image` filename not matching the item key
- Short or empty `description`
- Deprecated `combinable` field
- Missing `category` field

Custom `type` values (e.g. `stew`, `painkillers`) and zero-weight items are **not** flagged — these are common and often intentional on RSG servers.

---

## What Export produces

- One item per line with fields aligned into columns
- Double-quoted strings converted to single quotes; apostrophes and backslashes are re-escaped correctly, so strings like `"The World's"` export as `'The World\'s'` — always valid Lua
- Invalid escape sequences (e.g. `\S`) repaired during conversion
- Bare, unbracketed item keys (e.g. `bread = { ... }`)
- Duplicate keys removed (first occurrence kept)
- Items with no `category` field are automatically given `category = 'general'` before export
- Items sorted alphabetically and grouped by `category` (e.g. `-- tools`, `-- medical`, `-- general`)
- Output wrapped as:

```lua
RSGShared = RSGShared or {}
RSGShared.Items = {
    ...
}
```

If missing commas are detected, Export asks for confirmation first — the resulting file may still be malformed.

---

## Usage

1. Open `rex_item_tool.html` in a browser.
2. Paste your `items.lua` content into the left panel, or use **Upload items.lua**.
3. Click **Check Items** to see the per-item breakdown on the right.
4. Click **Fix** on any item (or **Fix All Items**) to repair issues directly in the editor.
5. Click **Export Clean items.lua** to download the cleaned version.

---

## Notes

- The version shown next to the title comes from `config.js` (`version: '1.0.0'`). Keep `config.js` in the same folder as `rex_item_tool.html`; edit it to bump the version.
- Expects entries in the form `["itemname"] = { ... }`, `itemname = { ... }`, or `RSGCore.Shared.Items["itemname"] = { ... }`.
- Designed for RSG-Core's item table format; adjust `REQUIRED_FIELDS` and `FIELD_ORDER` in the script if your fork uses different conventions.

---

## Support

If you like this script, you can support development here:

[![Buy Me a Coffee](https://img.shields.io/badge/Buy%20Me%20a%20Coffee-ffdd00?style=for-the-badge&logo=buy-me-a-coffee&logoColor=black)](https://buymeacoffee.com/rexshack)
