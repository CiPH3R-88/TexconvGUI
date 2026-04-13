# TexconvGUI

### A simple GUI for Microsoft's Texconv texture converter

---

## What is this?

**TexconvGUI** is a graphical interface for [Texconv](https://github.com/microsoft/DirectXTex/wiki/Texconv) — a powerful command-line texture conversion tool made by Microsoft.

Instead of typing complex commands, you just pick your files, choose your settings, and click Convert. The app builds and runs the command for you — and shows you exactly what command it's running in real time.

It supports converting image textures between formats like **DDS, PNG, JPG, WebP, BMP, TGA, TIF and HDR**.


---

## How to Use

### Step 1 — Choose a Mode

| Mode             | Use when                                      |
| ---------------- | --------------------------------------------- |
| **Single File**  | Converting one image at a time                |
| **Batch Folder** | Converting all images inside a folder at once |

---

### Step 2 — Select Input

**Single mode** → Click Browse and pick your image file

- Supported input formats: `PNG`, `JPG`, `DDS`, `WebP`
- The app will auto-detect and display the format

**Batch mode** → Click Browse and pick a folder

- Toggle **Include Subfolders** ON to also convert files inside nested folders
- The app scans and shows a summary of what was found (e.g. `12× PNG, 4× DDS`)
- Mixed formats in the same folder are fully supported

---

### Step 3 — Configure Settings

#### Resize Scale

A slider with 7 steps from `0.125x` to `8x`. Center position `1x` means no resize.

- Moving **left** scales down — `0.5x` makes a 1024×1024 into 512×512
- Moving **right** scales up — `2x` makes a 512×512 into 1024×1024
- Each image in batch mode is scaled based on its own dimensions — aspect ratio is always preserved
- Enable **Fit to Power-of-2** alongside the slider to also snap the result to the nearest power-of-2 value

#### Mip Levels

Only applies to **DDS output**

| Option        | What it does                                                 |
| ------------- | ------------------------------------------------------------ |
| Keep Existing | Preserves whatever mip levels are already in the source file |
| All Mipmaps   | Generates a full mip chain (recommended for game use)        |
| No Mipmaps    | Outputs just the base image, no mip chain                    |
| Custom        | Enter a specific mip count                                   |

#### Filter / Resize Quality

Only matters when the image is being resized

- `LINEAR` — default, smooth, works well for most textures
- `POINT` — sharp, no blending, good for pixel art or when you want zero interpolation

#### Format / Compression

Only applies to **DDS output**

| Format             | Best for                             |
| ------------------ | ------------------------------------ |
| Keep Source Format | No re-compression, output as-is      |
| BC1 / DXT1         | Opaque textures — smallest file size |
| BC2 / DXT3         | Textures with sharp transparency     |
| BC3 / DXT5         | Textures with smooth transparency    |

#### Alpha Handling

| Option                   | When to use                                      |
| ------------------------ | ------------------------------------------------ |
| Default                  | Leave alpha as-is — works for most cases         |
| Premultiplied (-pmalpha) | Use when converting DDS → PNG for correct colors |
| Straight (-alpha)        | Undo premultiply from source                     |
| Separate (-sepalpha)     | Alpha channel doesn't contain transparency info  |

#### Color Space

| Option                    | When to use                                                         |
| ------------------------- | ------------------------------------------------------------------- |
| None                      | Default — no color space change                                     |
| sRGB Input only (-srgbi)  | Source is sRGB encoded — use for DDS → PNG to fix washed-out output |
| sRGB Both (-srgb)         | Both input and output are sRGB                                      |
| sRGB Output only (-srgbo) | Only write sRGB tag to output                                       |

#### Output File Type

Choose the format you want the converted file saved as. DDS is the default.

| Format | Best used for                  |
| ------ | ------------------------------ |
| DDS    | Game engines, DirectX textures |
| PNG    | Lossless editing, modding      |
| JPG    | Smaller file size, no alpha    |
| BMP    | Legacy support                 |
| TGA    | Classic game texture format    |
| TIF    | High quality archival          |
| HDR    | High dynamic range images      |

> When you select PNG or JPG, the app automatically applies the correct Alpha and Color Space settings for you.

---

### Step 4 — AI Upscaling _(optional)_

Enable **AI Upscaling** to enhance texture quality using [upscayl-ncnn](https://github.com/xinntao/Real-ESRGAN) before the final conversion. This uses a **DDS Sandwich** workflow — three phases run automatically for each file.

#### How it works

```
Phase 1 — texconv      : Source texture  →  Lossless PNG  (temp file)
Phase 2 — upscayl-ncnn : Lossless PNG   →  4× Upscaled PNG  (temp file)
Phase 3 — texconv      : Upscaled PNG   →  Final output at your chosen scale + format + mipmaps
```

- The temp files are created in a system temp folder and **deleted automatically** when done
- Only PNG, JPG, DDS and WebP inputs are supported for AI upscaling

#### Scale Slider + AI Upscaling

The **Scale Slider** controls the **final output size** — the AI always runs internally at 4× and Phase 3 resizes back to your chosen scale.

| Scale Slider | Source size | Final output                                    |
| ------------ | ----------- | ----------------------------------------------- |
| 1× (default) | 512×256     | 512×256 — same size, but quality is AI-enhanced |
| 2×           | 512×256     | 1024×512 — double size, AI-enhanced             |
| 0.5×         | 512×256     | 256×128 — half size, AI-enhanced                |
| 4×           | 512×256     | 2048×1024 — full 4× size, native AI resolution  |

> At 1× the texture keeps its original dimensions but gains detail from the AI upscale. This is useful when source files have degraded quality or were originally downscaled.

#### Models

| Model                     | Best for                                       |
| ------------------------- | ---------------------------------------------- |
| `realesrgan-x4plus`       | Real-world photos, game textures (general use) |
| `realesrgan-x4plus-anime` | Anime-style art, flat-shaded textures, UI      |
| `realesr-animevideov3`    | Anime video frames, animated sprite sheets     |

#### Queue Modes

| Mode                 | How it works                                                 | Recommended for                   |
| -------------------- | ------------------------------------------------------------ | --------------------------------- |
| **Serial** (default) | One file at a time — Phase 1 → Phase 2 → Phase 3, repeat     | All GPUs, safe default            |
| **Parallel** ⚡      | All Phase 1 first → All Phase 2 simultaneously → All Phase 3 | High-end GPUs with plenty of VRAM |

> **Parallel mode** runs all upscaling jobs at the same time using multiple threads. This is significantly faster for large batches on powerful hardware, but will use much more GPU memory. Use Serial mode if your GPU crashes or runs out of VRAM.

#### Requirements

- `upscayl-ncnn.exe` must be present in the same folder as the app
- A `models` folder must exist next to it, containing the `.bin` and `.param` files for each model
- Minimum recommended: a dedicated GPU with at least 2 GB VRAM for serial mode

---

### Step 5 — Set Output Folder

- **Auto-create `output` folder** (ON by default) — creates an `output` folder next to your input file or folder automatically
- Turn it OFF and click Browse to choose your own destination folder

---

### Step 6 — Overwrite Toggle

- **OFF** (default) — skips files that already exist in the output folder
- **ON** — overwrites existing files with the same name

---

### Step 7 — Convert

Click the **Convert** button. The log panel on the right shows:

- The exact texconv command being run
- File-by-file progress
- AI upscaling progress bar (when AI mode is active)
- Any errors or warnings
- A final ✅ Done or ❌ Error message

---

## DDS → PNG (Correct Colors Fix)

When converting DDS to PNG, colors can appear **brighter or washed out** compared to the original. This is a known gamma/color space issue with how PNG handles color data.

**Fix — the app auto-applies these when you select PNG output:**

| Setting        | Value                          |
| -------------- | ------------------------------ |
| Alpha Handling | Premultiplied Alpha (-pmalpha) |
| Color Space    | sRGB Input only (-srgbi)       |

These are set automatically — you don't need to change them manually.

---

## Supported Input Formats

| Format     | Notes                                    |
| ---------- | ---------------------------------------- |
| PNG        | Lossless, fully supported                |
| JPG / JPEG | Lossy, common photo format               |
| DDS        | DirectX texture, most common in games    |
| WebP       | Requires WebP codec from Microsoft Store |

---

## Live Command Preview

Every time you change a setting, the **Live Command Preview** panel updates instantly showing the exact texconv command that will run. You can copy it using the clipboard button and run it manually in CMD if needed.

> When AI Upscaling is enabled, the preview shows all three phases of the DDS Sandwich workflow so you know exactly what will run.

---

## Tips

- In **batch mode**, mixed format folders work fine — PNG, JPG, DDS and WebP are all processed together
- The **output folder** mirrors subfolder structure when Include Subfolders is ON
- **Scale factor** in batch mode applies per-image — a 1024×1024 and a 512×256 in the same folder will each scale correctly to their own proportional size
- **Mip levels** — use `Keep Existing` when you just want to convert format without touching the mip chain
- **AI Upscaling + 1× scale** is a great way to improve quality on old or degraded textures without changing their resolution
- **AI Upscaling** works on single files too — not just batch mode
- If upscayl-ncnn crashes mid-batch, try switching from **Parallel** to **Serial** mode — it uses far less VRAM

---

## Credits

- **Texconv** — by Microsoft / [DirectXTex](https://github.com/microsoft/DirectXTex)
- **upscayl-ncnn** — AI upscaling engine by [Upscayl](https://github.com/upscayl/upscayl-ncnn)
- **TexconvGUI** — built with Python, pywebview, Bootstrap 5
