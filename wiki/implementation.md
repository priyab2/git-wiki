

# Implementation

## Implementing modularity

GenomeGPS consists of five component workflows. Each workflow may contain higher-level [modules](#modularity), which we call stages. For example, in BAM cleaning we have two stages: Alignment and Realignment/Recalibration.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img align="center" src="https://user-images.githubusercontent.com/4040442/34808679-108a8432-f656-11e7-856a-3542018692a0.png" alt="Image of Folder Structure" width="550">

Each *stage* consists of *tasks* - lowest complexity modules that represent meaningful bioinformatics processing steps (green boxes in the [figure](https://user-images.githubusercontent.com/4040442/34805599-9179e4aa-f644-11e7-993e-c0e9ece4f015.png) on the right), such as alignment against a reference or deduplication of aligned BAMs. Tasks are written as .wdl scripts:

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img align="right" src="https://user-images.githubusercontent.com/4040442/34805599-9179e4aa-f644-11e7-993e-c0e9ece4f015.png" alt="BAM cleaning with stages" height="850">

```WDL
#BWAMemSamtoolView.wdl

task ReadMappingTask {
   # Define variables

   command {
      BWA mem ReferenceFasta InputRead1 InputRead2 | Samtools view -> aligned.bam
   }

   output {
      File Aligned_Bam = "aligned.bam"
   }

   runtime {
      continueOnReturnCode: true
   }
}
```

Tasks can be called from other .wdl scripts to form workflows (such as the Alignment stage) or for testing purposes:


```WDL
#TestBWAMemSamtoolView.wdl

import "BWAMemSamtoolView.wdl" as BWAMEMSAMTOOLVIEW

workflow CallReadMappingTask {
   # Define inputs

   scatter(sample in inputsamples) {
      call BWASAMTOOLSORT.ReadMappingTask {
         input :
            sampleName = sample[0]
      }
   }
}
```


## Organization of the code

The /src folder is broken up by stages. Inside the folder for each stage (i.e. AlignmentStage_WDL/) we have three subfolders that contain: (1) the library of tasks (Tasks/), (2) the suite of unit tests, one for each task (TestTasks/), and (3) the resultant workflow stage (Workflows/), which can be used for testing the integration of individual tasks into workflows. The files are named as per the function that they perform.  


&nbsp;&nbsp;<img align ="center" src="https://user-images.githubusercontent.com/4040442/34808799-cd56402e-f656-11e7-960a-7cb5803b1d0e.png" alt="Modularity implementation" width="800"> 



## Special modules

We implemented the initial QC on executables and input data in a separate module that could be invoked from any workflow that is part of this package. A prototype of that module is currently here: https://github.com/ncsa/Genomics_MGC_GenomeGPS_CromwelWDL/blob/dev/src/AlignmentStage_WDL/Tasks/PreExec_QC.wdl.

Additionally, there is prototype of a module to notify the user of failure at the end of any workflow: https://github.com/ncsa/Genomics_MGC_GenomeGPS_CromwelWDL/blob/dev/src/AlignmentStage_WDL/Tasks/EndofBlock_Notify.wdl.



## Naming conventions

The naming convention used to name tasks, workflows and files is fairly straightforward. The bullets points below should provide a clear understanding how the files are named in this repository.

There are three types of files: 
* Files that represent a task. These files are named in reference to the command that the script executes. Every file name represents the function of the tool/tools that are used to run the script. For example, filename "BWAMemSamtoolView.wdl" represents that this file executes BWA Mem and Samtools View commands on the input samples. 
* Files that represent the testing of a task. These files are named in reference to the tasks they test. The name of the file starts with the word "Test" and then the name of the script it imports for testing. For example, filename "TestBWAMemSamtoolView.wdl" represents that this file imports the script "BWAMemSamtoolView.wdl" and checks the functionality of the Task defined therein.
* Files that represent the testing of a stage. These files are named in reference to the stage in the workflow that they run. The name of the file start with the word "Test" and then the name of stage in the workflow it executes and end with the word "Stage". For example, filename "TestAlignmentStage.wdl" shows that the Alignment stage of the workflow is being tested. 

* Task Names: The task modules within a file are named with respect to their functionality followed by the word "Task" (e.g. "ReadMappingTask").

* TestTask Names: The names of workflows that only call a specific task or tasks start with the word "Call" and then have the name of the task(s) called. For example, "CallReadMappingTask" is the name of the workflow that calls the task "ReadMappingTask." If the workflow represents a stage, the name of the Stage is used after the word "Call" instead.

* Alias Names: Each task is written as an individual script and these scripts are included into workflows which either test the functionality of each task or include multiple tasks into one workflow and test the functionality of a stage. Hence when importing these tasks into workflows, aliases have to be used in order access to the variables defined inside of the task. The alias names are the same as the name of the file being imported into the workflow except that they are all in CAPS. For example, import "BWAMemSamtoolView.wdl" as BWAMEMSAMTOOLVIEW. In this example the name of the alias is all in caps and is the same as the name of the file being imported.


## Scripting peculiarities imposed by WDL

### Bash

The command block in each task specifies the series of bash commands that will be run in series on each input sample. In order to script Bash variables in legible style, we have to use two tricks:
1. The command block needs to be delimited with `<<< >>>`, not `{ }`. This is because Bash variable names are best specified as ${variable}, not $variable, for legibility and correct syntax.
2. The Bash dollar sign for variables cannot be escaped. Therefore, the "dollar" has to be defined at the top of each .wdl script: `String dollar = "$"`.


### Calling of tasks 

When calling tasks from within workflows, one has to use the "import" statement and explicitly refer to the task using the specific folder path leading to it. This makes the workflow entirely un-portable. This problem may be alleviated in the server version of Cromwell by invoking the workflow with the -p flag. Running the server in an HPC cluster environment poses some security challenges (running as root, having access to all files on the filesystem without group restrictions). A udocker can be used for running the server version of Cromwell at the user level and thus circumventing the security issues.

In non-server mode, one can still invoke Cromwell with the -p option, and it will work, so long as it points to a zip archive containing the tasks that will be called from within the workflow. One should be able to zip up the entire folder tree for this code repository and supply it through this option. Then the task can be invoked in a workflow by specifying complete relative path to the task wdl script in the zip archive, i.e: `import "AlignmentStage_WDL/Tasks/Novosort.wdl" as NSORT`. 

We created a zip of the entire src/ folder tree and put it at the same folder level as the src/ folder, for download with the repository (via `git pull`). We are working on implementing automatic creation of this zip archive during nightly integration tests. When running Cromwell, use the -p option and specify the full path to the zip archive on your filesystem.


### Workflows of workflows

We would prefer to implement each stage of the BAM cleaning in GenomeGPS as a separate WDL workflow, and then use a global workflow to invoke the subworkflows (Alignment and Real/Recal). Thus the outputs of the last step of the previous workflow have to feed as the input to the first step of the next workflow. This introduces complexities because Cromwell generates its own output folder structure during runtime. In order to access these folders we would need a wrapper program which will parse out the run ID from the logs, traverse the respective output folder tree, find the output files and feed them to the next block. The workflow management system should be able to do that for us, but at present we do not see how. It is a TO-DO item.

Another issue with declaring subworkflows exists when there is a dependency between two tasks that belong to two separate workflows. A workflow which is included as a subworkflow inside another workflow will have issues with accessing variables from tasks in the imported workflow.

These issues can be resolved by specifying `output` block at the end of each component workflow. Then the master workflow can use those output variables to specify inputs to the downstream components. - Testing this functionality is a TO-DO item.

The example below will help explain how workflow of workflows(WOWs) are implemented. 

Consider two scripts: 
  1. A script which performs BWA mem and Samtools. This script has a task (Task1) 
     where the BWA and Samtools commands are specified. It also contains a workflow
     (WorkFlow1) whose output block has the "aligned.bam" output file.
  2. The second script which performs Novosort. This script also has a task (Task2)
     which where the Novosort commands are spcified. This script contains a workflow
     (WorkFlow2). The first script is imported into the second script and this is
     how workflow of workflows are created.

There are certain constraints that are to be followed while writing workflows-of-workflows (WOWs). All the variables that were declared in Task1 have to be a part of WorkFlow1. Inside WorkFlow1 the call to Task1 is made and in the input section of the call, Workflow1 variables are equated to Task1 variables. This is done because Script1 is imported into Script2 and when Script2 is compiled, then these variables from WorkFlow1 will be listed during the creation of the input json file. Inside WorkFlow2 is where we declare the TSV file which has Input Reads and the scatter block which creates multiple instances of the tasks for every sample. Inside WorkFlow2, calls to WorkFlow1 and the Novosort Task are made. WorkFlow1 is called inside WorkFlow2 for two reasons. One, because the Input Read files and the samplename are provided as input which in turns become the input to Task1. Second, the output of WorkFlow1 is input for the Novosort Task and by calling WorkFlow1, its output variable can be accessed.

The example scripts and its json input file are in the folder `/WOWScripts`.

----------------------


