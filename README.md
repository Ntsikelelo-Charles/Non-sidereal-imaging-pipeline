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

- Python
