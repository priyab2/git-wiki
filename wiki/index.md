


# Dependencies

# To-Dos

* As of Oct 24, 2017: Ram will work only on the bwa-mem module, to implement fully the template that could be used for other modules
    * multiple samples in parallel
    * all paths and parameters coded in runfile
    * loggery, user notification
    * error capture
    * checking inputs and executables
    * checking outputs
    * QC on input FASTQ
    * QC on outputs


7 Output Folder Structure
=============================

Cromwell creates a nested output folder structure, one for each tool, and for each sample inside:

* After every execution of a Cromwell tool, a folder named "cromwell-executions" is created in the directory from where the cromwell execution engine is run.
* Inside the "cromwell-executions" folder, a subfolder is created and its name corresponds to the name of the workflow specified in the .wdl file. For example, if the name of the workflow in the .wdl file is BWA_Mem_Run then a subfolder is created with the same name.
* After the workflow has successfully executed, an ID for the execution is created and this ID is unique for each run. In the example below "97cbc5de-ff36-4912-aa04-395b08702c85" is a unique ID name. 

```
  |-cromwell-executions
  |  |-BWA_Mem_Run
  |  |  |-97cbc5de-ff36-4912-aa04-395b08702c85
```

* Inside each folder with the unique ID name there are sub folders that represent the various tasks defined in the .wdl file.
* The various tasks have a "call" prefix as shown below. 

```
  |
  |-call-Samtools
  |-call-BWA-MEM
```

* Inside the individual task, every sample has an individual folder created for it.  "shard-0" in the example below represents Sample 1, "shard-1" represents Sample 2, and so on.

```
  |-call-Samtools
  |  |-shard-0
  |  |-shard-1
  |  |-shard-2
```
* Every shard folder contains an execution folder as seen below.

```
  |-shard-0
  |  |-execution
```

* A "glob-" folder is present inside every execution folder and it is inside this folder that the output of our program exists. The example below shows the where the output of our program resides at.

```
  |-execution
  |  |-glob-8fbfe7a84f921347caf0a4408578e296
  |  |  | -aligned.bam
```

* The example below shows how the entire directory tree and how the folders are created every time the cromwell excution engine executes a wdl program.

```
  |
  |-call-Samtools
  |  |-shard-2
  |  |  |-inputs
  |  |  |  |-projects
  |  |  |  |  |-mgc
  |  |  |  |  |  |-Project_1
  |  |  |  |  |  |  |-ram
  |  |  |  |  |  |  |  |-CromwellWDL
  |  |  |  |  |  |  |  |  |-MultiSampleMultiStepVC
  |  |  |  |  |  |  |  |  |  |-cromwell-executions
  |  |  |  |  |  |  |  |  |  |  |-BWA_Mem_Run
  |  |  |  |  |  |  |  |  |  |  |  |-97cbc5de-ff36-4912-aa04-395b08702c85
  |  |  |  |  |  |  |  |  |  |  |  |  |-call-BWA_Mem
  |  |  |  |  |  |  |  |  |  |  |  |  |  |-shard-2
  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |-execution
  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |-glob-8fbfe7a84f921347caf0a4408578e296
  |  |  |-execution
  |  |  |  |-tmp.Jh4vIo
  |  |  |  |-glob-e04fb83eeaea0b4b8fdde6282d30506c
  |  |-shard-0
  |  |  |-inputs
  |  |  |  |-projects
  |  |  |  |  |-mgc
  |  |  |  |  |  |-Project_1
  |  |  |  |  |  |  |-ram
  |  |  |  |  |  |  |  |-CromwellWDL
  |  |  |  |  |  |  |  |  |-MultiSampleMultiStepVC
  |  |  |  |  |  |  |  |  |  |-cromwell-executions
  |  |  |  |  |  |  |  |  |  |  |-BWA_Mem_Run
  |  |  |  |  |  |  |  |  |  |  |  |-97cbc5de-ff36-4912-aa04-395b08702c85
  |  |  |  |  |  |  |  |  |  |  |  |  |-call-BWA_Mem
  |  |  |  |  |  |  |  |  |  |  |  |  |  |-shard-0
  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |-execution
  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |-glob-e2984cba736e7acdf7c0536378232fca
  |  |  |-execution
  |  |  |  |-tmp.JOg8Ce
  |  |  |  |-glob-e04fb83eeaea0b4b8fdde6282d30506c
  |  |-shard-1
  |  |  |-inputs
  |  |  |  |-projects
  |  |  |  |  |-mgc
  |  |  |  |  |  |-Project_1
  |  |  |  |  |  |  |-ram
  |  |  |  |  |  |  |  |-CromwellWDL
  |  |  |  |  |  |  |  |  |-MultiSampleMultiStepVC
  |  |  |  |  |  |  |  |  |  |-cromwell-executions
  |  |  |  |  |  |  |  |  |  |  |-BWA_Mem_Run
  |  |  |  |  |  |  |  |  |  |  |  |-97cbc5de-ff36-4912-aa04-395b08702c85
  |  |  |  |  |  |  |  |  |  |  |  |  |-call-BWA_Mem
  |  |  |  |  |  |  |  |  |  |  |  |  |  |-shard-1
  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |-execution
  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |-glob-1b331778c20fad9525cbbad0d5f04486
  |  |  |-execution
  |  |  |  |-glob-e04fb83eeaea0b4b8fdde6282d30506c
  |  |  |  |-tmp.yCh857
  |-call-BWA_Mem
  |  |-shard-2
  |  |  |-inputs
  |  |  |  |-projects
  |  |  |  |  |-bioinformatics
  |  |  |  |  |  |-DataPacks
  |  |  |  |  |  |  |-human
  |  |  |  |  |  |  |  |-EllensSyntheticData
  |  |  |  |  |  |  |  |  |-WGS_chr1_5X_E0.005
  |  |  |  |  |  |  |  |  |  |-Chunks
  |  |  |  |  |  |-TestData
  |  |  |  |  |  |  |-HG19
  |  |  |  |  |  |  |  |-gatk_bundle_2.8
  |  |  |  |  |  |  |  |  |-2.8
  |  |  |  |  |  |  |  |  |  |-hg19
  |  |  |-execution
  |  |  |  |-glob-8fbfe7a84f921347caf0a4408578e296
  |  |  |  |-tmp.OWyZa0
  |  |-shard-0
  |  |  |-inputs
  |  |  |  |-projects
  |  |  |  |  |-bioinformatics
  |  |  |  |  |  |-DataPacks
  |  |  |  |  |  |  |-human
  |  |  |  |  |  |  |  |-EllensSyntheticData
  |  |  |  |  |  |  |  |  |-WGS_chr1_5X_E0.005
  |  |  |  |  |  |  |  |  |  |-Chunks
  |  |  |  |  |  |-TestData
  |  |  |  |  |  |  |-HG19
  |  |  |  |  |  |  |  |-gatk_bundle_2.8
  |  |  |  |  |  |  |  |  |-2.8
  |  |  |  |  |  |  |  |  |  |-hg19
  |  |  |-execution
  |  |  |  |-glob-e2984cba736e7acdf7c0536378232fca
  |  |  |  |-tmp.rrJiKx
  |  |-shard-1
  |  |  |-inputs
  |  |  |  |-projects
  |  |  |  |  |-bioinformatics
  |  |  |  |  |  |-DataPacks
  |  |  |  |  |  |  |-human
  |  |  |  |  |  |  |  |-EllensSyntheticData
  |  |  |  |  |  |  |  |  |-WGS_chr1_5X_E0.005
  |  |  |  |  |  |  |  |  |  |-Chunks
  |  |  |  |  |  |-TestData
  |  |  |  |  |  |  |-HG19
  |  |  |  |  |  |  |  |-gatk_bundle_2.8
  |  |  |  |  |  |  |  |  |-2.8
  |  |  |  |  |  |  |  |  |  |-hg19
  |  |  |-execution
  |  |  |  |-tmp.kbVcdk
  |  |  |  |-glob-1b331778c20fad9525cbbad0d5f04486
```

Email Notifications
====================

It's good practice to notify the analyst of failures in the workflow. Failure notifications can be emailed to the analyst. Analysts can be notified at each step as and when a sample fails, or a list of all the samples that have failed a step can be collected at the end of the step and that information can be emailed to the user.  

If notifications are sent for every sample, the analyst will be able to figure out that something is wrong and tend to the errors. On the other hand he/she ends up receiving numerous emails for all the samples that have failed a certain step. For example, if there are 1000 samples that run on a workflow and a majority of samples fail at the first step, the analyst can quickly look at the logs and figure out why the samples have failed that particular step. The disadvantage is that an email will be sent out for each sample, flooding the analyst's inbox.

A list of all the failed samples sent as one email will prevent flooding of the inbox. However, the analyst will have to wait for the end of the step to find out which samples have failed, which could take multiple hours (depending on the step of the workflow). These are some trade-offs which have to considered while designing the workflow.


Input Parsing and Type Validation
============

Although the workflow takes input in a JSON formatted file, it is more convenient to save input variables in a flat key="value" formatted file. The config_parser.py script (located in src/config) takes these flat configuration files and fills in a JSON file template provided by Cromwell/WDL.

(The template JSON file for a workflow can be created with the command `java -jar wdltool.jar inputs myWorkflow.wdl > myWorkflow_inputs.json`)

After parsing, the key_validation.py script (located in src/config) can be used to verify the types of the input arguments where possible. For example, the validator can verify that a key called "NumberOfThreads" was passed an integer as its value. The validator gets the type information for each key from a key types file (the workflow type information file is src/config/key_types.json) and confirms that the key's values match what is expected. However, for some types, such as strings, no pre-flight validation can be done.

Single Sample Workflow
======================

# Design Decision

The scripts for trimming sequences and alignment work on either single-ended or paired-end reads. Hence to eliminate complexities in the WDL workflow, the output for both the scripts include if checks for single-ended and paired-end reads. If the reads are single-ended then the right read is set to null. This helps remove branches in the workflow and keeps it simple. 
