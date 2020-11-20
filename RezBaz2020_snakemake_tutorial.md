# Workflow languages – your foundation for accuracy and reproducibility in data analysis

**This workshop is a part of [RezBaz 2020: Pick n Mix](https://resbaz.auckland.ac.nz/)**
**Date:** Friday 27 November 2020
**Time:** 2:30pm - 3:30pm

Are you working with big data? Do you need to pass your data through various software? If you’ve ever been in this situation (as I have in a population genetics masters project), you would know it can become very difficult to maintain reproducibility and accuracy; wait, have I updated this output file? The more manual steps we do, the more human errors are inevitably introduced into our analysis, hampering accuracy and reproducibility.

Be lazy, the machine does it better.

Workflow languages automate your data analysis workflow . But this isn’t all, they ensure all your analysis logs are captured in an organised fashion, they explicitly outline the software (and exact software versions) used, the input and output files at each step. Lastly, when your data inevitably becomes big data, you can easily scale up from running your analysis on your laptop, to running your analysis on a high performance cluster (HPC) such as NeSi.

In this workshop, we will work through an introduction to Snakemake, a workflow language with its basis in the popular programming language, Python. This Workshop is intended for anyone who has several steps in their data analysis workflow, particularly when many different software are involved. [Book here](https://vuw.libcal.com/event/5293465/).

# Table of contents

- [Workflow languages – your foundation for accuracy and reproducibility in data analysis](#workflow-languages--your-foundation-for-accuracy-and-reproducibility-in-data-analysis)
- [Table of contents](#table-of-contents)
- [Introduction](#introduction)
  - [My background](#my-background)
  - [Outcomes](#outcomes)
  - [Benefits of workflow languages](#benefits-of-workflow-languages)
  - [Benefits of Snakemake](#benefits-of-snakemake)
  - [Other workflow languages](#other-workflow-languages)
- [Tutorial](#tutorial)
  - [Setup](#setup)
    - [Check your OS](#check-your-os)
    - [Install Miniconda](#install-miniconda)
    - [Create an conda environment to work in](#create-an-conda-environment-to-work-in)
  - [Create a basic workflow](#create-a-basic-workflow)
    - [Aim](#aim)
    - [File structure](#file-structure)
    - [First rule](#first-rule)
    - [Run using the conda package management system](#run-using-the-conda-package-management-system)
    - [Capture our logs](#capture-our-logs)
    - [Scale our analyse to all of our samples](#scale-our-analyse-to-all-of-our-samples)
    - [Add more rules](#add-more-rules)
  - [Lets speed this up!](#lets-speed-this-up)
    - [Throw it more threads](#throw-it-more-threads)
    - [Deploy on a HPC](#deploy-on-a-hpc)
  - [Leveling up your workflow!](#leveling-up-your-workflow)
    - [Pull out parameters](#pull-out-parameters)
    - [Pull out user configurable options](#pull-out-user-configurable-options)
    - [Generating a snakemake report](#generating-a-snakemake-report)
    - [Leave messages for the user](#leave-messages-for-the-user)
    - [Create temporary files](#create-temporary-files)
  - [Version control](#version-control)
  - [Trouble shooting](#trouble-shooting)
  - [Final words](#final-words)

# Introduction

## My background

- Bioinformatician at the Institute of Environmental Science and Research (ESR)
- Analyse human genomic data
- MSc - population genetics

## Outcomes

By the end of this workshop, you should be able to:

- Use a systematic approach to build your own workflow
- Understand how to link multiple rules in a snakemake workflow
- Be able to use software using conda environments or containers
- Capture all your log files
- Identify and troubleshoot some common error messages
- Use wildcards to process all your data/samples
- Multithread software to speed it up
- Visualize your workflow by creating a diagram of jobs (DAG)
- Search for existing workflows you can use or adapt
- Store your workflow on github

## Benefits of workflow languages

- Reproducibility
- Automation
- Portability
- Scalability
- Interpretability
  
## Benefits of Snakemake

![Snakemake](https://avatars2.githubusercontent.com/u/33450111?s=400&v=4 "Snakemake")

- Based in the popular (and widely used) programming language, Python

## Other workflow languages

Choose your favourite flavour of workflow language!

- [Common Workflow Language (CWL)](https://www.commonwl.org/)
- [Nextflow](https://www.nextflow.io/)
- [Workflow Description Language (WDL)](https://openwdl.org/)
- [Guix Workflow Language](https://workflows.guix.info/)

# Tutorial

## Setup

### Check your OS

If you already use Linux or MacOS X, great! Ignore this paragraph!. If you use Windows, setup a Linux virtual machine (VM) with Vagrant (see instructions on how to do this [here](https://snakemake.readthedocs.io/en/stable/tutorial/setup.html#setup-a-linux-vm-with-vagrant-under-windows)).

### Install Miniconda

Information on how to install Miniconda can be found [on their website](https://docs.conda.io/en/latest/miniconda.html). Snakemake also provides information on installing Miniconda in [their documentation](https://snakemake.readthedocs.io/en/stable/tutorial/setup.html#step-1-installing-miniconda-3)

Once miniconda is installed, [set up your channels](https://bioconda.github.io/user/install.html#set-up-channels) (channels are locations where packages/software are can be installed from)

```bash
conda config --add channels defaults
conda config --add channels bioconda
conda config --add channels conda-forge
```

### Create an conda environment to work in

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

## Create a basic workflow

### Aim

---

Let's create a basic workflow that will do some of the typical quality control checks, pre-processing and mapping to a reference genome that is undertaken on paired-end sequence data

input data :arrow_right: [fastqc](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/) :arrow_right: [multiqc](https://multiqc.info/)
input data :arrow_right: [trim_galore](https://www.bioinformatics.babraham.ac.uk/projects/trim_galore/) :arrow_right: [bwa](http://bio-bwa.sourceforge.net/)

---

We have paired end sequencing data for three samples `NA24631` to process in the `./data` directory. Let's have a look:

```bash
ls -lh ./data/
```

Output:

```bash
-rw-rw-r-- 1 lkemp lkemp 2.1M Nov 18 14:56 NA24631_1.fastq.gz
-rw-rw-r-- 1 lkemp lkemp 2.3M Nov 18 14:56 NA24631_2.fastq.gz
-rw-rw-r-- 1 lkemp lkemp 2.1M Nov 18 14:56 NA24694_1.fastq.gz
-rw-rw-r-- 1 lkemp lkemp 2.3M Nov 18 14:56 NA24694_2.fastq.gz
-rw-rw-r-- 1 lkemp lkemp 1.8M Nov 18 14:56 NA24695_1.fastq.gz
-rw-rw-r-- 1 lkemp lkemp 1.9M Nov 18 14:56 NA24695_2.fastq.gz
```

### File structure

Workflow file structure:

```bash

demo_workflow/
      |_______results/
      |_______workflow/
                 |_______envs/
                 |_______Snakefile
```

We will work in the `workflow` directory send all of our file outputs/results to the `results/` directory

*See more information on the workflow structure that the Snakemake developers recommend [here](https://snakemake.readthedocs.io/en/stable/snakefiles/deployment.html#distribution-and-reproducibility)*

```bash
mkdir -p demo_workflow/{results,workflow/envs}
touch demo_workflow/workflow/Snakefile
```

### First rule

First lets run the first step ([fastqc](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/)) directly on the command line to get the syntax of the command right and check what outputs files we expect to get

```bash
# Install fastqc
conda install fastqc

# See what parameters are availble
fastqc --help

# Create a test directory
mkdir test

# Directly run fastqc on the command line
fastqc ./data/NA24631_1.fastq.gz ./data/NA24631_2.fastq.gz -o ./test -t 8
```

What are the output files of fastqc?

```bash
ls -lh ./test
```

Output:

```bash
-rw-rw-r-- 1 lkemp lkemp 250K Nov 18 15:53 NA24631_1_fastqc.html
-rw-rw-r-- 1 lkemp lkemp 327K Nov 18 15:53 NA24631_1_fastqc.zip
-rw-rw-r-- 1 lkemp lkemp 249K Nov 18 15:53 NA24631_2_fastqc.html
-rw-rw-r-- 1 lkemp lkemp 327K Nov 18 15:53 NA24631_2_fastqc.zip
```

Let's wrap this up in a Snakemake workflow! Start with the basic structure in the Snakefile:

```python
# Target rules
rule all:
    input:

# Workflow
rule my_rule:
    input:
        ""
    output:
        ""
    threads:
    shell:
        ""
```

- Name the rule
- Fill in the the input fastq files from the `data` directory (path relative to the Snakefile)
- Fill in the output files
- Set the number of threads
- Write the shell command and pass these variables to the shell command
- Set the final output files (`rule all:`)

```python
# Targets
rule all:
    input:
        "../results/fastqc/NA24631_1_fastqc.html",
        "../results/fastqc/NA24631_2_fastqc.html",
        "../results/fastqc/NA24631_1_fastqc.zip",
        "../results/fastqc/NA24631_2_fastqc.zip"

# Workflow
rule fastqc:
    input:
        R1 = "../../data/NA24631_1.fastq.gz",
        R2 = "../../data/NA24631_2.fastq.gz"
    output:
        html = ["../results/fastqc/NA24631_1_fastqc.html", "../results/fastqc/NA24631_2_fastqc.html"],
        zip = ["../results/fastqc/NA24631_1_fastqc.zip", "../results/fastqc/NA24631_2_fastqc.zip"]
    threads: 8
    shell:
        "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads}"
```

Test the workflow

```bash
# Move to the workflow directory where the Snakefile is
cd demo_workflow/workflow/

# Dryrun
snakemake -n -j 8
```

Output:

```bash
Building DAG of jobs...
Job counts:
        count   jobs
        1       all
        1       fastqc
        2

[Wed Nov 18 17:08:08 2020]
rule fastqc:
    input: ../../data/NA24631_1.fastq.gz, ../../data/NA24631_2.fastq.gz
    output: ../results/fastqc/NA24631_1_fastqc.html, ../results/fastqc/NA24631_2_fastqc.html, ../results/fastqc/NA24631_1_fastqc.zip, ../results/fastqc/NA24631_2_fastqc.zip
    jobid: 1
    threads: 8


[Wed Nov 18 17:08:08 2020]
localrule all:
    input: ../results/fastqc/NA24631_1_fastqc.html, ../results/fastqc/NA24631_2_fastqc.html, ../results/fastqc/NA24631_1_fastqc.zip, ../results/fastqc/NA24631_2_fastqc.zip
    jobid: 0

Job counts:
        count   jobs
        1       all
        1       fastqc
        2
This was a dry-run (flag -n). The order of jobs does not reflect the order of execution.
```

The output confirms that the workflow will run one sample (`count 1`) through the rule `fastqc`

We can also visualise our workflow by creating a diagram of jobs (DAG)

```bash
# Visualise workflow
snakemake --dag | dot -Tpdf > dag_1.pdf
```

This produces a diagram that lets us visualise our workflow

![DAG_1](./demo_workflow_diagrams/dag_1.pdf)

*Note. this diagram can be output to several other image formats such as svg or png*

```bash
# Fullrun
snakemake -j 8
```

What happens if we try a dryrun/run now?

```bash
snakemake -n -j 8
snakemake -j 8
```

### Run using the conda package management system

fastqc worked because we already had it installed locally. Let's specify a conda environment for fastqc so the user of the workflow doesn't need to install it manually.

Make a conda environment file for fastqc

```bash
# Create the file
touch ./envs/fastqc.yaml

# See what versions of fastqc are available
conda search fastqc

# Write the following to fastqc.yaml
channels:
  - bioconda
  - conda-forge
  - defaults
dependencies:
  - bioconda::fastqc=0.11.9
```

Update our rule to use it using the `conda:` directive

```python
# Targets
rule all:
    input:
        "../results/fastqc/NA24631_1_fastqc.html",
        "../results/fastqc/NA24631_2_fastqc.html",
        "../results/fastqc/NA24631_1_fastqc.zip",
        "../results/fastqc/NA24631_2_fastqc.zip"

# Workflow
rule fastqc:
    input:
        R1 = "../../data/NA24631_1.fastq.gz",
        R2 = "../../data/NA24631_2.fastq.gz"
    output:
        html = ["../results/fastqc/NA24631_1_fastqc.html", "../results/fastqc/NA24631_2_fastqc.html"],
        zip = ["../results/fastqc/NA24631_1_fastqc.zip", "../results/fastqc/NA24631_2_fastqc.zip"]
    threads: 8
    conda:
        "envs/fastqc.yaml"
    shell:
        "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads}"
```

Run again, now telling Snakemake to use to use [Conda](https://docs.conda.io/en/latest/) to get our software by using the `--use-conda` flag

```bash
# Remove output of last run
rm -r ../results/*

# Run dryrun/run again
snakemake -n -j 8 --use-conda
snakemake -j 8 --use-conda
```

### Capture our logs

What happens if we have a large workflow and an error in one of our rules? Let's create an error (remove the `-o` flag):

```python
# Targets
rule all:
    input:
        "../results/fastqc/NA24631_1_fastqc.html",
        "../results/fastqc/NA24631_2_fastqc.html",
        "../results/fastqc/NA24631_1_fastqc.zip",
        "../results/fastqc/NA24631_2_fastqc.zip"

# Workflow
rule fastqc:
    input:
        R1 = "../../data/NA24631_1.fastq.gz",
        R2 = "../../data/NA24631_2.fastq.gz"
    output:
        html = ["../results/fastqc/NA24631_1_fastqc.html", "../results/fastqc/NA24631_2_fastqc.html"],
        zip = ["../results/fastqc/NA24631_1_fastqc.zip", "../results/fastqc/NA24631_2_fastqc.zip"]
    log:
        "logs/fastqc/NA2431.tsv"
    threads: 8
    conda:
        "envs/fastqc.yaml"
    shell:
        "fastqc {input.R1} {input.R2} ../results/fastqc/ -t {threads}"
```

Run again

```bash
# Remove output of last run
rm -r ../results/*

# Run dryrun/run again
snakemake -n -j 8 --use-conda
snakemake -j 8 --use-conda
```

The logs are currently just printed to the screen and can be hard to find in a large automated workflow

- We can get the logs for *each sample* and *each rule* to be written to a log file via the `logs:` directive
- It's a good idea to organise the logs by:
  - putting the logs in a directory labelled by the rule
  - labelling the log files with the sample name

```python
# Targets
rule all:
    input:
        "../results/fastqc/NA24631_1_fastqc.html",
        "../results/fastqc/NA24631_2_fastqc.html",
        "../results/fastqc/NA24631_1_fastqc.zip",
        "../results/fastqc/NA24631_2_fastqc.zip"

# Workflow
rule fastqc:
    input:
        R1 = "../../data/NA24631_1.fastq.gz",
        R2 = "../../data/NA24631_2.fastq.gz"
    output:
        html = ["../results/fastqc/NA24631_1_fastqc.html", "../results/fastqc/NA24631_2_fastqc.html"],
        zip = ["../results/fastqc/NA24631_1_fastqc.zip", "../results/fastqc/NA24631_2_fastqc.zip"]
    log:
        "logs/fastqc/NA2431.log"
    threads: 8
    conda:
        "envs/fastqc.yaml"
    shell:
        "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads}"
```

Run again

```bash
# Remove output of last run
rm -r ../results/*

# Run dryrun/run again
snakemake -n -j 8 --use-conda
snakemake -j 8 --use-conda
```

It didn't write to our log file!

We haven't linked the `log:` directive to our shell command!

```bash
# Targets
rule all:
    input:
        "../results/fastqc/NA24631_1_fastqc.html",
        "../results/fastqc/NA24631_2_fastqc.html",
        "../results/fastqc/NA24631_1_fastqc.zip",
        "../results/fastqc/NA24631_2_fastqc.zip"

# Workflow
rule fastqc:
    input:
        R1 = "../../data/NA24631_1.fastq.gz",
        R2 = "../../data/NA24631_2.fastq.gz"
    output:
        html = ["../results/fastqc/NA24631_1_fastqc.html", "../results/fastqc/NA24631_2_fastqc.html"],
        zip = ["../results/fastqc/NA24631_1_fastqc.zip", "../results/fastqc/NA24631_2_fastqc.zip"]
    log:
        "logs/fastqc/NA24631.log"
    threads: 8
    conda:
        "envs/fastqc.yaml"
    shell:
        "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} &> {log}"
```

- StdOut: standard output
- StdErr: standard error

|  Syntax  | StdOut in terminal | StdErr in terminal | StdOut in file | StdErr in file |
|----------|--------------------|--------------------|----------------|----------------|
|    >     |  no                | yes                | yes            | no             |
|   2>     |  yes               | no                 | no             | yes            |
|   &>     |  no                | no                 | yes            | yes            |

(Table adapted from [here](https://askubuntu.com/questions/420981/how-do-i-save-terminal-output-to-a-file))

Run again

```bash
# Remove output of last run
rm -r ../results/*

# Run dryrun/run again
snakemake -n -j 8 --use-conda
snakemake -j 8 --use-conda
```

Worked great, all of our logs are now written to `logs/fastqc/NA24631.log`

```bash
head logs/fastqc/NA24631.log
```

Output:

```bash
Started analysis of NA24631_1.fastq.gz
Approx 5% complete for NA24631_1.fastq.gz
Approx 10% complete for NA24631_1.fastq.gz
Approx 15% complete for NA24631_1.fastq.gz
Approx 20% complete for NA24631_1.fastq.gz
Approx 25% complete for NA24631_1.fastq.gz
Approx 30% complete for NA24631_1.fastq.gz
Approx 35% complete for NA24631_1.fastq.gz
Approx 40% complete for NA24631_1.fastq.gz
Approx 45% complete for NA24631_1.fastq.gz
```

![logs](https://i.redd.it/d8qyw1j389e11.jpg)

### Scale our analyse to all of our samples

We are currently only analysing one of our three samples

Let's scale up to run all of our samples by using [wildcards](https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html#wildcards), this way we can grab all the samples/files in the `data` directory and analyse them

- Set a global wildcard that defines the samples to be analysed
- Generalise where this rule uses an individual sample (`NA24631`) to use this wildcard `{sample}`
- Use the [expand function](https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html#the-expand-function) (`expand()`) function to tell snakemake that `{sample}` is what we defined in our global wildcard `SAMPLES,`
- Snakemake can figure out what `{sample}` is in our rule since it's defined in the targets in `rule all:`

```python
# Define samples from data directory using wildcards
SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")

# Targets
rule all:
    input:
        expand("../results/fastqc/{sample}_1_fastqc.html", sample = SAMPLES),
        expand("../results/fastqc/{sample}_2_fastqc.html", sample = SAMPLES),
        expand("../results/fastqc/{sample}_1_fastqc.zip", sample = SAMPLES),
        expand("../results/fastqc/{sample}_2_fastqc.zip", sample = SAMPLES)

# Workflow
rule fastqc:
    input:
        R1 = "../../data/{sample}_1.fastq.gz",
        R2 = "../../data/{sample}_2.fastq.gz"
    output:
        html = ["../results/fastqc/{sample}_1_fastqc.html", "../results/fastqc/{sample}_2_fastqc.html"],
        zip = ["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"]
    log:
        "logs/fastqc/{sample}.log"
    threads: 8
    conda:
        "envs/fastqc.yaml"
    shell:
        "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} &> {log}"
```

Run again

```bash
# Remove output of last run
rm -r ../results/*

# Run dryrun/run again
snakemake -n -j 8 --use-conda
snakemake -j 8 --use-conda
```

All three samples were run through our workflow! And we have a log file for each sample for the fastqc rule

```bash
ls -lh ./logs/fastqc
```

Output:

```bash
-rw-rw-r-- 1 lkemp lkemp 1.8K Nov 19 15:17 NA24631.log
-rw-rw-r-- 1 lkemp lkemp 1.8K Nov 19 15:17 NA24694.log
-rw-rw-r-- 1 lkemp lkemp 1.8K Nov 19 15:17 NA24695.log
```

### Add more rules

- Make a conda environment file for multiqc

```bash
# Create the file
touch ./envs/multiqc.yaml

# See what versions of multiqc are available
conda search multiqc

# Write the following to multiqc.yaml
channels:
  - bioconda
  - conda-forge
  - defaults
dependencies:
  - bioconda::multiqc=1.9
```

- Connect the outputs of fastqc to the inputs of multiqc
- Add a new final target for `rule all:`

```python
# Define samples from data directory using wildcards
SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")

# Targets
rule all:
    input:
        expand("../results/fastqc/{sample}_1_fastqc.html", sample = SAMPLES),
        expand("../results/fastqc/{sample}_2_fastqc.html", sample = SAMPLES),
        expand("../results/fastqc/{sample}_1_fastqc.zip", sample = SAMPLES),
        expand("../results/fastqc/{sample}_2_fastqc.zip", sample = SAMPLES),
        "../results/multiqc_report.html"

# Workflow
rule fastqc:
    input:
        R1 = "../../data/{sample}_1.fastq.gz",
        R2 = "../../data/{sample}_2.fastq.gz"
    output:
        html = ["../results/fastqc/{sample}_1_fastqc.html", "../results/fastqc/{sample}_2_fastqc.html"],
        zip = ["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"]
    log:
        "logs/fastqc/{sample}.log"
    threads: 8
    conda:
        "envs/fastqc.yaml"
    shell:
        "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} &> {log}"
  
rule multiqc:
    input:
        ["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"]
    output:
        "../results/multiqc_report.html"
    conda:
        "envs/multiqc.yaml"
    log:
        "logs/multiqc/multiqc.log"
    shell:
        "multiqc {input} -o ../results/ &> {log}"
```

Run again

```bash
# Remove output of last run
rm -r ../results/*

# Run dryrun/run again
snakemake -n -j 8 --use-conda
snakemake -j 8 --use-conda
```

Didn't work? Error:

```bash
Building DAG of jobs...
WildcardError in line 30 of /home/lkemp/RezBaz2020/demo_workflow/workflow/Snakefile:
Wildcards in input files cannot be determined from output files:
'sample'
```

Since we haven't defined `{sample}` in `rule all:` for multiqc, we need to define it somewhere! Let do so in the multiqc rule

```python
# Define samples from data directory using wildcards
SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")

# Targets
rule all:
    input:
        expand("../results/fastqc/{sample}_1_fastqc.html", sample = SAMPLES),
        expand("../results/fastqc/{sample}_2_fastqc.html", sample = SAMPLES),
        expand("../results/fastqc/{sample}_1_fastqc.zip", sample = SAMPLES),
        expand("../results/fastqc/{sample}_2_fastqc.zip", sample = SAMPLES),
        "../results/multiqc_report.html"

# Workflow
rule fastqc:
    input:
        R1 = "../../data/{sample}_1.fastq.gz",
        R2 = "../../data/{sample}_2.fastq.gz"
    output:
        html = ["../results/fastqc/{sample}_1_fastqc.html", "../results/fastqc/{sample}_2_fastqc.html"],
        zip = ["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"]
    log:
        "logs/fastqc/{sample}.log"
    threads: 8
    conda:
        "envs/fastqc.yaml"
    shell:
        "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} &> {log}"
  
rule multiqc:
    input:
        expand(["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"], sample = SAMPLES)
    output:
        "../results/multiqc_report.html"
    conda:
        "envs/multiqc.yaml"
    log:
        "logs/multiqc/multiqc.log"
    shell:
        "multiqc {input} -o ../results/ &> {log}"
```

Run again

```bash
# Remove output of last run
rm -r ../results/*

# Run dryrun/run again
snakemake -n -j 8 --use-conda
snakemake -j 8 --use-conda
```

What happens if we only have the final target file (`../results/multiqc_report.html`) in `rule all:`

```python
# Define samples from data directory using wildcards
SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")

# Targets
rule all:
    input:
        "../results/multiqc_report.html"

# Workflow
rule fastqc:
    input:
        R1 = "../../data/{sample}_1.fastq.gz",
        R2 = "../../data/{sample}_2.fastq.gz"
    output:
        html = ["../results/fastqc/{sample}_1_fastqc.html", "../results/fastqc/{sample}_2_fastqc.html"],
        zip = ["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"]
    log:
        "logs/fastqc/{sample}.log"
    threads: 8
    conda:
        "envs/fastqc.yaml"
    shell:
        "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} &> {log}"
  
rule multiqc:
    input:
        expand(["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"], sample = SAMPLES)
    output:
        "../results/multiqc_report.html"
    conda:
        "envs/multiqc.yaml"
    log:
        "logs/multiqc/multiqc.log"
    shell:
        "multiqc {input} -o ../results/ &> {log}"
```

Run again

```bash
# Remove output of last run
rm -r ../results/*

# Run dryrun/run again
snakemake -n -j 8 --use-conda
snakemake -j 8 --use-conda
```

It still works because it is the last file in the workflow sequence, Snakemake will do all the steps necessary to get to this target file (therefore it runs fastqc and multiqc)

**Beware: Snakemake will also NOT run rules that is doesn't need to run in order to get the target files defined in rule: all**

For example if our fastqc outputs are defined as the target in `rule: all`

```bash
# Remove output of last run
rm -r ../results/*

# Run dryrun/run again
snakemake -n -j 8 --use-conda
snakemake -j 8 --use-conda
```

What happens if we only have the final target file (`../results/multiqc_report.html`) in `rule all:`

```python
# Define samples from data directory using wildcards
SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")

# Targets
rule all:
    input:
        expand("../results/fastqc/{sample}_1_fastqc.html", sample = SAMPLES),
        expand("../results/fastqc/{sample}_2_fastqc.html", sample = SAMPLES),
        expand("../results/fastqc/{sample}_1_fastqc.zip", sample = SAMPLES),
        expand("../results/fastqc/{sample}_2_fastqc.zip", sample = SAMPLES)

# Workflow
rule fastqc:
    input:
        R1 = "../../data/{sample}_1.fastq.gz",
        R2 = "../../data/{sample}_2.fastq.gz"
    output:
        html = ["../results/fastqc/{sample}_1_fastqc.html", "../results/fastqc/{sample}_2_fastqc.html"],
        zip = ["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"]
    log:
        "logs/fastqc/{sample}.log"
    threads: 8
    conda:
        "envs/fastqc.yaml"
    shell:
        "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} &> {log}"
  
rule multiqc:
    input:
        expand(["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"], sample = SAMPLES)
    output:
        "../results/multiqc_report.html"
    conda:
        "envs/multiqc.yaml"
    log:
        "logs/multiqc/multiqc.log"
    shell:
        "multiqc {input} -o ../results/ &> {log}"
```

Run again

```bash
# Remove output of last run
rm -r ../results/*

# Run dryrun/run again
snakemake -n -j 8 --use-conda
snakemake -j 8 --use-conda
```

Output:

```bash
Job counts:
        count   jobs
        1       all
        3       fastqc
        4
This was a dry-run (flag -n). The order of jobs does not reflect the order of execution.
```

Our multiqc rule won't be run/evaluated

Snakemake is lazy.

![Snakemake is lazy.](https://64.media.tumblr.com/0492923adeb79cb841e29968135305d5/tumblr_nzdagaL6EH1uavdlbo7_1280.png)

We also don't need to specify *ALL* the files output by a target, just at least one output file to convince Snakemake to not be lazy and run the rule. For example:

```python
# Define samples from data directory using wildcards
SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")

# Targets
rule all:
    input:
        expand("../results/fastqc/{sample}_1_fastqc.html", sample = SAMPLES)

# Workflow
rule fastqc:
    input:
        R1 = "../../data/{sample}_1.fastq.gz",
        R2 = "../../data/{sample}_2.fastq.gz"
    output:
        html = ["../results/fastqc/{sample}_1_fastqc.html", "../results/fastqc/{sample}_2_fastqc.html"],
        zip = ["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"]
    log:
        "logs/fastqc/{sample}.log"
    threads: 8
    conda:
        "envs/fastqc.yaml"
    shell:
        "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} &> {log}"
  
rule multiqc:
    input:
        expand(["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"], sample = SAMPLES)
    output:
        "../results/multiqc_report.html"
    conda:
        "envs/multiqc.yaml"
    log:
        "logs/multiqc/multiqc.log"
    shell:
        "multiqc {input} -o ../results/ &> {log}"
```

Run again

```bash
# Remove output of last run
rm -r ../results/*

# Run dryrun/run again
snakemake -n -j 8 --use-conda
```

Output:

```bash
Job counts:
        count   jobs
        1       all
        3       fastqc
        4
This was a dry-run (flag -n). The order of jobs does not reflect the order of execution.
```

Our fastqc rule will still be run/evaluated

Let's add the rest of the rules, currently we have:

input data :arrow_right: [fastqc](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/) :arrow_right: [multiqc](https://multiqc.info/) :heavy_check_mark:

We still need to add:

input data :arrow_right: [trim_galore](https://www.bioinformatics.babraham.ac.uk/projects/trim_galore/) :arrow_right: [bwa](http://bio-bwa.sourceforge.net/)

```python
# Define samples from data directory using wildcards
SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")

# Targets
rule all:
    input:
        "../results/multiqc_report.html",
        expand("../results/mapped/{sample}.bam", sample = SAMPLES)

# Workflow
rule fastqc:
    input:
        R1 = "../../data/{sample}_1.fastq.gz",
        R2 = "../../data/{sample}_2.fastq.gz"
    output:
        html = ["../results/fastqc/{sample}_1_fastqc.html", "../results/fastqc/{sample}_2_fastqc.html"],
        zip = ["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"]
    log:
        "logs/fastqc/{sample}.log"
    threads: 8
    conda:
        "envs/fastqc.yaml"
    shell:
        "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} &> {log}"
  
rule multiqc:
    input:
        expand(["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"], sample = SAMPLES)
    output:
        "../results/multiqc_report.html"
    conda:
        "envs/multiqc.yaml"
    log:
        "logs/multiqc/multiqc.log"
    shell:
        "multiqc {input} -o ../results/ &> {log}"

rule trim_galore:
    input:
        ["../../data/{sample}_1.fastq.gz", "../../data/{sample}_2.fastq.gz"]
    output:
        ["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"]
    log:
        "logs/trim_galore/{sample}.log"
    conda:
        "./envs/trim_galore.yaml"
    threads: 8
    shell:
        "trim_galore {input} -o ../results/trimmed/ --paired -j {threads} &> {log}"

rule bwa:
    input:
        fastq = ["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"],
        refgenome = "/store/lkemp/publicData/b37/human_g1k_v37_decoy.fasta"
    output: 
        "../results/mapped/{sample}.bam"
    log:
        "logs/bwa_mem/{sample}.log"
    conda:
        "./envs/bwa.yaml"
    threads: 8
    shell:
        "bwa mem -t {threads} {input.refgenome} {input.fastq} > {output} 2> {log}"
```

Conda env files

```bash
# Create file
touch ./envs/trim_galore.yaml

# Write the following to trim_galore.yaml
channels:
  - bioconda
  - conda-forge
  - defaults
dependencies:
  - bioconda::trim-galore=0.6.5

# Create file
touch ./envs/bwa.yaml

# Write the following to bwa.yaml
channels:
  - bioconda
  - conda-forge
  - defaults
dependencies:
  - bioconda::bwa=0.7.17
  - bioconda::gatk4=4.1.6.0
```

Run again

```bash
# Remove output of last run
rm -r ../results/*

# Visualise workflow
snakemake --dag | dot -Tpdf > dag.pdf
snakemake --rulegraph | dot -Tpdf > rulegraph.pdf

# Run dryrun/run again
snakemake -n -j 8 --use-conda
snakemake -j 8 --use-conda
```

Notice it will run only one rule/sample at a time...why is that?

---

Takeaways:

- Run your commands directly on the command line before wrapping it up in a Snakemake rule
- First do a dryrun with the `-n` flag to check the Snakemake structure is set up correctly
- Work iteratively (get each rule working before moving onto the next)
- File paths are relative to the Snakefile
- Visualise your workflow by creating a DAG (diagram of jobs) or a rulegraph
- Use the `--use-conda` flag when using conda to install software in your workflow
- Snakemake is lazy...
  - It will only do something if it hasn't already done it
  - It will pick up where it left off, rather than run the whole workflow again
  - It *won't* do any steps that aren't necessary to get to the target files defined in `rule: all`
- `input:` `output:` `log:` and `threads:` directives need to be called in the `shell` directive
- Use `&> {log}` in the shell command to capture your log files
- Organise your log files with rulenames and sample name
- You don't need to specify all the target files in `rule all:`, the final file in a given chain of tasks will suffice
- We can massively speed up our analyses by running them in parallel

---

## Lets speed this up!

### Throw it more threads

Run again allowing Snakemake to use more threads overall `-j 32` rather than `-j 8`

```bash
# Remove output of last run
rm -r ../results/*

# Run dryrun/run again
snakemake -n -j 32 --use-conda
snakemake -j 32 --use-conda
```

![Unlimited speed](https://www.driven.co.nz/media/46465/nt-unlimited-speed-sign.jpg?width=820)

### Deploy on a HPC

---

Takeaways:

- Run your commands directly on the command line before wrapping it up in a Snakemake rule
- First do a dryrun with the `-n` flag to check the Snakemake structure is set up correctly
- Work iteratively (get each rule working before moving onto the next)
- File paths are relative to the Snakefile
- View the workflow with the ` ` command
- Use the `--use-conda` flag when using conda to install software in your workflow
- Snakemake is lazy...
  - It will only do something if it hasn't already done it
  - It will pick up where it left off, rather than run the whole workflow again
  - It *won't* do any steps that aren't necessary to get to the target files defined in `rule: all`
- `input:` `output:` `log:` and `threads:` directives need to be called in the `shell` directive
- Use `&> {log}` in the shell command to capture your log files
- Organise your log files with rulenames and sample name
- You don't need to specify all the target files in `rule all:`, the final file in a given chain of tasks will suffice
- We can massively speed up our analyses by running them in parallel

---

## Leveling up your workflow!

### Pull out parameters

### Pull out user configurable options

We can separate the user configurable options for our workflow away from the workflow. This supports reproducibility by minimising the chance the user makes changes to the core workflow.

### Generating a snakemake report

### Leave messages for the user

### Create temporary files

## Version control

## Trouble shooting

## Final words

https://github.com/ESR-NZ/human_genomics_pipeline

https://github.com/ESR-NZ/vcf_annotation_pipeline

