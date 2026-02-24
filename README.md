# 🔬 TLC Analyzer

**A browser-based tool for quantitative analysis of Thin Layer Chromatography (TLC) images.**

No installation required. No server. No data leaves your computer. Just open the HTML file in any modern browser and start analyzing.

---

## What It Does

TLC Analyzer lets you load a photo of a TLC plate and:

- Mark the **solvent front** and **origin line** to calculate Rf values
- Define **vertical lanes** and name them (e.g., "Control", "Sample 1", "Fraction 3")
- **Auto-detect spots** using a flood-fill algorithm on pixel darkness, or draw them manually
- Measure **integrated optical density** (pixel-level darkness × area) as a proxy for compound quantity
- Compare **relative amounts** of the same compound across different lanes
- Visualize **lane densitograms** (optical density profile from origin to front)
- Export all data to **CSV**

---

## Quick Start

1. **Download** `tlc-analyzer.html`
2. **Open** it in Chrome, Firefox, Edge, or Safari
3. Follow the 6-step workflow:

```
① Load image → ② Mark front → ③ Mark origin → ④ Add lanes → ⑤ Detect spots → ⑥ Compare
```

That's it — no npm, no Python, no installation.

---

## Step-by-Step Tutorial

### Step 1 — Load Your TLC Image

Drag and drop your TLC photo onto the upload zone, or click to browse.

- Works with **JPG, PNG, TIFF**, or any standard image format
- The image is processed entirely in your browser; nothing is uploaded anywhere
- Higher resolution photos give more accurate optical density measurements

> **Tip:** Good lighting and a dark background on the plate give the best results. Avoid glare.

---

### Step 2 — Mark the Solvent Front

1. Select **▲ Front** mode (active by default)
2. Click on the image at the position of the **solvent front line** (the highest point the solvent reached)

A green dashed line labeled `FRENTE` will appear. This is the reference point for Rf calculation.

---

### Step 3 — Mark the Origin

1. Select **▼ Origin** mode
2. Click on the **application line** (where you spotted the samples)

An orange dashed line labeled `ORIGEN` will appear.

> **Why this matters:** Rf = (distance spot traveled) / (distance solvent traveled), both measured from the origin. Without both lines, Rf cannot be calculated.

---

### Step 4 — Define Lanes

1. Select **| Lanes** mode
2. Click once on the image **at the center of each lane** (vertical column)
3. Each lane gets a unique color and a default name ("Lane 1", "Lane 2", etc.)
4. **Rename lanes** in the left panel by clicking the name field — use meaningful names like `Control`, `Sample A`, `Std 10µg`

Lanes are sorted automatically by horizontal position. You can delete individual lanes with the ✕ button.

> **Tip:** Adjust the **Lane width** slider before auto-detection to match the physical width of your lanes. A wider value captures more of the lane; too wide and you'll pick up signal from neighboring lanes.

---

### Step 5 — Detect Spots

#### Option A: Auto-Detect (recommended)

1. Adjust the two sliders:
   - **Threshold** — how dark a pixel must be to count as a spot. Increase if you're getting false positives (background noise detected), decrease if real spots are being missed.
   - **Min area** — minimum spot size in pixels. Increase to filter out noise artifacts.
2. Click **⚡ Auto-detect**

The algorithm uses a **flood-fill (BFS) approach**: it finds connected dark-pixel regions within each lane's column, bounded by the origin and front lines.

#### Option B: Manual Drawing

1. Select **● Spots** mode
2. Click and drag on the image to draw a rectangle around a spot
3. The spot is automatically assigned to the nearest lane

You can combine both methods — auto-detect first, then manually add missed spots or remove false positives by clearing and redrawing.

---

### Step 6 — Compare & Analyze

Once spots are detected, the right panel shows three views:

#### 📊 Quantities Tab

This is the core quantitative comparison. For each group of spots with similar Rf values (within ±0.07), a bar chart shows **relative optical volume** across all lanes.

**Optical volume** is calculated as:

```
Volume = Σ (255 − pixel_gray_value)  for every pixel inside the spot bounding box
```

- A pitch-black pixel contributes 255; a white pixel contributes 0
- Larger and/or darker spots have higher volumes
- Within a Rf group, bars are normalized to the maximum (100%)

At the top of the tab you'll also see the **total optical load per lane** (sum of all spots), useful for checking loading consistency.

> **Methodological note:** Optical volume is proportional to concentration only in the linear range of the dye's response. For absolute quantification, include a dilution series of a known standard on the same plate and build a calibration curve.

#### 📈 Densitogram Tab

For each lane, a plot of average optical density vs. position (from origin to front) is shown. This is analogous to a chromatogram trace. Detected spot positions are marked with vertical dashed lines.

Useful for:
- Verifying spot detection accuracy
- Identifying overlapping or poorly-resolved spots
- Comparing peak widths between lanes

#### 🗃 Table Tab

A complete data table with:

| Column | Description |
|--------|-------------|
| Lane | Lane name |
| Rf | Retention factor (0 = origin, 1 = front) |
| Area px² | Number of pixels in the spot bounding box |
| Optical volume | Integrated optical density |
| Rel % | Percentage of maximum volume across all spots |

Click **⬇ Export CSV** to download the full table for further analysis in Excel, R, or Python.

---

## Parameters Reference

| Parameter | Default | Effect |
|-----------|---------|--------|
| Threshold | 35 | Min darkness for a pixel to be "dark". Range 5–100. |
| Min area | 300 px² | Smallest spot that will be reported. Filters noise. |
| Lane width | 40 px | Half-width of the column scanned per lane. |

---

## Limitations & Caveats

- **Bounding box approximation:** Spot area is measured as the bounding rectangle, not the exact spot shape. This slightly overestimates area for irregularly shaped spots.
- **Overlapping spots:** If two spots overlap, they may be detected as one. Manual separation by drawing individual rectangles is recommended in those cases.
- **Lighting uniformity:** Non-uniform illumination of the plate (e.g., shadows, glare) will introduce systematic error in optical density. If possible, scan plates on a UV transilluminator or use a flatbed scanner instead of a camera photo.
- **No absolute quantification:** Without a standard curve, results are relative comparisons only.
- **Color TLC plates:** The algorithm uses grayscale intensity. For colored spots (e.g., ninhydrin-stained amino acids), the tool still works but you may need to adjust the threshold to match spot visibility.

---

## Technical Details

- **Language:** Vanilla JavaScript + HTML5 Canvas — no frameworks, no dependencies
- **Pixel access:** Full-resolution `ImageData` is cached on load for fast repeated analysis
- **Spot detection:** Connected-component labeling via iterative BFS within each lane's bounding column
- **Rf grouping:** Spots within ±0.07 Rf units are treated as the same compound for cross-lane comparison
- **All processing is client-side** — your images never leave the browser

---

## File Structure

```
tlc-analyzer.html    ← The entire application (single self-contained file)
README.md            ← This file
```

---

## Example Use Cases

- Comparing **reaction yield** across multiple conditions (each lane = one condition)
- Monitoring **column fraction purity** (each lane = one fraction)
- Verifying **loading consistency** across lanes before densitometry
- Teaching TLC concepts — students can annotate and measure their own plate photos

---

## Contributing

Feel free to open issues or pull requests. Some ideas for future improvements:

- [ ] Gaussian peak fitting for overlapping spots
- [ ] Support for inverted plates (bright spots on dark background)
- [ ] Multi-image batch processing
- [ ] Calibration curve tool for absolute quantification
- [ ] Spot color channel selection (R/G/B) for colored stains

---

## License

MIT — free to use, modify, and share.
