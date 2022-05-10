# Appendix - Use conda environments

* Do not remove this line (it will not be displayed)
{:toc}

## Introduction

Conda environments can be used as a substitute to environment modules to run a rule in a self-contained and reproducible environment.
This page presents an alternative version of [Section 3.09: Run using environment modules](https://nesi.github.io/snakemake_workshop/workshop_material/03_create_a_basic_workflow.html#309-run-using-environment-modules).

## Activate conda on NeSI

First, you need to have [Conda](https://docs.conda.io/en/latest/miniconda.html) or [Mamba](https://github.com/mamba-org/mamba) installed on your machine.

On NeSI, use the Miniconda environment module:

```bash
module load Miniconda3
```

and check that the `conda` tool is now available:

```bash
conda --version
```

## Run using the conda package management system

Now let's run the example from [Section 3.09](https://nesi.github.io/snakemake_workshop/workshop_material/03_create_a_basic_workflow.html#309-run-using-environment-modules) using a conda environment.
First, we need to specify a conda environment for fastqc.

Make a conda environment file for fastqc

```bash
# create the file
touch ./envs/fastqc.yaml

# see what versions of fastqc are available
conda search fastqc

# write the following to fastqc.yaml
channels:
  - bioconda
  - conda-forge
  - defaults
dependencies:
  - bioconda::fastqc=0.11.9
```

This will install [fastqc (version 0.11.9)](https://anaconda.org/bioconda/fastqc) from bioconda into a 'clean' conda environment separate from the rest of your computer

See [here](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#creating-an-environment-file-manually) for information on creating conda environment files.

Update our rule to use it using the `conda:` directive, pointing the rule to the `envs` directory which has our conda environment file for fastqc (directory relative to the Snakefile)

```diff
# target OUTPUT files for the whole workflow
rule all:
    input:
        "../results/fastqc/NA24631_1_fastqc.html",
        "../results/fastqc/NA24631_2_fastqc.html",
        "../results/fastqc/NA24631_1_fastqc.zip",
        "../results/fastqc/NA24631_2_fastqc.zip"

# workflow
rule fastqc:
    input:
        R1 = "../../data/NA24631_1.fastq.gz",
        R2 = "../../data/NA24631_2.fastq.gz"
    output:
        html = ["../results/fastqc/NA24631_1_fastqc.html", "../results/fastqc/NA24631_2_fastqc.html"],
        zip = ["../results/fastqc/NA24631_1_fastqc.zip", "../results/fastqc/NA24631_2_fastqc.zip"]
    threads: 2
+   conda:
+       "envs/fastqc.yaml"
    shell:
        "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads}"
```

Run again, now telling Snakemake to use to use [Conda](https://docs.conda.io/en/latest/) to automatically install our software by using the `--use-conda` flag

```diff
# first remove output of last run
rm -r ../results/*

# Run dryrun again
- snakemake --dryrun --cores 2
+ snakemake --dryrun --cores 2 --use-conda
```

My output:

```bash
Building DAG of jobs...
Conda environment envs/fastqc.yaml will be created.
Job stats:
job       count    min threads    max threads
------  -------  -------------  -------------
all           1              1              1
fastqc        1              2              2
total         2              1              2


[Mon Sep 13 03:06:45 2021]
rule fastqc:
    input: ../../data/NA24631_1.fastq.gz, ../../data/NA24631_2.fastq.gz
    output: ../results/fastqc/NA24631_1_fastqc.html, ../results/fastqc/NA24631_2_fastqc.html, ../results/fastqc/NA24631_1_fastqc.zip, ../results/fastqc/NA24631_2_fastqc.zip
    jobid: 1
    threads: 2
    resources: tmpdir=/dev/shm/jobs/22281190


[Mon Sep 13 03:06:45 2021]
localrule all:
    input: ../results/fastqc/NA24631_1_fastqc.html, ../results/fastqc/NA24631_2_fastqc.html, ../results/fastqc/NA24631_1_fastqc.zip, ../results/fastqc/NA24631_2_fastqc.zip
    jobid: 0
    resources: tmpdir=/dev/shm/jobs/22281190

Job stats:
job       count    min threads    max threads
------  -------  -------------  -------------
all           1              1              1
fastqc        1              2              2
total         2              1              2

This was a dry-run (flag -n). The order of jobs does not reflect the order of execution.
```

Notice it now says that "Conda environment envs/fastqc.yaml will be created.". Now the software our workflow uses will be automatically installed!

Let's do a full run

```diff
- snakemake --cores 2
+ snakemake --cores 2 --use-conda
```

My output:

```bash
Building DAG of jobs...
Creating conda environment envs/fastqc.yaml...
Downloading and installing remote packages.
Environment for envs/fastqc.yaml created (location: .snakemake/conda/67c1376bae89b8de73037e703ea4b6f5)
Using shell: /usr/bin/bash
Provided cores: 2
Rules claiming more threads will be scaled down.
Job stats:
job       count    min threads    max threads
------  -------  -------------  -------------
all           1              1              1
fastqc        1              2              2
total         2              1              2

Select jobs to execute...

[Mon Sep 13 03:10:27 2021]
rule fastqc:
    input: ../../data/NA24631_1.fastq.gz, ../../data/NA24631_2.fastq.gz
    output: ../results/fastqc/NA24631_1_fastqc.html, ../results/fastqc/NA24631_2_fastqc.html, ../results/fastqc/NA24631_1_fastqc.zip, ../results/fastqc/NA24631_2_fastqc.zip
    jobid: 1
    threads: 2
    resources: tmpdir=/dev/shm/jobs/22281190

Activating conda environment: /scale_wlg_persistent/filesets/project/nesi99991/snakemake20210914/lkemp/snakemake_workshop/demo_workflow/workflow/.snakemake/conda/67c1376bae89b8de73037e703ea4b6f5
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
Approx 50% complete for NA24631_1.fastq.gz
Approx 55% complete for NA24631_1.fastq.gz
Approx 60% complete for NA24631_1.fastq.gz
Started analysis of NA24631_2.fastq.gz
Approx 65% complete for NA24631_1.fastq.gz
Approx 5% complete for NA24631_2.fastq.gz
Approx 70% complete for NA24631_1.fastq.gz
Approx 10% complete for NA24631_2.fastq.gz
Approx 75% complete for NA24631_1.fastq.gz
Approx 15% complete for NA24631_2.fastq.gz
Approx 80% complete for NA24631_1.fastq.gz
Approx 20% complete for NA24631_2.fastq.gz
Approx 85% complete for NA24631_1.fastq.gz
Approx 25% complete for NA24631_2.fastq.gz
Approx 90% complete for NA24631_1.fastq.gz
Approx 30% complete for NA24631_2.fastq.gz
Approx 95% complete for NA24631_1.fastq.gz
Approx 35% complete for NA24631_2.fastq.gz
Analysis complete for NA24631_1.fastq.gz
Approx 40% complete for NA24631_2.fastq.gz
Approx 45% complete for NA24631_2.fastq.gz
Approx 50% complete for NA24631_2.fastq.gz
Approx 55% complete for NA24631_2.fastq.gz
Approx 60% complete for NA24631_2.fastq.gz
Approx 65% complete for NA24631_2.fastq.gz
Approx 70% complete for NA24631_2.fastq.gz
Approx 75% complete for NA24631_2.fastq.gz
Approx 80% complete for NA24631_2.fastq.gz
Approx 85% complete for NA24631_2.fastq.gz
Approx 90% complete for NA24631_2.fastq.gz
Approx 95% complete for NA24631_2.fastq.gz
Analysis complete for NA24631_2.fastq.gz
[Mon Sep 13 03:10:33 2021]
Finished job 1.
1 of 2 steps (50%) done
Select jobs to execute...

[Mon Sep 13 03:10:33 2021]
localrule all:
    input: ../results/fastqc/NA24631_1_fastqc.html, ../results/fastqc/NA24631_2_fastqc.html, ../results/fastqc/NA24631_1_fastqc.zip, ../results/fastqc/NA24631_2_fastqc.zip
    jobid: 0
    resources: tmpdir=/dev/shm/jobs/22281190

[Mon Sep 13 03:10:33 2021]
Finished job 0.
2 of 2 steps (100%) done
Complete log: /scale_wlg_persistent/filesets/project/nesi99991/snakemake20210914/lkemp/snakemake_workshop/demo_workflow/workflow/.snakemake/log/2021-09-13T030734.543325.snakemake.log
```

## Additional information

Have a look at [bioconda's list of packages](https://bioconda.github.io/conda-package_index.html) to see the VERY extensive list of quality open source (free) bioinformatics software that is available for download and use. Note that is only one of the conda package repositories that exist, also have a look at the [conda-forge](https://conda-forge.org/feedstocks/) and [main](https://anaconda.org/anaconda/repo) conda package repositories.

Another option to run your code in self-contained and reproducible environments are containers.
Snakemake can use Singularity containers to execute the workflow, as detailed in the [official documentation](https://snakemake.readthedocs.io/en/latest/snakefiles/deployment.html#running-jobs-in-containers).