Python/IPython tool used for calculating NMR relaxation rates from molecular dynamics trajectories
=========

DynPy is a Python re-implementation and extension of a Fortran code called DynPro ('Dynamic Properties') developed previously in the computational chemistry lab of Dr. Jochen Autschbach. https://ja01.chem.buffalo.edu/
See Badu et al. (2013) in file NOTES. If you use the software provided in this repository for a research project, we would appreciate it if you cite one or more of the articles by Philips, Autschbach, et al. listed in file NOTES.

The current version of DynPy is capable of calculating quadrupolar NMR relaxation from molecular dynamics simulations and calculations of electric field gradient (EFG) tensors along the trajectories. Code for other relaxation mechanisms will be added at a later time. 


This repository contains the code and helper scripts used to 

(1) Extract clusters from MD and write inputs for EFG calculations to be run with the Amsterdam Density Functional (ADF, https://www.scm.com/) program or with the GIPAW module of QE.

(2) extract relevant data from EFG outputs,

(3) Calculate quadrupolar NMR relaxation rates.

(4) There is also functionality to compute dipolar and spin-rotation relaxation rates from MD trajectory data

Note: Input files and job scripts generated by this software are designed for use on the computing resources at the Center for Computational Research (CCR) at the University at Buffalo, State University of New York. Likely, you will have to adapt the templates in the input parameter file dynpy_EFG_inputs_from_MD.py if you are doing your computations somewhere else or if you want to use different programs for EFG calculations.

We will not be able to provide support for DynPy. However, if you find a bug, 
we would welcome a bug report with a minimal use case to reproduce it.



# Prerequisites, dependencies, and setup:
- anaconda python 3.X:
    - Navigate to https://docs.conda.io/en/latest/miniconda.html and download the installer for your operating system. 
    - For Linux, then run the downloaded script:
     `./Miniconda3-latest-Linux-x86_64.sh`

- Install the following packages via `conda install <package>`:
    - pandas
    - scipy
    - numba
    - networkx
    - datetime
    - psutil
    
    Note, to update all conda packages:
    
    `conda update --all`

    - Add dynpy directory to `$PYTHONPATH` environment variable. You may have to give yourself execute permission for the scripts in `dynpy/`
 
- We highly recommend downloading the dynpy-examples dataset to test the software and learn its main functionalities. It is available at https://zenodo.org/records/14226242
    - Add the unzipped dynpy-examples directory to the top-level dynpy directory for proper functionality of the examples (and notebooks)

Step-by-step instructions for running the examples in the above package via command line input are procided below. In addition, you may choose to utilize the Ipython notebook packaged in the notebooks directory of this repository.

# Extract clusters from MD, write EFG inputs, extract EFG results, and compute quadrupolar rates:
This example demonstrates how to generate snapshot configurations from an MD trajectory, generate EFG calc inputs, extract results from outputs, and finally, calculate quadrupolar relaxation rates.
The example system is a CPMD simulation of aqueous Iodide from Quantum Espresso. Again, this requires download of the above Zenodo repository to obtain both the MD data sets and the .py input parameter files used as the arguments in the following commands. Steps to run the example are as follows:

To generate snapshots (nearest neighbor clusters and full periodic) and corresponding EFG inputs (ADF and QE-PAW):

	python -m dynpy --inputs dynpy_EFG_inputs_from_MD.py

Input parameters and descriptions are found in the input file dynpy_EFG_inputs_from_MD.py. MD trajectory files and subsequently generated EFG inputs/outputs must be in top-level directories named '01','02',..'XX'.
Directory 'pre-computed-EFGs' exemplifies required directory structure for snapshot EFG inputs/outputs. Running the code above will generate inputs in directory ./'01' following this convention.

To extract EFG data from packaged pre-computed EFGs, run the following:

	python -m dynpy --parse-efgs dynpy_parse_efgs.py

This will write raw EFG time series data to .csv file.

To calculate autocorrelation functions and relaxation rates from EFGs:
	
	python -m dynpy --Qrelax dynpy_qrax_params.py

Directions for remaining examples are in individual README files located in the corresponding directory in dynpy-examples (download from Zenodo repo above). Details on individual input parameters are provided as comments in the input parameter .py files (e.g. dynpy_EFG_inputs_from_MD.py)

# Notebook functionality
Interactive use of this code along with plotting examples can be found in `./notebooks/`
To start up a jupyter notebook kernel, from a terminal enter the notebooks directory and type:
`jupyter-notebook`
If this does not automatically open your browser, open your internet browser and navigate to the 'localhost' address shown in the terminal messages.
