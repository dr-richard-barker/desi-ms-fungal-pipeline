Copilot Chat
Creating a comprehensive README file
Generate a README for this repository.
I'll load the repo overview ability first, then analyze the repository to generate a comprehensive README.


Gathered additional information
Now I'll gather evidence about the repository structure and contents.


Get data from /repos/sofialeivaaa/desi-ms-fungal-pipeline

Get file .

Get data from /repos/sofialeivaaa/desi-ms-fungal-pipeline/contents

Using results of get file and 1 other tool call

Results of /repos/sofialeivaaa/desi-ms-fungal-pipeline/contents/src and 1 other tool call

Using results of get file and 5 other tool calls

View directory .vscode
Now I have sufficient evidence to generate a comprehensive README. Let me create one based on the repository structure and code.I'm waiting for your confirmation to create the README file in the repository. This README provides:

Project overview — What the pipeline does and why it matters
Architecture — Directory structure with annotated roles
Data flow — How the pipeline orchestrates metadata, spectra, and analysis
Setup & usage — Step-by-step instructions for running the pipeline
Input requirements — Expected file formats and locations for plate metadata, heatmaps, and chemical libraries
Output details — Three-sheet Excel results with well-level and summary data
Would you like me to proceed with creating this README?

Make these code changes?
README.md

md
# DESI-MS Fungal Pipeline

A Python-based metabolomics analysis pipeline for processing DESI mass spectrometry (DESI-MS) data from fungal samples. This pipeline automates the detection and quantification of known fungal metabolites (targeted analysis) and identifies unexpected ions of interest (untargeted analysis) from 384-well microplate experiments.

## What It Does

1. **Plate Management**: Creates and tracks 384-well microplate layouts with sample metadata
2. **Targeted Metabolite Detection**: Matches experimental mass-to-charge ratios (m/z) against a reference chemical library
3. **Untargeted Ion Discovery**: Ranks unknown ions by intensity differences between experimental conditions
4. **Automated Reporting**: Exports results with well-level details, compound summaries, and actionable recommendations

This pipeline bridges spectrometry data with sample metadata to answer: *Which metabolites are present in my fungal samples, at what intensity, and under which conditions?*

### Stack

- **Language**: Python 3
- **Key Libraries**:
  - `pandas` — data manipulation and aggregation
  - `openpyxl` — Excel file I/O (plate maps, heatmaps, metadata)
  - `pymzml` — mass spectrometry data parsing
- **Data Format**: Excel (.xlsx) for all inputs and outputs
- **Entry Point**: `src/pipeline.py`

## How It's Organized

src/ pipeline.py Main orchestrator; loads plates, spectra, and runs analyses load_data.py Excel readers for heatmaps, plate metadata, chemical library, spectra analysis.py Targeted and untargeted metabolite analysis engines plate_layout.py Joins heatmap intensities with well metadata; signal classification generate_plate_map.py Creates PLATE_XX.xlsx from user-filled input templates reports.py Exports results to multi-sheet Excel output

data/ plates/ Generated PLATE_XX.xlsx files with Plate_Map and Sample_Metadata sheets plate_inputs/ User-filled input templates (PLATE_XX_input.xlsx) Spectra/ Filtered, centroided spectra from pre-processing (optional) raw/ Heatmaps/ Heatmap_*.xlsx files (one per compound per reaction per plate) (raw/ reserved for future .mzML spectrometry files)

.vscode/ settings.json VS Code workspace configuration .gitignore Excludes raw data files (.mzML, .mzXML, .RAW, .csv)

Code

### How It Fits Together

The pipeline follows this flow:

1. **Load plate metadata** — Read all `PLATE_XX.xlsx` files from `data/plates/`, each containing 384 wells with sample IDs, organism, condition, and other metadata
2. **Parse heatmap files** — Scan `data/raw/Heatmaps/` for `Heatmap_*.xlsx` files; extract compound, ion mode, and reaction metadata from filenames
3. **Join intensities with metadata** — For each well, combine heatmap intensity values with sample metadata
4. **Targeted analysis** — Match each well's compound (by name) against the chemical library; annotate with theoretical m/z, adduct type, and signal classification (strong/moderate/low/absent)
5. **Untargeted analysis** — Rank all unknown ions from `Spectra.xlsx` by intensity difference between conditions (Cont vs. Ex)
6. **Export results** — Write three-sheet Excel files to `data/processed/` with well-level results, summaries, and recommendations

## How to Run It

### Prerequisites

- Python 3.7+
- Install dependencies:
  ```bash
  pip install pandas openpyxl pymzml
Generate a Plate Map (First Time Setup)
Before running the pipeline, create a plate metadata file for each microplate:

bash
python src/generate_plate_map.py PLATE_01
python src/generate_plate_map.py PLATE_02
This script:

Reads a user-filled input template from data/plate_inputs/PLATE_XX_input.xlsx (with columns: well, sample_id, organism, condition, replicate, notes)
Generates data/plates/PLATE_XX.xlsx with two sheets:
Plate_Map: A visual 16×24 grid showing sample locations
Sample_Metadata: A flat table (384 rows) with all well metadata
Run the Pipeline
bash
python src/pipeline.py
This will:

Load all plate metadata from data/plates/
Scan for heatmap files in data/raw/Heatmaps/
For each heatmap, prompt you to confirm/correct the parsed metadata (compound, ion mode, reaction)
Run targeted analysis (matching compounds against the chemical library)
Run untargeted analysis (ranking unknown ions if spectra data is available)
Export results to data/processed/PLATE_XX_DESI_results.xlsx
Required Input Files
Before running the pipeline, prepare:

Chemical library → data/chemical_library.csv

Columns: molecule_name, target_mz, exact_mass, type, adduct_type
Example:

Code
molecule_name,target_mz,exact_mass,type,adduct_type
Arabitol,182.1094,180.0945,sugar,Na
Mannitol,187.1345,186.1256,sugar,Na
Plate input template → data/plate_inputs/PLATE_01_input.xlsx

Fill in the "input" sheet with one row per sample well
Columns: well, sample_id, organism, condition, replicate, notes
Example: A1, SAM_001, Aspergillus fumigatus, Ctrl, 1, control replicate
Heatmap files → data/raw/Heatmaps/Heatmap_*.xlsx

Excel files with 16 rows (A–P) × 24 columns intensity grid
Filename pattern: Heatmap_<plate_id>_<instrument>_<plate_type>_<compound>_<reaction>.xlsx
Example: Heatmap_DESI1_girardt1_FullP_Arabitol_FMPB.xlsx
(Optional) Spectra file → data/Spectra/Spectra.xlsx

Sheets: Neg_FiltCent, Pos_FiltCent
Columns: m/z, Cont, Ex, Dif, Y/N (ion enrichment flag)
Used for untargeted analysis of unexpected metabolites
Output
Results are written to data/processed/PLATE_XX_DESI_results.xlsx with three sheets:

Targeted_WellLevel: One row per well with intensity, signal flag, and compound annotations
Targeted_Summary: Aggregated statistics (mean intensity, max, detection count) per compound per condition
Untargeted_Ions: Top 30 ranked unknown ions sorted by intensity difference; includes enrichment recommendations
Try Asking
"How do I create a plate map for my first plate?" — See src/generate_plate_map.py usage; it expects a user-filled template in data/plate_inputs/
"Where should I put my .mzML spectrometry files?" — The pipeline currently works with pre-processed heatmap and spectra Excel files; .mzML support is reserved for future versions (see data/raw/)
"What does 'signal_flag' mean?" — Signals are classified in plate_layout.py:classify_signal() as strong (≥500), moderate (≥100), low (≥10), or absent (<10); thresholds are tunable after seeing real data
