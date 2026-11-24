# RetroWheel — Custom Skin Engine Specification

## 1. Overview

RetroWheel features an open, customizable hardware skinning architecture that enables users and designers to modify every aesthetic and tactile dimension of the player. Themes are packaged as `.rtskin` bundles (standard ZIP archives containing assets and a JSON manifest).

A companion Web Application (**Skin Studio**, located in `skin_studio/index.html`) provides a real-time 2D/3D hardware simulator and bundle exporter.

---

## 2. Bundle Archive Structure (`.rtskin`)

A valid `.rtskin` bundle is a ZIP archive containing:

```text
my_skin.rtskin (ZIP)
├── skin.json             # Manifest defining color palettes, typography, and asset mappings
├── chassis.png           # Hardware unibody enclosure texture overlay (optional)
├── wheel_face.png        # Rotating anisotropic metal disc texture (optional)
├── center_badge.png      # Center action button emblem (optional)
├── glass_glare.png       # CRT/OLED glass reflection overlay (optional)
└── click.wav             # 16-bit uncompressed mechanical impulse sound (optional)
```

---

## 3. Schema: `skin.json`

```json
{
  "id": "watch_dogs_dedsec",
  "name": "Watch Dogs // DedSec",
  "author": "RetroWheel Community",
  "version": 1,
  "colors": {
    "background": "#0A0D10",
    "surface": "#12171E",
    "primary": "#00FF66",
    "text": "#E2E8F0",
    "textSecondary": "#64748B",
    "divider": "#1E293B",
    "accent": "#00FF66"
  },
  "wheel": {
    "ringBase": "#0D1117",
    "ringHighlight": "#1F2937",
    "ringShadow": "#05070A",
    "centerBase": "#0F141C",
    "centerHighlight": "#1E293B",
    "centerShadow": "#080B0F",
    "labelColor": "#00FF66",
    "accentColor": "#00FF66"
  },
  "textures": {
    "chassis": "chassis.png",
    "wheelFace": "wheel_face.png",
    "centerBadge": "center_badge.png",
    "glassGlare": "glass_glare.png"
  },
  "acoustics": {
    "clickSound": "click.wav"
  }
}
```

---

## 4. Safety & Memory Constraints

Located in `app/src/main/java/com/retrowheel/player/data/skin/CustomSkinManager.kt`:

1. **Zip-Slip Traversal Prevention**:
   Every unzipped file path is verified against the canonical root directory of the application's internal skin cache (`/data/user/0/com.retrowheel.player/files/skins/`). Any entry containing `../` or targeting external directories throws an immediate `SecurityException`.

2. **Strict Texture Downsampling**:
   High-resolution user-provided textures (e.g., 4K or 8K smartphone camera photos) are downsampled on import using Android's `BitmapFactory.Options` with an upper bound constraint:
   $$\max(\text{width}, \text{height}) \le 1440\text{ px}$$
   This guarantees that custom skins never exceed device heap budgets or cause Out-Of-Memory (OOM) errors.

3. **Hot-Reloading**:
   Switching custom skins updates Compose state flows immediately without requiring an application restart. Custom rotating wheel textures are mapped directly to hardware GPU RenderNodes (`graphicsLayer`).
