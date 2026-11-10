# 3K FRET Analysis

Analysis code for 3K FRET imaging in the Kara Neurodegenerative Disease Lab. The notebook segments lipid droplets, synuclein inclusions, and FRET signal in three-channel `.czi` images, then compares inclusion holes between the donor and FRET channels and measures raw MFI per cell.

**Notebook:** [3k_FRET_final.ipynb](3k_FRET_final.ipynb). `3k_FRET_prototype.ipynb` is the earlier version (writes `.xlsx` instead of `.csv`) and is kept for reference.

## Channels

The notebook expects this CZI channel order (`CHANNEL_ORDER` in the parameters cell):

| Index | Channel | Fluorophore | Meaning |
|---|---|---|---|
| 0 | orange | AF555 | acceptor / lipid droplets |
| 1 | green | AF488 | donor / synuclein inclusions |
| 2 | blue | AF555-T3 | FRET channel |

## Setup

1. Clone the repo and create an environment (Python 3.12 was used):

   ```bash
   git clone https://github.com/timmyhsu07/3k_Fret_Analaysis.git
   cd 3k_Fret_Analaysis
   python -m venv .venv
   source .venv/bin/activate        # Windows: .venv\Scripts\activate
   pip install "cellpose<4" czifile tifffile numpy pandas scipy scikit-image matplotlib jupyter
   ```

   Cellpose must be below version 4. The notebook uses `models.Cellpose(model_type="cyto3")`, which was removed in Cellpose 4.

2. Put your images in a folder next to the notebook, or under `images/`:

   ```
   3k_Fret_Analaysis/
     3k_FRET_final.ipynb
     images/
       <your folder>/
         sample_01.czi
         ...
   ```

3. In the paths cell, set `IMAGE_FOLDER_NAMES` to your folder name(s). It currently lists `"test images"`. Each folder is processed separately.

4. If you have no GPU, set `USE_GPU = False`. Set `SHOW_PLOTS = False` for long batch runs.

5. Run the notebook top to bottom.

## Outputs

All written to `Results/`:

- `<folder>_3k_FRET_structure_hole_comparison.csv`: one row per green inclusion. Each green hole is compared with the blue hole at the same location. `Blue_Narrower_Than_Green` is True only when the blue hole is narrower; `Comparable` and `Comparison_Basis` explain rows where no comparison was possible.
- `<folder>_3k_FRET_per_cell_MFI.csv`: one row per segmented cell. Raw orange, green, and blue MFI under (droplets inside green) ∩ (green inclusion) ∩ (cell), plus green MFI for walls touching those droplets.
- `Results/masks/<folder>/<image>/`: binary (0/255) and label masks, numbered in analysis order.
- `Results/qc/<folder>/<image>_QC.png`: QC panels per image.
- `Results/masks/<folder>/run_info.json`: all parameters and per-image counts for the run.

## Notes

- MFIs are always measured on raw uint16 pixels. Normalization and smoothing are only used to build masks.
- Green and blue inclusions are segmented with the same function and parameters, independent of cell borders.
- Review the QC figures. Cell masks, touching-object splits, and very small holes still need a visual check.
