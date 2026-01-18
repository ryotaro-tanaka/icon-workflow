# App Icon Generation Workflow (ImageMagick)

## Purpose

Generate iOS / Android / PWA app icons from a generated image  
without breaking important visual elements.

This workflow assumes:

- iOS / Android apply rounded or adaptive masks
- Source images may not be square
- Some visual elements must not be cropped
- ImageMagick is available (WSL2 / Linux / macOS)

---

## Core Concepts

### 1. Trim

- Removes uniform background areas
- Often changes aspect ratio
- Must always be followed by square normalization

Command example:

```bash
convert icon.png -fuzz 6% -trim +repage icon-trimmed.png
```

---

### 2. Square Normalization

After trimming, images are often not square.

There are two valid strategies:

#### Crop (destructive)

Use only if losing edge content is acceptable.

- Pros: No added background
- Cons: Important details may be cut

#### Extent (non-destructive, recommended)

Pads the shorter side to make the image square.

- Pros: No visual loss
- Cons: Minimal background padding

Rule of thumb:

> If any important detail exists near the edges, always use extent.

---

## Recommended Workflow (No Cropping, Safe)

### Step 1: Trim background

```bash
convert icon.png -fuzz 6% -trim +repage icon-trimmed.png
```

### Step 2: Make the image square using extent

```bash
w=$(identify -format "%w" icon-trimmed.png)
h=$(identify -format "%h" icon-trimmed.png)
s=$(( w > h ? w : h ))

convert icon-trimmed.png \
    -gravity center \
    -background "<APP_BG_COLOR>" \
    -extent "${s}x${s}" \
    icon-square.png
```

Replace `<APP_BG_COLOR>` with your app’s background color  
(e.g. `#f6f5f1`).

### Step 3: Resize for standard icons

```bash
convert icon-square.png -resize 512x512 icon-512.png
convert icon-512.png -resize 192x192 icon-192.png
```

---

## Android Maskable Icon

Android maskable icons require the main content  
to fit inside the central 80% safe area.

### Generate maskable icon (512x512)

```bash
# 512 * 0.8 ≈ 410
convert icon-square.png \
    -resize 410x410 \
    -gravity center \
    -background "<APP_BG_COLOR>" \
    -extent 512x512 \
    icon-maskable-512.png
```

Optional tuning:

- If the icon looks too small, try 82–85% (420–435px)
- Never exceed ~88%

---

## Output Files

```
    icon-512.png            Standard app icon
    icon-192.png            PWA icon
    icon-maskable-512.png   Android maskable icon
```

---

## Best Practices

- Do not draw icon frames or borders
- Let the OS handle masking
- DPI is irrelevant; pixel size only matters
- Always verify on real devices

---

## Notes

This workflow is intentionally generic and reusable  
across different apps and visual styles.

Adjust only:

- Background color
- Fuzz percentage (for trim)
- Maskable scale ratio

Keep the structure unchanged.
