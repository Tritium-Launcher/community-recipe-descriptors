# community-recipe-descriptors

TOML recipe type descriptors for mods that don't implement the Tritium Mod API.
Used by [Tritium Launcher](https://github.com/Tritium-Launcher)'s Recipe Builder
to provide recipe support without requiring the full Tritium Mod API.

## Adding a mod

Create a directory under `mods/{namespace}/` and add a `descriptor.toml` file.

### Structure

```
mods/
  {ns}/
    descriptor.toml
  {ns}/
    21.4/descriptor.toml
  {ns}/
    21-26.2/descriptor.toml
```

Each mod directory contains either:

| Layout | When to use |
|---|---|
| `mods/{ns}/descriptor.toml` | Recipe structure never changes between updates |
| `mods/{ns}/{version}/descriptor.toml` | Single MC version |
| `mods/{ns}/{from}-{to}/descriptor.toml` | Version range |

### Index

After adding a mod, register it in `mods/index.json`:

```json
{
  "{ns}": [],
  "{ns}": ["21.4"],
  "{ns}": ["21-26.2"]
}
```

- Array of version strings. Each is a single version (e.g. `"21.4"`) or a range (e.g. `"21-26.2"`)
- Empty array `[]` means the descriptor works for **all** MC versions
- Supports multiple + mixed entries

### Descriptor format example

```toml
[recipe]
id = "ae2:charger"
display_name = "Charger"
width = 100
height = 44
catalysts = ["ae2:charger"]

[[slot]]
id = "input"
role = "input"
slot_type = "ITEM"
x = 10
y = 14
w = 16
h = 16
max_capacity = 1

[[slot]]
id = "output"
role = "output"
slot_type = "ITEM"
x = 70
y = 14
w = 16
h = 16
max_capacity = 1

[templates]

[templates.formats.json]
_ = """
{
  "type": "ae2:charger",
  "ingredient": {{ input }},
  "result": {{ output }}
}
"""

[templates.formats.kubejs]
_ = """
event.recipes.ae2.charger(
  {{ output }},
  {{ input }}
)
"""

[templates.requirements.kubejs]
mod = "applied-kubejs"
sources.modrinth = "applied-kubejs"
sources.curseforge = "1469724"

```

In this example, we make a descriptor for Applied Energistics' Charger recipe type. 

See [SCHEMA.md](SCHEMA.md) for the full field reference.

### Validation

Before submitting a PR:

```bash
taplo validate
```

A JSON Schema (`recipe-descriptor.schema.json`) is checked into the repo root.
GitHub Actions CI runs `taplo validate` on every PR. The config in `taplo.toml`
associates the schema with all descriptor files automatically.

### Template placeholders

| Placeholder | Description |
|---|---|
| `{{ id }}` | Filled slot value for slot `id` |
| `{{ option:key }}` | Option value from the toolbar widget |

Format is inferred from the template section name (`formats.json` → JSON, `formats.kubejs` → KubeJS).
Override with pipe syntax: `{{ slot_id | kubejs }}` for mixed-format templates.

### Custom types

For mods with custom value types (gas, mana, etc.) that can't be represented as items or fluids,
define the slot as `display_only = true` on the sprite and add `[[option]]` fields for the values:

```toml
[[slot]]
id = "gas"
role = "custom"
label = "Chemical"
x = 70
y = 30
w = 16
h = 16
display_only = true

[[option]]
key = "gasType"
label = "Gas"
type = "text"
placeholder = "mekanism:hydrogen"

[[option]]
key = "gasAmount"
label = "Amount"
type = "integer"
default = "1"

[templates.formats.kubejs]
_ = """
event.recipes.mekanism.separating(
  {{ fluid_input }},
  MekType.Gas.of('{{option:gasType}}', {{option:gasAmount}})
)
"""
```

The template wraps the option values in whatever function call the codegen format requires.

### Conventions

- **IDs** — always prefixed with the mod namespace: `thermal:smelter`
- **Catalysts** — objects which uses this recipe type
- **Options** — use `integer` for RF/energy values, `text` for string parameters
