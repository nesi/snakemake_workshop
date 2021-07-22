# Workflow languages – your foundation for accuracy and reproducibility in data analysis

**This workshop is a part of [RezBaz 2020: Pick n Mix](https://resbaz.auckland.ac.nz/)** <br />
**Date:** Friday 27 November 2020 <br />
**Time:** 2:30pm - 3:30pm <br />

Are you working with big data? Do you need to pass your data through various software? If you’ve ever been in this situation (as I have in a population genetics masters project), you would know it can become very difficult to maintain reproducibility and accuracy; wait, have I updated this output file? The more manual steps we do, the more human errors are inevitably introduced into our analysis, hampering accuracy and reproducibility.

Be lazy, the machine does it better.

Workflow languages automate your data analysis workflow . But this isn’t all, they ensure all your analysis logs are captured in an organised fashion, they explicitly outline the software (and exact software versions) used, the input and output files at each step. Lastly, when your data inevitably becomes big data, you can easily scale up from running your analysis on your laptop, to running your analysis on a high performance cluster (HPC) such as [NeSi](https://www.nesi.org.nz/).

In this workshop, we will work through an introduction to [Snakemake](https://snakemake.readthedocs.io/en/stable/), a workflow language with its basis in the popular programming language, [Python](https://www.python.org/). This Workshop is intended for anyone who has several steps in their data analysis workflow, particularly when many different software are involved. [Book here](https://vuw.libcal.com/event/5293465/).

**Workshop sections:**

- [01 - Introduction](./workshop_material/01_introduction.md)
- [02 - Setup](./workshop_material/02_setup.md)
- [03 - Create a basic workflow](./workshop_material/03_create_a_basic_workflow.md)
- [04 - Leveling up your workflow](./workshop_material/04_leveling_up_your_workflow.md)
- [05 - We want more!](./workshop_material/05_we_want_more.md)

**The workflows we will create:**

- [Basic demo workflow](https://github.com/leahkemp/RezBaz2020_snakemake_workshop/tree/main/basic_demo_workflow)
- [Leveled up demo workflow](https://github.com/leahkemp/RezBaz2020_snakemake_workshop/tree/main/leveled_up_demo_workflow)

<p align="center"><b><a href="https://leahkemp.github.io/RezBaz2020_snakemake_workshop/blob/master/workshop_material/01_introduction.md">Start the workshop!</a>
