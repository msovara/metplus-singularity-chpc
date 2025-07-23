# 🚀 METplus Singularity Setup on CHPC Lengau Cluster

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.15790478.svg)](https://doi.org/10.5281/zenodo.15790478)

This guide provides step-by-step instructions for installing and running METplus via Singularity (Apptainer) on the CHPC Lengau cluster.

## 📋 Prerequisites
- CHPC Lengau account
- Basic Linux command-line knowledge
- Project allocation on Lengau

## 🔧 Installation Steps - Developer Notes

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

ℹ️ **Note:** Could you check for newer versions and update the URL accordingly?

### 3. ✅ Verify the Image
Check if the image works and manually verify its metadata:
```bash
singularity run metplus_6.0-latest.sif
singularity inspect metplus_6.0-latest.sif
```

### 4. 📦 List Installed packages
For Debian/Ubuntu-based containers:
```bash
singularity exec metplus_6.0-latest.sif dpkg --list
```

### 5. 📚 Check Libraries Installed
```bash
singularity exec metplus_6.0-latest.sif ldconfig -p
```
... End of Developer Notes.
---
 
## 👤 User Notes
## 🚀 Quick Start

### 1. 🗂️ Prepare Your Directories
Create the required directories:
```sh
mkdir -p input config output logs
```
### 📁 Directory Structure
```text
your_project_directory/
├── 📂 input/                # Input data
│   ├── file1.grib
│   ├── file2.nc
│   └── ...
├── 📂 config/               # Configuration files
│   ├── your_metplus_config.conf
│   └── ...
├── 📂 output/               # Results directory
├── 📂 logs/                 # Execution logs
└── 📄 metplus_pbs_job.sh    # PBS job script
```
- Place your input data files in `input/`
- Place your METplus config files in `config/`
- `output/` and `logs/` will be populated by METplus

### 2. ✏️ Edit the PBS Job Script

Edit `metplus_job.pbs`:
- Set your project code, username, and email in the PBS directives.
- Set the correct path to your main METplus config file in the `CONFIG_FILE` variable.

### 3. ⚡ Submit the Job

```sh
qsub metplus_job.pbs
```
### 4. 🔍 Check Results
- Output files will be in `output/`
- Log files will be in `logs/`
- PBS job output/error will be in the paths specified in the script

## 📜 Modular PBS Job Script Example ```metplus_job.pbs```

```bash
#!/bin/bash
#PBS -l select=1:ncpus=1:mpiprocs=1
#PBS -P ERTH1022
#PBS -q serial
#PBS -l walltime=1:00:00
#PBS -o /mnt/lustre/users/lmakgati/METplus_validation/met.out
#PBS -e /mnt/lustre/users/lmakgati/METplus_validation/met.err
#PBS -m abe
#PBS -M youremailaddress

# === Unset limits ===
ulimit -s unlimited

# === Define Paths ===
WORKSPACE_DIR="$PBS_O_WORKDIR"
SINGULARITY_IMAGE="/home/apps/chpc/earth/metplus-6.0.0-singularity-container/metplus_6.0-latest.sif"
CONFIG_FILE="Rainfall.conf"

# Internal directories
INPUT_DIR="${WORKSPACE_DIR}/input"
CONFIG_DIR="${WORKSPACE_DIR}/config"
OUTPUT_DIR="${WORKSPACE_DIR}/output"
LOGS_DIR="${WORKSPACE_DIR}/logs"
#TMPDIR="${WORKSPACE_DIR}/tmp"

# External data sources
MODEL_DATA_DIR="/mnt/lustre/users/lmakgati/model_data"
OBS_DATA_DIR="/mnt/lutre/users/lmakgati/obs_data"

# === Validation Checks ===
if [ ! -f "${CONFIG_DIR}/${CONFIG_FILE}" ]; then
    echo "ERROR: Config file ${CONFIG_DIR}/${CONFIG_FILE} not found!"
    exit 1
fi

# === Load Environment Modules ===
module purge
module load chpc/singularity/3.5.3
module load chpc/earth/metplus/6.0.0

# === Log Run Info ===
echo "Starting METplus run at $(date)"
echo "Working directory       : $WORKSPACE_DIR"
echo "Singularity image       : $SINGULARITY_IMAGE"
echo "Configuration file      : ${CONFIG_DIR}/${CONFIG_FILE}"
echo "Model data mount        : $MODEL_DATA_DIR -> /model_data"
echo "Observation data mount  : $OBS_DATA_DIR -> /obs_data"

# === Run METplus in Singularity Container ===
cd "$WORKSPACE_DIR"
singularity exec \
  --env INPUT="$INPUT_DIR",OUTPUT="$OUTPUT_DIR",CONFIG="$CONFIG_DIR",LOGS="$LOGS_DIR" \
  --bind "$INPUT_DIR":/input \
  --bind "$CONFIG_DIR":/config \
  --bind "$OUTPUT_DIR":/output \
  --bind "$LOGS_DIR":/logs \
  --bind "$MODEL_DATA_DIR":/model_data \
  --bind "$OBS_DATA_DIR":/obs_data \
  "$SINGULARITY_IMAGE" run_metplus.py /config/"$CONFIG_FILE"
```
This script binds the following directories for use inside the container:
- input/ (for input data)
- config/ (for METplus configuration files)
- output/ (for results)
- logs/ (for execution logs)
Assumes these directories are subdirectories of your job submission directory ($PBS_O_WORKDIR).
- Loads the Singularity module 
- Checks for the container image
- Binds the current working directory (or a specified directory) to /workspace inside the container
- Passes all arguments to run_metplus.py
- Handles errors and provides helpful messages

---

## Notes

- The Singularity image is provided by CHPC at `/home/apps/chpc/earth/metplus-6.0.0-singularity-container/metplus_6.0-latest.sif`.
- You may need to adjust resource requests and paths for your project.
- For more information on METplus, see the [official documentation](https://metplus.readthedocs.io/en/latest/).

## Support

For issues or questions, please open an issue in this repository or contact the CHPC support team at: https://users.chpc.ac.za/helpdesk/tickets/submit/

--- 

## 📚 Citation
If you use this repository or its contents in your work, please cite it as follows:

Sovara, M. (2025). METplus Port to Lengau HPC – Job Scripts and Integration Examples (v1.0.0). Zenodo. https://doi.org/10.5281/zenodo.15790478

```bibtex
@software{sovara_2025,
  author       = {Sovara, Mthetho},
  title        = {{METplus Port to Lengau HPC – Job Scripts and Integration Examples}},
  version      = {v1.0.0},
  date         = {2025-07-02},
  publisher    = {Zenodo},
  doi          = {10.5281/zenodo.15790478},
  url          = {https://doi.org/10.5281/zenodo.15790478}
}
```

## 🔖 License
MIT License for job scripts and configuration files.
METplus itself is licensed separately, see METplus GitHub (https://github.com/dtcenter/METplus) for details.

## 🤝 Acknowledgements
This work builds on the open-source METplus framework developed by the Developmental Testbed Center (DTC) and NCAR.

---















