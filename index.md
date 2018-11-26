

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


## Fault tolerance and error handling

The workflow must be robust against hardware/software/data failure. It should:
* give the user the option to fail or continue the whole workflow when something goes wrong with one of the samples
* have the ability to move a task to a spare node in the event of hardware failure.

The latter is a function of Cromwell, but the workflow should support it by requesting a few extra nodes (beyond the nodes required based on user specifications).

To prevent avoidable failures and resource waste, the workflow should: 
* check that all the sample files exist and have nonzero size before the workflow runs
* check that all executables exist and have the right permissions before the workflow runs
* after running each module, check that output was actualy produced and has nonzero size
* perform QC on each output file, write results into log, give user option to continue even if QC failed.

[User notification](#email-notifications) of success/failure will be implemented by capturing exit codes, 
writing error messages into failure logs, and notifying the analyst of the success/failure status 
via email or another notification system. We envision three levels of granularity for user 
notification:
* total dump of success/failure messages at the end of the workflow
* notification at the end of a stage
* notification at the end of a task.

The level of granularity will be specified by users as an option in the runfile.


## Portability

The workflow should be able to port smoothly among the following four kinds of systems:
* grid clusters with PBS Torque
* grid clusters with OGE
* AWS
* MS Azure.


## Development and test automation 

The workflow should be constructed in such a way as to support multiple levels of automated [testing](#testing):
* Unit testing on each task
* Integration testing for each codepath in each workflow stage
* Integration testing for the main (i.e. most used) codepath in the workflow
* Regression testing on all of the above.

