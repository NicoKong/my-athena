# Athena++ Setup Guide 

This guide helps you set up and run Athena++, a radiation magnetohydrodynamics code.

## 1. Get Athena++
Download [Athena++](https://github.com/PrincetonUniversity/athena) and place it in your desired location. You may also use git to clone the repository.

## 2. Configure Environment

### 2.1 MacOS
#### 2.1.1 Install Homebrew
First, install [Homebrew](https://brew.sh/) if you haven't already.

#### 2.1.2 Install Dependencies
Install gcc, open-mpi and the parallelized version of hdf5:
```bash
brew install gcc open-mpi hdf5-mpi
```

### 2.2 Windows (WSL)

Athena++ runs on Windows via Windows Subsystem for Linux (WSL).

#### 2.2.1 Install WSL
Open PowerShell and run:
```PowerShell
wsl --install
```
Restart when prompted. By default, Ubuntu will be installed.

#### 2.2.2 Using VS Code with WSL

If you want to use Visual Studio Code as your editor for Athena++ inside WSL:
1. Install Visual Studio Code from https://code.visualstudio.com.
2. Install the Remote Development extension pack (includes Remote - WSL).
3. Open the Command Palette in VS Code (Ctrl+Shift+P) → Search for Remote-WSL: Connect to WSL.
4. Select your Ubuntu distribution.
5. Once connected, open your Athena++ project folder in VS Code.
6. Any terminal opened in VS Code will run inside WSL automatically.

#### 2.2.3 Install Dependencies

Open Ubuntu in WSL, then follow Linux installation (Section 2.3).

### 2.3 Linux
#### 2.3.1 Install Dependencies
```bash
sudo apt update
sudo apt install build-essential openmpi-bin libhdf5-openmpi-dev git wget vim
```

## 3. Install Miniconda and yt

### 3.1 Install Miniconda (or Anaconda)
Check [official Miniconda documentation](https://docs.anaconda.com/miniconda/) to install Miniconda according to your operating system.

### 3.2 Install yt
After Miniconda is installed and activated:
```bash
conda install -c conda-forge yt
```

## 4. Compile the Kelvin-Helmholtz Example

### 4.1 MacOS
```bash
cd athena
python3 configure.py --prob kh -mpi -hdf5 --hdf5_path=/opt/homebrew/opt/hdf5-mpi
make
```

### 4.2 Linux
First check which directory exists on your system (`/usr/lib/aarch64-linux-gnu`, `/usr/lib/x86_64-linux-gnu`, etc.), and replace the directory in the command accordingly:
```bash
cd athena
python3 configure.py --prob kh -mpi -hdf5 --include /usr/include/hdf5/openmpi \
                     --lib_path /usr/lib/x86_64-linux-gnu/hdf5/openmpi
make
```

## 5. Run the Kelvin-Helmholtz Example

### 5.1 Modify the Input File
Before running any simulation, locate the athinput file in the directory `athena/inputs/`, and change the line `file_type = vtk` in the athinput file you'll use (in this case `athena/inputs/hydro/athinput.kh-shear`) to `file_type = hdf5`. This is necessary to ensure that the output files are in hdf5 format, which is required for yt to read them.

### 5.2 Run the Simulation
```bash
./bin/athena -i inputs/hydro/athinput.kh-shear
```

## 6. Analyze the Simulation Data
Use [yt](https://yt-project.org) to analyze the generated simulation data in hdf5 format. The following is a basic example of how to loop over the output files and create slice plots of the density.
```python
import yt

for n in range(101):
    ds = yt.load("kh.out2.%05d.athdf" % n)
    slc = yt.SlicePlot(ds, "z", ["density"])
    slc.annotate_timestamp()
    slc.save()
```
Then you can execute the script with:
```bash
python3 script.py
```
