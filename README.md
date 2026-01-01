# INT-WFC-transit-pipeline
End-to-end INT/WFC transit photometry pipeline: calibration, cube building, aperture photometry, ensemble detrending, and BATMAN-based transit modelling.

# INT/WFC Transit Photometry Pipeline (Monolithic)

Single-file, end-to-end pipeline for INT/WFC time-series transit photometry:

**header ingestion → calibration (bias / flat / overscan / trim) → data cubes → DS9 regions → aperture photometry → ensemble detrending → transit modelling (BATMAN + MCMC)**

This repository intentionally keeps the pipeline as a **single script (`INT_All.py`)** to preserve behaviour, reproducibility, and reduce refactor risk.

---

## Requirements

Python **3.10+** recommended.

Install dependencies (example):

```bash
pip install numpy scipy astropy photutils matplotlib pandas tqdm emcee corner batman-package
```

---

## Repository contents

```
INT_All.py            # full pipeline (monolithic by design)
targets.json          # target database (periods, limb darkening, etc.)
instrument.json       # instrument-level configuration overrides
outputs/              # calibration products, cubes, manifests
photometry_outputs/   # photometry CSVs, plots, merged products
```

---

## Data directory layout

You may place your data anywhere and point the pipeline to it using `--data-dir`.

The pipeline **recursively scans** for FITS files, so the exact layout is flexible.

Example:

```
data_root/
  2023-09-14/
    *.fits
  2023-09-15/
    *.fits
```

Bias, flat, and science frames may be mixed; classification is performed from headers (with rescue heuristics if OBSTYPE is missing).

---

## Configuration

### `targets.json`

Defines target-level parameters (keyed by target name).

Example:

```json
{
  "TOI_1516_b": {
    "Period": 2.70863,
    "T0_BJD": 2459000.12345,
    "A_RS": 5.12,
    "INC_DEG": 86.3,
    "limb_darkening": {
      "r": { "U1_0": 0.32, "U2_0": 0.21, "SIG_U": 0.10 }
    },
    "aliases": ["TOI1516", "TOI 1516"]
  }
}
```

---

### `instrument.json`

Overrides instrument defaults **without editing code**.

Example:

```json
{
  "hdu_index": 4,
  "horizontal_overscan": 53,
  "vertical_overscan": 100,
  "measure_side": "left",
  "gain_e_per_adu": 2.9,
  "max_workers": 6
}
```

- Missing keys fall back to code defaults
- Unknown keys are warned and ignored

---

## Running the pipeline

### Interactive mode (default)

```bash
python INT_All.py
```

You will be prompted for:
- data directory
- target
- filter
- night
- DS9 region files

---

### CLI mode (recommended for reproducibility)

```bash
python INT_All.py \
  --data-dir "C:\\path\\to\\data_root" \
  --target TOI_1516_b \
  --band r \
  --night 2023-09-14 \
  --outdir outputs \
  --seed 42 \
  --verbose
```

Any missing option falls back to interactive selection.

---

## Outputs

The pipeline produces:

- `Header_Master.csv` (full header table with BJD_TDB)
- `master_bias.fits`
- `pix_flat_<filter>.fits`, `illum_flat_<filter>.fits`
- `cube_<filter>.fits`
- `cube_manifest_<filter>.csv`
- raw and detrended photometry CSVs
- diagnostic plots
- MCMC corner plots and fit summaries
- `run_metadata.json` (seed, package versions, timestamp)
- `photometry_frame_quality.txt` (frame rejection statistics)

---

## Notes / limitations

- Written specifically for **INT/WFC**
- Assumes overscan geometry consistent with WFC CCDs
- DS9 region files are in IMAGE coordinates (pixel space)

---

## License

MIT
