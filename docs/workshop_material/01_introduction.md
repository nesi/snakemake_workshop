# 01 - Introduction


## Benefits of workflow languages

- Reproducibility
- Automation
- Portability
- Scalability
- Interpretability

A good paper on [Scalable Workflows and Reproducible Data Analysis for Genomics](https://link.springer.com/protocol/10.1007/978-1-4939-9074-0_24) - although genomics focussed, it covers a lot of the concepts we will touch on in this workshop
  
## Benefits of Snakemake

![Snakemake](https://avatars2.githubusercontent.com/u/33450111?s=400&v=4 "Snakemake")

- Based in the popular (and widely used) programming language, Python
- Great documentation, actively maintained (but so are the other workflow languages mentioned below)
- Easier to learn (particularly if you're familiar with python)
- Flexible

See what other people think:

- [https://www.biostars.org/p/345998/](https://www.biostars.org/p/345998/)
- [https://www.reddit.com/r/bioinformatics/comments/f3rxel/snakemake_vs_nextflow/](https://www.reddit.com/r/bioinformatics/comments/f3rxel/snakemake_vs_nextflow/)
- [https://www.biostars.org/p/258436/](https://www.biostars.org/p/258436/)

## Other workflow languages

Choose your favourite flavour of workflow language!

- [Common Workflow Language (CWL)](https://www.commonwl.org/)
- [Nextflow](https://www.nextflow.io/)
- [Workflow Description Language (WDL)](https://openwdl.org/)
- [Guix Workflow Language (GWL)](https://workflows.guix.info/)

The real point is to use a workflow language (where applicable) and just use the flavour you like!

## This workshop

This workshop is designed with someone who had some familiarity with the command line. However, I've tried to make it as accessible as possible to anyone who wants to learn Snakemake.

Throughout this workshop, I'll be indicating the code to remove and the code to insert (relative to the previous step) with the following:

```diff
- Remove this line of code
+ Add this line of code
```

*However, the actual `+` and `-` symbols should not be included in your own code*

At each section of the workshop you can find a drop down box under "Current snakefile:" that will contain the main Snakefile that comprises the pipeline as a plain text file to copy and paste from if you need to catch up.

- - - 



<p align="center"><b><a class="btn" href="https://nesi.github.io/snakemake_workshop/" style="background: var(--bs-dark);font-weight:bold">Back to homepage</a></b></p>
