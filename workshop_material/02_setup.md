# 02 - Setup

# Table of contents

- [02 - Setup](#02---setup)
- [Table of contents](#table-of-contents)
  - [Install Miniconda](#install-miniconda)
    - [Check your OS](#check-your-os)
    - [Installing miniconda](#installing-miniconda)
  - [Create a conda environment](#create-a-conda-environment)
  - [Clone this repo](#clone-this-repo)

## Install Miniconda

For this workshop, will analyse our data using various software. However, the only software we will need to manually install is [Miniconda](https://docs.conda.io/en/latest/miniconda.html).

### Check your OS

If you already use Linux or MacOS X, great! Ignore this paragraph!. If you use Windows, setup a Linux virtual machine (VM) with Vagrant (see instructions on how to do this [here](https://snakemake.readthedocs.io/en/stable/tutorial/setup.html#setup-a-linux-vm-with-vagrant-under-windows)).

### Installing miniconda

Information on how to install Miniconda can be found [on their website](https://docs.conda.io/en/latest/miniconda.html). Snakemake also provides information on installing Miniconda in [their documentation](https://snakemake.readthedocs.io/en/stable/tutorial/setup.html#step-1-installing-miniconda-3)

Once miniconda is installed, [set up your channels](https://bioconda.github.io/user/install.html#set-up-channels) (channels are locations where packages/software are can be installed from)

```bash
conda config --add channels defaults
conda config --add channels bioconda
conda config --add channels conda-forge
```

## Create a conda environment

With Miniconda, we can create a conda environment which acts as a space contained from the rest of the machine in which our workflow will automatically install all the necessary software it uses, supporting the portability and reproducibility of your workflow.

Create a conda environment (called `demo_workflow_env`) that has python and Snakemake installed (and all it's dependant software)

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

To clone this repo (and use the example data I have provided), you will require you have git installed. See [this guide](https://www.atlassian.com/git/tutorials/install-git) for help with an installation of git.

Once git is installed, clone this repo with the following:

```bash
git clone https://github.com/leahkemp/RezBaz2020_snakemake_workshop.git
cd RezBaz2020_snakemake_workshop
```

See the [Git Guides](https://github.com/git-guides/git-clone) for information on cloning a repo

<p align="left"><b><a href="https://leahkemp.github.io/RezBaz2020_snakemake_workshop/blob/master/workshop_material/01_introduction.md">Previous page: 01 - Introduction</a>
<p align="right"><b><a href="https://leahkemp.github.io/RezBaz2020_snakemake_workshop/blob/master/workshop_material/03_create_a_basic_workflow.md">Next page: 03 - Create a basic workflow</a>
