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

The Non-Sidereal Imaging Pipeline is a Stimela-based workflow for MeerKAT radio interferometric calibration and imaging, designed to support non-sidereal observations.

The pipeline combines existing radio astronomy software and recipes with automated observation configuration.

Its main components are:

1. **First-generation calibration (1GC):** Derived from the [xgc-reborn pipeline](https://github.com/ratt-ru/xgc-reborn.git).
2. **Self-calibration and imaging:** Based on recipes from the [Parrot recipe repository](https://github.com/ratt-ru/parrot-stew-recipes.git).
3. **Observation configuration:** The `update_obs.py` script reads metadata from the input Measurement Set (MS) and updates `xgc-reborn_obs.yml` with observation-specific parameters.

The observation configuration script automatically identifies calibrator and target fields, determines the MeerKAT observing band, selects the appropriate Crystallball sky model, and identifies the calibration reference antenna.

The workflow is orchestrated using Stimela, with observation-specific settings defined in YAML configuration files.

## Acknowledgements

This pipeline builds on existing software and recipes developed by the radio astronomy community.

- **[xgc-reborn](https://github.com/ratt-ru/xgc-reborn.git):** Provides the basis for the first-generation calibration (1GC) component of the pipeline.
- **[Parrot stew recipes](https://github.com/ratt-ru/parrot-stew-recipes.git):** Provides recipes used for self-calibration and imaging.
- **[Stimela](https://github.com/caracal-pipeline/stimela):** Provides the workflow orchestration framework used to execute the pipeline.

We acknowledge the developers and contributors of these projects for making their software and recipes available to the community.


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



## Requirements

The following are required to run the pipeline:

- Python
