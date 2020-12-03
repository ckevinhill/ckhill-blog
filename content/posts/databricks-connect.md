---
title: "Databricks Connect"
date: 2020-12-03T09:17:20+08:00
tags: ["datascience", "devops"]
---


### Databricks Connect

Databricks provides an end-to-end Datascience development environment including:

* Notebooks
* Git integration
* Scheduler
* ML Flow
* Spark Runtime

Databricks is reasonably convenient for Data Science project start-up and experimentation as it does provide quick access to wide-range of tooling.

However when dealing with developing actual software, leveraging good DevOps practices, Databricks is somewhat deficient as a development platform.  In particular the paradigm of Notebook-based development and lack of IDE-based support (linting, testing, etc.) is limiting.

[Databricks-connect](https://docs.microsoft.com/en-us/azure/databricks/dev-tools/databricks-connect) is an effort to enable local development, supported by Databricks cluster execution.  This helps enable all of the benefits of Databricks (in particular optimized Spark execution) combined with the ability to use local, IDE-based development.

>For any production-oriented development it is recommended to not use the Databricks Notebook based interface but instead use a local IDE enabled with databricks-connect.

## Databricks-connect Setup (typical)

There is decent documentation on how to setup databricks-connect [here](https://docs.microsoft.com/en-us/azure/databricks/dev-tools/databricks-connect).

The only nugget missing is on how to acquire a Personal Access Token from Databricks.  This can best be executed via:

1. Click on top right user profile icon in Databricks workspace
2. Click _User Settings_
3. Click on _Access Tokens_ tab

![Tokens](/images/databricks-get-tokens.png)

4. Click on _Generate New Token_ button

If you are having issues executing `databricks-connect test` to confirm connection you may want to check the values in your ~/.databricks-connect file.  You may also want to try different standard databrick ports (e.g. 8787 or 15001).  You can force port use via `spark.databricks.service.port 8787` added to _Configuration_ if desired.

You can confirm that the Databricks remote server service has been specified in configuration by checking _Advanced Options_ in the Databricks cluster _Configuration_ tab.

![Config](/images/databricks-cluster-config-enabled.png)

>Databricks-connect errors are not always super helpful.  It may indicate that 'spark.databricks.service.server.enabled true' needs to be added to configuration even if it already has.

## Databricks-connect Setup (Remote Container)

You can also setup databricks-connect using the awesome [Remote Dev Container extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers).  

You can follow instructions [here from Data Thirst](https://datathirst.net/blog/2020/6/7/databricks-connect-in-a-container).  Warning:  I haven't used (yet).

## IDE integration

Once you activate the conda environment with databricks-connect installed and configured it will automatically route to the remote Databricks cluster once you have initated your spark session:

`spark = SparkSession.builder.getOrCreate()`

You can now debug locally with either debug terminal, typical break-points and/or Notebooks with execution leveraging the remote Databricks environment.

For possibly more convenience you can consider the [VS Code Databricks extension](https://marketplace.visualstudio.com/items?itemName=paiqo.databricks-vscode) - which also uses databricks-connect under the hood.  Warning:  I haven't used.

>Generally I recommend to think of Databricks as a remote execution environment optimized for Spark vs. a development environment.  This is consistent with something like the Microsoft Azure Machine Learning Workspace design that allows you to attach Databricks as one of many possible execution targets.

## Remaining Gaps

This helps close the most pressing issues with Databricks development by enabling local tool use (git/VS Code, Testing, etc.) and an ability to move away from Notebooks while still leveraging Databricks Spark optimization and supporting infrastructure.

However this does not easily provide a way to "deploy" code to Databricks if you were planning on using Databricks scheduler.  Honestly I don't really think you should probably be thinking of Databricks as a deployment environment so I don't see this as much of an issue.  If you do need to deploy into Databricks you can consider either manually copying files into the Databricks UI and/or using the Databricks Git sync functionality.