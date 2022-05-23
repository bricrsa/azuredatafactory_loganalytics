# Monitoring Data Factory from Log Analytics

Azure Data Factory has an in-built [monitoring capability](https://docs.microsoft.com/en-us/azure/data-factory/monitor-visually). This is focused on operational understanding in a single data factory.

If you want to do larger more flexible analysis of your pipelines, or across factories, then using the standard Log Analytics integration is very useful.
Add [Diagnostic settings](/media/DiagnosticSettings.png)

## *Contents*

- [Understanding the Logs](#understanding-the-logs)
- [Pipeline Runs](#pipeline-runs)
- [Activity Runs](#activity-runs)
- [Data processed by Copy and Dataflow activities](#data-processed-by-copy-and-dataflow-activities)
- [Cost per Activity](#cost-per-activity)
- [Gantt chart for Pipelines from Activity data](#gantt-chart-for-pipelines-from-activity-data)
- [Known issues](#known-issues)
- [Understanding the size of your Logs in Log Analytics](#understanding-the-size-of-your-logs-in-log-analytics)



## Understanding the logs

Ensure that the Resource specific log analytics integration is used.

Learn about the query language: https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/

This will show the following tables in Log Analytics. [Schemas](https://docs.microsoft.com/en-us/azure/data-factory/monitor-schema-logs-events).

- ADFActivityRun
- ADFPipelineRun
- ADFTriggerRun
- ADFSandboxActivityRun
- ADFSandboxPipelineRun

## Pipeline Runs

*ADFPipelineRun*
 - contains tags, status, user properties, annotations and information about predecessors and triggers
 - includes parameters and system parameters

Example query of Pipelines calling pipelines in pipelines.
- [Pipeline linkage](/loganalytics_queries/pipeline_linking.kql)

## Activity Runs

*ADFActivityRun*
 - contains status, billing information and details of inputs and outputs
 - includes parameters 

Example query of activities in 
 - [Copy activities](./loganalytics_queries/activity_copy.kql)
 - [Dataflow activities](./loganalytics_queries/activity_dataflows.kql)

 ## Data processed by Copy and Dataflow activities

Note that for ADF Data Flows, the read and write MB is not entirely accurate, as it reflects the total number of bytes read and written across multiple stages.
This is then summarised to give a total number to give an idea of how much data the data flow is processing.

The detail of each series of stages is displayed in the dsl column under run_status.

- [MB Processed](./loganalytics_queries/activity_mb_processed.kql)

## Cost per Activity
 
 Pipeline cost is available in the Monitor blade of the ADF UI. It is possible to break this down to activity level.
 For those activities that have a variable cost, such as Copy and Dataflow activities, we can mine the detailed costs out of the billing data json within each activity.
 Coupling this with the specific cost for the region, integration runtime type and your billing arrangement, the actual cost per run can be calculated.
 [Understand general costs in ADF](https://azure.microsoft.com/en-gb/pricing/details/data-factory/data-pipeline/). Find your actual region based costs for your EA or other billing option by logging into the Pricing Calculator.

 An example query to calculate the cost for each [activity]
 - [Activity cost](/loganalytics_queries/activity_cost.kql). 

 Test the output of these queries against the actual costs you see in cost management to validate it is being calculated correctly.

## Gantt chart for Pipelines from Activity data 

Use the [Excel workbook](/media/ADF_Gantt.xlsx) to show a rudimentary Gantt chart of Pipeline runs. Thas can be adapted to show only Dataflows to highlight opportunities for TTL based dataflow cluster re-use.


## Known issues

If there is too much information in certain activities (typically a dataflow), ADF will not pass only the logs appropriately. An error will appear in the Output section of the activity: {"errorMessage":"Value is too large to be parsed for logging"}

## Understanding the size of your Logs in Log Analytics

ADF generally does not generate a large volume of logs. Note that you should always use the Resource Specific option for Diagnostic settings if available.

Example query for [exploring usage](./loganalytics_queries/usage_explore.kql)