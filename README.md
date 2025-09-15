# CRPG Isometric Pipeline — Unity Starter

This project scaffolds a **local-first, camera-locked, pre-rendered isometric** content pipeline for Unity.

---

## 0) Prereqs (install once)

- **Unity Hub** + **Unity LTS** (choose latest LTS). Create project as **2D (URP optional)**.
- **IDE (pick one):**
  - **Visual Studio 2022 Community** with the *Game development with Unity* workload, or
  - **JetBrains Rider** (Unity plugin auto-installed), or
  - **VS Code** + *C# Dev Kit* + *Unity* extensions
- **Git** + **Git LFS** (https://git-scm.com / https://git-lfs.com)
- **GitHub** account (or GitLab/Bitbucket—your choice)

---

## 1) Create the repo (once)

```bash
# inside your dev folder
git clone <your-empty-remote> crpg-unity
cd crpg-unity

# drop this starter in (or unzip the provided zip at the repo root)
# then initialize LFS
git lfs install
git add .gitattributes .gitignore
git commit -m "chore: repo bootstrap (gitignore + gitattributes)"
git push -u origin main
```

> If you already created the Unity project via Hub, copy these folders/files into your project root. Otherwise, you can start the project **in this repo** and open it from Unity Hub.

---

## 2) Unity project settings (do this first)

In **Edit → Project Settings**:
- **Editor**
  - *Asset Serialization*: **Force Text**
  - *Version Control*: **Visible Meta Files**
- **Player**
  - Company/Product name to your preference
- **Graphics**
  - If using URP, ensure the URP pipeline asset is assigned

In **Edit → Preferences → External Tools**:
- Set your **External Script Editor** (VS/Rider/VS Code)

Create/open a scene in `Assets/Scenes` (e.g., `Main.unity`):
- Main Camera: **Projection = Orthographic**
- **Size**: start with ~8–12 (tune later)
- **PPU (Pixels Per Unit)** convention: **128 px = 1 tile = 1 Unity unit**
- Add `GameObject → 2D → Tilemap` (Grid + Tilemap), set **Cell Size = (1,1)**

---

## 3) Folder layout (already created)

```
Assets/
  Scenes/
  Sprites/
  Tilemaps/
  Prefabs/
  ScriptableObjects/
  Importer/         <-- Editor scripts will live here
  Generated/        <-- Drop AI outputs here (PNG + JSON sidecar)
Prompts/
  _globals.json     <-- Global camera/style defaults
  scenes/
  props/
  npcs/
Docs/
  StyleLock/        <-- 10–20 reference images you re-use in prompts
```

---

## 4) Prompt + metadata (local-first)

Place a **sidecar JSON** next to each generated image in `Assets/Generated/`:

```json
{
  "id": "prop.cart.v1",
  "type": "prop",
  "prompt_resolved": "FULL PROMPT WITH GLOBALS APPLIED...",
  "prompt_hash": "sha256:...",
  "image_hash": "sha256:...",
  "seed": 265726981,
  "tile_size": { "x": 2, "y": 2 },
  "pivot": { "x": 0.5, "y": 0.9 },
  "grid_dims": { "x": 24, "y": 14 },
  "tags": ["market","wood","damaged"]
}
```

**Why:** Git becomes your **source of truth** for the art’s recipe (prompt/seed) and in-engine configuration.

---

## 5) Branching & commits

- Branch model:
  - `main` (stable), `dev` (integration), `feature/<short-name>` for tasks
- Conventional commits:
  - `feat: importer creates prefabs from props`
  - `fix: correct pivot for 2x2 props`
  - `chore: add LFS patterns for .psd`

---

## 6) Next steps (what we'll implement next)

- `AssetMeta` ScriptableObject (per-asset metadata in-engine)
- `CameraLockConfig` ScriptableObject (PPU, tile size, pivots, grid dims)
- **Editor importer** (reads `/Assets/Generated/*.png/.json`, creates Sprites/Prefabs, writes AssetMeta)
- A sample Tilemap scene with orthographic camera configured

---

## 7) Tips

- Keep **StyleLock** references consistent (same camera). Use 2–3 refs per generation.
- Always **append** the camera/style lock block to prompts.
- Prefer **transparent PNGs** for props/NPCs; environment bases as full BG.
- Use **Git LFS** for all binary art to keep the repo fast.

