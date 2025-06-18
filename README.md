# METplus Singularity Setup on CHPC Lengau Cluster

This guide provides step-by-step instructions for installing and running METplus via Singularity (Apptainer) on the CHPC Lengau cluster.

## Prerequisites
- CHPC Lengau account
- Basic Linux command-line knowledge
- Project allocation on Lengau

## Installation Steps

### 1. Load Singularity Module
```bash
module load chpc/apptainer/1.2.4
```

### 2. Download METplus Container
```bash
cd $SCRATCH
wget https://dtcenter.org/sites/default/files/community-code/metplus/singularity/images/metplus-5.0.0.sif
```
**Note:** Check for newer versions and update the URL accordingly.

### 3. Create Workspace
```bash
mkdir -p metplus_workdir/{input,config,output,logs}
cd metplus_workdir
```
Where: 
- input/: Input data (GRIB/NetCDF files)
- config/: METplus configuration files
- output/: Results directory
- logs/: Execution logs
---

## Configuration Setup

### 1. Create METplus Config
Create your configuration file in config/:
```bash
nano config/my_config.conf
```

### 2. Sample Configuration Snippets
```ini
[dir]
INPUT_BASE = /input
OUTPUT_BASE = /output

[filename_templates]
FCST_GRID_STAT_INPUT_TEMPLATE = {init?fmt=%Y%m%d%H}/wrfprs_{init?fmt=%H}.grb
```
---

## Running METplus
### Basic Test
```bash
singularity exec \
  --bind $PWD/input:/input \
  --bind $PWD/config:/config \
  --bind $PWD/output:/output \
  $SCRATCH/metplus-5.0.0.sif \
  /usr/local/bin/run_metplus.py -c /config/my_config.conf
```

### Submit job:
```bash
qsub run_metplus.pbs
```

### Suggested GitHub Repository Structure 
```txt
METplus-Lengau-Guide/
├── METplus_Lengau_Guide.md # This documentation
├── sample_configs/ # Example METplus config files
│ ├── grid_stat.conf
│ └── point_stat.conf
├── job_scripts/ # PBS scripts
│ ├── serial_run.pbs
│ └── parallel_run.pbs
└── data_prep_scripts/ # Data preprocessing
├── grib_to_netcdf.sh
└── wps_met_prep.py
```












