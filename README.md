# xDupe

**xDupe** is a lightweight, open-source Paper plugin that brings back the classic piston-item frame duplication mechanic for anarchy servers. Originally patched by both Mojang and Paper, this mechanic is carefully re-implemented to bypass modern anti-dupe protections while keeping full NBT integrity. Fully configurable via both file and in-game commands.

- **Author:** EMIRLQQ1
- **Language:** Java 17+
- **Platform:** Paper 1.20 – 1.21.x (Spigot API)
- **License:** MIT

---

## Table of Contents

- [Features](#features)
- [How It Works](#how-it-works)
- [Installation](#installation)
- [Configuration](#configuration)
- [Commands](#commands)
- [Permissions](#permissions)
- [Building from Source](#building-from-source)
- [Supported Versions](#supported-versions)
- [Troubleshooting](#troubleshooting)
- [FAQ](#faq)
- [Notes](#notes)
- [License](#license)

---

## Features

- **Classic Dupe Mechanic** – Piston extends into an item frame → item is duplicated. Works exactly like the pre-1.20 vanilla behavior, but better.
- **Glow Item Frame Support** – Fully compatible with both normal item frames and glow item frames.
- **Full NBT Preservation** – Enchantments, custom names, lore, attributes, potion effects, and all other NBT data are preserved on every duplicated item.
- **Configurable Chance** – Set the duplication success rate from 0% to 100%. When the chance fails, the item frame breaks naturally as if nothing happened.
- **Configurable Amount** – Control how many extra copies are dropped. Set to 0 for no duplication at all, or set high values for extreme duplication.
- **Configurable Frame Drop** – Choose whether the item frame itself drops as an item when broken.
- **In-Game Configuration** – Change all settings on the fly with simple commands. No server restarts required.
- **Tab Completion** – Full tab completion support for every command and value.
- **Anti-Anti-Dupe** – Runs at `EventPriority.LOWEST` and removes the item frame instantly to prevent anti-cheat plugins from interfering.
- **Performance Optimized** – Uses `getNearbyEntities` with bounding box filtering instead of scanning the entire world. Minimal overhead.
- **Adventure API** – Built with Paper's modern Adventure API for reliable cross-version chat formatting.

---

## How It Works

```
                         ┌─────────────────────┐
                         │   Piston Extends     │
                         └──────────┬──────────┘
                                    │
                         ┌──────────v──────────┐
                         │  Item Frame Found   │
                         │  in front of piston │
                         └──────────┬──────────┘
                                    │
                         ┌──────────v──────────┐
                         │  Frame has item?    │
                         └──────┬──────────┬───┘
                                │          │
                              yes         no
                                │          │
                     ┌──────────v──┐       │
                     │  Chance     │       │
                     │  Check      │       │
                     └──────┬──────┘       │
                            │              │
                    ┌───────┴───────┐      │
                    │               │      │
                success          failed    │
                    │               │      │
            ┌───────v───────┐      │      │
            │ Drop original  │      │      │
            │ + extra copies │      │      │
            │ (+ frame if    │      │      │
            │  enabled)      │      │      │
            └───────┬───────┘      │      │
                    │              │      │
            ┌───────v───────┐      │      │
            │ Remove frame  │      │      │
            │ from world    │      │      │
            └───────┬───────┘      │      │
                    │              │      │
                    └──────┬───────┘      │
                           │              │
                    ┌──────v──────┐       │
                    │   Done     │       │
                    └────────────┘       │
                              ┌──────────v──────────┐
                              │  Natural Break      │
                              │  (vanilla behavior) │
                              └─────────────────────┘
```

### Technical Details

When a piston extends, the server fires a `BlockPistonExtendEvent`. xDupe listens at `EventPriority.LOWEST` — before any anti-cheat or anti-dupe plugin has a chance to cancel the event. At this point:

1. The plugin calculates the block position the piston head will occupy.
2. It searches for item frames at that exact block location using a bounding box query (`getNearbyEntities` with 0.75 radius).
3. If an item frame with an item is found, the configured chance is rolled.
4. On success: the item is cloned with `ItemStack.clone()`, which copies all NBT data including enchantments, custom names, lore, and attributes.
5. The original item, plus the configured number of duplicates, are dropped at the frame's location using `dropItemNaturally`.
6. If `drop-frame` is enabled, the item frame itself is also dropped.
7. The frame entity is immediately removed from the world with `remove()`, preventing any other plugin from interfering or creating additional drops.
8. On failure: the plugin does nothing, and the frame breaks naturally with vanilla behavior.

---

## Installation

### Step 1: Download

Download the latest `xDupe.jar` from the [Releases](https://github.com/EMIRLQQ1/xDupe/releases) page.

### Step 2: Install

Place the `xDupe.jar` file into your server's `plugins/` directory.

```
server/
├── plugins/
│   ├── xDupe.jar          ← place here
│   ├── ...
│   └── ...
├── world/
├── world_nether/
├── world_the_end/
├── server.properties
├── ...
```

### Step 3: Start or Reload

Restart your server completely, or if you're on Paper 1.20+, use:

```
/reload confirm
```

### Step 4: Verify

Run this command in-game or from the console:

```
/xdupe
```

You should see the xDupe status screen with default configuration values.

### Requirements

| Dependency | Minimum Version |
|---|---|
| **Server** | Paper 1.20 (or any Paper-compatible fork) |
| **Java** | Java 17 (JDK 17 or higher) |

---

## Configuration

On first run, xDupe creates `plugins/xDupe/config.yml` automatically.

### Default Configuration

```yaml
chance: 100
duplicate-amount: 1
drop-frame: true
```

### Options Reference

| Option | Type | Default | Range | Description |
|---|---|---|---|---|
| `chance` | `int` | `100` | 0 – 100 | The percentage chance that the duplication will trigger. At 100, it always triggers. At 0, it never triggers. |
| `duplicate-amount` | `int` | `1` | 0 – ∞ | How many EXTRA copies are dropped on top of the original item. Total items dropped = 1 (original) + duplicate-amount. |
| `drop-frame` | `boolean` | `true` | true / false | Whether the item frame itself drops as an item when the dupe triggers. |

### Examples

#### Fair Anarchy (50% chance, 1 extra copy)
```yaml
chance: 50
duplicate-amount: 1
drop-frame: true
```

#### Extreme Overpowered (100% chance, 10 extra copies)
```yaml
chance: 100
duplicate-amount: 10
drop-frame: false
```

#### Disabled (duplication off)
```yaml
chance: 0
duplicate-amount: 0
drop-frame: false
```

#### Vanilla-Like (100% chance, no extra, but frame drops)
```yaml
chance: 100
duplicate-amount: 0
drop-frame: true
```

---

## Commands

| Command | Description | Permission |
|---|---|---|
| `/xdupe` | Display help menu and current configuration | `xdupe.admin` |
| `/xdupe help` | Display help menu and current configuration | `xdupe.admin` |
| `/xdupe chance [value]` | View or set the duplication chance (0–100) | `xdupe.admin` |
| `/xdupe amount [value]` | View or set the number of extra copies (≥0) | `xdupe.admin` |
| `/xdupe dropframe [true\|false]` | View or set whether the frame drops | `xdupe.admin` |
| `/xdupe reload` | Reload configuration from `config.yml` | `xdupe.admin` |

**Alias:** `/dupe`

### Usage Examples

```
/xdupe
    ── shows current settings and all available commands

/xdupe chance
    ── shows current chance value (e.g., "current chance: 100%")

/xdupe chance 50
    ── sets chance to 50%

/xdupe amount 3
    ── sets duplicate-amount to 3 (total 4 items per dupe)

/xdupe dropframe false
    ── disables frame dropping

/xdupe reload
    ── reloads config.yml from disk
```

> **Note:** All commands have tab completion. Start typing and press Tab to see suggestions.

---

## Permissions

| Permission | Default | Description |
|---|---|---|
| `xdupe.admin` | `op` | Allows using all `/xdupe` commands |

Example for granting to a non-op player:

```
/manadd PlayerName xdupe.admin
```

---

## Building from Source

### Prerequisites

- Java 17 or higher (JDK)
- Apache Maven 3.8+


### Importing into IntelliJ IDEA

1. Open IntelliJ IDEA
2. Click **File → Open**
3. Select the `xDupe` directory
4. IntelliJ will automatically detect the Maven project and import dependencies
5. Use the Maven sidebar to run **Package** or run `mvn clean package` in the terminal

---

## Supported Versions

| Minecraft Version | Paper Support | Status |
|---|---|---|
| 1.20 | ✅ | Fully tested |
| 1.20.1 | ✅ | Fully tested |
| 1.20.2 | ✅ | Fully tested |
| 1.20.3 | ✅ | Fully tested |
| 1.20.4 | ✅ | Fully tested |
| 1.20.5 | ✅ | Compatible |
| 1.20.6 | ✅ | Compatible |
| 1.21 | ✅ | Compatible |
| 1.21.1 | ✅ | Compatible |
| 1.21.2+ | ✅ | Should work (API stable) |

Any Paper-compatible fork (Purpur, Pufferfish, etc.) should work as well.

---

## Troubleshooting

### Plugin doesn't load

- Make sure you're using **Paper**, not CraftBukkit or Spigot (xDupe uses Paper-specific APIs).
- Check that you have **Java 17 or higher** installed.
- Verify that the JAR file is not corrupted by checking its size (should be ~5-10 KB).
- Check the server console for any error messages.

### Dupe isn't working

- Confirm that `chance` is set to a value greater than 0.
- Make sure the item frame actually contains an item (right-click the frame with an item).
- Check that the piston is facing directly toward the item frame — the piston head must extend into the exact block where the frame is located.
- Try setting `chance: 100` and `drop-frame: true` for testing.
- If another plugin is interfering, try disabling other anti-dupe plugins temporarily.

### Command not found

- Verify that the plugin loaded successfully: `/plugins` should show `xDupe` in the list.
- If using `/dupe` alias and it doesn't work, try `/xdupe` directly.
- Make sure you have the `xdupe.admin` permission.

### Items lose enchantments or data

- This should not happen — xDupe uses `ItemStack.clone()` which preserves all NBT data.
- If you're experiencing data loss, it may be caused by another plugin modifying items after they drop.

---

## FAQ

**Q: Is this bannable?**  
A: On anarchy servers, no. That's what this plugin is made for. On vanilla or survival servers, duplication is typically against the rules.

**Q: Does this work with Spigot?**  
A: It should work on Spigot too, but Paper is recommended. The plugin uses Paper's `getNearbyEntities` with a predicate filter, which is available on Spigot as well. However, the anti-dupe bypass is specifically designed for Paper's built-in protections.

**Q: Can I use this with other dupe plugins?**  
A: Yes, but be aware they may conflict. If using multiple dupe plugins, test them together first.

**Q: Does this dupe shulker boxes with their contents?**  
A: Yes! Since `ItemStack.clone()` preserves all NBT data, shulker box contents, written books, enchanted items, and any other NBT data are all fully duplicated.

**Q: How do I reset the config to default?**  
A: Delete `plugins/xDupe/config.yml` and run `/xdupe reload` or restart the server. A fresh config will be generated.

**Q: Will items despawn?**  
A: Duplicated items are regular dropped items and follow standard Minecraft despawn timers (5 minutes by default). This is not controlled by xDupe.

---

## Notes

- **Purpose:** This plugin deliberately recreates a duplication mechanic that was patched by both Mojang and Paper. It is intended **only** for anarchy / chaos servers where duplication is part of the gameplay.
- **Anti-Dupe Bypass:** The plugin works by running at `EventPriority.LOWEST` and removing the item frame entity before any anti-cheat plugin can react. This is a bypass, not a fix — it's designed to circumvent protections.
- **Performance:** xDupe uses bounded entity queries (`getNearbyEntities` with a 0.75-block radius) rather than scanning all loaded entities. This means near-zero performance impact during normal gameplay. The only cost is when a piston actually extends near an item frame.
- **Server Logging:** The plugin does not log individual dupe events to avoid filling the console. Only enable/disable messages are logged.
- **Modification:** This is open source under the MIT license. Feel free to modify, redistribute, and use as you wish.

---

## License

```
MIT License

Copyright (c) 2026 EMIRLQQ1

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

**xDupe** — Made with ❤️ by EMIRLQQ1
