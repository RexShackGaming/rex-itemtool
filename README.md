# RSG Item Tool

- ✨ Store   : https://store.rexshack.dev/
- ✨ Tip Jar : https://buymeacoffee.com/rexshack
- ✨ Discord : https://discord.gg/Dmeh4dTQBT

A single-file HTML tool for validating and cleaning up RSG-Core `shared/items.lua` item tables. Open `rex_item_tool.html` in any browser — no server or build step required.

<img width="1898" height="871" alt="rex_item_tool" src="https://github.com/user-attachments/assets/7ec484e1-32b6-4864-ae88-ec3c57c6ded0" />

---

## What it does

Paste (or upload) your items table, hit **Check Items**, and the tool breaks down every item on the right side: clean items, suggestions, and errors. From there you can **Fix** issues one item at a time, **Fix All** in one click, or **Export** a clean, reformatted copy of the whole file.

| Button | What happens |
| --- | --- |
| **Upload items.lua** | Loads a `.lua`/`.txt` file into the editor. |
| **Check Items** | Validates every item and lists errors / suggestions per item, with summary stats up top. |
| **Fix** *(per item)* | Repairs that one item's block in the editor, then re-checks it automatically so it re-renders clean. |
| **Fix All Items** | Applies the same fixes to every item at once. |
| **Export Clean items.lua** | Downloads a reformatted, deduplicated, grouped version of the table. |
| **Load Sample** | Fills the editor with an example table so you can try the tool. |
| **Clear** | Wipes the editor and the results panel. |

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

- Expects entries in the form `["itemname"] = { ... }`, `itemname = { ... }`, or `RSGCore.Shared.Items["itemname"] = { ... }`.
- Designed for RSG-Core's item table format; adjust `REQUIRED_FIELDS` and `FIELD_ORDER` in the script if your fork uses different conventions.

---

## Recent fixes

- **Duplicate item keys are now flagged by Check** — items sharing the same key (e.g. two `["bandage"] = { ... }` entries) previously went unmentioned by Check even though Export silently dropped all but the first. Each duplicate now shows as an error on every occurrence, and the summary bar shows a Duplicate keys count.

- **Nested field tables no longer misread as items** — extra fields written as their own table (e.g. `blends = { ... }`) were previously picked up as bogus top-level item entries, throwing off Check counts, missing-comma detection, and Export. The scanner now skips past each item's full body before looking for the next one.
- **Safer rendering of untrusted item data** — item values (name, image, label, etc.) are now HTML-escaped before being shown in the results panel, so pasting or uploading a crafted/untrusted `items.lua` can't inject markup into the page.
