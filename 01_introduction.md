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