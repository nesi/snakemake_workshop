# 04 - Leveling up your workflow!

<p style="text-align:left;">
  <b><a class="btn" href="https://nesi.github.io/snakemake_workshop/workshop_material/03_create_a_basic_workflow.html" style="background: var(--bs-green);font-weight:bold">&laquo; 3 - Creating a Basic WF</a></b> 
  <span style="float:right;">
    <b><a class="btn" href="https://nesi.github.io/snakemake_workshop/workshop_material/05_we_want_more.html" style="background: var(--bs-green);font-weight:bold">5 - More &raquo;</a></b>
  </span>
</p>




## Catching up

From section 03, you should have the following Snakefile:

{% capture e4dot1 %}

```python
# define samples from data directory using wildcards
SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")

# target OUTPUT files for the whole workflow
rule all:
    input:
        "../results/multiqc_report.html",
        expand(["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"], sample = SAMPLES)

# workflow
rule fastqc:
    input:
        R1 = "../../data/{sample}_1.fastq.gz",
        R2 = "../../data/{sample}_2.fastq.gz"
    output:
        html = ["../results/fastqc/{sample}_1_fastqc.html", "../results/fastqc/{sample}_2_fastqc.html"],
        zip = ["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"]
    log:
        "logs/fastqc/{sample}.log"
    threads: 2
    envmodules:
        "FastQC/0.11.9"
    shell:
        "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} &> {log}"
  
rule multiqc:
    input:
        expand(["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"], sample = SAMPLES)
    output:
        "../results/multiqc_report.html"
    log:
        "logs/multiqc/multiqc.log"
    envmodules:
        "MultiQC/1.9-gimkl-2020a-Python-3.8.2"
    shell:
        "multiqc {input} -o ../results/ &> {log}"

rule trim_galore:
    input:
        ["../../data/{sample}_1.fastq.gz", "../../data/{sample}_2.fastq.gz"]
    output:
        ["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"]
    log:
        "logs/trim_galore/{sample}.log"
    envmodules:
        "TrimGalore/0.6.7-gimkl-2020a-Python-3.8.2-Perl-5.30.1"
    threads: 2
    shell:
        "trim_galore {input} -o ../results/trimmed/ --paired --cores {threads} &> {log}"
```

{% endcapture %}

{% include exercise.html title="e4dot1" content=e4dot1%}
<br>

## 4.1 Use a profile for HPC

In section 3.16, we have seen that a snakemake workflow can be run on an HPC cluster.
To reduce the boilerplate, we can use a [configuration profile](https://snakemake.readthedocs.io/en/stable/executing/cli.html#profiles) to configure default options.
In this case, we use it to set the `--cluster` and the `--jobs` options.

Make a `slurm` profile folder

```bash
# create the profile folder
mkdir slurm
touch slurm/config.yaml

# write the following to config.yaml
jobs: 20
cluster: "sbatch --time 00:10:00 --mem 512MB --cpus-per-task 8 --account nesi99991"
```

Then run the snakemake workflow using the `slurm` profile

```bash
# remove output of last run
rm -r ../results/*

# run dryrun/run again
snakemake --dryrun --profile slurm --use-envmodules
snakemake --profile slurm --use-envmodules
```

If you interrupt the execution of a snakemake workflow using CTRL-C, already submitted Slurm jobs won't be cancelled.
We tell snakemake how to cancel Slurm jobs using `scancel` via the `--cluster-cancel` option and adding `--parsable` to the `sbatch` command, to make it return the job ID.

```diff
jobs: 20
- cluster: "sbatch --time 00:10:00 --mem 512MB --cpus-per-task 8"
+ cluster: "sbatch --parsable --time 00:10:00 --mem 512MB --cpus-per-task 8"
+ cluster-cancel: scancel
```

You can specify different resources (memory, cpus, gpus, etc.) for each target in the workflow and refer to them in the `cluster` option using placeholders.
Default resources for all rules can also be set using the `default-resources` option.

Update the profile `slurm/config.yaml` file as follows (using a multiline option to improve readability)

```diff
jobs: 20
- cluster: "sbatch --parsable --time 00:10:00 --mem 512MB --cpus-per-task 8"
+ cluster:
+     sbatch
+         --parsable
+         --time {resources.time_min}
+         --mem {resources.mem_mb}
+         --cpus-per-task {resources.cpus}
+         --account nesi99991
+ default-resources: [cpus=2, mem_mb=512, time_min=10]
cluster-cancel: scancel
```

and add resources definitions in the workflow.
Here we give more CPU resources to `trim_galore` to make it run faster.

```diff
# define samples from data directory using wildcards
SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")

# target OUTPUT files for the whole workflow
rule all:
    input:
        "../results/multiqc_report.html",
        expand(["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"], sample = SAMPLES)

# workflow
rule fastqc:
    input:
        R1 = "../../data/{sample}_1.fastq.gz",
        R2 = "../../data/{sample}_2.fastq.gz"
    output:
        html = ["../results/fastqc/{sample}_1_fastqc.html", "../results/fastqc/{sample}_2_fastqc.html"],
        zip = ["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"]
    log:
        "logs/fastqc/{sample}.log"
    threads: 2
    envmodules:
        "FastQC/0.11.9"
    shell:
        "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} &> {log}"
  
rule multiqc:
    input:
        expand(["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"], sample = SAMPLES)
    output:
        "../results/multiqc_report.html"
    log:
        "logs/multiqc/multiqc.log"
    envmodules:
        "MultiQC/1.9-gimkl-2020a-Python-3.8.2"
    shell:
        "multiqc {input} -o ../results/ &> {log}"

rule trim_galore:
    input:
        ["../../data/{sample}_1.fastq.gz", "../../data/{sample}_2.fastq.gz"]
    output:
        ["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"]
    log:
        "logs/trim_galore/{sample}.log"
    envmodules:
        "TrimGalore/0.6.7-gimkl-2020a-Python-3.8.2-Perl-5.30.1"
    threads: 2
+   resources:
+       cpus=8
    shell:
        "trim_galore {input} -o ../results/trimmed/ --paired --cores {threads} &> {log}"
```

Current slurm profile:

{% capture e4dot2 %}

```
jobs: 20
cluster:
    sbatch
        --parsable
        --time {resources.time_min}
        --mem {resources.mem_mb}
        --cpus-per-task {resources.cpus}
        --account nesi99991
default-resources: [cpus=2, mem_mb=512, time_min=10]
cluster-cancel: scancel
```

{% endcapture %}

{% include exercise.html title="e4dot2" content=e4dot2%}
<br>

Current snakefile:

{% capture e4dot3 %}

```txt
# define samples from data directory using wildcards
SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")

# target OUTPUT files for the whole workflow
rule all:
    input:
        "../results/multiqc_report.html",
        expand(["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"], sample = SAMPLES)

# workflow
rule fastqc:
    input:
        R1 = "../../data/{sample}_1.fastq.gz",
        R2 = "../../data/{sample}_2.fastq.gz"
    output:
        html = ["../results/fastqc/{sample}_1_fastqc.html", "../results/fastqc/{sample}_2_fastqc.html"],
        zip = ["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"]
    log:
        "logs/fastqc/{sample}.log"
    threads: 2
    envmodules:
        "FastQC/0.11.9"
    shell:
        "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} &> {log}"
  
rule multiqc:
    input:
        expand(["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"], sample = SAMPLES)
    output:
        "../results/multiqc_report.html"
    log:
        "logs/multiqc/multiqc.log"
    envmodules:
        "MultiQC/1.9-gimkl-2020a-Python-3.8.2"
    shell:
        "multiqc {input} -o ../results/ &> {log}"

rule trim_galore:
    input:
        ["../../data/{sample}_1.fastq.gz", "../../data/{sample}_2.fastq.gz"]
    output:
        ["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"]
    log:
        "logs/trim_galore/{sample}.log"
    envmodules:
        "TrimGalore/0.6.7-gimkl-2020a-Python-3.8.2-Perl-5.30.1"
    threads: 2
    resources:
        cpus=8
    shell:
        "trim_galore {input} -o ../results/trimmed/ --paired --cores {threads} &> {log}"
```

{% endcapture %}

{% include exercise.html title="e4dot3" content=e4dot3%}
<br>

Run the workflow again

```bash
# remove output of last run
rm -r ../results/*

# run dryrun/run again
snakemake --dryrun --profile slurm --use-envmodules
snakemake --profile slurm --use-envmodules
```

If you monitor the progress of your jobs using `squeue`, you will notice that some jobs now request 2 or 8 CPUs.

My output:

{% capture e4dot4 %}

```
JOBID         USER     ACCOUNT   NAME        CPUS MIN_MEM PARTITI START_TIME     TIME_LEFT STATE    NODELIST(REASON)
26763281      lkemp    nesi99991 spawner-jupy   4      4G interac 2022-05-11T1     7:21:18 RUNNING  wbn003
26763492      lkemp    nesi99991 snakejob.fas   2    512M large   2022-05-11T1        9:59 RUNNING  wbn144
26763493      lkemp    nesi99991 snakejob.tri   8    512M large   2022-05-11T1        9:59 RUNNING  wbn212
26763494      lkemp    nesi99991 snakejob.fas   2    512M large   2022-05-11T1        9:59 RUNNING  wbn145
26763495      lkemp    nesi99991 snakejob.fas   2    512M large   2022-05-11T1        9:59 RUNNING  wbn146
26763496      lkemp    nesi99991 snakejob.tri   8    512M large   2022-05-11T1        9:59 RUNNING  wbn217
26763497      lkemp    nesi99991 snakejob.tri   8    512M large   2022-05-11T1        9:59 RUNNING  wbn229
```

{% endcapture %}

{% include exercise.html title="e4dot4" content=e4dot4%}
<br>

Now looking at the content of our workflow folder, it is getting cluttered with Slurm log files:

```bash
ls -lh
```

My output:

{% capture e4dot5 %}

```
total 1.8M
-rw-rw----+ 1 lkemp nesi99991 4.2K May 11 12:10 dag_1.png
-rw-rw----+ 1 lkemp nesi99991 3.8K May 11 12:13 dag_2.png
-rw-rw----+ 1 lkemp nesi99991  12K May 11 12:16 dag_3.png
-rw-rw----+ 1 lkemp nesi99991  20K May 11 12:19 dag_4.png
-rw-rw----+ 1 lkemp nesi99991  15K May 11 12:21 dag_5.png
-rw-rw----+ 1 lkemp nesi99991  12K May 11 12:23 dag_6.png
-rw-rw----+ 1 lkemp nesi99991  26K May 11 12:24 dag_7.png
drwxrws---+ 5 lkemp nesi99991 4.0K May 11 12:25 logs
-rw-rw----+ 1 lkemp nesi99991  11K May 11 12:24 rulegraph_1.png
drwxrws---+ 3 lkemp nesi99991 4.0K May 11 12:34 slurm
-rw-rw----+ 1 lkemp nesi99991  837 May 11 12:27 slurm-26763403.out
-rw-rw----+ 1 lkemp nesi99991  809 May 11 12:27 slurm-26763404.out
-rw-rw----+ 1 lkemp nesi99991  837 May 11 12:27 slurm-26763405.out
-rw-rw----+ 1 lkemp nesi99991  837 May 11 12:27 slurm-26763406.out
-rw-rw----+ 1 lkemp nesi99991  809 May 11 12:27 slurm-26763407.out
-rw-rw----+ 1 lkemp nesi99991  809 May 11 12:27 slurm-26763408.out
-rw-rw----+ 1 lkemp nesi99991  865 May 11 12:29 slurm-26763409.out
-rw-rw----+ 1 lkemp nesi99991  837 May 11 12:30 slurm-26763418.out
-rw-rw----+ 1 lkemp nesi99991  809 May 11 12:30 slurm-26763419.out
-rw-rw----+ 1 lkemp nesi99991  837 May 11 12:30 slurm-26763420.out
-rw-rw----+ 1 lkemp nesi99991  837 May 11 12:30 slurm-26763421.out
-rw-rw----+ 1 lkemp nesi99991  809 May 11 12:30 slurm-26763422.out
-rw-rw----+ 1 lkemp nesi99991  809 May 11 12:30 slurm-26763423.out
-rw-rw----+ 1 lkemp nesi99991  865 May 11 12:32 slurm-26763431.out
-rw-rw----+ 1 lkemp nesi99991  837 May 11 12:33 slurm-26763435.out
-rw-rw----+ 1 lkemp nesi99991  809 May 11 12:34 slurm-26763436.out
-rw-rw----+ 1 lkemp nesi99991  837 May 11 12:33 slurm-26763437.out
-rw-rw----+ 1 lkemp nesi99991  837 May 11 12:34 slurm-26763438.out
-rw-rw----+ 1 lkemp nesi99991  809 May 11 12:34 slurm-26763439.out
-rw-rw----+ 1 lkemp nesi99991  809 May 11 12:34 slurm-26763440.out
-rw-rw----+ 1 lkemp nesi99991  865 May 11 12:35 slurm-26763444.out
-rw-rw----+ 1 lkemp nesi99991  857 May 11 12:36 slurm-26763447.out
-rw-rw----+ 1 lkemp nesi99991  829 May 11 12:36 slurm-26763448.out
-rw-rw----+ 1 lkemp nesi99991  857 May 11 12:36 slurm-26763449.out
-rw-rw----+ 1 lkemp nesi99991  857 May 11 12:36 slurm-26763450.out
-rw-rw----+ 1 lkemp nesi99991  829 May 11 12:36 slurm-26763451.out
-rw-rw----+ 1 lkemp nesi99991  829 May 11 12:36 slurm-26763452.out
-rw-rw----+ 1 lkemp nesi99991  885 May 11 12:37 slurm-26763454.out
-rw-rw----+ 1 lkemp nesi99991 1.8K May 11 12:34 Snakefile
```

{% endcapture %}

{% include exercise.html title="e4dot5" content=e4dot5%}
<br>

Let's clean this and create a dedicated folder `logs/slurm` for future log files:

```bash
# remove slurm log files
rm *.out

# create a new folder for Slurm log files
mkdir logs/slurm
```

then instruct Slurm to save its log files in it, in the profile `slurm/config.yaml` file

```diff
jobs: 20
cluster:
    sbatch
        --parsable
        --time {resources.time_min}
        --mem {resources.mem_mb}
        --cpus-per-task {resources.cpus}
+       --output logs/slurm/slurm-%j-{rule}.out
        --account nesi99991
default-resources: [cpus=2, mem_mb=512, time_min=10]
cluster-cancel: scancel
```

Note that `logs/slurm/slurm-%j-{rule}.out` contains a placeholder `{rule}`, which will be replaced by the name of the rule during the execution of the workflow.

Finally, to improve the communication between Snakemake and Slurm, we meed an additional script translating Slurm job status for Snakemake.
The `--cluster-status` option is used to tell Snakemake which script to use.

Create an executable `status.py` file

```bash
# create an empty file
touch status.py

# make it executable
chmod +x status.py
```

and copy the following content in it

```
#!/usr/bin/env python
import subprocess
import sys

jobid = sys.argv[1]

output = str(subprocess.check_output("sacct -j %s --format State --noheader | head -1 | awk '{print $1}'" % jobid, shell=True).strip())

running_status=["PENDING", "CONFIGURING", "COMPLETING", "RUNNING", "SUSPENDED"]
if "COMPLETED" in output:
    print("success")
elif any(r in output for r in running_status):
    print("running")
else:
    print("failed")
```

Then modify the profile `slurm/config.yaml` file

```diff
jobs: 20
cluster:
    sbatch
        --parsable
        --time {resources.time_min}
        --mem {resources.mem_mb}
        --cpus-per-task {resources.cpus}
        --output logs/slurm/slurm-%j-{rule}.out
        --account nesi99991
default-resources: [cpus=2, mem_mb=512, time_min=10]
cluster-cancel: scancel
+ cluster-status: ./status.py
```

Current slurm profile:

{% capture e4dot6 %}

```
jobs: 20
cluster:
    sbatch
        --parsable
        --time {resources.time_min}
        --mem {resources.mem_mb}
        --cpus-per-task {resources.cpus}
        --output logs/slurm/slurm-%j-{rule}.out
        --account nesi99991
default-resources: [cpus=2, mem_mb=512, time_min=10]
cluster-cancel: scancel
cluster-status: ./status.py
```

{% endcapture %}

{% include exercise.html title="e4dot6" content=e4dot6%}
<br>

Once all of this is in place, we can:

- submit Slurm jobs with the right resources per Snakemake rule,
- cancel the workflow and Slurms jobs using CTRL-C,
- keep all slurm jobs log files in a dedicated folder,
- and make sure Snakemake reports Slurm jobs failures.

> **Exercise:**
>
> Run the snakemake workflow with Slurm jobs then use `scancel JOBID` to cancel some Slurm. See how Snakemake reacts with and without the `status.py` script.


## 4.2 Pull out parameters

```diff
# define samples from data directory using wildcards
SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")

# target OUTPUT files for the whole workflow
rule all:
    input:
        "../results/multiqc_report.html",
        expand(["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"], sample = SAMPLES)

# workflow
rule fastqc:
    input:
        R1 = "../../data/{sample}_1.fastq.gz",
        R2 = "../../data/{sample}_2.fastq.gz"
    output:
        html = ["../results/fastqc/{sample}_1_fastqc.html", "../results/fastqc/{sample}_2_fastqc.html"],
        zip = ["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"]
    log:
        "logs/fastqc/{sample}.log"
    threads: 2
    envmodules:
        "FastQC/0.11.9"
    shell:
        "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} &> {log}"
  
rule multiqc:
    input:
        expand(["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"], sample = SAMPLES)
    output:
        "../results/multiqc_report.html"
    log:
        "logs/multiqc/multiqc.log"
    envmodules:
        "MultiQC/1.9-gimkl-2020a-Python-3.8.2"
    shell:
        "multiqc {input} -o ../results/ &> {log}"

rule trim_galore:
    input:
        ["../../data/{sample}_1.fastq.gz", "../../data/{sample}_2.fastq.gz"]
    output:
        ["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"]
+   params:
+       "--paired"
    log:
        "logs/trim_galore/{sample}.log"
    envmodules:
        "TrimGalore/0.6.7-gimkl-2020a-Python-3.8.2-Perl-5.30.1"
    threads: 2
    resources:
        cpus=8
    shell:
-       "trim_galore {input} -o ../results/trimmed/ --paired --cores {threads} &> {log}"
+       "trim_galore {input} -o ../results/trimmed/ {params} --cores {threads} &> {log}"
```

Current snakefile:

{% capture e4dot7 %}

```txt
# define samples from data directory using wildcards
SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")

# target OUTPUT files for the whole workflow
rule all:
    input:
        "../results/multiqc_report.html",
        expand(["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"], sample = SAMPLES)

# workflow
rule fastqc:
    input:
        R1 = "../../data/{sample}_1.fastq.gz",
        R2 = "../../data/{sample}_2.fastq.gz"
    output:
        html = ["../results/fastqc/{sample}_1_fastqc.html", "../results/fastqc/{sample}_2_fastqc.html"],
        zip = ["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"]
    log:
        "logs/fastqc/{sample}.log"
    threads: 2
    envmodules:
        "FastQC/0.11.9"
    shell:
        "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} &> {log}"
  
rule multiqc:
    input:
        expand(["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"], sample = SAMPLES)
    output:
        "../results/multiqc_report.html"
    log:
        "logs/multiqc/multiqc.log"
    envmodules:
        "MultiQC/1.9-gimkl-2020a-Python-3.8.2"
    shell:
        "multiqc {input} -o ../results/ &> {log}"

rule trim_galore:
    input:
        ["../../data/{sample}_1.fastq.gz", "../../data/{sample}_2.fastq.gz"]
    output:
        ["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"]
    params:
        "--paired"
    log:
        "logs/trim_galore/{sample}.log"
    envmodules:
        "TrimGalore/0.6.7-gimkl-2020a-Python-3.8.2-Perl-5.30.1"
    threads: 2
    resources:
        cpus=8
    shell:
        "trim_galore {input} -o ../results/trimmed/ {params} --cores {threads} &> {log}"
```

{% endcapture %}

{% include exercise.html title="e4dot7" content=e4dot7%}
<br>

Run a dryrun to check it works

```bash
# remove output of last run
rm -r ../results/*

# run dryrun again
snakemake --dryrun --profile slurm --use-envmodules
```

## 4.3 Pull out user configurable options

We can separate the user configurable options away from the workflow. This supports reproducibility by minimising the chance the user makes changes to the core workflow.

Create a configuration file in a new directory `config/`

File structure:

```bash
demo_workflow/
      |_______results/
      |_______workflow/
      |          |_______logs/
      |          |_______slurm/
      |          |_______Snakefile
      |          |_______status.py
      |_______config
                 |_______config.yaml
```

```bash
# create config directory
mkdir ../config

# create configuration file
touch ../config/config.yaml
```

Now we need to pull out the parameters the user would likely need to configure. Let's give the user the option to pass any parameters they like to fastqc. In our `../config/config.yaml` file, add the configuration options and add a couple flags to be passed to fastqc and multiqc:

```yaml
# set software parameters for...
PARAMS:
  # ... fastqc
  FASTQC: "--kmers 5"
  # ... multiqc
  MULTIQC: "--flat"
```

In the Snakefile, tell Snakemake to grab the variables `PARAMS` from `../config/config.yaml`

```diff
# define samples from data directory using wildcards
SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")

# target OUTPUT files for the whole workflow
rule all:
    input:
        "../results/multiqc_report.html",
        expand(["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"], sample = SAMPLES)

# workflow
rule fastqc:
    input:
        R1 = "../../data/{sample}_1.fastq.gz",
        R2 = "../../data/{sample}_2.fastq.gz"
    output:
        html = ["../results/fastqc/{sample}_1_fastqc.html", "../results/fastqc/{sample}_2_fastqc.html"],
        zip = ["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"]
+   params:
+       fastqc_params = config['PARAMS']['FASTQC']
    log:
        "logs/fastqc/{sample}.log"
    threads: 2
    envmodules:
        "FastQC/0.11.9"
    shell:
-       "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} &> {log}"
+       "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} {params.fastqc_params} &> {log}"
  
rule multiqc:
    input:
        expand(["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"], sample = SAMPLES)
    output:
        "../results/multiqc_report.html"
+   params:
+       multiqc_params = config['PARAMS']['MULTIQC']
    log:
        "logs/multiqc/multiqc.log"
    envmodules:
        "MultiQC/1.9-gimkl-2020a-Python-3.8.2"
    shell:
-       "multiqc {input} -o ../results/ &> {log}"
+       "multiqc {input} -o ../results/ {params.multiqc_params} &> {log}"

rule trim_galore:
    input:
        ["../../data/{sample}_1.fastq.gz", "../../data/{sample}_2.fastq.gz"]
    output:
        ["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"]
    params:
        "--paired"
    log:
        "logs/trim_galore/{sample}.log"
    envmodules:
        "TrimGalore/0.6.7-gimkl-2020a-Python-3.8.2-Perl-5.30.1"
    threads: 2
    resources:
        cpus=8
    shell:
        "trim_galore {input} -o ../results/trimmed/ {params} --cores {threads} &> {log}"
```

Current snakefile:

{% capture e4dot8 %}

```txt
# define samples from data directory using wildcards
SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")

# target OUTPUT files for the whole workflow
rule all:
    input:
        "../results/multiqc_report.html",
        expand(["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"], sample = SAMPLES)

# workflow
rule fastqc:
    input:
        R1 = "../../data/{sample}_1.fastq.gz",
        R2 = "../../data/{sample}_2.fastq.gz"
    output:
        html = ["../results/fastqc/{sample}_1_fastqc.html", "../results/fastqc/{sample}_2_fastqc.html"],
        zip = ["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"]
    params:
        fastqc_params = config['PARAMS']['FASTQC']
    log:
        "logs/fastqc/{sample}.log"
    threads: 2
    envmodules:
        "FastQC/0.11.9"
    shell:
        "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} {params.fastqc_params} &> {log}"
  
rule multiqc:
    input:
        expand(["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"], sample = SAMPLES)
    output:
        "../results/multiqc_report.html"
    params:
        multiqc_params = config['PARAMS']['MULTIQC']
    log:
        "logs/multiqc/multiqc.log"
    envmodules:
        "MultiQC/1.9-gimkl-2020a-Python-3.8.2"
    shell:
        "multiqc {input} -o ../results/ {params.multiqc_params} &> {log}"

rule trim_galore:
    input:
        ["../../data/{sample}_1.fastq.gz", "../../data/{sample}_2.fastq.gz"]
    output:
        ["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"]
    params:
        "--paired"
    log:
        "logs/trim_galore/{sample}.log"
    envmodules:
        "TrimGalore/0.6.7-gimkl-2020a-Python-3.8.2-Perl-5.30.1"
    threads: 2
    resources:
        cpus=8
    shell:
        "trim_galore {input} -o ../results/trimmed/ {params} --cores {threads} &> {log}"
```

{% endcapture %}

{% include exercise.html title="e4dot8" content=e4dot8%}
<br>

Let's use our configuration file! Run workflow again:

```bash
# remove output of last run
rm -r ../results/*

# run dryrun/run again
snakemake --dryrun --profile slurm --use-envmodules
snakemake --profile slurm --use-envmodules
```

Didn't work?

My error:

{% capture e4dot9 %}

```bash
KeyError in line 19 of /scale_wlg_persistent/filesets/project/nesi99991/snakemake20220512/lkemp/snakemake_workshop/demo_workflow/workflow/Snakefile:
'PARAMS'
  File "/scale_wlg_persistent/filesets/project/nesi99991/snakemake20220512/lkemp/snakemake_workshop/demo_workflow/workflow/Snakefile", line 19, in <module>
```

{% endcapture %}

{% include exercise.html title="e4dot9" content=e4dot9%}
<br>

Snakemake can't find our 'Key' - we haven't told Snakemake where our config file is so it can't find our config variables. We can do this by passing the location of our config file to the `--configfile` flag

```diff
# remove output of last run
rm -r ../results/

# run dryrun/run again
- snakemake --dryrun --profile slurm --use-envmodules
- snakemake --profile slurm --use-envmodules
+ snakemake --dryrun --profile slurm --use-envmodules --configfile ../config/config.yaml
+ snakemake --profile slurm --use-envmodules --configfile ../config/config.yaml
```

Alternatively, we can define our config file in our Snakefile in a situation where the configuration file is likely to always be named the same and be in the exact same location `../config/config.yaml` and you don't need the flexibility for the user to specify their own configuration files:

```diff
# define our configuration file
+ configfile: "../config/config.yaml"

# define samples from data directory using wildcards
SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")

# target OUTPUT files for the whole workflow
rule all:
    input:
        "../results/multiqc_report.html",
        expand(["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"], sample = SAMPLES)

# workflow
rule fastqc:
    input:
        R1 = "../../data/{sample}_1.fastq.gz",
        R2 = "../../data/{sample}_2.fastq.gz"
    output:
        html = ["../results/fastqc/{sample}_1_fastqc.html", "../results/fastqc/{sample}_2_fastqc.html"],
        zip = ["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"]
    params:
        fastqc_params = config['PARAMS']['FASTQC']
    log:
        "logs/fastqc/{sample}.log"
    threads: 2
    envmodules:
        "FastQC/0.11.9"
    shell:
        "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} {params.fastqc_params} &> {log}"
  
rule multiqc:
    input:
        expand(["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"], sample = SAMPLES)
    output:
        "../results/multiqc_report.html"
    params:
        multiqc_params = config['PARAMS']['MULTIQC']
    log:
        "logs/multiqc/multiqc.log"
    envmodules:
        "MultiQC/1.9-gimkl-2020a-Python-3.8.2"
    shell:
        "multiqc {input} -o ../results/ {params.multiqc_params} &> {log}"

rule trim_galore:
    input:
        ["../../data/{sample}_1.fastq.gz", "../../data/{sample}_2.fastq.gz"]
    output:
        ["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"]
    params:
        "--paired"
    log:
        "logs/trim_galore/{sample}.log"
    envmodules:
        "TrimGalore/0.6.7-gimkl-2020a-Python-3.8.2-Perl-5.30.1"
    threads: 2
    resources:
        cpus=8
    shell:
        "trim_galore {input} -o ../results/trimmed/ {params} --cores {threads} &> {log}"
```

Current snakefile:

{% capture e4dot10 %}

```txt
# define our configuration file
configfile: "../config/config.yaml"

# define samples from data directory using wildcards
SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")

# target OUTPUT files for the whole workflow
rule all:
    input:
        "../results/multiqc_report.html",
        expand(["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"], sample = SAMPLES)

# workflow
rule fastqc:
    input:
        R1 = "../../data/{sample}_1.fastq.gz",
        R2 = "../../data/{sample}_2.fastq.gz"
    output:
        html = ["../results/fastqc/{sample}_1_fastqc.html", "../results/fastqc/{sample}_2_fastqc.html"],
        zip = ["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"]
    params:
        fastqc_params = config['PARAMS']['FASTQC']
    log:
        "logs/fastqc/{sample}.log"
    threads: 2
    envmodules:
        "FastQC/0.11.9"
    shell:
        "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} {params.fastqc_params} &> {log}"
  
rule multiqc:
    input:
        expand(["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"], sample = SAMPLES)
    output:
        "../results/multiqc_report.html"
    params:
        multiqc_params = config['PARAMS']['MULTIQC']
    log:
        "logs/multiqc/multiqc.log"
    envmodules:
        "MultiQC/1.9-gimkl-2020a-Python-3.8.2"
    shell:
        "multiqc {input} -o ../results/ {params.multiqc_params} &> {log}"

rule trim_galore:
    input:
        ["../../data/{sample}_1.fastq.gz", "../../data/{sample}_2.fastq.gz"]
    output:
        ["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"]
    params:
        "--paired"
    log:
        "logs/trim_galore/{sample}.log"
    envmodules:
        "TrimGalore/0.6.7-gimkl-2020a-Python-3.8.2-Perl-5.30.1"
    threads: 2
    resources:
        cpus=8
    shell:
        "trim_galore {input} -o ../results/trimmed/ {params} --cores {threads} &> {log}"
```

{% endcapture %}

{% include exercise.html title="e4dot10" content=e4dot10%}
<br>

Then we don't need to specify where the configuration file is on the command line

```diff
# remove output of last run
rm -r ../results/*

# run dryrun/run again
- snakemake --dryrun --profile slurm --use-envmodules --configfile ../config/config.yaml
- snakemake --profile slurm --use-envmodules --configfile ../config/config.yaml
+ snakemake --dryrun --profile slurm --use-envmodules
+ snakemake --profile slurm --use-envmodules
```

## 4.4 Leave messages for the user

We can provide the user of our workflow more information on what is happening at each stage/rule of our workflow via the `message:` directive. We are able to call many variables such as:

- Input and output files `{input}` and `{output}`
- Specific input and output files such as `{input.R1}`
- Our `{params}`, `{log}` and `{threads}` directives

```diff
# define our configuration file
configfile: "../config/config.yaml"

# define samples from data directory using wildcards
SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")

# target OUTPUT files for the whole workflow
rule all:
    input:
        "../results/multiqc_report.html",
        expand(["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"], sample = SAMPLES)

# workflow
rule fastqc:
    input:
        R1 = "../../data/{sample}_1.fastq.gz",
        R2 = "../../data/{sample}_2.fastq.gz"
    output:
        html = ["../results/fastqc/{sample}_1_fastqc.html", "../results/fastqc/{sample}_2_fastqc.html"],
        zip = ["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"]
    params:
        fastqc_params = config['PARAMS']['FASTQC']
    log:
        "logs/fastqc/{sample}.log"
    threads: 2
    envmodules:
        "FastQC/0.11.9"
+   message:
+       "Undertaking quality control checks {input}"
    shell:
        "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} {params.fastqc_params} &> {log}"
  
rule multiqc:
    input:
        expand(["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"], sample = SAMPLES)
    output:
        "../results/multiqc_report.html"
    params:
        multiqc_params = config['PARAMS']['MULTIQC']
    log:
        "logs/multiqc/multiqc.log"
    envmodules:
        "MultiQC/1.9-gimkl-2020a-Python-3.8.2"
+   message:
+       "Compiling a HTML report for quality control checks. Writing to {output}."
    shell:
        "multiqc {input} -o ../results/ {params.multiqc_params} &> {log}"

rule trim_galore:
    input:
        ["../../data/{sample}_1.fastq.gz", "../../data/{sample}_2.fastq.gz"]
    output:
        ["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"]
    params:
        "--paired"
    log:
        "logs/trim_galore/{sample}.log"
    envmodules:
        "TrimGalore/0.6.7-gimkl-2020a-Python-3.8.2-Perl-5.30.1"
    threads: 2
    resources:
        cpus=8
+   message:
+       "Trimming using these parameter: {params}. Writing logs to {log}. Using {threads} threads."
    shell:
        "trim_galore {input} -o ../results/trimmed/ {params} --cores {threads} &> {log}"
```

Current snakefile:

{% capture e4dot11 %}

```txt
# define our configuration file
configfile: "../config/config.yaml"

# define samples from data directory using wildcards
SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")

# target OUTPUT files for the whole workflow
rule all:
    input:
        "../results/multiqc_report.html",
        expand(["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"], sample = SAMPLES)

# workflow
rule fastqc:
    input:
        R1 = "../../data/{sample}_1.fastq.gz",
        R2 = "../../data/{sample}_2.fastq.gz"
    output:
        html = ["../results/fastqc/{sample}_1_fastqc.html", "../results/fastqc/{sample}_2_fastqc.html"],
        zip = ["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"]
    params:
        fastqc_params = config['PARAMS']['FASTQC']
    log:
        "logs/fastqc/{sample}.log"
    threads: 2
    envmodules:
        "FastQC/0.11.9"
    message:
        "Undertaking quality control checks {input}"
    shell:
        "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} {params.fastqc_params} &> {log}"
  
rule multiqc:
    input:
        expand(["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"], sample = SAMPLES)
    output:
        "../results/multiqc_report.html"
    params:
        multiqc_params = config['PARAMS']['MULTIQC']
    log:
        "logs/multiqc/multiqc.log"
    envmodules:
        "MultiQC/1.9-gimkl-2020a-Python-3.8.2"
    message:
        "Compiling a HTML report for quality control checks. Writing to {output}."
    shell:
        "multiqc {input} -o ../results/ {params.multiqc_params} &> {log}"

rule trim_galore:
    input:
        ["../../data/{sample}_1.fastq.gz", "../../data/{sample}_2.fastq.gz"]
    output:
        ["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"]
    params:
        "--paired"
    log:
        "logs/trim_galore/{sample}.log"
    envmodules:
        "TrimGalore/0.6.7-gimkl-2020a-Python-3.8.2-Perl-5.30.1"
    threads: 2
    resources:
        cpus=8
    message:
        "Trimming using these parameter: {params}. Writing logs to {log}. Using {threads} threads."
    shell:
        "trim_galore {input} -o ../results/trimmed/ {params} --cores {threads} &> {log}"
```

{% endcapture %}

{% include exercise.html title="e4dot11" content=e4dot11%}
<br>

```diff
# remove output of last run
rm -r ../results/*

# run dryrun/run again
snakemake --dryrun --profile slurm --use-envmodules
snakemake --profile slurm --use-envmodules
```

Now our messages are printed to the screen as our workflow runs

My output:

{% capture e4dot12 %}

```bash
Building DAG of jobs...
Using shell: /usr/bin/bash
Provided cluster nodes: 20
Job stats:
job            count    min threads    max threads
-----------  -------  -------------  -------------
all                1              1              1
fastqc             3              2              2
multiqc            1              1              1
trim_galore        3              2              2
total              8              1              2

Select jobs to execute...

[Wed May 11 13:20:52 2022]
Job 4: Undertaking quality control checks ../../data/NA24694_1.fastq.gz ../../data/NA24694_2.fastq.gz

Submitted job 4 with external jobid '26763840'.

[Wed May 11 13:20:52 2022]
Job 6: Trimming using these parameter: --paired. Writing logs to logs/trim_galore/NA24695.log. Using 2 threads.

Submitted job 6 with external jobid '26763841'.

[Wed May 11 13:20:52 2022]
Job 2: Undertaking quality control checks ../../data/NA24631_1.fastq.gz ../../data/NA24631_2.fastq.gz

Submitted job 2 with external jobid '26763842'.

[Wed May 11 13:20:52 2022]
Job 3: Undertaking quality control checks ../../data/NA24695_1.fastq.gz ../../data/NA24695_2.fastq.gz

Submitted job 3 with external jobid '26763843'.

[Wed May 11 13:20:52 2022]
Job 5: Trimming using these parameter: --paired. Writing logs to logs/trim_galore/NA24631.log. Using 2 threads.

Submitted job 5 with external jobid '26763844'.

[Wed May 11 13:20:52 2022]
Job 7: Trimming using these parameter: --paired. Writing logs to logs/trim_galore/NA24694.log. Using 2 threads.

Submitted job 7 with external jobid '26763845'.
[Wed May 11 13:22:08 2022]
Finished job 4.
1 of 8 steps (12%) done
[Wed May 11 13:22:08 2022]
Finished job 6.
2 of 8 steps (25%) done
[Wed May 11 13:22:08 2022]
Finished job 2.
3 of 8 steps (38%) done
[Wed May 11 13:22:08 2022]
Finished job 3.
4 of 8 steps (50%) done
Select jobs to execute...

[Wed May 11 13:22:09 2022]
Job 1: Compiling a HTML report for quality control checks. Writing to ../results/multiqc_report.html.

Submitted job 1 with external jobid '26763848'.
[Wed May 11 13:22:09 2022]
Finished job 5.
5 of 8 steps (62%) done
[Wed May 11 13:22:09 2022]
Finished job 7.
6 of 8 steps (75%) done
[Wed May 11 13:24:21 2022]
Finished job 1.
7 of 8 steps (88%) done
Select jobs to execute...

[Wed May 11 13:24:21 2022]
localrule all:
    input: ../results/multiqc_report.html, ../results/trimmed/NA24631_1_val_1.fq.gz, ../results/trimmed/NA24695_1_val_1.fq.gz, ../results/trimmed/NA24694_1_val_1.fq.gz, ../results/trimmed/NA24631_2_val_2.fq.gz, ../results/trimmed/NA24695_2_val_2.fq.gz, ../results/trimmed/NA24694_2_val_2.fq.gz
    jobid: 0
    resources: mem_mb=512, disk_mb=1000, tmpdir=/dev/shm/jobs/26763281, cpus=2, time_min=10

[Wed May 11 13:24:21 2022]
Finished job 0.
8 of 8 steps (100%) done
Complete log: .snakemake/log/2022-05-11T132052.454902.snakemake.log
```

{% endcapture %}

{% include exercise.html title="e4dot12" content=e4dot12%}
<br>

## 4.5 Create temporary files

In our workflow, we are likely to be creating files that we don't want, but are used or produced by our workflow (intermediate files). We can mark such files as temporary so Snakemake will remove the file once it doesn't need to use it anymore.

For example, we might not want to keep our fastqc output files since our multiqc report merges all of our fastqc reports for each sample into one report. Let's have a look at the files currently produced by our workflow with:

```bash
ls -lh ../results/fastqc/
```

My output:

{% capture e4dot13 %}

```bash
total 4.5M
-rw-rw----+ 1 lkemp nesi99991 250K May 11 13:22 NA24631_1_fastqc.html
-rw-rw----+ 1 lkemp nesi99991 327K May 11 13:22 NA24631_1_fastqc.zip
-rw-rw----+ 1 lkemp nesi99991 249K May 11 13:22 NA24631_2_fastqc.html
-rw-rw----+ 1 lkemp nesi99991 327K May 11 13:22 NA24631_2_fastqc.zip
-rw-rw----+ 1 lkemp nesi99991 254K May 11 13:22 NA24694_1_fastqc.html
-rw-rw----+ 1 lkemp nesi99991 334K May 11 13:22 NA24694_1_fastqc.zip
-rw-rw----+ 1 lkemp nesi99991 250K May 11 13:22 NA24694_2_fastqc.html
-rw-rw----+ 1 lkemp nesi99991 328K May 11 13:22 NA24694_2_fastqc.zip
-rw-rw----+ 1 lkemp nesi99991 252K May 11 13:22 NA24695_1_fastqc.html
-rw-rw----+ 1 lkemp nesi99991 328K May 11 13:22 NA24695_1_fastqc.zip
-rw-rw----+ 1 lkemp nesi99991 253K May 11 13:22 NA24695_2_fastqc.html
-rw-rw----+ 1 lkemp nesi99991 330K May 11 13:22 NA24695_2_fastqc.zip
```

{% endcapture %}

{% include exercise.html title="e4dot13" content=e4dot13%}
<br>

Let's mark all the trimmed fastq files as temporary in our Snakefile by wrapping it up in the `temp()` function

```diff
# define our configuration file
configfile: "../config/config.yaml"

# define samples from data directory using wildcards
SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")

# target OUTPUT files for the whole workflow
rule all:
    input:
        "../results/multiqc_report.html",
        expand(["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"], sample = SAMPLES)

# workflow
rule fastqc:
    input:
        R1 = "../../data/{sample}_1.fastq.gz",
        R2 = "../../data/{sample}_2.fastq.gz"
    output:
-       html = ["../results/fastqc/{sample}_1_fastqc.html", "../results/fastqc/{sample}_2_fastqc.html"],
+       html = temp(["../results/fastqc/{sample}_1_fastqc.html", "../results/fastqc/{sample}_2_fastqc.html"]),
        zip = ["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"]
    params:
        fastqc_params = config['PARAMS']['FASTQC']
    log:
        "logs/fastqc/{sample}.log"
    threads: 2
    envmodules:
        "FastQC/0.11.9"
    message:
        "Undertaking quality control checks {input}"
    shell:
        "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} {params.fastqc_params} &> {log}"
  
rule multiqc:
    input:
        expand(["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"], sample = SAMPLES)
    output:
        "../results/multiqc_report.html"
    params:
        multiqc_params = config['PARAMS']['MULTIQC']
    log:
        "logs/multiqc/multiqc.log"
    envmodules:
        "MultiQC/1.9-gimkl-2020a-Python-3.8.2"
    message:
        "Compiling a HTML report for quality control checks. Writing to {output}."
    shell:
        "multiqc {input} -o ../results/ {params.multiqc_params} &> {log}"

rule trim_galore:
    input:
        ["../../data/{sample}_1.fastq.gz", "../../data/{sample}_2.fastq.gz"]
    output:
        ["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"]
    params:
        "--paired"
    log:
        "logs/trim_galore/{sample}.log"
    envmodules:
        "TrimGalore/0.6.7-gimkl-2020a-Python-3.8.2-Perl-5.30.1"
    threads: 2
    resources:
        cpus=8
    message:
        "Trimming using these parameter: {params}. Writing logs to {log}. Using {threads} threads."
    shell:
        "trim_galore {input} -o ../results/trimmed/ {params} --cores {threads} &> {log}"
```

Current snakefile:

{% capture e4dot14 %}

```txt
# define our configuration file
configfile: "../config/config.yaml"

# define samples from data directory using wildcards
SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")

# target OUTPUT files for the whole workflow
rule all:
    input:
        "../results/multiqc_report.html",
        expand(["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"], sample = SAMPLES)

# workflow
rule fastqc:
    input:
        R1 = "../../data/{sample}_1.fastq.gz",
        R2 = "../../data/{sample}_2.fastq.gz"
    output:
        html = temp(["../results/fastqc/{sample}_1_fastqc.html", "../results/fastqc/{sample}_2_fastqc.html"]),
        zip = ["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"]
    params:
        fastqc_params = config['PARAMS']['FASTQC']
    log:
        "logs/fastqc/{sample}.log"
    threads: 2
    envmodules:
        "FastQC/0.11.9"
    message:
        "Undertaking quality control checks {input}"
    shell:
        "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} {params.fastqc_params} &> {log}"
  
rule multiqc:
    input:
        expand(["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"], sample = SAMPLES)
    output:
        "../results/multiqc_report.html"
    params:
        multiqc_params = config['PARAMS']['MULTIQC']
    log:
        "logs/multiqc/multiqc.log"
    envmodules:
        "MultiQC/1.9-gimkl-2020a-Python-3.8.2"
    message:
        "Compiling a HTML report for quality control checks. Writing to {output}."
    shell:
        "multiqc {input} -o ../results/ {params.multiqc_params} &> {log}"

rule trim_galore:
    input:
        ["../../data/{sample}_1.fastq.gz", "../../data/{sample}_2.fastq.gz"]
    output:
        ["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"]
    params:
        "--paired"
    log:
        "logs/trim_galore/{sample}.log"
    envmodules:
        "TrimGalore/0.6.7-gimkl-2020a-Python-3.8.2-Perl-5.30.1"
    threads: 2
    resources:
        cpus=8
    message:
        "Trimming using these parameter: {params}. Writing logs to {log}. Using {threads} threads."
    shell:
        "trim_galore {input} -o ../results/trimmed/ {params} --cores {threads} &> {log}"
```

{% endcapture %}

{% include exercise.html title="e4dot14" content=e4dot14%}
<br>

```diff
# remove output of last run
rm -r ../results/*

# run dryrun/run again
snakemake --dryrun --profile slurm --use-envmodules
snakemake --profile slurm --use-envmodules
```

Now when we have a look at the `../results/fastqc/` directory with:

```bash
ls -lh ../results/fastqc/
```

These html files have been removed once Snakemake no longer needs the files for another rule/operation, and we've saved some space on our computer (from 4.5 megabytes to 3 megabytes in this directory).

My output:

{% capture e4dot15 %}

```bash
total 3.0M
-rw-rw----+ 1 lkemp nesi99991 327K May 11 13:26 NA24631_1_fastqc.zip
-rw-rw----+ 1 lkemp nesi99991 327K May 11 13:26 NA24631_2_fastqc.zip
-rw-rw----+ 1 lkemp nesi99991 334K May 11 13:26 NA24694_1_fastqc.zip
-rw-rw----+ 1 lkemp nesi99991 328K May 11 13:26 NA24694_2_fastqc.zip
-rw-rw----+ 1 lkemp nesi99991 328K May 11 13:26 NA24695_1_fastqc.zip
-rw-rw----+ 1 lkemp nesi99991 330K May 11 13:26 NA24695_2_fastqc.zip
```

{% endcapture %}

{% include exercise.html title="e4dot15" content=e4dot15%}
<br>

*This becomes particularly important when our data become big data, since we don't want to keep any massive intermediate output files that we don't need. Otherwise this can start to clog up the memory on our computer. It ensures our workflow is scalable when our data becomes big data.*

## 4.6 Generating a snakemake report

With Snakemake, we can automatically generate detailed self-contained HTML reports after we run our workflow with the following command:

```bash
snakemake --report ../results/snakemake_report.html
```

*Note. you won't be able to view a rendered version of this html while it is on the remote server, however after you transfer it to your local computer you should be able to view it in your web browser*

In our report:

- We get an interactive version of our directed acyclic graph (DAG).
- When you click on a node in the DAG, the input and output files are fully outlined, the exact software used and the exact shell command that was run.
- You are also provided with runtime information under the `Statistics` tab outlining how long each rule/sample ran for, and the date/time each file was created.

My report:

{% capture e4dot16 %}

![snakemake_report](./images/snakemake_report.gif)

{% endcapture %}

{% include exercise.html title="e4dot16" content=e4dot16%}
<br>

These reports are highly configurable, have a look at an example of what can be done with a report [here](https://koesterlab.github.io/resources/report.html)

*See more information on creating Snakemake reports [in the Snakemake documentation](https://snakemake.readthedocs.io/en/stable/snakefiles/reporting.html)*

## 4.7 Linting your workflow

Snakemake has a built in linter to support you building best practice workflows, let's try it out:

```bash
snakemake --lint
```

My output:

{% capture e4dot17 %}

```bash
Lints for rule fastqc (line 21, /scale_wlg_persistent/filesets/project/nesi99991/snakemake20220512/lkemp/snakemake_workshop/demo_workflow/workflow/Snakefile):
    * Additionally specify a conda environment or container for each rule, environment modules are not enough:
      While environment modules allow to document and deploy the required software on a certain platform, they lock your workflow in there, disabling easy reproducibility on
      other machines that don't have exactly the same environment modules. Hence env modules (which might be beneficial in certain cluster environments), should allways be
      complemented with equivalent conda environments.
      Also see:
      https://snakemake.readthedocs.io/en/latest/snakefiles/deployment.html#integrated-package-management
      https://snakemake.readthedocs.io/en/latest/snakefiles/deployment.html#running-jobs-in-containers

Lints for rule multiqc (line 61, /scale_wlg_persistent/filesets/project/nesi99991/snakemake20220512/lkemp/snakemake_workshop/demo_workflow/workflow/Snakefile):
    * Additionally specify a conda environment or container for each rule, environment modules are not enough:
      While environment modules allow to document and deploy the required software on a certain platform, they lock your workflow in there, disabling easy reproducibility on
      other machines that don't have exactly the same environment modules. Hence env modules (which might be beneficial in certain cluster environments), should allways be
      complemented with equivalent conda environments.
      Also see:
      https://snakemake.readthedocs.io/en/latest/snakefiles/deployment.html#integrated-package-management
      https://snakemake.readthedocs.io/en/latest/snakefiles/deployment.html#running-jobs-in-containers

Lints for rule trim_galore (line 96, /scale_wlg_persistent/filesets/project/nesi99991/snakemake20220512/lkemp/snakemake_workshop/demo_workflow/workflow/Snakefile):
    * Additionally specify a conda environment or container for each rule, environment modules are not enough:
      While environment modules allow to document and deploy the required software on a certain platform, they lock your workflow in there, disabling easy reproducibility on
      other machines that don't have exactly the same environment modules. Hence env modules (which might be beneficial in certain cluster environments), should allways be
      complemented with equivalent conda environments.
      Also see:
      https://snakemake.readthedocs.io/en/latest/snakefiles/deployment.html#integrated-package-management
      https://snakemake.readthedocs.io/en/latest/snakefiles/deployment.html#running-jobs-in-containers
```

{% endcapture %}

{% include exercise.html title="e4dot17" content=e4dot17%}
<br>

We have a few things we could improve in our workflow!

Writing a best practice workflow is more important than having Marie Kondo level tidiness, it increases the chance your workflow will continue to be used and maintained by others (and ourselves), making the code we write useful (it's exciting seeing someone else using your code!). If your workflow was used in scientific research, it makes your workflow accessible for people to reproduce your research findings; it isn't going to be a nightmare for them to run and they are more likely to try and have success doing so.

Read more about the [best practices for Snakemake](https://snakemake.readthedocs.io/en/stable/snakefiles/best_practices.html)

# Takeaways

---

- Pull out your parameters and put them in `params:` directive
- Pulling the user configurable options away from the core workflow will support reproducibility by reducing the chance of changes to the core workflow
- Leaving messages for the user of your workflow will help them understand what is happening at each stage and follow the workflows progress
- Mark files you won't need once the workflow completes to reduce the memory usage - *particularly* when dealing with big data
- Generate a snakemake report to get a summary of the workflow run - these are highly configurable
- Lint your workflow and check it complies with best practices - this supports reproducibility and portability
- There is so much more to explore, such as creating [modular workflows](https://snakemake.readthedocs.io/en/stable/snakefiles/modularization.html), automatically grabbing [remote files](https://snakemake.readthedocs.io/en/stable/snakefiles/remote_files.html) from places like Google Cloud Storage and Dropbox, [run various types of scripts](https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html#external-scripts) such as [python scripts](https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html#python), [R and RMarkdown scripts](https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html#r-and-r-markdown) and [Jupyter Notebooks](https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html#jupyter-notebook-integration)

---

# Summary commands

Use the parameter directive (`params`) to keep the parameters and flags of your programs separate from your shell command, for example:

```bash
params:
    "--paired"
```

Run your snakemake workflow (using environment modules to load your software AND with a configuration file) with:

```bash
snakemake --cores 2 --use-envmodules --configfile ../config/config.yaml
```

Alternatively, define your config file in the Snakefile:

```bash
configfile: "../config/config.yaml"
```

Use the `message` directive to provide information to the user on what is happening real time, for example:

```bash
message:
    "Undertaking quality control checks {input}"
```

Mark temporary files to remove (once they are no longer needed by the workflow) with `temp()`, for example:

```bash
temp(["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"])
```

Create a basic interactive Snakemake report after running your workflow with:

```bash
snakemake --report ../results/snakemake_report.html
```

# Our final snakemake workflow!

See [leveled_up_demo_workflow](https://github.com/nesi/snakemake_workshop/tree/main/leveled_up_demo_workflow) for the final Snakemake workflow we've created up to this point

- - - 
<p style="text-align:left;">
  <b><a class="btn" href="https://nesi.github.io/snakemake_workshop/workshop_material/03_create_a_basic_workflow.html" style="background: var(--bs-green);font-weight:bold">&laquo; 3 - Creating a Basic WF</a></b> 
  <span style="float:right;">
    <b><a class="btn" href="https://nesi.github.io/snakemake_workshop/workshop_material/05_we_want_more.html" style="background: var(--bs-green);font-weight:bold">5 - More &raquo;</a></b>
  </span>
</p>

<p align="center"><b><a class="btn" href="https://nesi.github.io/snakemake_workshop/" style="background: var(--bs-dark);font-weight:bold">Back to homepage</a></b></p>
