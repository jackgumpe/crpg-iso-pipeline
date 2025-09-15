# CRPG Isometric Prompt Pipeline — v0.1

**Goal:** Deterministic, reusable prompts that output art assets we can drop into Unity with minimal hand‑fixing.  
**Why:** Consistency = fewer art seams, faster iteration, and clean diffs for code review.

---

## 1) Principles (What & Why)
- **Consistency over flair.** A locked camera/style prevents drift across scenes; we get a coherent world.
- **Modularity.** Background “plates” + prop/character overlays allow hot‑swaps without re‑rendering plates.
- **Traceability.** Every asset carries a prompt JSON + seed so we can reproduce/repair it later.
- **Reversibility.** Nothing is final until it passes the checklist; we can roll back by seed/version.
- **Source control friendly.** Force Text & meta files; binaries in LFS; predictable filenames.

---

## 2) Asset Taxonomy
- **BG (Background Plate):** Wide, parallax‑friendly isometric backdrop. No characters. Delivered at 4096×3072 PNG.
- **PROP:** Foreground/midground, transparent PNG with clean alpha, 256–2048 px on the long edge (scene‑dependent).
- **CHAR:** Character/NPC sprites or pose plates, transparent PNG.
- **OVERLAY/VFX:** Fog, god‑rays, particles, color‑grading masks, transparent PNG.
- **MASKS:** Optional matte layers to assist compositing in Unity (e.g., occlusion masks).

---

## 3) House Style (“Torment‑like” Camera) — token: `CAMERA_TON_ISO`
**What:** 3/4 isometric look, ~45° yaw, ~35° pitch, compressed perspective (orthographic‑like), long shadows at oblique angles.  
**Why:** Reads like a classic CRPG, avoids extreme perspective distortion, keeps scale consistent.

**Prompt token expands to:**
- View: isometric / three‑quarter / elevated
- Yaw: 45°, Pitch: 35° (±3° tolerance)
- Lens: orthographic *or* narrow perspective mimicking ortho (≈ 80–120 mm equivalent; minimal convergence)
- Horizon control: low‑to‑mid frame; single dominant vanishing trend only
- Keep verticals near‑parallel; avoid wide‑angle distortion

> In Unity we’ll use **orthographic** for gameplay; the prompt token just enforces the look from the generator.

---

## 4) Lighting Sets (tokens)
- `LIGHT_DAWN_SOFT` — 5000–6000K, soft blue‑peach rim, long cool shadows, low haze.
- `LIGHT_NOON_NEUTRAL` — 6500K, short shadows, neutral contrast, minimal haze.
- `LIGHT_DUSK_AMBER` — 3500–4500K, warm key with cool fill, long dramatic shadows.
- `LIGHT_NIGHT_MOON` — 7000–8000K, high contrast moonlight + sparse warm practicals.
- `FOG_LOW`, `FOG_MED`, `FOG_HIGH` — atmospheric depth for parallax reads.

**Why:** Named sets allow rapid swapping *without* rewriting the whole prompt; color scripts stay consistent.

---

## 5) Deliverable Specs
- **BG:** PNG, 4096×3072 (or 4096×4096 square when needed), sRGB, no premultiplication, no embedded color‑grading.
- **PROP/CHAR/VFX:** PNG, alpha straight, edge‑aware (no dark halos), max 2048 on long edge unless specified.
- **Naming:** `{type}_{sceneSlug}_{shot}_{tokenSet}_{v}_{seed}.png`
  - Example: `BG_blackEarthEntrance_A01_CAMERA_TON_ISO_LIGHT_DUSK_AMBER_v01_seed4312.png`
- **Metadata:** Commit the prompt JSON alongside the asset (same name + `.prompt.json`).

**Why:** Reproducibility and diff‑ability. QA can re‑render by seed/version, and Unity import stays predictable.

---

## 6) JSON Prompt Schema (see `Assets/PromptPipeline/Templates/schema.prompt.json`)
Key sections:
- `meta` — id, version, author, createdAt, seed, assetType (BG/PROP/CHAR/VFX).
- `style` — model, styleTokens (e.g., `CAMERA_TON_ISO`, `LIGHT_DUSK_AMBER`), palette, mood.
- `scene` — setting, focalElements, geometry cues, material cues.
- `composition` — camera block (yaw/pitch/lens), framing, depth/foreground, rule‑of‑thirds, negative space.
- `tech` — resolution, format, sampler/steps/guidance, tiling flags, inpaint/outpaint bounds.
- `neg` — explicit “do‑not‑want” elements (e.g., modern garbage, cars, signage, UI text).
- `post` — mild sharpening/denoise policies; grade = none (grading happens in‑engine if needed).
- `outputs` — target filenames, paths, LFS hints.

**Why:** We separate art direction (`style`, `scene`, `composition`) from reproducibility knobs (`tech`, `seed`).

---

## 7) Review Checklist (pre‑commit)
1. **Look:** Camera matches `CAMERA_TON_ISO`; horizon & verticals clean.
2. **Scale:** Doors, stairs, props obey world scale (tell: character proxy fits).
3. **Lighting:** Token matches; shadow direction consistent; no broken contact shadows.
4. **Edges:** No halos on alpha; no double‑bakes; no text baked into art.
5. **Noise:** No AI artifacts (warped symbols, melting geometry); fix or iterate.
6. **Filename & metadata:** Naming pattern + `.prompt.json` present.
7. **Unity import:** sRGB on; compression OK; background plate not accidentally mipmapped if used as UI.

---

## 8) Workflow (Background example)
1. Duplicate `examples/scene_background.prompt.json` → fill `scene` details, choose tokens, set `seed`.
2. Generate the BG image → save as per naming rule + export `.prompt.json` next to it.
3. Add to repo under `Assets/Art/Backgrounds/` (tracked via LFS) → `git add` → PR.
4. In Unity: import plate, set Texture Type **Default**, sRGB **on**, MipMaps **off** if used as UI; create Scene prefab.
5. Review with checklist; iterate by bumping `v` or changing `seed` (record both).

---

## 9) Roadmap
- v0.2: Unity Editor tooling to auto‑import with defaults and generate Scene prefab stubs.
- v0.3: Token pack for materials (obsidian/ash/sandstone) and biome kits.
- v0.4: Prop library & hot‑swap harness (addressable variants).
