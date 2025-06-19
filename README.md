# METplus Singularity Setup on CHPC Lengau Cluster

This guide provides step-by-step instructions for installing and running METplus via Singularity (Apptainer) on the CHPC Lengau cluster.

## Prerequisites
- CHPC Lengau account
- Basic Linux command-line knowledge
- Project allocation on Lengau

## Installation Steps - Developer Notes

### 1. Load Singularity Module
```bash
module avail 2>&1 | grep -i singularity
module load chpc/singularity/3.5.3
```

### 2. Find the METplus Singularity Image
You can either:
- Pull the official METplus container from a registry (e.g., Sylabs Cloud or Docker Hub: https://hub.docker.com/r/dtcenter/metplus/tags), or
- Use a pre-existing image if one is available on Lengau.

To pull from Sylabs Cloud:
```bash
singularity pull docker://dtcenter/metplus:6.0-latest
```

**Note:** Could you check for newer versions and update the URL accordingly?

### 3. Verify the Image
Check if the image works and manually verify its metadata:
```bash
singularity run metplus_6.0-latest.sif
singularity inspect metplus_6.0-latest.sif
```

### 4. List Installed packages
For Debian/Ubuntu-based containers
```bash
singularity exec metplus_6.0-latest.sif dpkg --list
```

### 5. Check Libraries Installed
```bash
singularity exec metplus_6.0-latest.sif ldconfig -p
```
... End of Developer Notes
---
 User Notes
### 3. Run the Container
You can now run the container, mounting your working directory:
```bash
mkdir -p metplus_workdir/{input,config,output,logs}
cd metplus_workdir
singularity exec --bind $PWD:/workspace metplus_latest.sif bash
```
- Replace metplus_latest.sif with the actual filename if different.
- This command drops you into a shell inside the container with your current directory mounted at /workspace.
  
Where: 
- input/: Input data (GRIB/NetCDF files)
- config/: METplus configuration files
- output/: Results directory
- logs/: Execution logs
---

### 4. Use METplus Inside the Container
Inside the container, you can run METplus commands as usual. For example:
```bash
cd /workspace
run_metplus.py /workspace/path/to/your/configs
```


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












