# Generate a Benchmarking Report from Pipeline Results

## Table of Contents
1. [Prerequisites](#1-prerequisites)
2. [Overview](#2-overview)
3. [Generate the Report](#3-generate-the-report)
   - [1. Fetch Run Dumps from Seqera Platform](#1-fetch-run-dumps-from-seqera-platform)
   - [2. Generate a Samplesheet](#2-generate-a-samplesheet)
   - [3. Compile the Benchmark Report](#3-compile-the-benchmark-report)
4. [Interpreting the Results](#4-interpreting-the-results)

---

### 1. Prerequisites

- You have successful executions of your Nextflow pipeline(s) with both Fusion on-demand and Fusion snapshots.
- You have access to the Seqera Platform workspace with the completed runs.
- You have access to an AWS cost and usage report (CUR) in parquet format, containing cost information for your benchmarking runs with the resource labels we have set up with you.

### 2. Overview
To compile an interactive report comparing your Fusion on-demand and Fusion snapshots runs, we will utilize the [nf-aggregate](https://github.com/seqeralabs/nf-aggregate) pipeline developed by Seqera. This pipeline will fetch the detailed report logs for your runs directly from Seqera Platform using the [tw cli](https://github.com/seqeralabs/tower-cli) and generate a report using [Quarto](https://quarto.org/), an open-source publishing system, similar to RMarkdown in R. 

This containerized Nextflow pipeline requires a samplesheet that includes the workflow IDs, the workspace names where you runs were executed, and the grouping assignment for each run (either 'Fusion on-demand' or 'Fusion snapshots'). A template for the samplesheet is available in `nf_aggregate_samplesheet.csv`.

```
id,workspace,group
run_id_1,org/workspace,fusion_snapshots
run_id_2,org/workspace,fusion_ondemand
```

Below is an example from the community/showcase, featuring real workflow IDs and workspace declaration to illustrate the correct formatting.

```
id,workspace,group
3VcLMAI8wyy0Ld,community/showcase,group1
4VLRs7nuqbAhDy,community/showcase,group2
```

You can directly enter your information in `nf_aggregate_samplesheet.csv` and overwrite the template so you can plug and play the seqerakit scripts in this folder.

### 3. Configuring nf-aggregate
Now that you have your samplesheet ready we will start the nf-aggregate run using seqerakit, similar to how we executed the benchmarking runs. For the seqerakit scripts in this folder, we assume you are reusing the already configured compute environment (CE) used for the fusion runs.

This directory contains YAML configuration files to add nf-aggregate to your Seqera Platform Launchpad and use your configured samplesheet as input to nf-aggregate for compiling the benchmarking reports:

- `datasets/nf-aggregate-dataset.yml` : This configuration will add the samplesheet for your benchmark runs as a dataset to Seqera Platform.
- `pipelines/nf-aggregate-pipeline.yml`: This configuration is to setup the nf-aggregate workflow for compiling benchmark reports from your workflow runs and your AWS cost and usage report (CUR).
- `launch/nf-aggregate-launch.yml`: This configuration is to launch the nf-aggregate workflow on the Seqera Platform with the Fusion V2 compute environment.

The YAML configurations utilize environment variables defined in the `env.sh` file found in [01_setup_environment](../01_setup_environment/env.sh). Here's a breakdown of the variables used in this context:

| Variable | Description | Usage in YAML |
|----------|-------------|---------------|
| `$ORGANIZATION_NAME` | Seqera Platform organization | `workspace` field |
| `$WORKSPACE_NAME` | Seqera Platform workspace | `workspace` field |
| `$COMPUTE_ENV_PREFIX` | Prefix for compute environment name | `compute-env` field |
| `$PIPELINE_OUTDIR_PREFIX` | Prefix for pipeline output directory | `params.outdir` field |
Beside these environment variables, there are a few nextflow parameters that need to be configured based on your setup. Go directly in to `./pipelines/nextflow.config` and modify the following variables:

1) If you are an enterprise customer, please change `seqera_api_endpoint` to your Seqera Platform deployment URL. The person who set up your Enterprise deployment will know this address.
2) Set `benchmark_aws_cur_report` to the AWS CUR report containing the cost information for your runs. You can provide the direct S3 path to this file if your credentials in Seqera Platform have access to this file. Otherwise, please upload the parquet report to a S3 bucket accessible by the AWS credentials associated with your compute environment.
> **Note**: If you are using a Seqera Platform Enterprise instance that is secured with a private CA SSL certificate not recognized by default Java certificate authorities, you will need to amend the params section in the [nf-aggregate.yml](../launch/nf-aggregate-launch.yml) file before running the above seqerakit command, to specify a custom cacerts store path through `--java_truststore_path` and optionally, a password with the `--java_truststore_password` pipeline parameters. This certificate will be used to achieve connectivity with your Seqera Platform instance through API and CLI.
2) Set `benchmark_aws_cur_report` to the AWS CUR report containing your runs cost information. This can be the direct S3 link to this file if your credentials in Seqera Platform have access to this file, otherwise, please upload the parquet report to a bucket accesible by the AWS credentials associated with your compute environment.

### 4. Add the samplesheet to Seqera Platform
To add the samplesheet to Seqera Platform, run the following command:

```shell
seqerakit -j ./report/datasets/nf-aggregate-dataset.yml
```
This will return a JSON object with the dataset name, workspace, and dataset ID.

### 5. Retrieve the Dataset URL
You will need the Dataset URL from the Platform in order to launch nf-aggregate from the command-line. Use the following command to get the Dataset URL and create a new environment variable called $DATASET_URL:

```shell
export DATASET_URL=$(tw -o json datasets url --name fusion-benchmark-samplesheet --workspace $ORGANIZATION_NAME/$WORKSPACE_NAME | jq .datasetUrl | tr -d '"')
```
You can double-check the value of $DATASET_URL with the command below:

```shell 
echo $DATASET_URL

# Example output:
# https://api.cloud.seqera.io/workspaces/100452700310173/datasets/4f2d6orAHG5j7YY1DQtEzP/v/1/n/nf_aggregate_samplesheet.csv

```

### 6. Add nf-aggregate to the Launchpad

Add nf-aggregate with the proper configuration to the launchpad and then launch nf-aggregate, we will use seqerakit as we have done for running the benchmarks.

This configuration includes an `input` parameter that is set to the `DATASET_URL` environment variable.

```shell
seqerakit ./report/pipelines/nf-aggregate-pipeline.yml
```

### 7. Launch nf-aggregate
Once the pipeline is added to the Launchpad, you can launch nf-aggregate from the Launchpad. 

This configuration includes an `outdir` parameter that is set to the `PIPELINE_OUTDIR_PREFIX` environment variable to store the reports generated by the nf-aggregate run.

```shell 
seqerakit ./report/launch/nf-aggregate-launch.yml
```

This will launch nf-aggregate to compile the benchmarking report. Once the pipeline finishes, head to the 'Runs' tab to download the HTML report.

### 8. Interpreting the results

The benchmark report includes various sections, ranging from high-level comparisons to detailed task-level analyses. Each section contains comments and explanations to guide you through the results.

Upon completion, please share a copy of the report with the Seqera team to discuss the findings and walk you through the details.

To learn more about each section of the report, see the [Appendix](#report-sections).

## Appendix

### Report sections

<details>
<summary>Benchmark overview</summary>
This section provides a general overview of the pipeline run IDs used in the report for each group. If a `runUrl` is found in the logs, the run IDs will be clickable links. Please note that access to the specific Seqera Platform deployment and workspace is required for these links to work.
</details>
<br>

<details>
<summary>Run overview</summary>
This section contains detailed information about the runs included in the report. It features a sortable and filterable table with technical details such as version numbers for pipelines and Nextflow, as well as information about the compute environment setup. Below the table, bar plots provide a visual comparison of key performance characteristics at the pipeline level.

- **Accurate compute cost**: The total expense incurred by Nextflow tasks for AWS elastic compute instances (EC2) consumed during workflow execution, including both actively used and idle but allocated resources, retrieved from the AWS cost and usage report. This does not include cost for the Nextflow head job or any costs other than EC2 (S3 transfer costs, VPC costs, FSx costs etc.).
- **Accurate used cost**: The cost of vCPU and memory resources that were actually allocated to and consumed by Amazon ECS tasks during the workflow execution period.
- **Accurate unused cost**: The cost of vCPU and memory resources that were allocated to EC2 instances but remained unutilized by ECS tasks during the workflow execution period. This represents capacity that was reserved and paid for but was not used.
- **Total run time**: The cumulative execution duration across all Nextflow tasks used in the workflow, calculated by summing the run time of all individual tasks.
- **CPU efficiency**: The percentage of allocated CPU resources that were actively utilized during task execution, calculated as (CPU time consumed / CPU time allocated) × 100%. Higher percentages indicate better utilization of provisioned CPU capacity.
- **Memory efficiency**: The percentage of allocated memory that was actively used during task execution, calculated as (memory consumed / memory allocated) × 100%. Higher percentages indicate better utilization of provisioned memory resources.

</details>
<br>

<details>
<summary>Process overview</summary>
This section presents an overview of run times, combining both staging time and real execution time for all processes. It displays the mean run time, with one standard deviation range around the mean for each task.
</details>
<br>
<details>
<summary>Task overview</summary>
This section provides insights into instance type usage and task staging and execution times.

- **Task Instance Usage**: This subsection shows the number of tasks that ran on different instance types during pipeline runs, allowing for quick comparisons of instance type usage between groups. Users can hover over the stacked bar plots to view the detailed distribution of instance types and can use the legend to highlight or hide specific instance types.

  - **Task metrics**: The plots show pairwise correlations between the plainS3 run and the Fusion run for both staging time (staging in and staging out) and real execution time. The dashed diagonal line represents perfect correlation between the two runs, meaning that if the tasks in both runs were exactly the same, all points would lie on the diagonal line.