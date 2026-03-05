# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Project Zomboid mod ("Murcia Pack Beta", id: `MurciaPackBeta`) targeting Build 42. Adds Murcia-region themed food items and cooking recipes. Currently in development/beta. No build system, no dependencies, no automated tests.

## Mod Directory Layout (Build 42 format)

```
Murcia Pack Beta/
├── common/                                          # Required by B42 loader (empty, .gitkeep)
└── 42/
    ├── mod.info                                     # Mod metadata (has versionMin=42)
    ├── poster.png                                   # Steam Workshop poster
    └── media/
        ├── scripts/
        │   └── MurcianPackModule.txt                # Items, models, craftRecipes (module Base)
        ├── textures/
        │   ├── Item_MurcianCrisps.png               # Inventory icon
        │   └── WorldItems/                          # 3D world model textures
        │       ├── MurcianChips.png
        │       ├── ZarangolloPan.png
        │       └── ZarangolloPanRotten.png
        └── lua/shared/Translate/
            ├── EN/
            │   ├── ItemName_EN.txt                  # English item name overrides
            │   └── Recipes_EN.txt                   # English recipe names
            └── ES/
                ├── ItemName_ES.txt                  # Spanish item name overrides
                └── Recipes_ES.txt                   # Spanish recipe names
```

The `common/` directory must exist (even if empty) for Build 42 to detect the mod. Version-specific assets and metadata go inside `42/`.

## Script Format (MurcianPackModule.txt)

Everything is defined inside a single `module Base { }` block. The file contains three types of blocks:

### model blocks
Define 3D world representations. Reference base game meshes with custom textures:
```
model ModelName { mesh = WorldItems/BaseMesh, texture = WorldItems/CustomTexture, scale = 0.4, }
```

### item blocks
Define food items with `property = value,` syntax. Key food properties used in this mod:
- Nutritional: `Calories`, `Carbohydrates`, `Proteins`, `Lipids`, `HungerChange`, `ThirstChange`, `UnhappyChange`
- Perishable food: `DaysFresh`, `DaysTotallyRotten`, `IsCookable`, `MinutesToCook`, `MinutesToBurn`, `DangerousUncooked`
- Non-perishable: `Packaged = TRUE`, `CantBeFrozen = TRUE` (no DaysFresh/DaysTotallyRotten needed)
- Display: `DisplayName`, `DisplayCategory = Food`, `Icon`, `WorldStaticModel`
- Behavior: `ReplaceOnUse` (returns item when eaten, e.g., Pan), `GoodHot`, `BadInMicrowave`, `CookingSound`, `CustomEatSound`

### craftRecipe blocks (Build 42 format)
Replaced the old Build 41 `recipe` blocks. Use CamelCase IDs with no spaces:
```
craftRecipe RecipeID {
    time = <ticks>,
    category = Cooking,
    xpAward = <Skill>:<amount>,
    inputs {
        item <count> [Base.ItemName],                    # consume specific item
        item <count> [Base.A;Base.B;Base.C],             # consume one of alternatives
        item <count> tags[TagName],                      # consume by tag (e.g., Egg)
        item <count> [Base.ItemName] mode:keep,          # use but don't consume (tool)
    }
    outputs {
        item <count> Base.ResultItem,
    }
}
```

## Current Mod Content

### Items
| Item ID | Type | Description |
|---|---|---|
| `MurcianCrisps` | Non-perishable snack | Chips with lemon and pepper. 725 cal. Uses base game Crisps mesh. |
| `ZarangolloPan` | Perishable cookable | Zarangollo (murcian egg+zucchini dish). 280 cal, 3d fresh / 5d rotten. Cookable in pan (30 min cook, 50 min burn). Returns Pan on eat. |

### Recipes
| Recipe ID | Inputs | Output | XP |
|---|---|---|---|
| `MakeChipsWithLemonAndPepper` | 1x any Crisps variant + 1x Pepper (kept) + 1x Lemon | MurcianCrisps | Cooking:3 |
| `MakeZarangollo` | 1x Pan + 2x any Egg + 3x Zucchini + 1x Onion | ZarangolloPan | Cooking:10 |
| `EmptyZarangolloPan` | 1x ZarangolloPan (allows rotten) | Pan | none |

### Models
| Model ID | Base mesh | Custom texture |
|---|---|---|
| `MurcianCrispsModel` | `WorldItems/Chips` (scale 0.28) | `WorldItems/MurcianChips` |
| `ZarangolloPanModel_Ground` | `WorldItems/PanFriedVegetables` (scale 0.4) | `WorldItems/ZarangolloPan` |
| `ZarangolloPanModel_GroundRotten` | `WorldItems/PanFriedVegetables` (scale 0.4) | `WorldItems/ZarangolloPanRotten` |

## Translation Key Patterns

- Item names: `ItemName_Base.<ItemID>` in `ItemName_{lang}.txt` files (key prefix matches `module Base`)
- Recipe names: `Recipe_<CraftRecipeID>` in `Recipes_{lang}.txt` files (CamelCase, matching the craftRecipe block ID)
- All translation files must be UTF-8 encoded
- Both EN and ES translations must be kept in sync

## Adding a New Item

1. In `MurcianPackModule.txt` inside the `module Base {}` block:
   - Add a `model` block if the item needs a custom world appearance
   - Add an `item` block with all food/display properties
   - Add a `craftRecipe` block with inputs/outputs
2. In `EN/ItemName_EN.txt`: add `ItemName_Base.<NewItemID> = "English Name",`
3. In `ES/ItemName_ES.txt`: add `ItemName_Base.<NewItemID> = "Spanish Name",`
4. In `EN/Recipes_EN.txt`: add `Recipe_<NewRecipeID> = "English Recipe Name",`
5. In `ES/Recipes_ES.txt`: add `Recipe_<NewRecipeID> = "Spanish Recipe Name",`
6. Place icon PNG in `42/media/textures/` and world texture PNGs in `42/media/textures/WorldItems/`

## Testing

No automated tests. Manual testing only:
1. Copy or symlink the `Murcia Pack Beta/` folder into the PZ mods directory (`~/Zomboid/mods/` on Linux/Mac, `%USERPROFILE%\Zomboid\mods\` on Windows)
2. Launch Project Zomboid (Build 42), enable the mod in the mod manager
3. Verify items appear in-game and recipes work in the crafting menu

## Modding References

- [PZ Wiki Modding](https://pzwiki.net/wiki/Modding)
- [PZ Wiki craftRecipe](https://pzwiki.net/wiki/CraftRecipe)
- [PZ Wiki Mod Structure](https://pzwiki.net/wiki/Mod_structure)
- [Build 42 Mod Template](https://github.com/LabX1/ProjectZomboid-Build42-ModTemplate)
