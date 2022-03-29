# Online workflow code-along: intro to Snakemake

Are you working with big data? Do you need to pass your data through various software? If you’ve ever been in this situation, you would know it can become very difficult to maintain reproducibility and accuracy; wait, have I updated this output file? The more manual steps we do, the more human errors are inevitably introduced into our analysis, hampering accuracy and reproducibility.

Be lazy, the machine does it better.

Workflow languages automate your data analysis workflow. But this isn’t all, they ensure all your analysis logs are captured in an organised fashion, explicitly outline the software used, capture the input and output files at each step and even allow you to restart the pipeline from where it errored out, avoiding wasted time and resources re-running your workflow from the start. Additionally, when your data inevitably becomes big data, workflow languages allow you to easily scale up meaning you can move your analysis to a high performance cluster (HPC) without stress.

In this hands-on workshop, ESR’s Leah Kemp and the NeSI team will guide you through an introduction to [Snakemake](https://snakemake.readthedocs.io/en/stable/), a workflow language with its basis in the popular programming language, [Python](https://www.python.org/). Attendees can expect to learn:

- The benefits of using Snakemake or other workflow languages,
- how to create a workflow to organise your computations, and
- how an HPC scheduler (such as Slurm) fits into your workflow

## Who should attend

This workshop is intended for anyone who has several steps in their data analysis workflow. It is a hands-on workshop, meaning you will be coding along with Leah and the NeSI team.

## Setup

This is an online workshop that will use Jupyter on [NeSI](https://www.nesi.org.nz/). You will need to have a NeSI account in order to fully participate in the workshop. Those that register will be sent instructions on how to do this approximately 10 days before the event. 

## Workshop

**Workshop sections:**

- [01 - Introduction](./workshop_material/01_introduction.md)
- [02 - Setup](./workshop_material/02_setup.md)
- [03 - Create a basic workflow](./workshop_material/03_create_a_basic_workflow.md)
- [04 - Leveling up your workflow](./workshop_material/04_leveling_up_your_workflow.md)
- [05 - We want more!](./workshop_material/05_we_want_more.md)

**The workflows we will create:**

- [Basic demo workflow](https://github.com/nesi/snakemake_workshop/tree/main/basic_demo_workflow)
- [Leveled up demo workflow](https://github.com/nesi/snakemake_workshop/tree/main/leveled_up_demo_workflow)

<p align="center"><b><a href="https://nesi.github.io/snakemake_workshop/workshop_material/01_introduction.html">Start the workshop!</a>
