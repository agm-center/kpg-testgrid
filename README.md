# KPG 193 Test System

![License: ODbL](https://img.shields.io/badge/License-ODbL_1.0-blue.svg)
![Version: v1.5](https://img.shields.io/badge/Version-v1.5-green.svg)
![Status: Active](https://img.shields.io/badge/Status-Active-brightgreen.svg)

![KPG 193 System Overview](kpg193_v1_5/kpg193.png)
*Network topology of the KPG 193 test system showing 193 buses across the Korean peninsula.*

## What is KPG 193?

KPG 193 is a **synthetic Korean power grid test system** developed by the [AGM Center](https://agm.kentech.ac.kr/) at [KENTECH](https://www.kentech.ac.kr/) using publicly available, open data sources. It provides a foundational framework for modeling and analyzing the Korean power grid — filling a gap where no publicly available, standardized test system existed for the Korean transmission network.

The test system is designed to support a broad range of power system research, including:

- **Decarbonization studies** — Evaluate renewable integration pathways and carbon reduction strategies.
- **Optimal power flow (OPF)** — Benchmark optimization algorithms on a realistic Korean network topology.
- **Resource adequacy and reliability** — Assess generation sufficiency under diverse operating conditions.
- **Renewable energy integration** — Analyze the impact of wind, solar, and hydro generation with full temporal resolution.

KPG 193 is built from publicly available Korean energy statistics — and is released under the Open Database License (ODbL 1.0) to enable reproducible, transparent research.

## Key Features

- **MATPOWER-compatible**: Standard MATPOWER v2 case file for immediate use in MATLAB, Octave, and MATPOWER-based tools.
- **Geo-referenced buses**: All 193 buses include latitude/longitude coordinates and bilingual (Korean/English) names.
- **Full temporal resolution**: 8,760 hourly data points (365 daily CSV files) for demand, weather, and renewable profiles.
- **Renewable capacity by bus**: Wind, solar, and hydro generation capacity data at each bus.
- **Weather-coupled profiles**: Meteorological data (temperature, wind speed, longwave flux) linked to renewable generation profiles.
- **Open and reproducible**: All data released under ODbL 1.0 with transparent sourcing.
- **Versioned and maintained**: Regular updates with documented version history.

## System Overview

| Component | Count |
|-----------|------:|
| Buses | 193 |
| Conventional generators | 122 |
| Transmission lines (AC) | 359 |
| DC lines | 1 |
| Voltage levels | 154 / 345 / 765 kV |
| Renewable types | Wind, Solar, Hydro |
| Temporal resolution | 8,760 hours (24 h × 365 days) |

## Repository Structure

```
kpg193_v1_5/
├── network/
│   ├── m/
│   │   └── KPG193_ver1_5.m              # MATPOWER v2 case file
│   └── location/
│       └── bus_location.csv              # Bus geo-coordinates (lat/lon)
├── renewables_capacity/
│   ├── wind_generators_2022.csv          # Wind capacity by bus
│   ├── solar_generators_2022.csv         # Solar capacity by bus
│   └── hydro_generators_2022.csv         # Hydro capacity by bus
└── profile/
    ├── demand/
    │   └── daily_demand_{1..365}.csv     # Hourly demand (P, Q) per bus
    ├── weather/
    │   └── weather_{1..365}.csv          # Hourly meteorological data per bus
    └── renewables/
        └── renewables_{1..365}.csv       # Hourly renewable generation ratios per bus
```

## Data Description

### Network Case File — `KPG193_ver1_5.m`

A standard MATPOWER v2 case file (`baseMVA = 100`) containing the following data structures:

| Structure | Description |
|-----------|-------------|
| `mpc.bus` | 193 buses with type, demand (Pd/Qd), voltage limits, base kV, and area |
| `mpc.gen` | 122 conventional generators with dispatch limits and voltage setpoints |
| `mpc.branch` | 359 AC transmission lines/transformers with impedance and rating data |
| `mpc.gencost` | Generator cost curves (polynomial format) |
| `mpc.genthermal` | Thermal generator parameters |
| `mpc.areas` | Area definitions with price reference buses |
| `mpc.dcline` | HVDC line data |
| `mpc.dclinecost` | HVDC line cost data |

### Bus Locations — `bus_location.csv`

| Column | Description |
|--------|-------------|
| `bus_id` | Bus number (1–193) |
| `Latitude` | Geographic latitude (WGS 84) |
| `Longitude` | Geographic longitude (WGS 84) |
| `name_Korean` | Bus name in Korean |
| `name_English` | Bus name in English |

### Renewable Capacity — `{wind,solar,hydro}_generators_2022.csv`

| Column | Description |
|--------|-------------|
| `bus_ID` | Bus number (1–193) |
| `Type` | Generator type (`Wind`, `Solar`, or `Hydro`) |
| `Pmax [MW]` | Maximum active power capacity |
| `Pmin [MW]` | Minimum active power output |

### Demand Profiles — `daily_demand_{d}.csv`

One file per day (`d` = 1–365), each containing 24 × 193 rows.

| Column | Description |
|--------|-------------|
| `hour` | Hour of the day (1–24) |
| `bus_id` | Bus number (1–193) |
| `demandP` | Active power demand (MW) |
| `demandQ` | Reactive power demand (Mvar) |

### Weather Profiles — `weather_{d}.csv`

One file per day (`d` = 1–365), each containing 24 × 193 rows.

| Column | Description |
|--------|-------------|
| `hour` | Hour of the day (1–24) |
| `bus_id` | Bus number (1–193) |
| `net_downward_longwave_flux_W/m^2` | Net downward longwave radiation flux (W/m²) |
| `temperature_2m_K` | Air temperature at 2 m height (K) |
| `wind_u_93m_m/s` | Eastward wind component at 93 m (m/s) |
| `wind_v_93m_m/s` | Northward wind component at 93 m (m/s) |
| `wind_speed_93m_m/s` | Wind speed at 93 m (m/s) |

### Renewable Profiles — `renewables_{d}.csv`

One file per day (`d` = 1–365), each containing 24 × 193 rows. Values represent normalized generation ratios (0–1) relative to installed capacity.

| Column | Description |
|--------|-------------|
| `hour` | Hour of the day (1–24) |
| `bus_id` | Bus number (1–193) |
| `pv_profile_ratio` | Solar PV generation ratio |
| `wind_profile_ratio` | Wind generation ratio |
| `hydro_profile_ratio` | Hydro generation ratio |

## Getting Started

### 1. Download the Data

Clone this repository:

```bash
git clone https://github.com/agm-center/kpg-testgrid.git
cd kpg-testgrid
```

### 2. Load in MATLAB / Octave

Add the case file to your MATLAB path and load it directly:

```matlab
addpath('kpg193_v1_5/network/m');
mpc = KPG193_ver1_5;

% Run a power flow
results = runpf(mpc);

% Run an optimal power flow
results = runopf(mpc);
```

### 3. Load in Python

Use pandas to work with the CSV data:

```python
import pandas as pd

# Load bus locations
buses = pd.read_csv('kpg193_v1_5/network/location/bus_location.csv')

# Load renewable capacity
wind = pd.read_csv('kpg193_v1_5/renewables_capacity/wind_generators_2022.csv')
solar = pd.read_csv('kpg193_v1_5/renewables_capacity/solar_generators_2022.csv')

# Load a single day's demand profile (e.g., day 1)
demand = pd.read_csv('kpg193_v1_5/profile/demand/daily_demand_1.csv')

# Load all 365 days into one DataFrame
demand_all = pd.concat(
    [pd.read_csv(f'kpg193_v1_5/profile/demand/daily_demand_{d}.csv').assign(day=d)
     for d in range(1, 366)],
    ignore_index=True,
)
```

To load the MATPOWER case file in Python, you can use [oct2py](https://oct2py.readthedocs.io/) (requires GNU Octave):

```python
from oct2py import octave
octave.addpath('kpg193_v1_5/network/m')
mpc = octave.KPG193_ver1_5()
```

## Data Sources

For a detailed description of the data construction methodology, see the accompanying paper [https://arxiv.org/abs/2411.14756](https://arxiv.org/abs/2411.14756).

## Version History

| Version | Updated | Major Changes |
|---------|---------|---------------|
| **v1.5** | 2025-12 | Generator locations corrected |
| **v1.4** | 2025-09 | Bus numbering standardized, improved clustering |
| **v1.3** | 2025-07 | Renewable generation profiles fixed |
| **v1.2** | 2025-02 | Updated generator costs/capacities |
| **v1.1** | 2024-12 | Changed slack bus (bus 1 → bus 52) |
| **v1.0** | 2024-11 | Initial public release |

## Citation

If you use the KPG 193 test system in your work, we kindly request that you cite the following paper:

```bibtex
@article{song2024kpg,
  title={KPG 193: A Synthetic Korean Power Grid Test System for Decarbonization Studies},
  author={Song, Geonho and Kim, Jip},
  journal={arXiv preprint arXiv:2411.14756},
  year={2024}
}
```

## Documentation

For detailed documentation, tutorials, and additional resources, visit the [AGM Center Documentation](https://agm.kentech.ac.kr/documentation/kpg-test-system/).

## License

The test system is made available under the [Open Database License (ODbL 1.0)](http://opendatacommons.org/licenses/odbl/1.0/).

## Contact

If you have any questions or suggestions, email us at: egolab.kpg@gmail.com
