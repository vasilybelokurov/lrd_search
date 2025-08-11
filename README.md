# Little Red Dots (LRDs) Local Analog Search

A systematic search for local (low-redshift) analogs of Little Red Dots - compact, red objects that may be related to high-redshift quasars or early galaxy formation processes.

## Project Overview

This project identifies rare, red point sources from a comprehensive multi-survey cross-match and validates them spectroscopically. The analysis combines UV, optical, and near-infrared photometry with astrometric and spectroscopic data to find objects with Little Red Dot-like properties at accessible redshifts.

## Key Results

- **69 LRD candidates** identified from ~20M sources
- **23 candidates** (33%) are previously known SDSS quasars  
- **35 candidates** (51%) have DESI spectroscopic observations
- **21 candidates** (30%) have successfully retrieved DESI spectra
- **Redshift range**: 0.17-3.83 for spectroscopically confirmed objects
- **Mix of object types**: QSOs and galaxies, confirming genuine extragalactic nature

## Notebooks

### 1. `all_sky_query.ipynb` - Data Collection
**Status: ✅ Complete - search file already exists, no need to re-run**

Performs comprehensive all-sky cross-match between:
- **GALEX** UV photometry (FUV, NUV bands)
- **DECaLS** optical imaging (g, r, z bands) 
- **Gaia DR2/DR3** astrometry (proper motions, parallax)
- **SDSS DR9** photometry (u, g, r, i, z bands)

**Features:**
- Galactic latitude cut |b| > 30° to avoid disk contamination
- Extinction corrections using SFD dust maps
- Returns ~20.5M GALEX-DECaLS pairs → 645k unique sources

**Output:** `all_sky_data.parquet` (~20M objects)

### 2. `lrd_select_candidates.ipynb` - Candidate Selection

Applies stringent selection criteria to identify LRD candidates:

**Selection Criteria:**
- **Morphology**: Point-like sources (DECaLS type = 'PSF')
- **Astrometry**: Low proper motion (S/N < 2) and parallax (S/N < 2) 
- **Color cuts**:
  - BP-RP > 1.3 (red in Gaia)
  - g-r > 0.6 (red in optical) 
  - -0.6 < NUV-u < 0.6 (moderate UV-optical color)
  - 0.5 < u-g < 2 (blue-red transition)

**Features:**
- 12-panel color-color diagnostic plots
- DECaLS cutout image downloads
- Cross-match with SDSS-DR16 QSO catalog

**Outputs:**
- `lrd_can_radec.fits` (candidate coordinates)
- `lrd_candidates.parquet` (full candidate data)
- `decals_cutouts_jpg/` (69 DECaLS images)
- Color-color diagnostic plots

### 3. `desi_bulk_retrieval_fixed.ipynb` - Spectroscopic Follow-up

Retrieves and analyzes DESI DR1 spectra for the LRD candidates:

**Process:**
1. Searches DESI DR1 within 1" of each candidate using wsdb
2. Retrieves spectra using SPARCL client from NOIRLab DataLab
3. Creates publication-quality spectrum plots

**Features:**
- Redshift measurements from DESI pipeline
- Signal-to-noise ratios and spectral classifications
- Emission line identification ([OII], Hβ, [OIII], Hα, [NII])
- Individual spectrum plots with metadata

**Outputs:**
- `desi_bulk_spectra.pkl` (spectrum data)
- `desi_spectra/` (21 individual spectrum plots)
- `desi_bulk_search_results.csv` (search results)

## Data Products

### Images
- `decals_cutouts_jpg/` - 69 DECaLS gri color cutouts (15" × 15")
- `desi_spectra/` - 21 DESI spectrum plots with redshifts and line IDs
- `plots/` - Color-color diagnostic plots and candidate gallery

### Data Files
- `aux_data/desi_bulk_spectra.pkl` - Retrieved DESI spectra with metadata
- `lrd_can_radec.fits` - RA/Dec coordinates of 69 candidates
- `desi_bulk_search_results.csv` - DESI cross-match results

## Requirements

### Python Environment
Recommended: Use the `desi-dr1` virtual environment which has all required packages:

```bash
source ~/Work/venvs/desi-dr1/bin/activate
```

**Key packages:**
- `astropy` - Astronomical data handling
- `pandas` - Data manipulation
- `matplotlib` - Plotting
- `sqlutilpy` - Database queries
- `dustmaps` - Extinction corrections
- `astro-datalab` (imports as `dl`) - NOIRLab DataLab client
- `sparcl` - Spectral retrieval client

### Data Access
Requires authentication tokens for:
- **NOIRLab DataLab** - for DESI spectrum retrieval
- **wsdb** - for database queries (typically pre-configured)

## Usage

### Quick Start
1. **Skip the data collection** - `all_sky_query.ipynb` has already been run
2. **Run candidate selection**:
   ```bash
   jupyter notebook lrd_select_candidates.ipynb
   ```
3. **Retrieve DESI spectra**:
   ```bash
   jupyter notebook desi_bulk_retrieval_fixed.ipynb
   ```

### Modifying Selection Criteria
Edit the `sel_criteria` dictionary in `lrd_select_candidates.ipynb`:
```python
sel_criteria = {
    'bp_rp': (1.3, None),       # BP-RP  > 1.3
    'g_r'  : (0.6, None),      # g-r   > 0.6
    'nuv_u': (-0.6, 0.6),       # −0.6 < NUV-u < 0.6
    'u_g'  : (0.5, 2),         # 0.5 < u-g < 2
}
```

## File Structure

```
lrd_search/
├── README.md
├── all_sky_query.ipynb              # Data collection (complete)
├── lrd_select_candidates.ipynb       # Candidate selection
├── desi_bulk_retrieval_fixed.ipynb   # Spectroscopic follow-up
├── aux_data/
│   └── desi_bulk_spectra.pkl         # DESI spectra data
├── decals_cutouts_jpg/               # 69 DECaLS cutout images
│   ├── lrdcand_0000_RA354.4802_Dec2.0135.jpg
│   └── ...
├── desi_spectra/                     # 21 DESI spectrum plots  
│   ├── desi_ra030.5459_dec-04.4510.png
│   └── ...
└── plots/                           # Analysis plots
    ├── lrd_candidates_gallery.png
    └── lrd_colour_colour_density.png
```

## Scientific Context

Little Red Dots (LRDs) are compact, red objects discovered in deep JWST surveys at high redshift (z > 3). Their nature is debated - they could be:
- Dust-obscured active galactic nuclei
- Early supermassive black holes  
- Compact starbursting galaxies
- Transitional objects in galaxy evolution

This project searches for similar objects at lower redshifts where detailed follow-up is feasible, providing crucial insights into LRD physics and evolution.

## Citation

If you use this code or data products, please cite the relevant surveys:
- **GALEX**: Martin et al. 2005, ApJ, 619, L1
- **DECaLS**: Dey et al. 2019, AJ, 157, 168  
- **Gaia**: Gaia Collaboration et al. 2018, A&A, 616, A1
- **DESI**: DESI Collaboration et al. 2016, arXiv:1611.00036

---
*Generated with Claude Code - A systematic approach to finding rare astronomical objects*