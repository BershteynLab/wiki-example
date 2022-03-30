
# Getting Started with EMOD

## Introduction

Welcome to EMOD!

This repository and the documents contained herein are intended to serve as a basic walkthrough of how to install and run EMOD.

We will walk through the steps required to use EMOD and DTK-Tools as a black box software model. The full scope of how to build and customize an EMOD model from scratch will not be covered here.  The next recommended steps for understanding the different features and nuances of the epidemiological model are covered elsewhere.

[EMOD Software Overview](https://docs.idmod.org/projects/emod-hiv/en/latest/software-overview.html)

## A guide to the files in this directory

The present directory contains the following files and directories:

* `simtools.ini`
* `Data/calibration_ingest_form_Malawi_3_node_AB2_no_inc.xlsm` - This is the "ingest file." It contains a great deal of information which the simulation uses as inputs. This includes:
  * the list of dynamic parameters which will be fit during the calibration stage,
  * the list of calibration target variables and how the calibration stage should weight them
  * all calibration target data (ie, empirical data used to calibrate the model)
* `InputFiles`
  * `Static/Demographics_Malawi.json` - Malawi demographics file; includes all mortality and fertility data
* `InputFiles/Templates`
  * `PFA_Overlay_Malawi.json` - Malawi demographics file which adjusts the pair formation algorithm according to likelihood of pair formation between different demographic groups
  * `Risk_Assortivity_Overlay_Malawi.json` - Malawi demographics file which adjusts assortativity between agents according to risk group (ie, risk-prone agents may be more likely to form pairs with other risk-prone agents)
  * `Accessibility_and_Risk_IP_Overlay_Malawi.json` - Malawi demographics file which adjusts accessibility and risk
  * `campaign_Malawi_1Dec2021_recalib.json` - "Campaign file," which defines the full structure of the network of events which occur in the simulation. Together, this network of events describes the events which lead to transmission; the course of infection within infected individuals; and all the different ways individual agents interact with the health care system --- testing, treatment, care, etc.
  * `config.json` - a (mostly) alphabetical list of all parameters which are defined in the simulation. Not all of these parameters are necessary to build a simulation of HIV, but 
* `optim_script.py` 
  * Script for configuring the calibration stage of the simulation
* `run_scenarios.py`
  * Script for configuring and executing the simulation runs
* `scenarios.csv`
  * List of scenarios, which are run as part of `run_scenarios`

All together, these files can be used to build and run a simulation of HIV transmission in Malawi using EMOD and DTK-Tools.

It is recommended that you clone this repo in your personal folder in the Bershteyn lab group directory:

`/gpfs/data/bershteynlab/EMOD/[YOUR FOLDER]`

## Steps for Installing EMOD and DTK-Tools

Follow these steps to install EMOD and DTK-Tools on BigPurple (NYU Langone's cluster):

### Configure bash and python environment

1. `ssh` into BigPurple
2. Create a virtual python environment for the purpose of holding all the python packages that DTK-Tools depends on. Run the following in the command line:

```
python -m venv ~/environments/dtk-tools-p36
source ~/environments/dtk-tools-p36/bin/activate
```

3. Modify the bash environment to automatically launch the python environment just created. Paste the following into the `.bashrc` file:

```
# .bashrc

# Source global definitions
if [ -f /etc/bashrc ]; then
        . /etc/bashrc
fi

# User specific aliases and functions

module load singularity/3.1
module load python/cpu/3.6.5
module load git/2.17.0

source ~/environments/dtk-tools-p36/bin/activate
export PYTHONPATH=~/environments/dtk-tools-p36/lib/python3.6/site-packages
```

Run in command line: `source ~/.bashrc`

### Install DTK-Tools and Disease-Specific Packages

Use git to install DTK-Tools and packages specific to HIV. It is recommended that you install these in the `home` directory on BigPurple: `gpfs/home/[YOUR NAME]

1. Make sure that you have access to the Institute for Disease Modeling's github repos. Both dtk-tools and HIV-Analyzers are private, so your github account must be given permission to view and clone 
2. Set up a public key on BigPurple and add the key to github. This will allow you to clone and directly download EMOD and DTK-Tools from github onto BigPurple.
    * [Generate a new ssh key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
    * [Add your key to your github account](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)
3. Clone the DTK-Tools repo.
In your `home` directory on BigPurple, run the following in the command line:
    ```
    cd ~
    git clone git@github.com:InstituteforDiseaseModeling/dtk-tools.git dtk-tools-p36
    cd dtk-tools-p36
    python setup_manual.py
    ```
In the command line run `dtk -h` to confirm installation.

4. Clone the HIV-analyzers repo
In your `home` directory on BigPurple, run the following in the command line:

    ```
    cd ~
    git clone git clone git@github.com:InstituteforDiseaseModeling/HIV-Analyzers.git
    cd HIV-Analyzers
    python setup.py develop
    ```

5. Install Docker image to run simulations.
The Docker image creates a virtual environment where all of the packages and extensions required to run EMOD smoothly already exist. This saves the trouble of needing to re-install and re-configure them. Run the following in the command line:

    ```
    cd ~
    mkdir images
    cd images
    singularity pull docker://idm-docker-public.packages.idmod.org/nyu/dtk:20200306
    ```

## Guide to Simulation Setup

The `simtools.ini` file defines the paths where the simulation looks for inputs and writes outputs.

It is recommended practice to store simulation outputs in the `scratch` directory on BigPurple. This is a private directory which can be used for temporary storage. In `simtools.ini`, set the simulation root path to this location:

    ```
    # Path where the experiment/simulation outputs will be stored
    sim_root = /gpfs/scratch/[YOUR NAME]/experiments`
    ```

Set the `input_root` as the path to the current directory

    ```
    input_root = /gpfs/data/bershteynlab/EMOD/[YOUR FOLDER]/malawi2022_tutorial/InputFiles/Static
    ```

## How to calibrate

The first stage of the simulation will be to calibrate the model. This process will determine a probability distribution for each of the model parameters designated as "dynamic" which will then be used to run the simulation and produce output.

1. Open a virtual environment with `screen`
2. Request a compute node on BigPurple: 
    * `srun -p cpu_short -n 2 --mem-per-cpu=8G --time=11:00:00 --pty bash` 
    * More documentation on how to configure the different parts of this command [here](https://hpcmed.org/guide/slurm)
3. Run in command line: `python optim_script.py`

After the simulation runs, a number of new files will appear:
* A directory called `Malawi--0.001--rep3--test0`
  * Contains all of the outputs of each of 10 iterations performed by the simulation (`iter0`, `iter1`, etc)
  * Inside each `iter` folder is a set of PDF outputs which record the log-likelihood distribution of each dynamic parameter, and shows how the maximum likelihood parameter value evolves over the course of each iteration during calibration.
* Can be ignored:
  * `post_channel_config.json` 
    * This defines how the simulation population is subdivided demographically in order to match each demographic subgroup to its corresponding calibration target. A schema for the simulation output.
  * `output`
    * A directory which contains a number of `HIVCalibSite` .csv files.

## How to run a scenario

The second stage of the simulation will be to run an ensemble of simulations. The simulation will draw from the calibrated parameter values from the previous stage and produce model output which may then be analyzed and interpreted.

Run the following python command inside the compute node on BigPurple:

    ```
    python run_scenarios.py -c optim_script.py --resample-method roulette --nsamples 25 --output-dir MWI_baseline --suite-name MWI_baseline --table scenarios.csv --calib-dir Malawi--0.001--rep3--test0
    ```

* `-c` and `--calib-dir` : The command references the script used to perform the calibration (`optim_script.py` and the directory where the calibration results were stored.) 
* `--resample-method` and `--nsamples` : The command allows the user to define how the parameter values will be resampled, and the number of times to resample each parameter
* `--table` : The user provides a table of scenarios to run. In this case, only the baseline will run.
* --output-dir`: Where simulation outputs will go

After the simulation runs, a number of new files and directories will appear. The output directory will contain a set of reports. These reports will also appear in the `sim_root` directory defined in `simtools.ini`
