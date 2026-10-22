# Non-Sidereal Imaging Pipeline

A Stimela-based calibration and imaging pipeline for MeerKAT radio interferometric observations, designed to support non-sidereal imaging workflows.

The pipeline automates observation configuration by extracting calibrator and target information directly from a Measurement Set (MS). It updates the observation configuration before executing the calibration and imaging recipe.

## Table of Contents

- [Overview](#overview)
- [Requirements](#requirements)
- [Installation](#installation)
- [Preparing an Observation](#preparing-an-observation)
- [Running the Pipeline](#running-the-pipeline)
- [Automatic Observation Configuration](#automatic-observation-configuration)
- [Configuration Files](#configuration-files)
- [Troubleshooting](#troubleshooting)

## Overview

The pipeline consists of two main steps:

1. **Observation configuration:** `update_obs.py` reads metadata from the input Measurement Set and updates `xgc-reborn_obs.yml`.
2. **Calibration and imaging:** Stimela executes the workflow defined in `xgc-reborn_recipe.yml` using the populated observation configuration.

The observation configuration script automatically identifies calibrator and target fields, determines the MeerKAT observing band, selects the appropriate Crystallball sky model, and identifies the calibration reference antenna.

## Requirements

The following are required to run the pipeline:

- Python 3.10–3.12
- Git
- pip
- Access to the input Measurement Set
- A compatible container runtime and the software dependencies required by the Stimela recipe
- GitHub SSH access if installing PrepKAT through its Git repository

The Python dependencies are declared in `pyproject.toml`.

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/Ntsikelelo-Charles/Non-sidereal-imaging-pipeline.git

cd Non-sidereal-imaging-pipeline
```

### 2. Create and activate a virtual environment

Create an isolated Python environment:

```bash
python3.10 -m venv .venv
source .venv/bin/activate
```

Upgrade pip:

```bash
python -m pip install --upgrade pip
```

### 3. Install the project

Install the project and its declared dependencies from `pyproject.toml`:

```bash
python -m pip install .
```

This command reads the project metadata and dependency declarations from `pyproject.toml`, builds the project using the configured build backend, and installs it into the active Python environment.

**Note:** Some scientific software dependencies may require system libraries or additional configuration. In particular, `python-casacore` and the tools used by the Stimela recipe must be compatible with your system.

### 4. Verify the installation

Check that Stimela is available:

```bash
stimela --version
```

Check that the observation script's Python dependencies can be imported:

```bash
python -c "from casacore.tables import table; import yaml; print('Dependencies OK')"
```

Display the observation script's command-line options:

```bash
python update_obs.py --help
```

## Preparing an Observation

Before running the pipeline, execute `update_obs.py` with the path to the Measurement Set you want to process.

For example:

```bash
python update_obs.py \
    --ms /net/ziggy/data/oms/killick-cv/data/raw/1674958551_sdp_l0.ms
```

Replace the example path with the location of your own Measurement Set.

The script reads the Measurement Set metadata and automatically updates the `obs.default` section in `xgc-reborn_obs.yml`.

The following parameters are populated automatically:

- Measurement Set path
- Bandpass calibrator
- Flux calibrator
- Phase calibrator
- Polarization calibrator
- Target source
- MeerKAT observing band
- Crystallball sky model
- Calibration reference antenna

**Important:** Run the script from the repository directory, where `xgc-reborn_obs.yml` is located. The script modifies this file in place.

Review the updated configuration before starting the pipeline.

## Running the Pipeline

Once the observation configuration has been populated, run the Stimela recipe:

```bash
stimela run xgc-reborn_recipe.yml obs=default
```

This executes the workflow defined in `xgc-reborn_recipe.yml` using the `default` observation configuration.

### Complete workflow

For a new observation, the complete procedure is:

```bash
# 1. Clone the repository (first-time setup only)
git clone https://github.com/Ntsikelelo-Charles/Non-sidereal-imaging-pipeline.git

# 2. Enter the repository
cd Non-sidereal-imaging-pipeline

# 3. Create and activate the environment (first-time setup only)
python3.10 -m venv .venv
source .venv/bin/activate

# 4. Install the project (first-time setup only)
python -m pip install --upgrade pip
python -m pip install .

# 5. Populate the observation configuration
python update_obs.py \
    --ms /path/to/your/observation.ms

# 6. Run calibration and imaging
stimela run xgc-reborn_recipe.yml obs=default
```

Replace `/path/to/your/observation.ms` with the actual Measurement Set path.

For subsequent observations, activate the environment and repeat steps 5 and 6.

## Automatic Observation Configuration

The `update_obs.py` script reads the `FIELD`, `STATE`, main Measurement Set, `ANTENNA`, and `SPECTRAL_WINDOW` tables.

### Calibrator and target identification

The script uses the observing modes recorded in the Measurement Set to identify the required fields.

| Parameter | Description |
|---|---|
| `bpcal-field` | Bandpass calibrator |
| `fcal-field` | Flux calibrator |
| `gcal-field` | Phase calibrator |
| `xcal-field` | Polarization calibrator |
| `target-field` | Target source |

If multiple fields match an observing mode, the first matching field is selected. If a required field cannot be found, the script raises an error. For the polarization calibrator, it attempts to use a field associated with the `UNKNOWN` observing mode if no polarization calibrator is identified.

### Observing band detection

The script determines the minimum and maximum channel frequencies from the `SPECTRAL_WINDOW` table.

The currently supported bands are:

- **UHF:** Frequency coverage between 500 and 1150 MHz.
- **L band:** Frequency coverage between 800 and 1800 MHz.

Observations outside these ranges are not supported by the current band-detection implementation.

### Crystallball sky model selection

The script selects a sky model based on the detected band and the identified bandpass and flux calibrators.

The currently supported calibrator options are:

- `J1939-6342` (PKS 1934-638)
- `J0408-6545` / `0408-65` (PKS 0408-65)

The corresponding sky model is selected from the `data/crystallball/` directory. The required model file must exist at the configured path.

### Calibration reference antenna

The first antenna listed in the Measurement Set's `ANTENNA` table is selected as the calibration reference antenna and assigned to `cal_refant`.

## Configuration Files

| File | Purpose |
|---|---|
| `pyproject.toml` | Defines project metadata, build configuration, and Python dependencies. |
| `update_obs.py` | Extracts observation metadata and updates the observation configuration. |
| `xgc-reborn_obs.yml` | Stores observation-specific parameters under `obs.default`. |
| `xgc-reborn_recipe.yml` | Defines the Stimela calibration and imaging workflow. |

The `xgc-reborn_obs.yml` file is overwritten with updated observation parameters whenever `update_obs.py` runs. Keep a backup of any configuration you need to preserve for another observation.

## Troubleshooting

### Installation fails

Check that your Python version is supported:

```bash
python --version
```

Confirm that your virtual environment is active and pip is using the expected interpreter:

```bash
which python
python -m pip --version
```

Inspect the installation error for missing system libraries, unavailable packages, or dependency conflicts.

### PrepKAT cannot be installed

Ensure Git is installed and that you have SSH access to the repository:

```bash
git ls-remote git@github.com:JSKenyon/PrepKAT.git
```

If the repository requires authentication, configure your GitHub SSH key before attempting installation again.

### Measurement Set not found

Check that the path is correct and accessible:

```bash
ls -ld /path/to/your/observation.ms
```

### Observation configuration file is missing

Run `update_obs.py` from the repository directory and verify that `xgc-reborn_obs.yml` exists.

### Crystallball sky model cannot be determined

Check the detected observing band and the identified calibrators. The current implementation supports only the configured calibrator combinations and observing bands described above.

### Stimela execution fails

Inspect the terminal output and Stimela logs. Check that the required container runtime, container images, external tools, sky model files, input data, and output directories are available.

## Author

**Ntsikelelo Charles**

GitHub: [Ntsikelelo-Charles](https://github.com/Ntsikelelo-Charles)

## License

Specify the applicable software license for this repository before distributing the code for reuse.


## Requirements

The following are required to run the pipeline:

- Python
