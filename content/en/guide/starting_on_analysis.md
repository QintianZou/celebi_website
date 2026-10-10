---
title: "Starting an analysis"
summary: "Start an analysis using Celebi with data on the server and code on your local machine."
---

This file demonstrates how to start an analysis using Celebi. In this example, all data is stored on the server, but the code resides on our local computer.

## Prerequisite: Start Yuki

First, run the following command in WSL to start Yuki:

```WSL
❯ yuki docker run yuki:dev --dev-dir ~/Yuki --celebi-dir ~/Celebi
```
Then run `celebi`
to start our software. Type `Ctrl+D` when you want to exit.

## Initialize the Project

Create a new folder in your workdir to contain the project:
```celebi
>>>> mkdir B02K3pi
>>>> cd B02K3pi
>>>> celebi init
```
This new folder becomes your analysis project.

## 1. Bring in Raw Data
Raw data on the server should be linked to the analysis system on your local computer. 
Use the following commands to build the data file, which connects to the data on the server:
```celebi
>>>> create-data Raw
>>>> cd Raw
>>>> register-data pkufarm212 /home/user/workdir/TestData
```
## 2. Connect Data and Task

We need a task to process the data.
### Create a task
Create a new task in the project folder:
```celebi
>>>> create-task filter0
```
Create a `filter0.py` program inside the `filter0` directory and add `commands` to the YAML file in `filter0`:
```yaml
commands:
- python3 filter0.py
descriptor: filter0
environment: env_root_6.38.04
memory_limit: 256Mi
```
`Commands`: These are the commands that will be 
executed when running the workflow. Note that a dash (`-`) followed by a space must precede each command.


Add the input data to the task:

```celebi
>>>> cd filter0
>>>> add-input ../Raw raw_data
```

Here we create a new name for the input data ("raw_data" here), which is only for the task and can be arbitrary.
The new name is to ensure that even if we change the names or relative addresses of the tasks or data, 
the workflow we have defined can remain unchanged.

To remove the input data when you change your mind, use
```celebi
>>>> remove-input raw_data
```
Notice that you should use the new name of the input.

After we `add-input` to `filter0`, a basic workflow has been set up. To enable the `filter0.py` can get input data and create output files,
ensure:
- The input folder is `raw_data/stageout`
- The output folder is `stageout`

Because, from the point of view of `filter0.py`, the input files are in `raw_data/stageout`, and it will create output files in `stageout`.


## 3. Run the Workflow
### Set the Environment
Check what environments are available on the server:
```celebi
>>>> runner-envs pkufarm212
Conda environments on 'pkufarm212' (3):
  base                          /home/zouqt/miniconda3
  env_root_6.38.04              /home/zouqt/miniconda3/envs/env_root_6.38.04
  snakemake                     /home/zouqt/miniconda3/envs/snakemake
```

Copy `env_root_6.38.04` to the YAML file of the task.

To set a default environment for the project, type
```celebi
[Celebi][B02K3pi][.]
>>>> user-config
```
you'll see:
```
# Celebi user configuration.
#
# The values below are the built-in defaults. Change one to override
# it, or delete its line to fall back to the default.

# Editor used by `config`, `edit-script` and `readme`.
editor: vi

# Program used to open a local file, e.g. by `view local:...`.
file_opener: xdg-open

# Command used to open a URL. Leave empty to use the system default browser.
browser: ''

# Runner assigned to newly created tasks. Existing tasks keep whatever they
# were created with.
default_runner: pkufarm212

# Whether newly created tasks download their outputs automatically.
auto_download: true

# Whether newly created tasks cache their results on the runner.
cache_on_runner: true

# Directory where `draw-dag` writes its output.
dag_output_dir: ~/Downloads

# Environment written into a new task's celebi.yaml. Changing this changes
# the impression of tasks created afterwards.
task_environment: env_root_6.38.04

# Environment written into a new algorithm's celebi.yaml.
algorithm_environment: script
```

Change `task_environment` and save it, then every task will use this environment.
### Configure the Runner
If "Cache on runner" is `False`, enable it:

```celebi
>>>> cache-on-runner on
```
If there is no runner configured, request one:
```celebi
>>>> request-runner pkufarm212
```
### View the Workflow
Use ``ls`` in the task folder to see the workflow you just built:
```celebi
>>>> ls
>>>> DITE: [connected]
README: 
Please write README for task filter0
o--> Predecessors:
[0] (task)       raw_data: @/TestData
---- Task files:
filter0.py
Environment: env_root_6.38.04
Memory limit: 256Mi
Validated: True
Request runner: pkufarm212
Request cache: True
---- Commands:
python3 filter0.py
```
### Submit the Job
When everything is ready, submit the task:
```celebi
>>>> submit --runner pkufarm212
```
The task will be executed by the server.
### Check Job Status
Use the `status` command to monitor the job:
```celebi
>>>> status
Status of : filter0_task
Impression: [9017b6082037087e287fa89b76caca7b]
DITE: [connected]
Job status: [in movement][running]
Details: Executing workflow steps
**** Workflow: 
Workflow: [pkufarm212][494f0f6c61b34204b2e8910404c772e9]
Stageout files:
    (nothing to show yet — run 'collect', or the runner may be unreachable)
```
To see realtime log, type `log -f`, and the log will display below.

When the job is finished, the status will show:
```celebi
>>>> status
Status of : filter0_task
Impression: [9017b6082037087e287fa89b76caca7b]
DITE: [connected]
Job status: [coda][finished]
Details: Remote execution completed
**** Workflow: 
Workflow: [pkufarm212][494f0f6c61b34204b2e8910404c772e9]
Stageout files:
    NAME                              SIZE  TYPE   IN YUKI
    24r1_down_cut0.root           758.7 MB  data   ✗
    24r1_up_cut0.root               1.2 GB  data   ✗
    25c1_down_cut0.root           135.6 MB  data   ✗
    25c1_up_cut0.root             555.9 MB  data   ✗
    25c2_down_cut0.root             1.3 GB  data   ✗
    25c3_up_cut0.root             867.2 MB  data   ✗
    25c4_down_cut0.root           328.7 MB  data   ✗
    25c4_up_cut0.root             442.7 MB  data   ✗
```


You can use `log` to review the log. 
### Visualize the Workflow
Run the following command to generate a sketch of the workflow:
```celebi
>>>> draw-dag-graphviz
```
<img width="1764" height="822" alt="image" src="https://github.com/user-attachments/assets/8528a658-381c-43e8-82e1-692ccbd70f72" />


Note: If you don't have Graphviz installed, install it in WSL using:
```celebi
sudo apt update && sudo apt install graphviz
```
