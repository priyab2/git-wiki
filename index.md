

# Design principles

## Modularity

This workflow is modular by design, with each bioinformatics task in its own module. 
WDL makes this easy by defining "tasks" and "workflows." [Tasks](#workflow-architecture)
in our case will wrap individual bioinformatics steps comprising the workflow.
Tasks can be run individually and also strung together into workflows.

The variant calling workflow is complex, so we break it up into smaller subworkflows, or 
[stages](#workflow-architecture) that are easier to develop and maintain. 
Stages can be run individually and also called sequentially to execute the workflow fully or partially. 

Reasons for modular design:
* flexibility: can execute any part of the workflow 
    * useful for testing or after failure
    * can swap tools in and out for every task based on user's choice
* optimal resource utilization: can specify ideal number of nodes, walltime, etc. for every stage
* maintainability: can edit modules without breaking the rest of the workflow 
    * modules like QC and user notification, which serve as plug-ins for other modules, can be changed without updating multiple places in the workflow


## Data parallelism and scalability

The workflow should run as a single multi-node job, handling the placement of tasks 
across the nodes using embedded parallel mechanisms. Support is required for:
* running one sample per node 
* running multiple samples per node on clusters with and without node sharing.

The workflow must support repetitive fans and merges in the code (conditional on user choice in the runfile):
* splitting of the input sequencing data into chunks, performing alignment in parallel on all chunks, 
and merging the aligned files per sample for sorting and deduplication
* splitting of aligned/dedupped BAMs for parallel realignment and recalibration per chromosome.

The workflow should scale well with the number of samples - although that is a function 
of the Cromwell execution engine. We will be benchmarking this feature (see Testing section below).



## Real-time logging and monitoring, data provenance tracking

The workflow should have a good system for logging and monitoring progress of the jobs. 
At any moment during the run, the analyst should be able to assess: 
* which stage of the workflow is running for every sample batch 
* which samples may have failed and why 
* which nodes the analyses are running on, and their health status. 

Additionally, a well-structured post-analysis record of all events 
executed on each sample is necessary to ensure reproducibility of 
the analysis. 
