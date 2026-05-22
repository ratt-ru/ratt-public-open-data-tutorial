# MeerKAT Radio Observations of ESO 137-006

## Overview

This dataset contains MeerKAT radio telescope visibility data for the radio galaxy
**ESO 137-006**, located in the Norma galaxy cluster (Abell 3627) at a distance of
approximately 70 megaparsecs (~228 million light-years).

These observations led to the serendipitous discovery of multiple **collimated
synchrotron threads (CSTs)** — extremely narrow, thread-like structures of radio
emission linking the galaxy's radio lobes. This configuration had not been
previously observed in radio galaxies and is described in detail in
[Ramatsoku et al. (2020)](#citation).

The dataset is hosted on Amazon S3 at `s3://ratt-public-data/ESO137/` in the
`af-south-1` (Cape Town) region and can be read directly from the cloud without
downloading.

---

## Observations

| Parameter | Value |
|---|---|
| Telescope | MeerKAT (64 antennas) |
| Target | ESO 137-006 (Norma cluster / Abell 3627) |
| Target coordinates | RA 16h 15m 04s, Dec −60° 54′ 21″ |
| Observation date | May 2019 |
| On-target integration | 14 hours |
| Frequency band 1 | 980–1080 MHz (centre ~1030 MHz) |
| Frequency band 2 | 1356–1440 MHz (centre ~1398 MHz) |
| Angular resolution | ~10 arcsec at 1030 MHz; ~7.5 arcsec at 1398 MHz |
| Image noise (band 1) | 30.8 µJy/beam |
| Image noise (band 2) | 20.8 µJy/beam |

---

## Data Organisation

The dataset is stored under the S3 prefix `s3://ratt-public-data/ESO137/` and
consists of three Measurement Sets:

```
s3://ratt-public-data/ESO137/
├── ms1_primary.zarr     # Flux/bandpass calibrator (PKS 1934-638)
├── ms1_target.zarr      # Target field — scan group 1
└── ms2_target.zarr      # Target field — scan group 2
```

Each `.zarr` entry is a Zarr directory hierarchy containing one or more
xarray Datasets, each corresponding to a scan group within the observation.

| File | Approx. compressed size | Description |
|---|---|---|
| `ms1_primary.zarr` | ~277.5 GB | Primary calibrator scans |
| `ms1_target.zarr` | ~1.3 TB | First set of target-field scans |
| `ms2_target.zarr` | ~1.3 GB | Second set of target-field scans |
| **Total** | **~2.8 TB** |  |

---

## Data Format

The data follow the **CASA Measurement Set v2** data model
([CASA Memo 229](https://casa.nrao.edu/Memos/229.html)), serialised as
**Zarr** arrays. Zarr is a cloud-native chunked array format that supports
direct, parallel access from object storage without staging data locally.

The recommended access library is
[dask-ms](https://github.com/ratt-ru/dask-ms), which presents each
Measurement Set as a list of [xarray](https://xarray.pydata.org) Datasets
backed by [Dask](https://dask.org) arrays.

### Accessing the data

```python
from daskms import xds_from_storage_ms

url = "s3://ratt-public-data/ESO137/ms1_target.zarr"
datasets = xds_from_storage_ms(url)
```

No AWS credentials are required — the bucket is publicly readable. The
`dask-ms[s3,zarr]` package handles S3 access transparently via
[s3fs](https://s3fs.readthedocs.io).

### Minimum dependencies

```
dask-ms[s3,zarr] >= 0.2.30
dask[dataframe]  >= 2026.3.0
xarray           >= 2026.4.0
```

---

## Data Dictionary

Each xarray Dataset returned by `xds_from_storage_ms` represents one scan
group and exposes the following data variables and coordinates:

### Data variables

| Variable | Shape | dtype | Description |
|---|---|---|---|
| `DATA` | `(row, chan, corr)` | complex64 | Raw correlated visibilities (voltages) |
| `CORRECTED_DATA` | `(row, chan, corr)` | complex64 | Calibrated visibilities after direction-independent calibration |
| `WEIGHT_SPECTRUM` | `(row, chan, corr)` | float32 | Per-channel inverse-noise weights |
| `FLAG` | `(row, chan, corr)` | bool | `True` where data are flagged (bad or missing) |
| `UVW` | `(row, 3)` | float64 | Baseline UVW coordinates in metres (columns: U, V, W) |
| `ANTENNA1` | `(row,)` | int32 | Index of first antenna in baseline |
| `ANTENNA2` | `(row,)` | int32 | Index of second antenna in baseline |
| `TIME` | `(row,)` | float64 | Measurement time (Modified Julian Date in seconds) |

### Coordinates / dimensions

| Dimension | Size | Description |
|---|---|---|
| `row` | varies per scan | One row per (baseline, time) sample |
| `chan` | 4096 | Frequency channel index |
| `corr` | 4 | Polarisation correlation index (XX, XY, YX, YY) |

### Dataset-level attributes

Each Dataset also carries subtable references (as xarray attributes or
companion Datasets): `ANTENNA`, `FIELD`, `SPECTRAL_WINDOW`,
`POLARIZATION`, and `OBSERVATION`, providing metadata about antenna
positions, field pointing, spectral configuration, and observation details.

---

## Processing Pipeline

The raw MeerKAT data were calibrated and imaged using the
[CARACal](https://github.com/caracal-pipeline/caracal) pipeline, which
orchestrates [CASA](https://casa.nrao.edu),
[WSClean](https://wsclean.readthedocs.io), and
[CubiCal](https://github.com/ratt-ru/CubiCal) via the
[Stimela](https://github.com/caracal-pipeline/stimela) framework.

The `CORRECTED_DATA` column in the target Measurement Sets contains
direction-independent (DI) calibrated visibilities. Full calibration
details are described in Ramatsoku et al. (2020).

---

## Community Challenge

A key challenge associated with this dataset is performing full distributed
radio interferometric imaging — gridding, Fast Fourier Transform (FFT), and
CLEAN deconvolution — operating directly on the cloud-stored visibilities at
scale (~3 TB). This is directly relevant to the imaging pipelines being
developed for the Square Kilometre Array (SKA).

Related work: [pfb-imaging](https://www.sciencedirect.com/science/article/pii/S2213133725000691)

---

## Citation

If you use this dataset in a scientific publication, please cite:

> Ramatsoku, M., Murgia, M., Vacca, V., Serra, P., et al. (2020).
> *Collimated synchrotron threads linking the radio lobes of ESO 137-006.*
> Astronomy & Astrophysics, 636, L1.
> https://doi.org/10.1051/0004-6361/202037800

### BibTeX

```bibtex
@article{Ramatsoku2020,
  author  = {Ramatsoku, M. and Murgia, M. and Vacca, V. and Serra, P. and others},
  title   = {Collimated synchrotron threads linking the radio lobes of {ESO} 137-006},
  journal = {Astronomy \& Astrophysics},
  year    = {2020},
  volume  = {636},
  pages   = {L1},
  doi     = {10.1051/0004-6361/202037800}
}
```

---

## Further Reading

- [SARAO Press Release](https://www.sarao.ac.za/media-releases/astronomers-stumble-upon-unexpected-features-in-a-distant-galaxy-using-meerkat-data/)
- [MeerKAT telescope overview](https://www.sarao.ac.za/science/meerkat/)
- [dask-ms documentation](https://dask-ms.readthedocs.io)
- [CASA Measurement Set v2 specification](https://casa.nrao.edu/Memos/229.html)

---

## Contact

For questions about this dataset, contact [sperkins@sarao.ac.za](mailto:sperkins@sarao.ac.za)
or open an issue at https://github.com/ratt-ru/ratt-public-open-data-tutorial/issues.
