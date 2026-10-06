# Radiosonde temperature and pressure profiles

A Python notebook that investigates how atmospheric pressure, temperature, moisture and wind vary vertically. The example compares Singapore and Ulaanbaatar on **7 January and 7 July 2021**, with **00Z and 12Z** launches at each station. It brings together fixed-width data parsing, scientific modelling, interpolation and visual comparison in one reproducible workflow.

**Start with [radiosonde_profiles.ipynb](radiosonde_profiles.ipynb).** Its saved charts and tables can be viewed directly on GitHub. The notebook contains the analysis code; no external Python module or WRF installation is needed.

## What the investigation covers

- Measured pressure–height and temperature–pressure profiles across stations and launch times.
- Layer-average lapse rates calculated from temperature interpolated in height.
- Hypsometric layer thickness, separating the measured-temperature term from the water-vapour correction.
- Wind-speed profiles and maxima within a selected pressure interval.

These are individual sounding comparisons, not seasonal averages. January and July labels identify the example dates rather than representative climatological seasons.

The notebook follows the original report's investigation: pressure–height comparisons at both launch times, January pressure-layer thickness, then temperature, lapse-rate and wind comparisons at 12Z. Exploratory exponential fits, full-profile log-pressure trials and duplicate charts are omitted.

## Repository layout

```text
radiosonde-profile-investigation/
├── README.md
├── requirements.txt
├── .gitignore
├── radiosonde_profiles.ipynb     # Main analysis and saved example results
└── data/
    ├── README.md                # Input format and provenance notes
    ├── soundings.csv            # Station/date/launch/file manifest
    └── *.txt                   # Eight small example soundings
```

Running the notebook creates an ignored `outputs/` directory containing regenerated figures and CSV tables. The original assignment PDF, trial PNGs, duplicate figures, LaTeX explanation, WRF files and scratch notebook are excluded from this package.

## Set up the environment

The example was validated with **Python 3.13.7** and the exact dependency versions in `requirements.txt`. Using Python 3.13 is recommended for reproducing this environment. No network connection is needed during the analysis once dependencies are installed and inputs are present.

Open a terminal in this repository folder.

**Windows PowerShell:**

```powershell
py -3.13 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m ipykernel install --user --name radiosonde --display-name "Python (radiosonde)"
.\.venv\Scripts\python.exe -m jupyter lab
```

**macOS / Linux** (with Python 3.13 available as `python3.13`):

```bash
python3.13 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
.venv/bin/python -m ipykernel install --user --name radiosonde --display-name "Python (radiosonde)"
.venv/bin/python -m jupyter lab
```

In JupyterLab, open `radiosonde_profiles.ipynb`, select **Python (radiosonde)**, then choose **Restart Kernel and Run All Cells**. Save the notebook after a successful run to refresh the results shown on GitHub. Launch Jupyter from the repository root so its kernel can find `data/soundings.csv`.

For a noninteractive rerun, use the virtual environment's Python from the repository root. This executes from a fresh kernel and updates the notebook with new results:

```bash
python -c "from pathlib import Path; import nbformat; from nbclient import NotebookClient; p=Path('radiosonde_profiles.ipynb'); n=nbformat.read(p,as_version=4); NotebookClient(n,timeout=180,kernel_name='radiosonde',resources={'metadata':{'path':str(Path.cwd())}}).execute(); nbformat.write(n,p)"
```

Replace `python` with `.\.venv\Scripts\python.exe` on Windows or `.venv/bin/python` on macOS/Linux. The command uses the `radiosonde` kernel registered above.

## Prepare data for another area

1. Obtain soundings for the station(s), dates and UTC launches you want to compare. The [University of Wyoming Upper Air service](https://weather.uwyo.edu/upperair/) provides a Soundings entry point. [NOAA's Integrated Global Radiosonde Archive](https://www.ncei.noaa.gov/products/weather-balloon/integrated-global-radiosonde-archive) is another source, but its native files require a separate conversion to this notebook's schema; they cannot be loaded directly.
2. Save one plain-text, fixed-width table per launch under `data/`. Keep the eleven-column header, units row and seven-character field widths described in [data/README.md](data/README.md). Remove station-information blocks and other prose following the observations. Leading title lines are accepted.
3. Check the original units and missing-value conventions. Convert wind speeds in knots to m/s by multiplying by `0.514444`; convert pressures, temperatures and heights if necessary. Replace source missing-value sentinels with blank fields. Preserve missing fields and column alignment. Changing a units label without converting the values produces incorrect results.
4. Replace or extend `data/soundings.csv`. Each row supplies `station`, ISO `date` (`YYYY-MM-DD`), `time_utc` (`00Z` or `12Z`) and a `file` path relative to `data/`. Add the source URL and notes about station identifier, download date, original units and any transformations. Filenames are arbitrary; plot labels come from the manifest. Check the labels against the source because the stripped text tables cannot verify them.
5. Edit the notebook's parameter cell for the available coverage, restart the kernel, and run all cells. The notebook accepts one or more stations and dates. Each station needs at least one launch at `COMPARISON_TIME_UTC`; both launch times are not required for every station/date pair.

For example, a manifest for another location could start with:

```csv
station,date,time_utc,file,source_url,notes
Example Station,2024-01-15,00Z,example_00Z_20240115.txt,,Record station identifier and original source here
Example Station,2024-07-15,12Z,example_12Z_20240715.txt,,Record station identifier and original source here
```

The referenced files must exist before running; the notebook does not automatically download data.

### Parameters to review

| Parameter | Default | How to adapt it |
| --- | --- | --- |
| `REFERENCE_LEVELS_HPA` | 1000, 925, 850, 700, 500, 400, 300, 200, 100 | Exact observed levels shown in reference tables. Unobserved levels remain blank. |
| `COMPARISON_TIME_UTC` | `12Z` | Launch time for temperature, lapse-rate and wind comparisons. Set to `00Z` to use that launch instead. |
| `THICKNESS_DATE` | `None` | Uses the earliest manifest date (January in the example). Set an ISO date to investigate another date. |
| `LAYER_BOTTOM_HPA`, `LAYER_TOP_HPA` | 850, 500 | Both boundaries must be observed in every sounding on the selected thickness date; selected layer rows need finite height, temperature and mixing ratio. |
| `LAPSE_STEP_M`, `LAPSE_TOP_M` | 1000, 10000 | Use a positive step and a top within each sounding's valid temperature-height range. The first boundary is chosen from common station coverage. |
| `WIND_MIN_HPA`, `WIND_MAX_HPA` | 70, 850 | Choose an interval observed across the launches you compare; each needs at least one valid speed. |
| `EXPORT_RESULTS` | `True` | Set to `False` to display results without saving additional files. |

All heights are source-reported altitudes, not metres above the station. Comparisons of lapse rates between stations must account for different first-layer elevations and thicknesses.

## Methods and interpretation

The loader checks headers and units, keeps the original observation order and preserves blanks as `NaN`. It does not repair source data. Pressure–height plots and standard-pressure reference tables use recorded observations without interpolation or extrapolation.

Lapse rates use linear interpolation in recorded height and actual layer thicknesses, with no extrapolation. Positive values mean cooling with height; negative values mean a layer-average inversion. Pressure-layer thickness uses trapezoidal integration over log pressure and the approximation `Tv ≈ T × (1 + 0.61r)`, with temperature in kelvin and mixing ratio in kg/kg. Virtual potential temperature (`THTV`) is not used as virtual temperature.

The [hypsometric equation in Stull's *Practical Meteorology*, chapter 1, section 1.8](https://www.eoas.ubc.ca/books/Practical_Meteorology/prmet102/Ch01-atmos-v102b.pdf) provides the physical basis for the pressure-layer calculation. Recorded height may already have been derived using hydrostatic balance, so agreement is a consistency check rather than independent validation. The model decomposition does not establish the causes of climatic differences between stations.

The example data were copied unchanged from the original project. The accompanying previous report identifies the **University of Wyoming Atmospheric Science Radiosonde Archive** as their source. Their station, date and launch labels come from filenames and the report; exact retrieval URLs, station identifiers and download/transformation history were not supplied. See [data/README.md](data/README.md) for these provenance limits. The NOAA link above is an alternative source for new data.

## Reproducibility and troubleshooting

The notebook has been run from a fresh kernel with all eight included inputs. Its final cell checks positive recorded thicknesses, finite lapse rates, and the dry-plus-moisture decomposition. Figures and CSVs are regenerated from the inputs; exported tables preserve unrounded calculations. Calculations use the actual first-layer thicknesses of 0.984 km (Singapore) and 0.694 km (Ulaanbaatar), even where report tables rounded or treated the first interval as a full kilometre.

| Error | What to check |
| --- | --- |
| Manifest or file not found | Start in the repository root; check manifest paths relative to `data/`. |
| Expected header or units | Match the documented schema and convert actual values to the expected units. |
| Text outside table / invalid numeric field | Restore seven-character widths and remove trailing metadata or prose. |
| No comparison launch for a station | Supply its selected UTC launch, or change `COMPARISON_TIME_UTC`. |
| Missing layer boundary or measurements | Change the layer to observed boundaries and inspect temperature, height and mixing-ratio gaps. |
| Lapse boundaries exceed observations | Reduce `LAPSE_TOP_M` or choose inputs with sufficient height coverage. |
| Notebook displays old results | Restart the kernel, run all cells and save before committing. |

The repository contains no access tokens, automatic downloads or machine-specific input paths. An open-source licence has not been assigned; choose one for your own code and separately establish the source terms for data you publish.

## Publish your copy on GitHub

Create an empty GitHub repository and open a terminal **inside this folder**:

```bash
git init
git add .
git status
git commit -m "Add reproducible radiosonde profile investigation"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPOSITORY.git
git push -u origin main
```

Review `git status` before committing: `outputs/`, local environments and notebook checkpoints should be excluded. Suggested repository description: **Reproducible radiosonde analysis of temperature, pressure, lapse rates and moisture effects across atmospheric profiles.** Suggested topics: `radiosonde`, `atmospheric-science`, `meteorology`, `python`, `jupyter-notebook`, `data-visualization`.
