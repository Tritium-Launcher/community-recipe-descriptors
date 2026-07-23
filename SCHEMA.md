# Schema Reference

## `[recipe]`

| Field | Required | Default | Description |
|---|---|---|---|
| `id` | yes | — | Recipe type ID (`namespace:name`) |
| `display_name` | no | `id` | Human-readable name |
| `width` | no | `176` | Sprite width in pixels |
| `height` | no | `166` | Sprite height in pixels |
| `catalysts` | no | `[]` | Item IDs that unlock this recipe type |

Sprite is auto-resolved from a `sprite.png` file in the same directory as the descriptor.

## `[[slot]]`

| Field | Required | Default | Description |
|---|---|---|---|
| `id` | yes | — | Slot identifier |
| `label` | no | `id` | Display label |
| `role` | yes | — | `input`, `output`, `fuel`, `energy`, `duration`, `custom` |
| `slot_type` | no | `"ITEM"` | Value type (`ITEM`, `FLUID`, or custom) |
| `x` | yes | — | X position on the sprite (pixels) |
| `y` | yes | — | Y position on the sprite (pixels) |
| `w` | no | `18` | Width (pixels) |
| `h` | no | `18` | Height (pixels) |
| `max_capacity` | no | `64` | Stack size for items, millibuckets for fluids |
| `display_only` | no | `false` | Render as label/icon only |

## `[[option]]`

| Field | Required | Default | Description |
|---|---|---|---|
| `key` | yes | — | Option identifier |
| `label` | no | `key` | UI label |
| `type` | no | `"text"` | Widget type: `text`, `integer`, `checkbox` |
| `placeholder` | no | `""` | Placeholder text |
| `default` | no | `""` | Default value |

## `[templates]`

### `[templates.formats.{format}]`

Template strings keyed by format (`json`, `kubejs`, `crafttweaker`).
Use `_` for single-variant formats, named keys (`shaped`, `shapeless`) for multi-variant.

Placeholders: `{{ slot_id }}` for filled slot values, `{{ option:key }}` for toolbar options.
Format is inferred from the section name. Override per-placeholder with `| format` syntax.

### `[templates.variant_defaults.{variant}]`

Per-variant default values for options. Applied when the user switches variants.

### `[templates.requirements.{format}]`

Declares a mod dependency for a template format. When the user selects that format
in the recipe builder, the launcher checks whether the required mod is installed.
If missing, a warning banner is shown with optional download links.

| Field | Required | Default | Description |
|---|---|---|---|
| `mod` | yes | — | Mod ID to check in the launcher's mod database |
| `sources` | no | `{}` | Map of source platform → project ID/slug for download links |

The `sources` map is extensible — known platforms include `modrinth` and `curseforge`,
but custom source plugins can register their own key.

```toml
[templates.requirements.kubejs]
mod = "applied-kubejs"
sources.modrinth = "applied-kubejs"
sources.curseforge = "1469724"
```
