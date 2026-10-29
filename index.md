---
title: "mlwp-tools"
subtitle: "Tools for machine-learning weather prediction"
toc: false
---

mlwp-tools is a collection of Python packages for working with machine-learning weather prediction (MLWP) data: defining dataset conventions, loading data into them, aligning datasets in space and time, and verifying forecasts.

## Packages

| Package | Description |
|---|---|
| [mlwp-data-specs](https://github.com/mlwp-tools/mlwp-data-specs) | Dataset specifications for MLWP applications for time, coordinate and uncertainty traits based on `xr.Dataset`s and CF Zarr |
| [mlwp-data-loaders](https://github.com/mlwp-tools/mlwp-data-loaders) | General-purpose data-loaders to create mlwp-data-spec conforming `xr.Dataset`s |
| [mxalign](https://github.com/mlwp-tools/mxalign) | xarray-based space/time/uncertainty-alignment for meteorological datasets |
| [mlwp-harp-writer](https://github.com/mlwp-tools/mlwp-harp-writer) | Write mxalign-aligned, mlwp-data-specs conforming datasets to HARP parquet datasets |
| [mlwp-scorecard](https://github.com/mlwp-tools/mlwp-scorecard) | Easy weather model scorecard creation in Python |

See the [roadmap](roadmap.md) for where the project is heading.
