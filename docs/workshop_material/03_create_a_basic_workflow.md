# 03 - Create a basic workflow




## 3.01 Aim

---
!!! info ""

    Let's create a basic workflow that will do some of the analysis steps for genetic data. We will have three samples with two files each - six files in total. These files will be processed through the below workflow, passing through three software.

---
<center>
![rulegraph_1](./images/rulegraph_1.png)
</center>

We have paired end sequencing data for three samples `NA24631` to process in the `./data` directory. Let's have a look:

!!! terminal "code"

    ```bash
    ls -lh ./data/
    ```

    ??? success "output"
    
        ```bash
        total 13M
        -rw-rw----+ 1 lkemp nesi99991 2.1M May 11 12:06 NA24631_1.fastq.gz
        -rw-rw----+ 1 lkemp nesi99991 2.3M May 11 12:06 NA24631_2.fastq.gz
        -rw-rw----+ 1 lkemp nesi99991 2.1M May 11 12:06 NA24694_1.fastq.gz
        -rw-rw----+ 1 lkemp nesi99991 2.3M May 11 12:06 NA24694_2.fastq.gz
        -rw-rw----+ 1 lkemp nesi99991 1.8M May 11 12:06 NA24695_1.fastq.gz
        -rw-rw----+ 1 lkemp nesi99991 1.9M May 11 12:06 NA24695_2.fastq.gz
        ```

<br>

## 3.02 Snakemake workflow file structure

Workflow file structure:

```bash
demo_workflow/
      |_______results/
      |_______workflow/
                 |_______Snakefile
```

We will create and run our workflow from the `workflow` directory send all of our file outputs/results to the `results` directory

*Read up on the best practice workflow structure [here](https://snakemake.readthedocs.io/en/stable/snakefiles/deployment.html#distribution-and-reproducibility)*

Create this file structure and our main Snakefile with:

!!! terminal "code"

    ```bash
    mkdir -p demo_workflow/{results,workflow}
    ```
    ```bash
    touch demo_workflow/workflow/Snakefile
    ```

Now you should have the very beginnings of your Snakemake workflow in a `demo_workflow` directory. Let's have a look:

!!! terminal "code"

    ```bash
    ls -lh demo_workflow/
    ```

    ??? success "output"
    
    
        ```bash
        total 1.0K
        drwxrws---+ 2 lkemp nesi99991 4.0K May 11 12:07 results
        drwxrws---+ 2 lkemp nesi99991 4.0K May 11 12:07 workflow
        ```
<br>
!!! terminal "code"

    ```bash
    ls -lh demo_workflow/workflow/
    ```

    ??? success "output"
    
        ```bash
        total 0
        -rw-rw----+ 1 lkemp nesi99991 0 May 11 12:07 Snakefile
        ```

<br>

Within the `workflow` directory (where we will create and run our workflow), we have a `Snakefile` file that will be the backbone of our workflow.

## 3.03 Run the software on the command line

First lets run the first step in our workflow ([fastqc](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/)) directly on the command line to get the syntax of the command right and check what outputs files we expect to get. Knowing what files the software will output is important for Snakemake since it is a lazy "pull" based system where software/rules will only run if you tell it to create the output file. We will talk more about this later!

!!! terminal "code"

    - First make sure to have [fastqc](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/) available. On NeSI, load the corresponding module
    ```bash
    module load FastQC/0.11.9
    ```
    
    - See what parameters are available so we know how we want to run this software before we put it in a Snakemake workflow
    ```bash
    fastqc --help
    ```
    
    - Create a test directory to put the output files
    ```bash
    mkdir fastqc_test
    ```
    
    Run fastqc directly on the command line on one of the samples
    ```bash
    fastqc ./data/NA24631_1.fastq.gz ./data/NA24631_2.fastq.gz -o ./fastqc_test -t 2
    ```
    
    ??? success "output"
    
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
        Approx 25% complete for NA24631_2.fastq.gz
        Approx 85% complete for NA24631_1.fastq.gz
        Approx 90% complete for NA24631_1.fastq.gz
        Approx 30% complete for NA24631_2.fastq.gz
        Approx 35% complete for NA24631_2.fastq.gz
        Approx 95% complete for NA24631_1.fastq.gz
        Approx 40% complete for NA24631_2.fastq.gz
        Analysis complete for NA24631_1.fastq.gz
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
        ```
    
    <br>
    
    - What are the output files of fastqc? Find out with:
    ```bash
    ls -lh ./fastqc_test
    ```
    
    ??? success "output"

        ```bash
        total 2.5M
        -rw-rw----+ 1 lkemp nesi99991 718K May 11 12:08 NA24631_1_fastqc.html
        -rw-rw----+ 1 lkemp nesi99991 475K May 11 12:08 NA24631_1_fastqc.zip
        -rw-rw----+ 1 lkemp nesi99991 726K May 11 12:08 NA24631_2_fastqc.html
        -rw-rw----+ 1 lkemp nesi99991 479K May 11 12:08 NA24631_2_fastqc.zip
        ```
    
<br>

## 3.04 Create the first rule in your workflow

Let's wrap this up in a Snakemake workflow! Start with the basic structure of a Snakefile:

```sh
# target OUTPUT files for the whole workflow
rule all:
    input:

# workflow
rule my_rule:
    input:
        ""
    output:
        ""
    threads:
    shell:
        ""
```

!!! scale-balance "Now add our fastqc rule, let's:"

    - Name the rule
    - Fill in the the input fastq files from the `data` directory (*path relative to the Snakefile*)
    - Fill in the output files (now you can see it's useful to know what files fastqc outputs!)
    - Set the number of threads
    - Write the fastqc shell command in the `shell:` section and pass the input/output variables to the shell command
    - Set the final output files for the whole workflow in `rule all:`

---

The use of the word `input` in `rule all` can be confusing, but in this context, it is referring to the final *output* files of the whole workflow

---
!!! terminal ""
    ```diff
    # target OUTPUT files for the whole workflow
    rule all:
        input:
    +       "../results/fastqc/NA24631_1_fastqc.html",
    +       "../results/fastqc/NA24631_2_fastqc.html",
    +       "../results/fastqc/NA24631_1_fastqc.zip",
    +       "../results/fastqc/NA24631_2_fastqc.zip"
    
    # workflow
    - rule my_rule:
    + rule fastqc:
        input:
    +       R1 = "../../data/NA24631_1.fastq.gz",
    +       R2 = "../../data/NA24631_2.fastq.gz"
        output:
    +       html = ["../results/fastqc/NA24631_1_fastqc.html", "../results/fastqc/NA24631_2_fastqc.html"],
    +       zip = ["../results/fastqc/NA24631_1_fastqc.zip", "../results/fastqc/NA24631_2_fastqc.zip"]
    +   threads: 2
        shell:
    +       "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads}"
    ```

Current snakefile:

!!! terminal ""

    ```txt
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
        shell:
            "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads}"
    ```


<br>

When you have multiple input and output files:

- You can "name" you inputs/outputs, they can be called separately in the shell command
- Remember to use commas between multiple inputs/outputs, it's a common source of error!

Let's test the workflow! First we need to be in the `workflow` directory, where the Snakefile is

!!! terminal "code"

    ```bash
    cd demo_workflow/workflow/
    ```

## 3.05 Dryrun

Then let's carry out a dryrun of the workflow, where no actual analysis is undertaken (fastqc is *not* run) but the overall Snakemake structure is run/validated. This is a good way to check for errors in your Snakemake workflow before actually running your workflow.

!!! terminal "code"

    ```bash
    snakemake --dryrun
    ```

    ??? success "output"

        ```bash
        Building DAG of jobs...
        Job stats:
        job       count    min threads    max threads
        ------  -------  -------------  -------------
        all           1              1              1
        fastqc        1              1              1
        total         2              1              1
        
        
        [Wed May 11 12:09:56 2022]
        rule fastqc:
            input: ../../data/NA24631_1.fastq.gz, ../../data/NA24631_2.fastq.gz
            output: ../results/fastqc/NA24631_1_fastqc.html, ../results/fastqc/NA24631_2_fastqc.html, ../results/fastqc/NA24631_1_fastqc.zip, ../results/fastqc/NA24631_2_fastqc.zip
            jobid: 1
            resources: tmpdir=/dev/shm/jobs/26763281
        
        
        [Wed May 11 12:09:56 2022]
        localrule all:
            input: ../results/fastqc/NA24631_1_fastqc.html, ../results/fastqc/NA24631_2_fastqc.html, ../results/fastqc/NA24631_1_fastqc.zip, ../results/fastqc/NA24631_2_fastqc.zip
            jobid: 0
            resources: tmpdir=/dev/shm/jobs/26763281
        
        Job stats:
        job       count    min threads    max threads
        ------  -------  -------------  -------------
        all           1              1              1
        fastqc        1              1              1
        total         2              1              1
        
        This was a dry-run (flag -n). The order of jobs does not reflect the order of execution.
        ```



<br>

The last table in the output confirms that the workflow will run one sample (`count 1`) through fastqc (`job fastqc`)

## 3.06 Create a DAG

We can also visualise our workflow by creating a [directed acyclic graph](https://en.wikipedia.org/wiki/Directed_acyclic_graph) (DAG). We can tell snakemake to create a DAG with the `--dag` flag, then pipe this output to the [dot software](https://graphviz.org/) and write the output to the file, `dag_1.png`

!!! terminal "code"

    ```bash
    snakemake --dag | dot -Tpng > dag_1.png
    ```

    ??? success "DAG:"

        <center>![DAG_1](./images/dag_1.png)</center>

<br>

Our diagram has a node for each job which are connected by edges representing dependencies

*Note. this diagram can be output to several other image formats such as svg or pdf*

## 3.07 Fullrun

Let's do a full run of our workflow (by removing the `--dryrun` flag). We will also now need to specify the maximum number of cores to use at one time with the `--cores` flag before snakemake will run

!!! terminal "code"

    ```bash
    snakemake --cores 2
    ```

    ??? success "output"

        ```bash
        Building DAG of jobs...
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
        
        [Wed May 11 12:10:44 2022]
        rule fastqc:
            input: ../../data/NA24631_1.fastq.gz, ../../data/NA24631_2.fastq.gz
            output: ../results/fastqc/NA24631_1_fastqc.html, ../results/fastqc/NA24631_2_fastqc.html, ../results/fastqc/NA24631_1_fastqc.zip, ../results/fastqc/NA24631_2_fastqc.zip
            jobid: 1
            threads: 2
            resources: tmpdir=/dev/shm/jobs/26763281
        
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
        Approx 65% complete for NA24631_1.fastq.gz
        Started analysis of NA24631_2.fastq.gz
        Approx 70% complete for NA24631_1.fastq.gz
        Approx 5% complete for NA24631_2.fastq.gz
        Approx 75% complete for NA24631_1.fastq.gz
        Approx 10% complete for NA24631_2.fastq.gz
        Approx 80% complete for NA24631_1.fastq.gz
        Approx 15% complete for NA24631_2.fastq.gz
        Approx 85% complete for NA24631_1.fastq.gz
        Approx 20% complete for NA24631_2.fastq.gz
        Approx 90% complete for NA24631_1.fastq.gz
        Approx 25% complete for NA24631_2.fastq.gz
        Approx 95% complete for NA24631_1.fastq.gz
        Approx 30% complete for NA24631_2.fastq.gz
        Analysis complete for NA24631_1.fastq.gz
        Approx 35% complete for NA24631_2.fastq.gz
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
        [Wed May 11 12:10:48 2022]
        Finished job 1.
        1 of 2 steps (50%) done
        Select jobs to execute...
        
        [Wed May 11 12:10:48 2022]
        localrule all:
            input: ../results/fastqc/NA24631_1_fastqc.html, ../results/fastqc/NA24631_2_fastqc.html, ../results/fastqc/NA24631_1_fastqc.zip, ../results/fastqc/NA24631_2_fastqc.zip
            jobid: 0
            resources: tmpdir=/dev/shm/jobs/26763281
        
        [Wed May 11 12:10:48 2022]
        Finished job 0.
        2 of 2 steps (100%) done
        Complete log: .snakemake/log/2022-05-11T121044.745212.snakemake.log
        ```


<br>

It worked! Now in our results directory we have our output files from fastqc. Let's have a look:

!!! terminal "code"
    ```bash
    ls -lh ../results/fastqc/
    ```

    ??? success "output"
        ```bash
        total 2.5M
        -rw-rw----+ 1 lkemp nesi99991 718K May 11 12:10 NA24631_1_fastqc.html
        -rw-rw----+ 1 lkemp nesi99991 475K May 11 12:10 NA24631_1_fastqc.zip
        -rw-rw----+ 1 lkemp nesi99991 726K May 11 12:10 NA24631_2_fastqc.html
        -rw-rw----+ 1 lkemp nesi99991 479K May 11 12:10 NA24631_2_fastqc.zip
        ```



{% include exercise.html title="e3dot10" content=e3dot10%}
<br>

## 3.08 Lazy evaluation

What happens if we try a dryrun or full run now?

!!! terminal "code"

    ```bash
    snakemake --dryrun --cores 2
    ```

    !!! success "output"
    
        ```bash
        Building DAG of jobs...
        Nothing to be done (all requested files are present and up to date).
        ```


<br>

!!! terminal "code"

    ```bash
    snakemake --cores 2
    ```
    
    ??? success "output"
    
        ```bash
        Building DAG of jobs...
        Nothing to be done (all requested files are present and up to date).
        Complete log: .snakemake/log/2022-05-11T121300.251492.snakemake.log
        ```

<br>

Nothing happens, all the target files in `rule all` have already been created so Snakemake does nothing

Also, what happens if we create another directed acyclic graph (DAG) after the workflow has been run?

```bash
snakemake --dag | dot -Tpng > dag_2.png
```

My DAG:

{% capture e3dot13 %}

![DAG_2](./images/dag_2.png)



{% include exercise.html title="e3dot13" content=e3dot13%}
<br>

Notice our workflow 'job nodes' are now dashed lines, this indicates that their output is up to date and therefore the rule doesn't need to be run. We already have our target files!

This can be quite informative if your workflow errors out at a rule. You can visually check which rules successfully ran and which didn't.

## 3.09 Run using environment modules

fastqc worked because we loaded it in our current shell session. Let's specify the environment module for fastqc so the user of the workflow doesn't need to load it manually.

Update our rule to use it using the `envmodules:` directive

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
+   envmodules:
+       "FastQC/0.11.9"
    shell:
        "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads}"
```

Current snakefile:

{% capture e3dot14 %}

```txt
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
    envmodules:
        "FastQC/0.11.9"
    shell:
        "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads}"
```



{% include exercise.html title="e3dot14" content=e3dot14%}
<br>

Run again, now telling Snakemake to use [environment modules](https://nesi.github.io/hpc-intro/14-modules/index.html) to automatically load our software by using the `--use-envmodules` flag

```diff
# first remove output of last run
rm -r ../results/*

# Run dryrun again
- snakemake --dryrun --cores 2
+ snakemake --dryrun --cores 2 --use-envmodules
```

My output:

{% capture e3dot15 %}

```bash
Building DAG of jobs...
Job stats:
job       count    min threads    max threads
------  -------  -------------  -------------
all           1              1              1
fastqc        1              2              2
total         2              1              2


[Wed May 11 12:13:52 2022]
rule fastqc:
    input: ../../data/NA24631_1.fastq.gz, ../../data/NA24631_2.fastq.gz
    output: ../results/fastqc/NA24631_1_fastqc.html, ../results/fastqc/NA24631_2_fastqc.html, ../results/fastqc/NA24631_1_fastqc.zip, ../results/fastqc/NA24631_2_fastqc.zip
    jobid: 1
    threads: 2
    resources: tmpdir=/dev/shm/jobs/26763281


[Wed May 11 12:13:52 2022]
localrule all:
    input: ../results/fastqc/NA24631_1_fastqc.html, ../results/fastqc/NA24631_2_fastqc.html, ../results/fastqc/NA24631_1_fastqc.zip, ../results/fastqc/NA24631_2_fastqc.zip
    jobid: 0
    resources: tmpdir=/dev/shm/jobs/26763281

Job stats:
job       count    min threads    max threads
------  -------  -------------  -------------
all           1              1              1
fastqc        1              2              2
total         2              1              2

This was a dry-run (flag -n). The order of jobs does not reflect the order of execution.
```



{% include exercise.html title="e3dot15" content=e3dot15%}
<br>

Let's do a full run

```diff
- snakemake --cores 2
+ snakemake --cores 2 --use-envmodules
```

My output:

{% capture e3dot16 %}

```bash
Building DAG of jobs...
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

[Wed May 11 12:14:22 2022]
rule fastqc:
    input: ../../data/NA24631_1.fastq.gz, ../../data/NA24631_2.fastq.gz
    output: ../results/fastqc/NA24631_1_fastqc.html, ../results/fastqc/NA24631_2_fastqc.html, ../results/fastqc/NA24631_1_fastqc.zip, ../results/fastqc/NA24631_2_fastqc.zip
    jobid: 1
    threads: 2
    resources: tmpdir=/dev/shm/jobs/26763281

Activating environment modules: FastQC/0.11.9

The following modules were not unloaded:
   (Use "module --force purge" to unload all):

  1) XALT/minimal   2) slurm   3) NeSI
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
Approx 65% complete for NA24631_1.fastq.gz
Approx 70% complete for NA24631_1.fastq.gz
Approx 75% complete for NA24631_1.fastq.gz
Started analysis of NA24631_2.fastq.gz
Approx 5% complete for NA24631_2.fastq.gz
Approx 80% complete for NA24631_1.fastq.gz
Approx 10% complete for NA24631_2.fastq.gz
Approx 85% complete for NA24631_1.fastq.gz
Approx 15% complete for NA24631_2.fastq.gz
Approx 90% complete for NA24631_1.fastq.gz
Approx 20% complete for NA24631_2.fastq.gz
Approx 95% complete for NA24631_1.fastq.gz
Approx 25% complete for NA24631_2.fastq.gz
Analysis complete for NA24631_1.fastq.gz
Approx 30% complete for NA24631_2.fastq.gz
Approx 35% complete for NA24631_2.fastq.gz
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
[Wed May 11 12:14:26 2022]
Finished job 1.
1 of 2 steps (50%) done
Select jobs to execute...

[Wed May 11 12:14:26 2022]
localrule all:
    input: ../results/fastqc/NA24631_1_fastqc.html, ../results/fastqc/NA24631_2_fastqc.html, ../results/fastqc/NA24631_1_fastqc.zip, ../results/fastqc/NA24631_2_fastqc.zip
    jobid: 0
    resources: tmpdir=/dev/shm/jobs/26763281

[Wed May 11 12:14:26 2022]
Finished job 0.
2 of 2 steps (100%) done
Complete log: .snakemake/log/2022-05-11T121422.744466.snakemake.log
```



{% include exercise.html title="e3dot16" content=e3dot16%}
<br>

Notice it now says that "Activating environment modules: FastQC/0.11.9". Now the software our workflow uses will be automatically loaded!

## 3.10 Capture our logs

So far our logs (for fastqc) have been simply printed to our screen. As you can imagine, if you had a large automated workflow (that you might not be sitting at the computer watching run) you'll want to capture all that information. Therefore, any information the software spits out (including error messages!) will be kept and can be looked at once you return to your machine from your coffee break.

We can get the logs for each rule to be written to a log file via the `log:` directive:

- It's a good idea to organise the logs by:
  - Putting the logs in a directory labelled after the rule/software that was run
  - Labelling the log files with the sample name the software was run on

- Also make sure you tell the software (fastqc) to write the standard output and standard error to this log file we defined in the `log:` directive in the shell script (eg. `&> {log}`)

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
+   log:
+       "logs/fastqc/NA24631.log"
    threads: 2
    envmodules:
        "FastQC/0.11.9"
    shell:
-       "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads}"
+       "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} &> {log}"
```

Current snakefile:

{% capture e3dot17 %}

```txt
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
    log:
        "logs/fastqc/NA24631.log"
    threads: 2
    envmodules:
        "FastQC/0.11.9"
    shell:
        "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} &> {log}"
```



{% include exercise.html title="e3dot17" content=e3dot17%}
<br>

---

A tangent about [standard streams](https://en.wikipedia.org/wiki/Standard_streams)

- These are standard streams in which information is returned by a computer process - in our case the logs that we see returned to us on our screen when we run fastqc
- There are two main streams:
  - standard output (the log messages)
  - standard error (the error messages)

Different ways to write log files:

|  Syntax  | standard output in terminal | standard error in terminal | standard output in file | standard error in file |
|----------|-----------------------------|----------------------------|-------------------------|------------------------|
|   `>`    |  NO                         | YES                        | YES                     | NO                     |
|   `2>`   |  YES                        | NO                         | NO                      | YES                    |
|   `&>`   |  NO                         | NO                         | YES                     | YES                    |

(Table adapted from [here](https://askubuntu.com/questions/420981/how-do-i-save-terminal-output-to-a-file))

---

Run again

```bash
# remove output of last run
rm -r ../results/*

# run dryrun/run again
snakemake --dryrun --cores 2 --use-envmodules
snakemake --cores 2 --use-envmodules
```

My output:

{% capture e3dot18 %}

```bash
Building DAG of jobs...
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

[Wed May 11 12:15:16 2022]
rule fastqc:
    input: ../../data/NA24631_1.fastq.gz, ../../data/NA24631_2.fastq.gz
    output: ../results/fastqc/NA24631_1_fastqc.html, ../results/fastqc/NA24631_2_fastqc.html, ../results/fastqc/NA24631_1_fastqc.zip, ../results/fastqc/NA24631_2_fastqc.zip
    log: logs/fastqc/NA24631.log
    jobid: 1
    threads: 2
    resources: tmpdir=/dev/shm/jobs/26763281

Activating environment modules: FastQC/0.11.9

The following modules were not unloaded:
   (Use "module --force purge" to unload all):

  1) XALT/minimal   2) slurm   3) NeSI
[Wed May 11 12:15:20 2022]
Finished job 1.
1 of 2 steps (50%) done
Select jobs to execute...

[Wed May 11 12:15:20 2022]
localrule all:
    input: ../results/fastqc/NA24631_1_fastqc.html, ../results/fastqc/NA24631_2_fastqc.html, ../results/fastqc/NA24631_1_fastqc.zip, ../results/fastqc/NA24631_2_fastqc.zip
    jobid: 0
    resources: tmpdir=/dev/shm/jobs/26763281

[Wed May 11 12:15:20 2022]
Finished job 0.
2 of 2 steps (100%) done
Complete log: .snakemake/log/2022-05-11T121516.368334.snakemake.log
```



{% include exercise.html title="e3dot18" content=e3dot18%}
<br>

We now have a log file, lets have a look at the first 10 lines of our log with:

```bash
head ./logs/fastqc/NA24631.log
```

My output:

{% capture e3dot19 %}

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



{% include exercise.html title="e3dot19" content=e3dot19%}
<br>

<p align="center"><b>We have logs. Tidy logs.</b><br></p>

![logs](https://miro.medium.com/max/2560/1*ohWUB5snJRaMe-vJ8HaoiA.png)

> **Exercise:**
>
> Try creating an error in the shell command (for example remove the `-o` flag) and use the three different syntaxes for writing to your log file. What is and isn't printed to your screen and to your log file?

## 3.11 Scale up to analyse all of our samples

We are currently only analysing one of our three samples

Let's scale up to run all of our samples by using [wildcards](https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html#wildcards), this way we can grab all the samples/files in the `data` directory and analyse them

- Set a global wildcard that defines the samples to be analysed
- Generalise where this rule uses an individual sample (`NA24631`) to use this wildcard `{sample}`
- Use the [expand function](https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html#the-expand-function) (`expand()`) function to tell snakemake that `{sample}` is what we defined in our global wildcard `SAMPLES,`
- Snakemake can figure out what `{sample}` is in our rule since it's defined in the targets in `rule all:`

```diff
# define samples from data directory using wildcards
+ SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")

# target OUTPUT files for the whole workflow
rule all:
    input:
-       "../results/fastqc/NA24631_1_fastqc.html",
-       "../results/fastqc/NA24631_2_fastqc.html",
-       "../results/fastqc/NA24631_1_fastqc.zip",
-       "../results/fastqc/NA24631_2_fastqc.zip"
+       expand("../results/fastqc/{sample}_1_fastqc.html", sample = SAMPLES),
+       expand("../results/fastqc/{sample}_2_fastqc.html", sample = SAMPLES),
+       expand("../results/fastqc/{sample}_1_fastqc.zip", sample = SAMPLES),
+       expand("../results/fastqc/{sample}_2_fastqc.zip", sample = SAMPLES)

# workflow
rule fastqc:
    input:
-       R1 = "../../data/NA24631_1.fastq.gz",
-       R2 = "../../data/NA24631_2.fastq.gz"
+       R1 = "../../data/{sample}_1.fastq.gz",
+       R2 = "../../data/{sample}_2.fastq.gz"
    output:
-       html = ["../results/fastqc/NA24631_1_fastqc.html", "../results/fastqc/NA24631_2_fastqc.html"],
-       zip = ["../results/fastqc/NA24631_1_fastqc.zip", "../results/fastqc/NA24631_2_fastqc.zip"]
+       html = ["../results/fastqc/{sample}_1_fastqc.html", "../results/fastqc/{sample}_2_fastqc.html"],
+       zip = ["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"]
    log:
-       "logs/fastqc/NA24631.log"
+       "logs/fastqc/{sample}.log"
    threads: 2
    envmodules:
        "FastQC/0.11.9"
    shell:
        "fastqc {input.R1} {input.R2} -o ../results/fastqc/ -t {threads} &> {log}"
```

Current snakefile:

{% capture e3dot20 %}

```txt
# define samples from data directory using wildcards
SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")

# target OUTPUT files for the whole workflow
rule all:
    input:
        expand("../results/fastqc/{sample}_1_fastqc.html", sample = SAMPLES),
        expand("../results/fastqc/{sample}_2_fastqc.html", sample = SAMPLES),
        expand("../results/fastqc/{sample}_1_fastqc.zip", sample = SAMPLES),
        expand("../results/fastqc/{sample}_2_fastqc.zip", sample = SAMPLES)

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
```



{% include exercise.html title="e3dot20" content=e3dot20%}
<br>

Visualise workflow

```bash
snakemake --dag | dot -Tpng > dag_3.png
```

Now we have three samples running though our workflow, one of which has already been run in our last run (NA24631) indicated by the dashed lines

My DAG:

{% capture e3dot21 %}

![DAG_3](./images/dag_3.png)



{% include exercise.html title="e3dot21" content=e3dot21%}
<br>

Run workflow again

```bash
# remove output of last run
rm -r ../results/*

# run dryrun again
snakemake --dryrun --cores 2 --use-envmodules
```

See how it now runs over all three of our samples in the output of the dryrun

My output:

{% capture e3dot22 %}

```bash
Building DAG of jobs...
Job stats:
job       count    min threads    max threads
------  -------  -------------  -------------
all           1              1              1
fastqc        3              2              2
total         4              1              2


[Wed May 11 12:16:46 2022]
rule fastqc:
    input: ../../data/NA24695_1.fastq.gz, ../../data/NA24695_2.fastq.gz
    output: ../results/fastqc/NA24695_1_fastqc.html, ../results/fastqc/NA24695_2_fastqc.html, ../results/fastqc/NA24695_1_fastqc.zip, ../results/fastqc/NA24695_2_fastqc.zip
    log: logs/fastqc/NA24695.log
    jobid: 2
    wildcards: sample=NA24695
    threads: 2
    resources: tmpdir=/dev/shm/jobs/26763281


[Wed May 11 12:16:46 2022]
rule fastqc:
    input: ../../data/NA24631_1.fastq.gz, ../../data/NA24631_2.fastq.gz
    output: ../results/fastqc/NA24631_1_fastqc.html, ../results/fastqc/NA24631_2_fastqc.html, ../results/fastqc/NA24631_1_fastqc.zip, ../results/fastqc/NA24631_2_fastqc.zip
    log: logs/fastqc/NA24631.log
    jobid: 1
    wildcards: sample=NA24631
    threads: 2
    resources: tmpdir=/dev/shm/jobs/26763281


[Wed May 11 12:16:46 2022]
rule fastqc:
    input: ../../data/NA24694_1.fastq.gz, ../../data/NA24694_2.fastq.gz
    output: ../results/fastqc/NA24694_1_fastqc.html, ../results/fastqc/NA24694_2_fastqc.html, ../results/fastqc/NA24694_1_fastqc.zip, ../results/fastqc/NA24694_2_fastqc.zip
    log: logs/fastqc/NA24694.log
    jobid: 3
    wildcards: sample=NA24694
    threads: 2
    resources: tmpdir=/dev/shm/jobs/26763281


[Wed May 11 12:16:46 2022]
localrule all:
    input: ../results/fastqc/NA24631_1_fastqc.html, ../results/fastqc/NA24695_1_fastqc.html, ../results/fastqc/NA24694_1_fastqc.html, ../results/fastqc/NA24631_2_fastqc.html, ../results/fastqc/NA24695_2_fastqc.html, ../results/fastqc/NA24694_2_fastqc.html, ../results/fastqc/NA24631_1_fastqc.zip, ../results/fastqc/NA24695_1_fastqc.zip, ../results/fastqc/NA24694_1_fastqc.zip, ../results/fastqc/NA24631_2_fastqc.zip, ../results/fastqc/NA24695_2_fastqc.zip, ../results/fastqc/NA24694_2_fastqc.zip
    jobid: 0
    resources: tmpdir=/dev/shm/jobs/26763281

Job stats:
job       count    min threads    max threads
------  -------  -------------  -------------
all           1              1              1
fastqc        3              2              2
total         4              1              2

This was a dry-run (flag -n). The order of jobs does not reflect the order of execution.
```



{% include exercise.html title="e3dot22" content=e3dot22%}
<br>

```bash
# full run again
snakemake --cores 2 --use-envmodules
```

All three samples were run through our workflow! And we have a log file for each sample for the fastqc rule

```bash
ls -lh ./logs/fastqc
```

My output:

{% capture e3dot23 %}

```bash
total 1.5K
-rw-rw----+ 1 lkemp nesi99991 1.8K May 11 12:17 NA24631.log
-rw-rw----+ 1 lkemp nesi99991 1.8K May 11 12:17 NA24694.log
-rw-rw----+ 1 lkemp nesi99991 1.8K May 11 12:17 NA24695.log
```



{% include exercise.html title="e3dot23" content=e3dot23%}
<br>

## 3.12 Add more rules

- Connect the outputs of fastqc to the inputs of multiqc
- Add a new final target for `rule all:`

```diff
# define samples from data directory using wildcards
SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")

# target OUTPUT files for the whole workflow
rule all:
    input:
        expand("../results/fastqc/{sample}_1_fastqc.html", sample = SAMPLES),
        expand("../results/fastqc/{sample}_2_fastqc.html", sample = SAMPLES),
        expand("../results/fastqc/{sample}_1_fastqc.zip", sample = SAMPLES),
-       expand("../results/fastqc/{sample}_2_fastqc.zip", sample = SAMPLES)
+       expand("../results/fastqc/{sample}_2_fastqc.zip", sample = SAMPLES),
+       "../results/multiqc_report.html"

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
  
+ rule multiqc:
+   input:
+       expand(["../results/fastqc/{sample}_1_fastqc.zip", "../results/fastqc/{sample}_2_fastqc.zip"], sample = SAMPLES)
+   output:
+       "../results/multiqc_report.html"
+   log:
+       "logs/multiqc/multiqc.log"
+   envmodules:
+       "MultiQC/1.9-gimkl-2020a-Python-3.8.2"
+   shell:
+       "multiqc {input} -o ../results/ &> {log}"
```

Current snakefile:

{% capture e3dot24 %}

```txt
# define samples from data directory using wildcards
SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")

# target OUTPUT files for the whole workflow
rule all:
    input:
        expand("../results/fastqc/{sample}_1_fastqc.html", sample = SAMPLES),
        expand("../results/fastqc/{sample}_2_fastqc.html", sample = SAMPLES),
        expand("../results/fastqc/{sample}_1_fastqc.zip", sample = SAMPLES),
        expand("../results/fastqc/{sample}_2_fastqc.zip", sample = SAMPLES),
        "../results/multiqc_report.html"

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
```



{% include exercise.html title="e3dot24" content=e3dot24%}
<br>

Run workflow again

```bash
# remove output of last run
rm -r ../results/*

# run dryrun/run again
snakemake --dryrun --cores 2 --use-envmodules
snakemake --cores 2 --use-envmodules
```

Visualise workflow

```bash
snakemake --dag | dot -Tpng > dag_4.png
```

Now we have two rules in our workflow (fastqc and multiqc), we can also see that multiqc isn't run for each sample (since it merges the output of fastqc for all samples)

My DAG:

{% capture e3dot25 %}

![DAG_4](./images/dag_4.png)



{% include exercise.html title="e3dot25" content=e3dot25%}
<br>

## 3.13 More about Snakemake's lazy behaviour

What happens if we only have the final target file (`../results/multiqc_report.html`) in `rule all:`

```diff
# define samples from data directory using wildcards
SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")

# target OUTPUT files for the whole workflow
rule all:
    input:
-       expand("../results/fastqc/{sample}_1_fastqc.html", sample = SAMPLES),
-       expand("../results/fastqc/{sample}_2_fastqc.html", sample = SAMPLES),
-       expand("../results/fastqc/{sample}_1_fastqc.zip", sample = SAMPLES),
-       expand("../results/fastqc/{sample}_2_fastqc.zip", sample = SAMPLES),
        "../results/multiqc_report.html"

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
```

Current snakefile:

{% capture e3dot26 %}

```txt
# define samples from data directory using wildcards
SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")

# target OUTPUT files for the whole workflow
rule all:
    input:
        "../results/multiqc_report.html"

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
```



{% include exercise.html title="e3dot26" content=e3dot26%}
<br>

Run workflow again

```bash
# remove output of last run
rm -r ../results/*

# run dryrun again
snakemake --dryrun --cores 2 --use-envmodules
```

It still works because it is the last file in the workflow sequence, Snakemake will do all the steps necessary to get to this target file (therefore it runs both fastqc and multiqc)

Visualise workflow

```bash
snakemake --dag | dot -Tpng > dag_5.png
```

Although the workflow ran the same, the DAG actually changed slightly, now there is only one file target and only the output of multiqc goes to `rule all`

My DAG:

{% capture e3dot27 %}

![DAG_5](./images/dag_5.png)



{% include exercise.html title="e3dot27" content=e3dot27%}
<br>

<p align="center"><b>Beware: Snakemake will also NOT run rules that it doesn't need to run in order to get the target files defined in rule: all</b><br></p>

For example if only our fastqc outputs are defined as the target in `rule: all`

```diff
# define samples from data directory using wildcards
SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")

# target OUTPUT files for the whole workflow
rule all:
    input:
+       expand("../results/fastqc/{sample}_1_fastqc.html", sample = SAMPLES),
+       expand("../results/fastqc/{sample}_2_fastqc.html", sample = SAMPLES),
+       expand("../results/fastqc/{sample}_1_fastqc.zip", sample = SAMPLES),
+       expand("../results/fastqc/{sample}_2_fastqc.zip", sample = SAMPLES)
-       "../results/multiqc_report.html"

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
```

Current snakefile:

{% capture e3dot28 %}

```txt
# define samples from data directory using wildcards
SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")

# target OUTPUT files for the whole workflow
rule all:
    input:
        expand("../results/fastqc/{sample}_1_fastqc.html", sample = SAMPLES),
        expand("../results/fastqc/{sample}_2_fastqc.html", sample = SAMPLES),
        expand("../results/fastqc/{sample}_1_fastqc.zip", sample = SAMPLES),
        expand("../results/fastqc/{sample}_2_fastqc.zip", sample = SAMPLES)

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
```



{% include exercise.html title="e3dot28" content=e3dot28%}
<br>

Run again

```bash
# run dryrun again
snakemake --dryrun --cores 2 --use-envmodules
```

My partial output:

{% capture e3dot29 %}

```bash
Job stats:
job       count    min threads    max threads
------  -------  -------------  -------------
all           1              1              1
fastqc        3              2              2
total         4              1              2
```



{% include exercise.html title="e3dot29" content=e3dot29%}
<br>

Our multiqc rule won't be run/evaluated

Visualise workflow

```bash
snakemake --dag | dot -Tpng > dag_6.png
```

Now we are back to only running fastqc in our workflow, despite having our second rule (multiqc) in our workflow

My DAG:

{% capture e3dot30 %}

![DAG_6](./images/dag_6.png)



{% include exercise.html title="e3dot30" content=e3dot30%}
<br>

<p align="center"><b>Snakemake is lazy.</b><br></p>

![Snakemake is lazy.](https://64.media.tumblr.com/0492923adeb79cb841e29968135305d5/tumblr_nzdagaL6EH1uavdlbo7_1280.png)

## 3.14 Add even more rules

Let's add the rest of the rules. We want to get to:

![rulegraph_1](./images/rulegraph_1.png)

We currently have fastqc and multiqc, so we still need to add trim_galore

```diff
# define samples from data directory using wildcards
SAMPLES, = glob_wildcards("../../data/{sample}_1.fastq.gz")

# target OUTPUT files for the whole workflow
rule all:
    input:
-       expand("../results/fastqc/{sample}_1_fastqc.html", sample = SAMPLES),
-       expand("../results/fastqc/{sample}_2_fastqc.html", sample = SAMPLES),
-       expand("../results/fastqc/{sample}_1_fastqc.zip", sample = SAMPLES),
-       expand("../results/fastqc/{sample}_2_fastqc.zip", sample = SAMPLES)
+       "../results/multiqc_report.html",
+       expand(["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"], sample = SAMPLES)

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

+ rule trim_galore:
+   input:
+       ["../../data/{sample}_1.fastq.gz", "../../data/{sample}_2.fastq.gz"]
+   output:
+       ["../results/trimmed/{sample}_1_val_1.fq.gz", "../results/trimmed/{sample}_2_val_2.fq.gz"]
+   log:
+       "logs/trim_galore/{sample}.log"
+   envmodules:
+       "TrimGalore/0.6.7-gimkl-2020a-Python-3.8.2-Perl-5.30.1"
+   threads: 2
+   shell:
+       "trim_galore {input} -o ../results/trimmed/ --paired --cores {threads} &> {log}"
```

Current snakefile:

{% capture e3dot31 %}

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
    shell:
        "trim_galore {input} -o ../results/trimmed/ --paired --cores {threads} &> {log}"
```



{% include exercise.html title="e3dot31" content=e3dot31%}
<br>

Visualise workflow

```bash
snakemake --dag | dot -Tpng > dag_7.png
```

Fantastic, we are starting to build a workflow!

My DAG:

{% capture e3dot32 %}

![DAG_7](./images/dag_7.png)



{% include exercise.html title="e3dot32" content=e3dot32%}
<br>

However, when analysing many samples, our DAG can become messy and complicated. Instead, we can create a rulegraph that will let us visualise our workflow without showing every single sample that will run through it

```bash
snakemake --rulegraph | dot -Tpng > rulegraph_1.png
```

My rulegraph:

{% capture e3dot33 %}

![rulegraph_1](./images/rulegraph_1.png)



{% include exercise.html title="e3dot33" content=e3dot33%}
<br>

An aside: another option that will show all your input and output files at each step:

```bash
snakemake --filegraph | dot -Tpng > filegraph.png
```

My filegraph:

{% capture e3dot34%}

![filegraph](./images/filegraph.png)



{% include exercise.html title="e3dot34" content=e3dot34%}
<br>

Run the rest of the workflow

```bash
# run dryrun/run again
snakemake --dryrun --cores 2 --use-envmodules
snakemake --cores 2 --use-envmodules
```

Notice it will run only one rule/sample/file at a time...why is that?

## 3.15 Throw it more cores

Run again allowing Snakemake to use more cores overall `--cores 4` rather than `--cores 2`

```bash
# remove output of last run
rm -r ../results/*

# run dryrun/run again
snakemake --dryrun --cores 4 --use-envmodules
snakemake --cores 4 --use-envmodules
```

Notice the whole workflow ran much faster and several samples/files/rules were running at one time. This is because we set each rule to run with 2 threads. Initially we specified that the *maximum* number of cores to be used by the workflow was 2 with the `--cores 2` flag, meaning only one rule and sample can be run at one time. When we increased the *maximum* number of cores to be used by the workflow to 4 with `--cores 4`, up to 2 samples could be run through at one time.

## 3.16 Throw it even more cores

With a high performance cluster such as [NeSi](https://www.nesi.org.nz/), you can start to REALLY scale up, particularly when you have many samples to analyse or files to process. This is because the number of cores available in a HPC is HUGE compared to a laptop or even an high end server.

<p align="center"><b>Boom! Scalability here we come!</b><br></p>

<center>![image](./nesi_images/mahuika_maui_real.png){width="800"}


To run the workflow on the cluster, we need to ensure that each step is run as a dedicated job in the queuing system of the HPC. On NeSI, the queuing system is managed by [Slurm](https://slurm.schedmd.com/documentation.html).

Use the `--cluster` option to specify the job submission command, using `sbatch` on NeSI.
This command defines resources used for each job (maximum time, memory, number of cores...).
In addition, you need to specify a maximum number of concurrent jobs using `--jobs`.

```bash
# remove output of last run
rm -r ../results/*

# run again on the cluster
snakemake --cluster "sbatch --time 00:10:00 --mem 512MB --cpus-per-task 8 --account nesi99991" --jobs 10 --use-envmodules
```

My output:

{% capture e3dot35 %}

```bash
Building DAG of jobs...
Using shell: /usr/bin/bash
Provided cores: 4
Rules claiming more threads will be scaled down.
Job stats:
job            count    min threads    max threads
-----------  -------  -------------  -------------
all                1              1              1
fastqc             3              2              2
multiqc            1              1              1
trim_galore        3              2              2
total              8              1              2

Select jobs to execute...

[Wed May 11 12:26:39 2022]
rule fastqc:
    input: ../../data/NA24694_1.fastq.gz, ../../data/NA24694_2.fastq.gz
    output: ../results/fastqc/NA24694_1_fastqc.html, ../results/fastqc/NA24694_2_fastqc.html, ../results/fastqc/NA24694_1_fastqc.zip, ../results/fastqc/NA24694_2_fastqc.zip
    log: logs/fastqc/NA24694.log
    jobid: 4
    wildcards: sample=NA24694
    threads: 2
    resources: tmpdir=/dev/shm/jobs/26763281

Activating environment modules: FastQC/0.11.9

[Wed May 11 12:26:39 2022]
rule trim_galore:
    input: ../../data/NA24694_1.fastq.gz, ../../data/NA24694_2.fastq.gz
    output: ../results/trimmed/NA24694_1_val_1.fq.gz, ../results/trimmed/NA24694_2_val_2.fq.gz
    log: logs/trim_galore/NA24694.log
    jobid: 7
    wildcards: sample=NA24694
    threads: 2
    resources: tmpdir=/dev/shm/jobs/26763281

Activating environment modules: TrimGalore/0.6.7-gimkl-2020a-Python-3.8.2-Perl-5.30.1

The following modules were not unloaded:
   (Use "module --force purge" to unload all):

  1) XALT/minimal   2) slurm   3) NeSI

The following modules were not unloaded:
   (Use "module --force purge" to unload all):

  1) XALT/minimal   2) slurm   3) NeSI
[Wed May 11 12:26:44 2022]
Finished job 4.
1 of 8 steps (12%) done
Select jobs to execute...

[Wed May 11 12:26:44 2022]
rule fastqc:
    input: ../../data/NA24631_1.fastq.gz, ../../data/NA24631_2.fastq.gz
    output: ../results/fastqc/NA24631_1_fastqc.html, ../results/fastqc/NA24631_2_fastqc.html, ../results/fastqc/NA24631_1_fastqc.zip, ../results/fastqc/NA24631_2_fastqc.zip
    log: logs/fastqc/NA24631.log
    jobid: 2
    wildcards: sample=NA24631
    threads: 2
    resources: tmpdir=/dev/shm/jobs/26763281

Activating environment modules: FastQC/0.11.9

The following modules were not unloaded:
   (Use "module --force purge" to unload all):

  1) XALT/minimal   2) slurm   3) NeSI
[Wed May 11 12:26:47 2022]
Finished job 7.
2 of 8 steps (25%) done
Select jobs to execute...

[Wed May 11 12:26:47 2022]
rule trim_galore:
    input: ../../data/NA24631_1.fastq.gz, ../../data/NA24631_2.fastq.gz
    output: ../results/trimmed/NA24631_1_val_1.fq.gz, ../results/trimmed/NA24631_2_val_2.fq.gz
    log: logs/trim_galore/NA24631.log
    jobid: 5
    wildcards: sample=NA24631
    threads: 2
    resources: tmpdir=/dev/shm/jobs/26763281

Activating environment modules: TrimGalore/0.6.7-gimkl-2020a-Python-3.8.2-Perl-5.30.1

The following modules were not unloaded:
   (Use "module --force purge" to unload all):

  1) XALT/minimal   2) slurm   3) NeSI
[Wed May 11 12:26:50 2022]
Finished job 2.
3 of 8 steps (38%) done
Select jobs to execute...

[Wed May 11 12:26:50 2022]
rule fastqc:
    input: ../../data/NA24695_1.fastq.gz, ../../data/NA24695_2.fastq.gz
    output: ../results/fastqc/NA24695_1_fastqc.html, ../results/fastqc/NA24695_2_fastqc.html, ../results/fastqc/NA24695_1_fastqc.zip, ../results/fastqc/NA24695_2_fastqc.zip
    log: logs/fastqc/NA24695.log
    jobid: 3
    wildcards: sample=NA24695
    threads: 2
    resources: tmpdir=/dev/shm/jobs/26763281

Activating environment modules: FastQC/0.11.9

The following modules were not unloaded:
   (Use "module --force purge" to unload all):

  1) XALT/minimal   2) slurm   3) NeSI
[Wed May 11 12:26:54 2022]
Finished job 3.
4 of 8 steps (50%) done
Select jobs to execute...

[Wed May 11 12:26:54 2022]
rule trim_galore:
    input: ../../data/NA24695_1.fastq.gz, ../../data/NA24695_2.fastq.gz
    output: ../results/trimmed/NA24695_1_val_1.fq.gz, ../results/trimmed/NA24695_2_val_2.fq.gz
    log: logs/trim_galore/NA24695.log
    jobid: 6
    wildcards: sample=NA24695
    threads: 2
    resources: tmpdir=/dev/shm/jobs/26763281

Activating environment modules: TrimGalore/0.6.7-gimkl-2020a-Python-3.8.2-Perl-5.30.1

The following modules were not unloaded:
   (Use "module --force purge" to unload all):

  1) XALT/minimal   2) slurm   3) NeSI
[Wed May 11 12:26:56 2022]
Finished job 5.
5 of 8 steps (62%) done
Select jobs to execute...

[Wed May 11 12:26:56 2022]
rule multiqc:
    input: ../results/fastqc/NA24631_1_fastqc.zip, ../results/fastqc/NA24695_1_fastqc.zip, ../results/fastqc/NA24694_1_fastqc.zip, ../results/fastqc/NA24631_2_fastqc.zip, ../results/fastqc/NA24695_2_fastqc.zip, ../results/fastqc/NA24694_2_fastqc.zip
    output: ../results/multiqc_report.html
    log: logs/multiqc/multiqc.log
    jobid: 1
    resources: tmpdir=/dev/shm/jobs/26763281

Activating environment modules: MultiQC/1.9-gimkl-2020a-Python-3.8.2

The following modules were not unloaded:
   (Use "module --force purge" to unload all):

  1) XALT/minimal   2) slurm   3) NeSI
[Wed May 11 12:27:01 2022]
Finished job 6.
6 of 8 steps (75%) done
[Wed May 11 12:27:03 2022]
Finished job 1.
7 of 8 steps (88%) done
Select jobs to execute...

[Wed May 11 12:27:03 2022]
localrule all:
    input: ../results/multiqc_report.html, ../results/trimmed/NA24631_1_val_1.fq.gz, ../results/trimmed/NA24695_1_val_1.fq.gz, ../results/trimmed/NA24694_1_val_1.fq.gz, ../results/trimmed/NA24631_2_val_2.fq.gz, ../results/trimmed/NA24695_2_val_2.fq.gz, ../results/trimmed/NA24694_2_val_2.fq.gz
    jobid: 0
    resources: tmpdir=/dev/shm/jobs/26763281

[Wed May 11 12:27:03 2022]
Finished job 0.
8 of 8 steps (100%) done
Complete log: .snakemake/log/2022-05-11T122639.019945.snakemake.log
```



{% include exercise.html title="e3dot35" content=e3dot35%}
<br>

If you open another terminal on the HPC, you can use the `squeue` command to list of your jobs and their state (pending, running, etc.):

```bash
squeue --me
```

My output:

{% capture e3dot36 %}

```bash
JOBID         USER     ACCOUNT   NAME        CPUS MIN_MEM PARTITI START_TIME     TIME_LEFT STATE    NODELIST(REASON)    
26763281      lkemp    nesi99991 spawner-jupy   4      4G interac 2022-05-11T1     7:30:33 RUNNING  wbn003              
26763418      lkemp    nesi99991 snakejob.fas   8    512M large   2022-05-11T1        9:59 RUNNING  wbn096              
26763419      lkemp    nesi99991 snakejob.tri   8    512M large   2022-05-11T1        9:59 RUNNING  wbn096              
26763420      lkemp    nesi99991 snakejob.fas   8    512M large   2022-05-11T1        9:59 RUNNING  wbn110              
26763421      lkemp    nesi99991 snakejob.fas   8    512M large   2022-05-11T1        9:59 RUNNING  wbn069              
26763422      lkemp    nesi99991 snakejob.tri   8    512M large   2022-05-11T1        9:59 RUNNING  wbn070              
26763423      lkemp    nesi99991 snakejob.tri   8    512M large   2022-05-11T1        9:59 RUNNING  wbn090  
```

<br>

An additional trick is to use the `watch` command to repeatedly call any command in the terminal, giving you a lightweight monitoring tool ;-).
Here we will use it to see your jobs gets queued and executed in real time:

```bash
watch squeue --me
```

You can exit the view create by `watch` by pressing CTRL+C.


# Takeaways

---

- Once familiar with environment modules, the software are very straightforward to integrate in your snakemake workflow
- Run your commands directly on the command line before wrapping it up in a Snakemake rule
- First do a dryrun to check the Snakemake structure is set up correctly
- Work iteratively (get each rule working before moving onto the next)
- File paths are relative to the Snakefile
- Run your workflow from where your Snakefile is
- Visualise your workflow by creating a DAG (directed acyclic graph), a rulegraph or filegraph
- Use environment modules to load software in your workflow - this improves reproducibility
- Snakemake is lazy...
  - It will only do something if it hasn't already done it
  - It will pick up where it left off, rather than run the whole workflow again
  - It *won't* do any steps that aren't necessary to get to the target files defined in `rule: all`
- `input:` `output:` `log:` and `threads:` directives need to be called in the `shell` directive
- Capture your log files
- Organise your log files by naming them after the rule that was run and sample that was analysed
- You don't need to specify all the target files in `rule all:`, the final file in a given chain of tasks will suffice
- We can massively speed up our analyses by running our samples in parallel

---

# Summary commands

Create a directed acyclic graph (DAG) with:

```bash
snakemake --dag | dot -Tpng > dag.png
```

Create a rulegraph with:

```bash
snakemake --rulegraph | dot -Tpng > rulegraph.png
```

Create a filegraph with:

```bash
snakemake --filegraph | dot -Tpng > filegraph.png
```

Run a dryrun of your snakemake workflow with:

```bash
snakemake --dryrun
```

Run your snakemake workflow with:

```bash
snakemake --cores 2
```

Run a dryrun of your snakemake workflow (using environment modules to load your software) with:

```bash
snakemake --dryrun --cores 2 --use-envmodules
```

Run your snakemake workflow (using environment modules to load your software) with:

```bash
snakemake --cores 2 --use-envmodules
```

Run your snakemake workflow using multiple jobs on NeSI:

```bash
snakemake --cluster "sbatch --time 00:10:00 --mem=512MB --cpus-per-task 8" --jobs 10 --use-envmodules
```

Create a global wildcard to get process all your samples in a directory with:

```bash
SAMPLES, = glob_wildcards("../relative/path/to/samples/{sample}_1.fastq.gz")
```

Combine this with the expand function to tell Snakemake to look at your global wildcard to figure out what you refer to as `{sample}` in your workflow

```bash
expand("../results/{sample}_1.fastq.gz", sample = SAMPLES)
```

Increase the number of samples that can be analysed at one time in your workflow by increasing the maximum number of cores to be used at one time with the `--cores` command

```bash
snakemake --cores 4 --use-envmodules
```

# Our final snakemake workflow!

See [basic_demo_workflow](https://github.com/nesi/snakemake_workshop/tree/main/basic_demo_workflow) for the final Snakemake workflow we've created up to this point

- - - 

<p align="center"><b><a class="btn" href="https://nesi.github.io/snakemake_workshop/" style="background: var(--bs-dark);font-weight:bold">Back to homepage</a></b></p>
