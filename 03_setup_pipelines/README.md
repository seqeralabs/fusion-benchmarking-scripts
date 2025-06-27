## Setup pipelines on Seqera Platform

### Table of contents
1. [Prerequisites](#1-prerequisites)
2. [Overview](#2-overview)
3. [Tutorial: Adding a test pipeline to the Launchpad](#3-tutorial-adding-a-test-pipeline-to-the-launchpad)
   - [YAML format description](#yaml-format-description)
     - [Environment variables in YAML](#1-environment-variables-in-yaml)
     - [Pipeline YAML definition](#2-pipeline-yaml-definition)
   - [Dry run mode](#dry-run-mode)
   - [Adding the pipeline](#adding-the-pipeline)
4. [Add your workflow to the Launchpad](#4-add-your-workflow-to-the-launchpad)

### 1. Prerequisites

- You have setup a Fusion V2 and plain S3 compute environment in the Seqera Platform in the [previous section](../02_setup_compute/README.md).
- You have created an S3 bucket for saving the workflow outputs.
- For effective use of resource labels, you have setup Split Cost Allocation tracking in your AWS account and activated the tags as mentioned in [this guide](../docs/assets/aws-split-cost-allocation-guide.md).
- If using private repositories, you have added your GitHub (or other VCS provider) credentials to the Seqera Platform workspace.
- You have reviewed and updated the environment variables in [env.sh](../01_setup_environment/env.sh) to match your specific Platform setup.

### 2. Overview

This directory contains YAML configuration files to add your workflow to the Seqera Platform Launchpad, as well as add the [nf-core/demo](https://github.com/nf-core/demo) workflow to the Seqera Platform Launchpad:

- `example_workflow_A_fusion.yml`: This configuration is to setup your custom workflow for benchmarking to run on Fusion V2 with the 6th generation intel instance type with NVMe storage. This workflow will use the `aws_fusion_nvme` compute environment created in the [previous section](../02_setup_compute/README.md#1-fusion-enabled-compute-environment).
- `example_workflow_A_plains3.yml`: This configuration is to setup your custom workflow for benchmarking to run on plain AWS Batch with S3 storage. This workflow will use the `aws_plain_s3` compute environment created in the [previous section](../02_setup_compute/README.md#2-plain-s3-compute-environment).
- `nf-core-demo.yml`: This configuration is to setup the [nf-core/demo](https://github.com/nf-core/demo) workflow to run on the Seqera Platform. This workflow will use the `aws_fusion_nvme` compute environment created in the [previous section](../02_setup_compute/README.md#1-fusion-enabled-compute-environment).

> **Note:** You can benchmark multiple workflows by copying and modifying these example configuration files. Simply create new YAML files for each workflow you want to test, adjusting the relevant parameters as needed.

We can start by adding a simple nf-core/demo pipeline to the Launchpad and then launching this in your chosen Workspace. This will ensure that `seqerakit` is working as expected and you are able to correctly add and launch a pipeline.

### 3. Tutorial: Adding a test pipeline to the Launchpad

Before we add our custom workflow to the Launchpad, let's start by adding the nf-core/demo pipeline to the Launchpad as defined in [`nf-core-demo.yml`](../seqerakit/pipelines/nf-core-demo.yml).

### YAML format description

#### 1. Environment variables in YAML

The YAML configurations utilize environment variables defined in the `env.sh` file. Here's a breakdown of the variables used in this context:

| Variable | Description | Usage in YAML |
|----------|-------------|---------------|
| `$ORGANIZATION_NAME` | Seqera Platform organization | `workspace` field |
| `$WORKSPACE_NAME` | Seqera Platform workspace | `workspace` field |
| `$COMPUTE_ENV_PREFIX` | Prefix for compute environment name | `compute-env` field |
| `$PIPELINE_PROFILE` | Config profile to run your pipeline with | `profile` field |

The `$PIPELINE_PROFILE` variable is defined in the `env.sh` file and can be used to specify a particular configuration profile for your pipeline. This allows you to easily switch between different sets of configuration options (e.g., 'test', 'production') without modifying the pipeline code or YAML files.

#### 2. Pipeline YAML definition

We can start by checking the YAML configuration file which defines the pipeline we will add to the workspace. The pipeline definition can be found at [`nf-core-demo.yml`](./pipelines/nf-core-demo.yml). Inspecting the contents here the file contains the following values:

```yaml
pipelines:
  - name: "nf-core-demo-test"
    url: "nf-core-demo.yml"
    workspace: '$ORGANIZATION_NAME/$WORKSPACE_NAME'
    description: "nf-core/demo is a simple nf-core style bioinformatics pipeline for workshops and demos."
    compute-env: "$COMPUTE_ENV_PREFIX_fusion_nvme"
    revision: "dev"
    overwrite: True
```

<details>
<summary>Click to expand: YAML format explanation</summary>

The YAML file begins with a block starting with the key `pipelines` which mirrors the command available on the Seqera Platform CLI to add pipelines to the Launchpad i.e. `tw pipelines add`. To give you another example, if you would like to create a Compute Environment in the Seqera Platform, you would use the `tw add compute-envs` command and hence the `compute-envs` key in your YAML file, and so on.

The nested options in the YAML also correspond to options available for that particular command on the Seqera Platform CLI. For example, if you run `tw pipelines add --help` you will see that `--name`, `--workspace`, `--description`, `--compute-env` and `--revision` are available as options, and will be provided to the `tw launch` command as defined in this YAML via `seqerakit`. However, other options defined in the YAML such as `url` and `overwrite` have been added specifically to extend the functionality in `seqerakit`.
<details>

#### 3. Dry run mode

Before we add the pipeline to the Launchpad let's run `seqerakit` in dry run mode. This will print the CLI commands that will be executed by `seqerakit` without actually deploying anything to the platform.

Run the following command in the 03_setup_pipelines directory of this tutorial material:

```bash
seqerakit --dryrun ./pipelines/nf-core-demo.yml
```

You should see the following output appear in the shell:

```shell
INFO:root:DRYRUN: Running command tw pipelines add --name nf-core-demo-test --workspace $ORGANIZATION_NAME/$WORKSPACE_NAME --description 'nf-core/demo is a simple nf-core style bioinformatics pipeline for workshops and demos.' --compute-env ${COMPUTE_ENV_PREFIX}_fusion_nvme --revision dev https://github.com/nf-core/demo
```

This indicates seqerakit is interpreting the YAML file and is able to run some commands. Check the commands written to the console. Do they look reasonble? If so, we can proceed to the next step.

#### 4. Adding the pipeline

We will now add the pipeline to the Launchpad by removing the `--dryrun` option from the command-line:

```bash
seqerakit ./seqerakit/pipelines/hello_world.yml
```
Output will be like:

```shell
DEBUG:root: Overwrite is set to 'True' for pipelines

DEBUG:root: Running command: tw -o json pipelines list -w $ORGANIZATION_NAME/$WORKSPACE_NAME
DEBUG:root: Running command: tw pipelines add --name nf-hello-world-test --workspace $ORGANIZATION_NAME/$WORKSPACE_NAME --description 'Classic Hello World script in Nextflow language.' --compute-env aws_fusion_nvme --revision master https://github.com/nextflow-io/hello
```

Go to the Launchpad page on your workspace on Seqera platform. You should see the hello world pipeline available to launch.

![Hello World added to Launchpad](../docs/images/hello-world-pipelines-add.png)

### 4. Add your workflow to the Launchpad

Now that you've confirmed your seqerakit setup and added the hello world pipeline, configure your custom workflows for the Launchpad:

1. Go to the `pipelines/` directory.
2. Edit `example_workflow_A_fusion.yml` with your workflow details:

    - `url`: GitHub repository URL (ensure credentials are added for private repos)
    - `description`: Brief workflow description
    - `profile`: Workflow profile (e.g., `test` or `test_full` for nf-core/rnaseq)
    - `revision`: Branch name, tag, or commit hash
    - `params`: Workflow parameters (inline or via `params-file:`)
    - `pre-run`: Path to pre-run script (optional)
    - `labels`: Workflow labels for organization

    Other details are optional for customization.

    A few of the details have been set for you in the example workflows. These are to ensure that the workflow run is configured to run on the Seqera Platform with the appropriate compute environment.

    ---

    **_NOTE:_** We have [specified a local path](../03_setup_pipelines/pipelines/example_workflow_A_fusion.yml#L9) to a [Nextflow config file](./pipelines/nextflow.config) through the `config:` option. This config file includes custom configuration settings for attaching resource labels to each process in the workflow. These resource labels will attach metadata such as the unique run id, pipeline name, process name, and so on, to each task submitted to AWS Batch. 
    
    ```json
    process {
    resourceLabels = {[
            uniqueRunId: System.getenv("TOWER_WORKFLOW_ID"),
            pipelineProcess: task.process.toString(),
    ...
    ```

    ---

3. Save the file and close the text editor. Feel free to rename the file to something more descriptive of your workflow.
4. Use these YAML files to add your workflows to the Seqera Platform Launchpad by running the following command:

```bash
seqerakit *.yml
```

This will add all pipelines to the Seqera Platform Launchpad and you will be able to see it in the Launchpad UI. Confirm your pipelines have been added to the Launchpad before moving onto the next step of launching them.
