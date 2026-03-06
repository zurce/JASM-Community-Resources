---
description: How to add new characters to the Games Assets repository (Genshin, Honkai, ZZZ, WuWa)
---

# Adding New Characters Workflow

This workflow documents the standard operating procedure for properly adding new characters to this repository based on established conventions.

### 1. Identify the Target Game
The repository manages assets for four games, each with its own root directory:
- `Genshin` (Genshin Impact)
- `Honkai` (Honkai: Star Rail)
- `ZZZ` (Zenless Zone Zero)
- `WuWa` (Wuthering Waves)

### 2. Fetch Character Metadata
- Scrape the character's exact stats from reliable sources (e.g., Prydwen.gg for ZZZ/WuWa/Honkai, Fandom Wiki for Genshin or as a fallback).
- Required stats: `ReleaseDate`, `Rarity`, `Element`, `Class`/`Weapon` (Path/Style), and `Region` (World/Faction).
- Formatting: Format `ReleaseDate` strictly as an ISO 8601 string: `YYYY-MM-DDT00:00:00` (e.g., `2026-02-25T00:00:00`).

### 3. Update Supporting JSONs
- **Regions**: Check `<Game>/regions.json` to see if the character's Region/Faction/World already exists. If it does not, append a new entry using `InternalName` and `DisplayName`.
- **Classes**: For Honkai specifically, also check `<Game>/weaponClasses.json` to verify the Path/Class exists.

### 4. Image Processing (`Images/Characters/`)
- Obtain a high-resolution transparent render (e.g., "card" asset from Prydwen or "Portrait" from Wiki).
- Crop the image to a `256x256` "profile picture" format, tightly framing the head and upper torso. Use OpenCV face detection or a bounded top-down smart crop.
- Save the cropped image in `.webp` format directly into the `<Game>/Images/Characters/` folder.

### 5. Update characters.json
Append the new character's block to the bottom of the `<Game>/characters.json` array following this strict schema:
- `Keys`: Array containing the character name and any common aliases (e.g., `["Yao_Guang", "Yaoguang"]`).
- `ReleaseDate`: The formatted ISO string.
- `Image`: **CRITICAL**: Only output the raw filename (e.g., `varka.webp`). **DO NOT** prefix it with `Characters/` or any other path. The app assumes the `Images/Characters/` path implicitly.
- `Rarity`: Integer value (e.g., `4` or `5`).
- `Element`: The character's element as a Title Case string.
- `Class`: The character's weapon type, path, or combat style as a Title Case string.
- `Region`: Array containing the exact matching Region string from `regions.json` (e.g., `["Mondstadt"]`).
- `ModFilesName`: PascalCase internal naming convention.
- `InGameSkins`: Typically `[]` for new entries.
- `InternalName`: Internal string name.
- `DisplayName`: The human-readable name rendered in the UI.

### 6. Git Protocol
- **Permission required**: Never automatically push commits to `master` (or upstream remotes) unless explicitly commanded by the user.
- **Single-use scope**: Every instruction to "push" is treated as a one-time operation for that specific changeset, rather than an open ticket. Do not assume permission carries over to future work.

### 7. Automation Tools (`tools/`)
This repository contains a suite of automation scripts stored in the root `tools/` directory (ignored by git explicitly) built to streamline the character ingestion process:
- **`verify_honkai_missing.py`**: Compares the local Honkai `<Game>/characters.json` inventory against Prydwen's live website to print any missing character names.
- **`parse_zzz_wiki.py` / `scrape_varka.py` / `parse_wiki_nation.py`**: A collection of Fandom Wiki API scrapers built to parse raw HTML and Infoboxes to extract `ReleaseDate`, `Element`, `Rarity`, `Path`, and `Region`.
- **`analyze_zzz_images.py` / `analyze_honkai.py`**: Analysis scripts that scan existing character portraits, map their alpha channels, and determine mathematical averages for image centering and bounding box offsets.
- **`recrop_honkai.py` / `iterative_honkai_crop.py`**: Intelligent "Smart Croppers" that use NumPy alpha-masking to anchor at the exact top of an illustration and pull a square 256x256 bounding box around the character's head and torso.
- **`lbpcascade_animeface.xml`**: An OpenCV HAAR cascade model used to dynamically recognize 2D anime facial geometry, enabling exact tracking of character faces even in chaotic, full-body splash art.
