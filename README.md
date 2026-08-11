# RSG Item Tool

A single-file HTML tool (`rex_item_tool.html`) for validating and cleaning up RSG-Core `shared/items.lua` item tables. Open the file in a browser — no server or build step required.

<img width="1898" height="871" alt="rex_item_tool" src="https://github.com/user-attachments/assets/7ec484e1-32b6-4864-ae88-ec3c57c6ded0" />

## Features

- **Paste or upload** a `shared/items.lua` file (or any Lua table of item entries).
- **Check Items** — validates every item and reports errors and suggestions:
  - Missing required fields: `name`, `label`, `weight`, `type`, `image`, `unique`, `useable`, `shouldClose`, `description`.
  - `name` not matching the item's table key.
  - Empty or lowercase `label` (suggests Title Case).
  - Invalid, negative, zero, or unusually high `weight`.
  - `type` not one of `item`, `weapon`, `ammo`.
  - `image` not ending in `.png`/`.jpg`, or not matching the item key.
  - `unique` / `useable` / `shouldClose` not `true`/`false`.
  - Short or empty `description`.
  - Deprecated `combinable` field present.
  - **Missing commas** between item entries, and missing commas *between fields within an item* (e.g. `label = 'Water'  weight = 100`) — both silently break the Lua table and are flagged as errors.
- **Export Clean items.lua** — generates a reformatted, deduplicated version of the table:
  - One item per line, fields aligned into columns for readability.
  - Double-quoted string values converted to single quotes.
  - Bare (unbracketed, unquoted) item keys, e.g. `bread = { ... }`.
  - Duplicate item keys removed (first occurrence kept).
  - Items grouped into comment blocks, in this order:
    1. Standard items
    2. `-- weapons` (keys starting with `weapon_`)
    3. `-- ammo` (keys starting with `ammo_`)
    4. `-- perishables` (items with a `decay` field set to something other than `nil`/`false`)
  - Each group sorted alphabetically by key.
  - Output is wrapped as:
    ```lua
    RSGShared = RSGShared or {}
    RSGShared.Items = {
        ...
    }
    ```
  - If missing commas are detected, export prompts for confirmation before proceeding (since the resulting file may still be malformed).
- **Load Sample** — fills the input with a small example table to try the tool out.
- **Clear** — resets the input and results.

## Usage

1. Open `rex_item_tool.html` in a browser.
2. Paste your `items.lua` content into the left panel, or use **Upload items.lua**.
3. Click **Check Items** to see a per-item breakdown of errors and suggestions on the right.
4. Fix any issues in the input (or in your source file and re-upload).
5. Click **Export Clean items.lua** to download a reformatted, grouped, deduplicated version.

## Notes

- Expects item entries in the form `["itemname"] = { ... }` or a bare key form `itemname = { ... }`.
- Designed for RSG-Core's item table format; adjust `VALID_TYPES` and `FIELD_ORDER` in the script if your fork uses different conventions.
