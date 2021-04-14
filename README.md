Python/IPython tool used for calculating NMR relaxation rates from molecular dynamics trajectories
=========

This is a reimplementation of a Fortran-based code called DynPro (for Dynamic Properties) developed previously in the computational chemistry lab of Dr. Jochen Autschbach.
This repository contains the code and helper scripts used to 

(1) Pack simulation boxes and write Quantum Espresso (QE) aimd inputs,

(2) Extract clusters from aimd and write inputs for EFG calculations to be run in ADF and/or QE-GIPAW, 

(3) Submit EFG calcs, extract relevant data from EFG outputs, and resubmit any missing, 

(4) Calculate quadrupolar NMR relaxation rates.

Note: Input files and submission scripts generated by this code are designed for use on the Center for Computational Research (CCR) at the University at Buffalo, State University of New York. For purposes of internal use, this is the default. Take care and make changes to those generated inputs according to your architecture if necessary.

# Prerequisites, dependencies, and setup:
- anaconda python 3.X:
    - Navigate to https://docs.conda.io/en/latest/miniconda.html, choose the installer for your operating system, and download it.
    - For Linux, then run the downloaded script:
     `./Miniconda3-latest-Linux-x86_64.sh`
- exa/exatomic (https://github.com/exa-analytics):

     `conda install pip`
     
     `pip install exa`
     
     `pip install exatomic`
    - For jupyter notebook functionality:
    
     `conda install -c conda-forge notebook`
     
     `jupyter nbextension install --py --symlink --sys-prefix exatomic`
     
     `jupyter nbextension enable --py --sys-prefix exatomic`
- Install the following packages via `conda install <package>`:
    - pandas
    - scipy
    - seaborn
    - datetime
    
    Note, to update all conda packages:
    
    `conda update --all`

    - Tinker Molecular modeling package:
          -Download the proper compiled binary (executable) packages for your operating system from https://dasher.wustl.edu/tinker/
          -Add directory containing executables to `$PATH`

     - Add dynpy directory to `$PYTHONPATH` environment variable. You may have to give yourself permissions on the scripts in `dynpy/`

Step-by-step instructions for running the code via command line input are procided below. In addition, for steps (2)-(4), and for generating various useful plots,
you may choose to utilize the Ipython notebook packaged in the notebooks directory of this repository.
# (1) Pack simulation boxes and write qe aimd inputs:
`cd trajectories`

`./mkbox.sh`

See helper bash script mkbox.sh, which calls tinker executables and functions from `tinkertoys.py` to prepare MD simulation boxes for Quantum espresso. Steps are commented in `mkbox.sh`. Define user variables at top of script. Coordinate files for solute, solvent, and counterion must be in tinker (.txyz) format and in `./trajectories` directory
along with `tinker.key` and parameter file `oplsaa.prm`. Currently, the packing code is only tested for monatomic solutes and non-cyclical solvent molecules. In past projects, packing with cyclical molecules resulted in interlocking rings. If you want to construct a system other than Iodide in water, currently one block of the QE MD inputs must be changed manually. That is the `ATOMIC_SPECIES` block which defines which Pseudopotentials to use for each atom type. This can be done in the tinkertoys.py script or (more safely) in the written QE input files. The script writes first four steps of aimd: initial wf opt (`*.inp` and `*inp.2`), NVT heating (`*.inp.3`), and first 5ps NVE production (`*inp.4`) as well as slurm script for submission on CCR.

The slurm script generated is a starting point but you may need to change it to fit your environment. Use the same `.slm` when submitting each subsequent MD input. Simply change the name of the input you want to submit to `*.inp` (no number suffix). Also, if you want to save the standard output files, make sure after each step to rename them to `*.out.$StepNumber` so that the subsequent runs don't overwrite it.

# (2) Extract clusters from aimd and write EFG inputs:
`python dynpy.py --inputs`
    
Use to generate ADF and/or QE-PAW inputs for EFG calculations after aiMD has been run. Requires `.pos` trajectory file from QE scratch to parse coordinates.

`neighbors_input.py` is used as an input file for this module. See `neighbors_input.py` for description of input parameters and to adjust them. By default, parses `I-01.pos` trajectory file in `./example-data/01`

Note that this package comes with example inputs already generated for 4 frames of the example trajectory. As a check, you may choose to inspect these or save them elsewhere before running this module as they will be overwritten.

# (3)  Submit batch of EFG calculations:
`ipython batch.ipy`

Note: This script was made for use at the Center for Computational Research (CCR) at the University at Buffalo, State University of New York. Likely, you will have to adapt this script if you are doing your computations somewhere else.

Use to submit set of ADF or QE-PAW EFG calcs on CCR. This script and all directories containing inputs and .slm files must be uploaded to CCR first.
Some parameters to define are described at the top of `batch.ipy`. The initial run will submit all inputs. After parsing data in step (4), frames with missing data
will be written to a file, and subsequent runs of batch.ipy will submit only those missing calculations.

# To extract relevant data from EFG outputs:
`python dynpy.py --parse-efgs <ADF/GIPAW> <path to trajectories> <system/calc description>`

Use to parse EFG data from ADF or GIPAW outputs after calculations are completed. First argument is either 'ADF' or 'GIPAW' telling which outputs to parse.
Last two arguments are optional. Defaults are `./example-data/`, and same as <ADF/GIPAW> respectively. Use these to test with example data.

# (4)  Calculate quadrupolar NMR relaxation rates and related quantities:
`python dynpy.py --relax <efg data file> <analyte symbol> <numeric analyte label>`
      
 Analyte symbol is mass number followed by element symbol e.g. 127I.
 Last argument is optional (use if you know atom labels are consistent in efg data and you want to designate analyte by numeric label)
 To test, use the data in `example-data/I-QZ4P-efg.csv` (analyte = 127I) to generate data which can be compared to our published work (will insert doi later...)

# Notebook functionality
Interactive use of this code along with plotting examples can be found in `./notebooks/dynpy.ipynb`
To start up a jupyter notebook kernel, from a terminal enter the notebooks directory and type:
`jupyter-notebook`
If this does not automatically open your browser, open your internet browser and navigate to the 'localhost' address shown in the terminal messages.

Other available notebooks packages here are:
 Structure-dynamics.ipynb: For calculating RDFs, and correlation functions based on the molecular dynamics trajectories.
 Dipolar.ipynb: For dipole-dipole proton NMR relaxation from MD
