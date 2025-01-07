---
title: Launch VSCode on Casper Compute Nodes
description: The qvscode script launches a new VSCode window that runs on a Casper compute node with user defined settings.
draft: false
authors:
  - neumanbrett
date: 2025-01-07
categories:
  - Casper
  - Script
  - VSCode
---

# qvscode

NCAR CISL Consulting Services Group has added the *qvscode* script that starts a new VSCode session on a Casper compute node.  The script can either take user input via terminal prompts or source a settings file to allocate resources for a compute node job.

<!-- more -->

## Motivation

VSCode uses significant resources on the login nodes when connecting with the RemoteSSH extension.  Even basic tasks like editing code with Intellisense can flag users for login node abuse.  The current method to connect to a compute node with VSCode requires creating an interactive job with the scheduler, identifying the compute node you are placed on, and then launching a new window to this node that will likely be different with the next job submission.  This script attempts to make requesting a compute node for VSCode easier and incentivize users to quickly move from the login node to a compute node when they are executing code, navigating large file spaces, need an exclusive node, or need additional compute, memory, or GPU capability from within VSCode.

## Procedure

1. Open VSCode on your local machine
2. In VSCode, connect to a Casper login node using RemoteSSH as described [here](https://ncar-hpc-docs.readthedocs.io/en/latest/environment-and-software/vscode/#connecting-to-derecho-or-casper)
3. Once connected to the login node, open up a new terminal window (Ctrl+Shift+\`)
4. Load the ncarenv/24.12 using `module load ncarenv/24.12` in the terminal
5. Call `qvscode`
6. Enter a valid project code and follow the prompts to launch a PBS job
7. A new VSCode window will open and connect to the compute node when your PBS job has started

A new VSCode window will launch on the compute node after user input or reading from the settings file.  The above procedure can be simplified by using a settings file as described in the [Settings Mode](#settings-mode) section.

!!! info
  Step 6 will not prompt you for a project code if you have PBS_ACCOUNT defined.

!!! warning 
    You *must* have a running VSCode Casper login session to launch the *qvscode* script. [Connect to a login node](https://ncar-hpc-  docs.readthedocs.io/en/latest/environment-and-software/vscode/#connecting-to-derecho-or-casper) and then launch the *qvscode* script from VSCode's built-in terminal.  This script will not work if you are connected to your local machine or launch it from outside of VSCode.

## Operating Modes

[Prompt Mode]($prompt-mode) will prompt the user for the values needed to launch the PBS job.  It contains default values to make launching a job faster.

[Settings Mode]($settings-mode) reads in variables from a user defined settings file and quickly launches a compute node with these settings.  It requires a specific path and format.

### Prompt Mode

This mode will be used when the script does not find the file `.qvscode_settings` in your home directory or you use the bypass argument.  You will be prompted for the PBS select statement arguments. 

```
bneuman@casper-login2:~> qvscode
Submitting job to Casper
Enter Project []: 
```

If you do not have a variable `PBS_ACCOUNT` setup then you will always be prompted to enter a valid project.  After the project prompt you will be asked if you would like to use the default values for the PBS select statements.  Answering 'N' to the default values prompts the user to enter variables for each of these basic job settings.  Note that the bracketed values are the default values and will be used if you do not enter any value when prompted.  

The defaults for the basic settings are for a serial CPU session with values of:

```
Account:  $PBS_ACCOUNT
Nodes:    1         
CPUs:     1         
Memory:   10GB         
GPUs:     0         
Walltime: 02:00:00         
Path:     $(pwd)
```

After finishing the basic settings you will be prompted to enter advanced options.  The advanced options are:

```
CPU Type:    
GPU Type:    
MPI Procs:   
OMP Threads: 
```

The advanced options have no default values and will only accept CPU and GPU types that are defined in PBS.

### Settings Mode

If the `qvscode` script finds a `.qvscode_settings` file in `$HOME` then it will import the variables into the script. The template requires specific keywords to pull values in.  It is *highly* recommended to copy the repository's `qvscode_settings_template` to your `$HOME` directory, rename it to `.qvscode_settings`, and then modify the values for each argument instead of manually creating the settings file.  You can use this command to copy the existing template into your home directory:

`cp /glade/work/csgteam/qvscode/qvscode_settings_template $HOME/.qvscode_settings`

The format for the template:

```
system=casper
project=SCSG0001
nodes=1
num_cpus=1
cpu_type=
memory=4GB
mpi_procs=1
ompthreads=1
num_gpus=
gpu_type=
walltime=01:00:00
path=$(pwd)

```

The `qvscode` script will use the default values for any variable that is not set.  If the path variable is empty then you will create a VSCode session that does not connect to GLADE and you won't be able to use the File Explorer but can still navigate the filesystem from the terminal.

As an example, if you wanted to request a single node of four Casper 80GB A100s then your settings would look like this:

```
system=casper
project=SCSG0001
nodes=1
num_cpus=4
cpu_type=milan
memory=200GB
mpi_procs=4
ompthreads=1
num_gpus=4
gpu_type=a100
walltime=01:00:00
path=$(pwd)

```

With your modified project code and memory requirements.

### Bypass and Custom Settings File Path Modes

You can bypass any existing `$HOME/.qvscode_settings` file by adding the `-b` or `--bypass` argument when calling `qvscode`.  You will then enter Prompt Mode.

You can use a custom settings file by providing the full path to the file with the `-p` or `--path` argument when calling `qvscode`.  Your custom .qvscode_settings file must use the same template format to set the values correctly.

```
Usage: qvscode [options]
  options:
    -b | --bypass      # Use prompt mode to enter job arguments and bypass any user settings files
    -p | --path path   # Provide the full path to a qvscode settings file to use in settings mode
    -h | --help        # Display this help message
```

## The New VSCode Compute Node Job

If the qsub arguments are valid then a new VSCode window will launch and connect to the compute node hostname.  You will be required to enter your NCAR two-factor authentication and may be asked to verify the SSH connection if you have not connected to the node previously.  You can close the new window and reconnect using the command provided by the script at job launch from the login node.  The format will be:

```code --remote ssh-remote+$USER@<computenode_hostname>.hpc.ucar.edu```

Pressing `Ctrl+C` from the login node terminal will kill the PBS job and end your compute node session.

## Log Files and Other Considerations

Log files are stored in `$SCRATCH/.qvscode_logs` and show the user arguments and job submission details.

This script will only work on Casper due to security guidelines set for Derecho's compute nodes.  We're investigating ways to connect to Derecho compute nodes using VSCode but there is no timeline.

Please pass along any errors or improvements to NHUG Slack channel.
