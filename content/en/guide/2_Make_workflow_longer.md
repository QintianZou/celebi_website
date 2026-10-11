---
title: "Making Workflows Longer"
weight: 2
summary: "Extend your analysis pipeline by chaining multiple tasks — add a second filter and pass data between tasks."
---
This guide walks you through extending an existing CELEBI workflow by adding a new filter step (`filter1`) 
and a merge step, creating a longer processing chain.



## 1. Create and Configure a New Filter Task

Create a task for `filter1` and link it to the algorithm:
```celebi
create-task filter1
cd filter1
```
Then create a `filter1.py` in `filter1` file.

Now set up the input:
```celebi
add-input ../filter0/ data_filter0
```
This means  `filter1` takes its input from the output of `filter0` and we name the input `data_filter0`. 
**Therefore, the input folder for `filter1.py` becomes `data_filter0/stageout` and the output folder is still `stageout`.**
Then configure the runner settings:
```celebi
cache-on-runner on
request-runner pkufarm212
```
To avoid setting this every time, write it in the main directory `[.]` and all tasks can be configured.

Verify the configuration with `ls`. You should see:
```celebi
o--> Predecessors:
[0] (task)       data_filter0: @/filter0
Environment: env_root_6.38.04
Cache on runner: True
Default runner: pkufarm212
```

## 3. Submit and Monitor the Filter1 Task
Submit the task to the remote runner:
```celebi
submit --runner pkufarm212
```

Use `draw-dag-graphviz` to visualize the workflow:

<img width="1916" height="806" alt="image" src="https://github.com/user-attachments/assets/77211ea5-9418-4e06-82a0-fd3a36e300e7" />


This way, you can create workflows as long as you need. 
Note that impressions are only generated for new tasks or tasks that have been changed in the directory from which you run `submit`.



