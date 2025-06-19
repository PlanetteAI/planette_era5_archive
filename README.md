# Planette's ERA5 Archive

<img src="planette_icon.png" alt="Planette Logo" width="300"/>

## Overview

Planette's ERA5 archive provides a cloud-native version of the ERA5 reanalysis dataset with multiple temporal aggregations, spanning from 1940 to present. This archive differs from other versions of ERA5 because it is stored using zarr stores with icechunk and offers data at various temporal resolutions for enhanced flexibility in climate analysis.

This dataset can be used for:
- **Evaluating climate variability and change** according to ERA5
- **Training data-driven algorithms** aimed at emulating ERA5 weather/climate variability
- **Multi-scale climate analysis** using different temporal aggregations

The dataset includes **six temporal aggregations**: daily means, 7-day rolling means, 30-day rolling means, 90-day rolling means, monthly means, and 3-month means.

## Variables

### Surface and Near-Surface Variables

| Variable                | Units     | Description                                 |
|-------------------------|-----------|---------------------------------------------|
| t2m                     | K         | Daily mean 2 meter temperature              |
| t2m_max                 | K         | Daily maximum 2 meter temperature          |
| t2m_min                 | K         | Daily minimum 2 meter temperature          |
| td2m                    | K         | 2 meter dew point temperature              |
| ts                      | K         | Surface skin temperature                    |
| sst                     | K         | Sea surface temperature                     |
| u10m                    | m/s       | 10 meter zonal wind                         |
| v10m                    | m/s       | 10 meter meridional wind                    |
| u100m                   | m/s       | 100 meter zonal wind                        |
| v100m                   | m/s       | 100 meter meridional wind                   |
| slp                     | hPa       | Sea level pressure                          |
| tcwv                    | kg/m²     | Total column water vapor                    |
| cape                    | J/kg      | Convective available potential energy       |
| cdd                     | K·day     | Cooling degree days                         |
| hdd                     | K·day     | Heating degree days                         |

### Atmospheric Variables at Pressure Levels

| Variable                | Units     | Pressure Levels (hPa)     | Description                                 |
|-------------------------|-----------|---------------------------|---------------------------------------------|
| w                       | Pa/s      | 850, 500, 200, 50        | Vertical velocity                           |
| z                       | m²/s²     | 850, 500, 200, 50        | Geopotential height                         |

## Temporal Aggregations

| Aggregation Type        | Description                                          | Example                                      |
|-------------------------|------------------------------------------------------|----------------------------------------------|
| **day**                 | Daily means (24-hour average)                       | Average of all 24 hours for 2020-01-01     |
| **7day**                | 7-day rolling means                                  | Average of 2020-01-01 to 2020-01-07        |
| **30day**               | 30-day rolling means                                 | Average of 2020-01-01 to 2020-01-30              |
| **90day**               | 90-day rolling means                                 | Average of 2020-01-01 to 2020-03-30              |
| **month**               | Monthly means                                        | Average of all days in January 2020        |
| **3month**              | 3-month means                                        | Average of all days in january, feburay, and march of 2020             |

## Temporal Coverage

- **Time Period:** 1940–present
- **Update Frequency:** Regular updates as new ERA5 data becomes available

## Spatial Coverage

- **Resolution:** 0.25°×0.25° global grid
- **Coverage:** Global

## Data Format and Access

- **Format:** Zarr (written with [icechunk](https://github.com/earth-mover/icechunk))
- **Storage:** Amazon S3 (`s3://planettebaikal/reanalysis/era5/prod/`)
- **Organization:** Data is organized by variable, temporal aggregation, and spatial resolution

### Data Structure

```
s3://planettebaikal/reanalysis/era5/prod/
├── {variable}/
│   ├── day/
│   ├── 7day/
│   ├── 30day/
│   ├── 90day/
│   ├── month/
│   └── 3month/
│       └── 0p25latx0p25lon/
│           └── era5_{variable}_{frequency}_0p25latx0p25lon.zarr
```

**Example paths:**
- Daily t2m: `s3://planettebaikal/reanalysis/era5/prod/t2m/day/0p25latx0p25lon/era5_t2m_day_0p25latx0p25lon.zarr`
- Monthly SLP: `s3://planettebaikal/reanalysis/era5/prod/slp/month/0p25latx0p25lon/era5_slp_month_0p25latx0p25lon.zarr`
- 7-day wind: `s3://planettebaikal/reanalysis/era5/prod/u10m/7day/0p25latx0p25lon/era5_u10m_7day_0p25latx0p25lon.zarr`

## Data Processing

### Daily Means
Daily mean files are created by averaging the hourly ERA5 data over all 24 hours for any given variable.

### Rolling Means
Rolling means are created by averaging the daily data over the specified time window. For example:
- **7-day rolling mean** for 2020-01-07 = average of daily data from 2020-01-01 to 2020-01-07
- **30-day rolling mean** for 2020-01-30 = average of daily data from 2020-01-01 to 2020-01-30

### Monthly and Seasonal Means
- **Monthly means** = average of all daily values within a calendar month
- **3-month means** = average of 3 consecutive calendar months

## Getting Started

### Prerequisites

Install the required Python packages:

```bash
pip install xarray zarr icechunk matplotlib s3fs
```

### Quick Start Example

```python
import xarray as xr
import icechunk as ic
import matplotlib.pyplot as plt

# get the bucket and prefix
variable = "t2m"  # 2-meter temperature (K)
frequency = "day"  # daily means
bucket = "planettebaikal"
prefix = f"reanalysis/era5/prod/{variable}/{frequency}/0p25latx0p25lon/era5_{variable}_{frequency}_0p25latx0p25lon.zarr"

# Get icechunk session and repo
storage = ic.s3_storage(bucket=bucket, prefix=prefix, from_env=True)
repo = ic.Repository.open(storage=storage)

# return readonly session
session = repo.readonly_session("main")

# open the data with xarray
ds = xr.open_dataset(session.store, engine="zarr", consolidated=False, decode_timedelta=True, chunks={})

# Explore the data
print(ds)

# Select a specific date and create a plot
t2m_20200101 = ds.t2m.sel(time='2020-01-01')
t2m_20200101.plot(cmap='RdBu_r', robust=True, extend='both')
plt.title('ERA5 Daily Mean 2m Temperature - 2020-01-01')
plt.show()
```

For a complete tutorial with examples of different temporal aggregations, see the [Jupyter notebook tutorial](era5_archive_tutorial.ipynb) in this repository.

## Usage Notes

- Data is derived from the NSF-NCAR ERA5 reanalysis (https://registry.opendata.aws/nsf-ncar-era5/)
- Please cite both the original ERA5 dataset, the NSF-NCAR ERA5 reanlaysis, and Planette's cloud-native archive (see citations below)
- All temporal aggregations maintain the original 0.25° spatial resolution
- Data is provided "as-is" with the same quality and limitations as the original ERA5

## License

This dataset is derived from ERA5 and follows the same licensing terms:

**CC BY 4.0:** This data is published under a Creative Commons Attribution 4.0 International (CC BY 4.0) license. You are free to share and adapt the material for any purpose, even commercially, provided that you give appropriate credit.

**Copyright:** European Centre for Medium-Range Weather Forecasts (ECMWF)

For full license details, see: https://creativecommons.org/licenses/by/4.0/
For license details from NSF-NCAR, see: https://www.ucar.edu/terms-of-use/data

## Contact

For questions about this dataset or Planette's ERA5 archive:
- **Email:** aodhan.sweeney@planette.ai
- **Organization:** Planette.ai

## Citations

### Primary ERA5 Citation
> Hersbach, H., Bell, B., Berrisford, P., Hirahara, S., Horányi, A., Muñoz‐Sabater, J., Nicolas, J., Peubey, C., Radu, R., Schepers, D., Simmons, A., Soci, C., Abdalla, S., Abellan, X., Balsamo, G., Bechtold, P., Biavati, G., Bidlot, J., Bonavita, M., De Chiara, G., Dahlgren, P., Dee, D., Diamantakis, M., Dragani, R., Flemming, J., Forbes, R., Fuentes, M., Geer, A., Haimberger, L., Healy, S., Hogan, R.J., Hólm, E., Janisková, M., Keeley, S., Laloyaux, P., Lopez, P., Lupu, C., Radnoti, G., de Rosnay, P., Rozum, I., Vamborg, F., Villaume, S., Thépaut, J.N. (2020): The ERA5 global reanalysis. Quarterly Journal of the Royal Meteorological Society, 146(730), 1999-2049. [https://doi.org/10.1002/qj.3803](https://doi.org/10.1002/qj.3803)

### Dataset Citation
> Copernicus Climate Change Service (C3S) (2017): ERA5: Fifth generation of ECMWF atmospheric reanalyses of the global climate. Copernicus Climate Change Service Climate Data Store (CDS). [https://cds.climate.copernicus.eu/cdsapp#!/home](https://cds.climate.copernicus.eu/cdsapp#!/home)
> NSF-NCAR ERA5 reanalysis [https://rda.ucar.edu/datasets/d633000/#](https://rda.ucar.edu/datasets/d633000/#)

### Planette Archive Citation
> Planette.ai (2025): Planette's ERA5 Archive - Cloud-native multi-temporal ERA5 reanalysis dataset (1940-present). Available at: [repository URL]

## References

- [ERA5 Documentation](https://confluence.ecmwf.int/display/CKB/ERA5%3A+data+documentation)
- [NSF-NCAR ERA5 Documentation](https://rda.ucar.edu/datasets/d633000/#)
- [Copernicus Climate Data Store](https://cds.climate.copernicus.eu/)
- [icechunk Documentation](https://github.com/earth-mover/icechunk)

## Acknowledgments

This work is supported by the European Centre for Medium-Range Weather Forecasts (ECMWF), the Copernicus Climate Change Service (C3S), and the AWS Open Data Sponsorship Program. 