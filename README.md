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
