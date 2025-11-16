---
topic: containers
title: Tutorial2 - Hello-world Nextflow example
---

# Hello-world Nextflow tutorial on Puhti Supercomputer
Running and managing workflows for bioinformatics applications can be challenging as the workflows usually are fragile eco-systems of several software tools and their dependencies. We therefore need a workflow manager like Nextflow to manage our scientific workflow. 

> This tutorial is done on Puhti, which requires that:
   - you have a user account at CSC
   - your account belongs to a project that has access to the Puhti service.


## Learning Objectives
Upon completion of these tutorials in course, you will be able to learn: 
- The basic understanding of how Nextflow works
- The deployment of Nextflow scripts on interactive node
- The inspection of results


## Setup your work environment for tutorials on Puhti
This hands-on tutorial uses Puhti supercomputer for executing Nextflow scripts for interactive and batch jobs. One therefore needs to have either a training or user account at CSC to access Puhti.

Login to Puhti using `ssh` command followed by making a work directory named `nextflow_tutorial` on *scratch* drive as shown below: 

```nextflow
ssh <your_csc_username>@puhti.csc.fi
mkdir -p  /scratch/project_xxxx/$USER/nextflow_tutorial && cd /scratch/project_xxxx/$USER/nextflow_tutorial 
```

### Start an interactive session on Puhti

Lanuch an [interactive session](https://docs.csc.fi/computing/running/interactive-usage/) on Puhti as below:
```
sinteractive -c 2 -m 4G -d 250 -A project_2xxxx  # replace actual project number here
module load nextflow/23.04.3                     # Load nextflow module
```
‼️ Please note that one has to load a module (in this case nextflow) with a version. Otherwise, the latest version of stable module installed at that point is used. For the reproducibility point of view, make sure to load versions of all tools including the nextflow module.

## Tutorial 1: Hello-world example 

This "Hello-world" minimalist example demonstrates the basic syntax of Nextflow for a process. In this tutorial, you will learn how to run a Nextflow script as well as understand the default location of resulting output files. Copy the below script to a file named, hello-world.nf

```nextflow
#!/usr/bin/env nextflow
  
greets = Channel.fromList(["Moi", "Ciao", "Hello", "Hola","Bonjour"])

/*
 * Use echo to print 'Hello !' in different languages to a file
 */

process sayHello {

  input:
    val greet

  output:
    path "${greet}.txt"

  script:
    """
    echo ${greet} > ${greet}.txt
    """
}

workflow {

    // Print  a greeting
    sayHello(greets)
}

```

 Execute the script by entering the following command on your interactive Puhti terminal: 

```nextflow
nextflow run hello-world.nf
```
This script defines one process named `sayHello`. This process takes a set of greetings from different languages and then writes each one to a separate file.

The resulting terminal output would look similar to the text shown below:

```nextflow
N E X T F L O W  ~  version 23.04.3
Launching `hello-world.nf` [intergalactic_panini] DSL2 - revision: 880a4a2dfd
executor >  local (5)
[a0/bdf83f] process > sayHello (5) [100%] 5 of 5 ✔
```
#### Where are output files created from the above hello-world example?

The hexadecimal number, like e2/9aa8c8, identifies a unique process execution and the number is also the prefix of directory where `sayHello` process is executed. You can inspect the files produced by above script by changing to directory $PWD/work.

Execute the following command on your terminal:
```
ls -l work/*/*
```
You can see that there is a separate output file created under each directory. 

#### What kind of hidden files exist inside $PWD/work directory?

Hidden files are present in each process directory, and the files are very handy when you want to debug a failed process.

you can find the hidden files as shown below:

```
ls -la work/*/*
```

