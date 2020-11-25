# Setup

# Table of contents

- [Setup](#setup)
- [Table of contents](#table-of-contents)
  - [Check your OS](#check-your-os)
  - [Install Miniconda](#install-miniconda)
  - [Create an conda environment to work in](#create-an-conda-environment-to-work-in)
  - [Clone this repo](#clone-this-repo)

## Check your OS

If you already use Linux or MacOS X, great! Ignore this paragraph!. If you use Windows, setup a Linux virtual machine (VM) with Vagrant (see instructions on how to do this [here](https://snakemake.readthedocs.io/en/stable/tutorial/setup.html#setup-a-linux-vm-with-vagrant-under-windows)).

## Install Miniconda

Information on how to install Miniconda can be found [on their website](https://docs.conda.io/en/latest/miniconda.html). Snakemake also provides information on installing Miniconda in [their documentation](https://snakemake.readthedocs.io/en/stable/tutorial/setup.html#step-1-installing-miniconda-3)

Once miniconda is installed, [set up your channels](https://bioconda.github.io/user/install.html#set-up-channels) (channels are locations where packages/software are can be installed from)

```bash
conda config --add channels defaults
conda config --add channels bioconda
conda config --add channels conda-forge
```

## Create an conda environment to work in

With Miniconda, we can create a conda environment which acts as a space contained from the rest of the machine in which to install all the necessary software to run our workflow, supporting the portability and reproducibility of your workflow


Create a conda environment (called `demo_workflow_env`) that has python and Snakemake installed (and all it dependant software)

```bash
conda create -n demo_workflow_env python=3.7.6 snakemake=5.28.0
```

Respond yes to the following prompt to install the necessary software in the new conda environment:

```bash
Proceed ([y]/n)?
```

Activate the conda environment we just created

```bash
conda activate demo_workflow_env
```

Now we can see which conda environment we are in on the command line

```bash
(demo_workflow_env) lkemp@Wintermute:~$
```

*Snakemake has been installed within your `demo_workflow_env` environment, so you won't be able to see or use your Snakemake install unless you are within this environment*

## Clone this repo

```bash
git clone https://github.com/leahkemp/RezBaz2020_snakemake_workshop.git
cd RezBaz2020_snakemake_workshop
```
