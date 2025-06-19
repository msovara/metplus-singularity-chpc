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
 
## User Notes


## Quick Start
### 1. Prepare Your Directories

Create the required directories:
```sh
mkdir -p input config output logs
```

- Place your input data files in `input/`
- Place your METplus config files in `config/`
- `output/` and `logs/` will be populated by METplus

### 2. Edit the PBS Job Script

Edit `metplus_pbs_job.sh`:
- Set your project code, username, and email in the PBS directives.
- Set the correct path to your main METplus config file in the `CONFIG_FILE` variable.

### 3. Submit the Job

```sh
qsub metplus_pbs_job.sh
```

### 4. Check Results

- Output files will be in `output/`
- Log files will be in `logs/`
- PBS job output/error will be in the paths specified in the script

## Example PBS Job Script

```bash
#!/bin/bash
#PBS -l select=1:ncpus=24:mpiprocs=24
#PBS -P PRJT1234
#PBS -q smp
#PBS -l walltime=4:00:00
#PBS -o /mnt/lustre/users/USERNAME/OMP_test/test1.out
#PBS -e /mnt/lustre/users/USERNAME/OMP_test/test1.err
#PBS -m abe
#PBS -M youremail@ddress

ulimit -s unlimited

# === User configuration ===
SINGULARITY_IMAGE="/home/apps/chpc/earth/metplus-6.0.0-singularity-container/metplus_6.0-latest.sif"
WORKSPACE_DIR="$PBS_O_WORKDIR"
INPUT_DIR="${WORKSPACE_DIR}/input"
CONFIG_DIR="${WORKSPACE_DIR}/config"
OUTPUT_DIR="${WORKSPACE_DIR}/output"
LOGS_DIR="${WORKSPACE_DIR}/logs"
CONFIG_FILE="your_metplus_config.conf"  # Edit as needed

# === Load modules ===
module purge
module load chpc/singularity/3.5.3
module load chpc/earth/metplus/6.0.0

# === Run METplus in the container ===
cd "$WORKSPACE_DIR"
singularity exec \
  --bind "$INPUT_DIR":/input \
  --bind "$CONFIG_DIR":/config \
  --bind "$OUTPUT_DIR":/output \
  --bind "$LOGS_DIR":/logs \
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

Here is a recommended directory structure for your METplus PBS job, showing the main directories and typical subdirectories/files you might have:

```text
your_project_directory/
├── input/                # Input data (GRIB/NetCDF files)
│   ├── file1.grib
│   ├── file2.nc
│   └── ...               # More input files
├── config/               # METplus configuration files
│   ├── your_metplus_config.conf   # Main config file
│   ├── other_config.conf
│   └── ...               # More config files
├── output/               # Results directory (created by METplus)
│   └── ...               # Output files will be written here
├── logs/                 # Execution logs (created by METplus)
│   └── ...               # Log files will be written here
└── metplus_pbs_job.sh    # Your PBS job script
```

## Notes

- The Singularity image is provided by CHPC at `/home/apps/chpc/earth/metplus-6.0.0-singularity-container/metplus_6.0-latest.sif`.
- You may need to adjust resource requests and paths for your project.
- For more information on METplus, see the [official documentation](https://metplus.readthedocs.io/en/latest/).

## Support

For issues or questions, please open an issue in this repository or contact the CHPC support team at: https://users.chpc.ac.za/helpdesk/tickets/submit/

---















