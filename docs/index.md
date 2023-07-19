<center>
# Introduction to Snakemake
</center>

<center>![image](./theme_images/sm_logo)</center>

!!! circle-info "Automate Your Workflow With Snakemake"

    Are you working with big data?
    
    Do you need to pass your data through various software?
    
    Oh wait! Have I updated this output file?
    
    If you’ve ever been in this situation, you would know that it can become quite difficult to maintain consistency and accuracy.
    
    The more manual steps we execute, the more human errors that are inevitably introduced into our analysis - hampering accuracy and reproducibility.
    
    Sit back and let the machines do their magic.
    
    Workflow languages automate your data analysis workflow. They also ensure that all your analysis logs are captured in an organized fashion, explicitly outline the software used, capture the input and output files at each step and even allow you to restart the pipeline from where it errored out. The process ensures higher productivity and decreases loss of resources re-running your workflow from the start. Additionally, when your data inevitably becomes big data, workflow languages allow you to easily scale up - meaning, you can move your analysis to a high performance cluster (HPC) without stress!
    
    In this hands-on workshop, ESR’s Leah Kemp and the NeSI team will guide you through an introduction to Snakemake, a workflow language with its basis in the popular programming language, Python. Attendees can expect to learn:
    
    - The benefits of using Snakemake or other workflow languages,
    - How to create a workflow to organize your computations, and
    - How an HPC scheduler (such as Slurm) fits into your workflow

!!! three-users "Who should attend"

    This Workshop is intended for anyone who has several steps in their data analysis workflow. It is a hands-on workshop, meaning you will be coding along with Leah and the NeSI team.

    If you have any questions or would like more information about this session, please email training@nesi.org.nz


!!! tags "Attribution notice"
    
    * Material used in this workshop is based on the following repository
        * Author     : Leah Kemp (ESR)
        * Repository : [https://leahkemp.github.io/RezBaz2020_snakemake_workshop/](https://leahkemp.github.io/RezBaz2020_snakemake_workshop/)
    
!!! screwdriver-wrench "Setup"

    This is an online workshop that will use Jupyter on NeSI. Attendees will be required to set up an account on NeSI in order to participate. Full instructions will be sent to registrants closer to the time of the workshop.

!!! calendar-days "Workshop"


    **Workshop sections:**
    
    - [01 - Introduction](./workshop_material/01_introduction.md)
    - [02 - Setup](./workshop_material/02_setup.md)
    - [03 - Create a basic workflow](./workshop_material/03_create_a_basic_workflow.md)
    - [04 - Leveling up your workflow](./workshop_material/04_leveling_up_your_workflow.md)
    - [05 - We want more!](./workshop_material/05_we_want_more.md)
    
    **The workflows we will create:**
    
    - [Basic demo workflow](https://github.com/nesi/snakemake_workshop/tree/main/basic_demo_workflow)
    - [Leveled up demo workflow](https://github.com/nesi/snakemake_workshop/tree/main/leveled_up_demo_workflow)
    
    **Appendices:**
    
    - [Setup for running on your machine using conda](./workshop_material/supplementary/99_appendix_setup_on_your_machine.md)
    - [Use conda environments](./workshop_material/supplementary/99_appendix_use_conda_environments.md)
