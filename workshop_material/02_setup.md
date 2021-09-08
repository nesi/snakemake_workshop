# 02 - Setup

# Table of contents

- [02 - Setup](#02---setup)
- [Table of contents](#table-of-contents)
  - [Setup for running on NeSI](#setup-for-running-on-nesi)
    - [Connect to Jupyter on NeSI](#connect-to-jupyter-on-nesi)
    - [Setting up Miniconda](#setting-up-miniconda)
  - [Setup for running on your machine](#setup-for-running-on-your-machine)
    - [Install Miniconda](#install-miniconda)
      - [Check your OS](#check-your-os)
      - [Installing miniconda](#installing-miniconda)
  - [Create a conda environment](#create-a-conda-environment)
  - [Clone this repo](#clone-this-repo)


## Setup for running on NeSI

Choose between this section and [Setup for running on your machine](#setup-on-your-machine) (don't do both).

### Connect to Jupyter on NeSI

1. Follow https://jupyter.nesi.org.nz/hub/login
2. <p>Enter NeSI username, HPC password and 6 digit second factor token<br><img src="" alt="drawing" width="720"/></p>
3. <p>Choose server options as below<br><img src="workshop_material/images/nesi99991_screenshot.png" alt="drawing" width="700"/></p>


Connect to [Jupyter on NeSI](https://jupyter.nesi.org.nz/) and login with your **NeSI HPC account** credentials (username, password and second factor as set on [MyNeSI](https://my.nesi.org.nz/account/hpc-account)).


From the launcher within JupyterLab you can select the "Terminal" button to launch a terminal session.

When you connect to NeSI JupyterLab you always start in a new hidden directory. To make sure you can find your work next time, first switch to your home directory in the terminal:

```bash
cd
```

### Setting up Miniconda

On NeSI we have lots of software preinstalled to simplify things for our users. To load Miniconda run the following in the terminal:

```bash
ml purge
ml Miniconda
```

Next, [set up your channels](https://bioconda.github.io/user/install.html#set-up-channels) (channels are locations where packages/software are can be installed from)

```bash
conda config --add channels defaults
conda config --add channels bioconda
conda config --add channels conda-forge
```

**TODO**: setup conda to install to project dir??? other conda config???


Continue to [Create a conda environment](#creating-a-conda-environment).


## Setup for running on your machine

Choose between this section and [Setup for running on NeSI](#setup-for-running-on-nesi) (don't do both).

### Install Miniconda

For this workshop, will analyse our data using various software. However, the only software we will need to manually install is [Miniconda](https://docs.conda.io/en/latest/miniconda.html).

#### Check your OS

If you already use Linux or MacOS X, great! Ignore this paragraph!. If you use Windows, setup a Linux virtual machine (VM) with Vagrant (see instructions on how to do this [here](https://snakemake.readthedocs.io/en/stable/tutorial/setup.html#setup-a-linux-vm-with-vagrant-under-windows)).

#### Installing miniconda

Information on how to install Miniconda can be found [on their website](https://docs.conda.io/en/latest/miniconda.html). Snakemake also provides information on installing Miniconda in [their documentation](https://snakemake.readthedocs.io/en/stable/tutorial/setup.html#step-1-installing-miniconda-3)

Once miniconda is installed, [set up your channels](https://bioconda.github.io/user/install.html#set-up-channels) (channels are locations where packages/software are can be installed from)

```bash
conda config --add channels defaults
conda config --add channels bioconda
conda config --add channels conda-forge
```

## Create a conda environment

With Miniconda, we can create a conda environment which acts as a space contained from the rest of the machine in which our workflow will automatically install all the necessary software it uses, supporting the portability and reproducibility of your workflow.

Create a conda environment (called `snakemake_env`) that has Snakemake installed (and all it's dependant software)

```bash
conda create -n snakemake_env snakemake
```

Respond yes to the following prompt to install the necessary software in the new conda environment:

```bash
Proceed ([y]/n)?
```

**Note. this installed Snakemake version 6.7.0 for me, you can use the same version this workshop was created with `conda create -n snakemake_env snakemake=6.7.0`**

Activate the conda environment we just created

```bash
conda activate snakemake_env
```

Now we can see which conda environment we are in on the command line, `(base)` has been replaced with `(snakemake_env)`

```bash
(snakemake_env) orac$ 
```

*Snakemake has been installed within your `snakemake_env` environment, so you won't be able to see or use your Snakemake install unless you are within this environment*

## Clone this repo

**TODO**: Provide instructions for loading a git module on NeSi?

To clone this repo (and use the example data I have provided), you will need to have git installed. See [this guide](https://www.atlassian.com/git/tutorials/install-git) for help with an installation of git.

Once git is installed, clone this repo with the following:

```bash
git clone https://github.com/nesi/snakemake_workshop.git
cd snakemake_workshop
```

See the [Git Guides](https://github.com/git-guides/git-clone) for information on cloning a repo

<p align="left"><b><a href="https://nesi.github.io/snakemake_workshop/workshop_material/01_introduction.html">Previous page: 01 - Introduction</a>
<p align="right"><b><a href="https://nesi.github.io/snakemake_workshop/workshop_material/03_create_a_basic_workflow.html">Next page: 03 - Create a basic workflow</a>
<p align="center"><b><a href="https://nesi.github.io/snakemake_workshop/">Homepage</a>
