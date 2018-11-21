

# Testing
## Unit testing
### Pre-flight QC

To conduct unit tests on the Python scripts that handle pre-flight QC, cd into /path/to/MayomicsVC/src/config and run
```
python3 -m unittest discover
```
This will automatically detect all unit testing files and run their tests, but only if you are in the config directory.

### Workflow 

Every task is a unit, and is tested by running as its own workflow. These unit tests can be found in `src/{Name}Stage_WDL/TestTasks`. The json runfiles that specify inputs and paths to executables are provided in `json_inputs` folder. The following steps have to be followed to perform Unit Testing on individual tasks using Cromwell:

1. Download `source.zip` and the workflow script of the task which is to be checked. The unit test scripts are located in `src/{Name}Stage_WDL/TestTasks`. For example, if the BWAMemSamtoolView task is to be checked, then we require the workflow script which calls this task inside it, namely "TestBWAMemSamtoolView.wdl." 

2. To execute a wdl script using Cromwell we need two inputs:

   a) The wdl script to perform Unit Testing on (e.g. "TestBWAMemSamtoolView.wdl")

   b) The json input files that specify where the executables are located for the tools used. The json input files for our workflow are located in the folder `json_inputs` (e.g. json_inputs/BWAMemSamtoolView_inputs.json). If the user wants to create json input files of their own, the following link provides information on how to do so: 
   https://software.broadinstitute.org/wdl/documentation/article?id=6751.

3. Once the json input file is created, it will contain the list of variables to which hard-coded paths are to be provided. Hence, open the .json file using a text editor and input the paths for the executables, input file paths, output file paths, etc. 

4. The Cromwell command used to execute a wdl script is as follows:-

   `java -jar "Path to the cromwell jar" run "Input WDL file" -i "Corresponding json input file" -p source.zip`

   `For eg: java -jar cromwell.jar run BWAMemSamtoolView.wdl -i BWAMemSamtoolView_inputs.json -p source.zip`
   
   In the above command, "run" mode will execute a single workflow, and exit when the workflow completes (successfully or not). The "-i" is a flag which specifies the user to include a workflow input file.
   The "-p" flag points to a directory or zipfile to search for workflow imports. In the case of our workflow, use of the "-p" flag is mandatory. It specifies that source.zip is where the scripts to individual tasks are located. Information on how to execute a wdl script using cromwell can be found on the following link: 
   https://software.broadinstitute.org/wdl/documentation/execution.

## Integration testing

Every code path through the overall workflow should be tested for integration. For example, a user may choose BWA or Novoalign for alignment, and both options must be tested within the Align stage. We provide the complete set of json files specifying the various workflow configurations here: {insert path}. Thus each integration test can be invoked with the same command, just varying the json config file. 


