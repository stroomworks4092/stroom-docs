---
title: "Pipeline Debugging"
linkTitle: "Pipeline Debugging"
weight: 21
description: >
  How to check if a pipeline is working, and what to do if it isn't doing what you expect.
tags:
  - pipeline
---


## Debugging with Stepping Mode


The **Stepper** is Stroom's most powerful debugging feature.
Unlike traditional logs, it allows you to visualize the data transformation as it happens, one record at a time.


### How to Start Stepping


1.  Navigate to a {{< stroom-doc "Pipeline" >}}, Structure tab. 
1.  Click the **Step** {{< stroom-icon "step.svg" >}} button in the toolbar.


### What the Stepper Shows


The UI will split into multiple panes, each representing an element in your pipeline (e.g., Source, Parser, XSLTFilter, XMLWriter).


*   **Input vs. Output**:
    You can click on any element to see exactly what data it received and what data it produced.
*   **Navigation**:
    You can move forward or backward through the stream record-by-record using the arrow buttons.
*   **Finding Errors**:
    If an XSLT transformation fails on record #502, you can jump to that specific record and see the input XML that caused the crash.
*   **Markers**:
    Fatal errors and warnings are highlighted with red and yellow markers in the margin, allowing you to quickly identify "bad" data.


### Use Cases for Stepping


*   **Developing XSLT**:
    See the immediate effect of your stylesheet changes on real data.
*   **Fixing Parsers**:
    Identify why a specific log line isn't being split into the correct XML fields.
*   **Validating Lookups**:
    Verify that reference data lookups are returning the expected values.


## How to Tell if it Worked


Once you've configured your Processor Filter and enabled the Pipeline, you can monitor its progress through several screens in the Stroom UI:


### Processor Filter Status


When you look at the **Processor** tab of a Pipeline, you will notice two levels of "Enabled" state:


1.  **The Processor**:
    This is the top-level parent (shown in the top pane).
    Enabling/Disabling this affects all filters associated with the pipeline.
1.  **The Filter**:
    This is the specific set of criteria (shown in the bottom pane, e.g., Feed = X).


{{% note %}}
Both the Processor (parent) and the Filter (child) must be checked for data to start flowing.
{{% /note %}}


This design allows you to pause all processing for a pipeline globally while keeping specific filter configurations ready.
You can also pause just one specific feed's filter without stopping others.


*   **Progress**:
    The progress bar and stream counts will show you how many streams have been processed vs. how many are remaining.
*   **Last Poll**:
    Tells you when the system last searched for new data to process.


### The Task Manager


Open **Monitoring -> Tasks** to see real-time pipeline execution.
If a pipeline is currently running, you will see a `PipelineProcessor` task here.
This is useful for seeing if a pipeline is "stuck" or taking an unusually long time.


### Data Tracking


The most definitive way to verify success is to look at the **Output Feed**.


1.  Navigate to the feed specified in your `StreamAppender`.
1.  Check the **Data** tab.
    You should see new streams with the timestamp and stream type you configured.
1.  Click on a stream to view its content and ensure the transformation (XML, JSON, etc.) looks correct.


## What to do if it Fails


If your pipeline isn't producing output or is behaving unexpectedly, follow these steps:


### Check for Errors (Stream Status)


If a stream fails during processing, it will often be marked as **Error** in the stream store.


1.  Go to **Monitoring -> Streams**.
1.  Look for streams with a status of `Error`.
1.  Select the stream and look at the **Attributes** or **Logs**.
    Stroom often attaches an "Error Stream" to the failed task containing the specific SAX or XSLT error message.


You can also look at the {{< stroom-doc "Feed" >}} Data tab and see if there is a stream of Type=`Error` within the Feed.
This is where errors raised in processing the feed will be written.


### Reprocessing Streams


It is possible to reprocess streams, but the option is located in a different part of the UI than where you create standard "standing" Processor Filters.


To reprocess a stream (or a set of streams):


1. Go to the Data tab: Navigate to the Data tab of a Feed, Pipeline, or Folder.
1. Select the Stream(s): Select the specific stream(s) you wish to reprocess. 
   You can use filters to narrow down the list if needed.
1. Click the Process button: Click the Process {{< stroom-icon "process.svg" >}} button in the toolbar above the stream list.
1. Enable "Reprocess data": In the Process Choice dialog that appears, you will see a checkbox labeled Reprocess data. 
   Checking this ensures that Stroom ignores any existing processing history for those streams and processes them again.
1. Choose Pipeline: If you initiated this from a Feed or Folder, you will be prompted to choose the Pipeline to use for the reprocessing.


{{% note %}}
To see the Process {{< stroom-icon "process.svg" >}} button, your user account must have the `Manage Processors` application permission.
{{% /note %}}

#### Why is it there?


Stroom distinguishes between Standing Filters (which automatically process new data as it arrives) and Reprocess Filters (which are targeted at specific existing data).

When you use the "Process" button with "Reprocess data" checked, Stroom creates a temporary Processor Filter configured specifically for your selection. 
Once it has finished processing that batch of data, the filter has completed its job.

Note: If you are reprocessing data to fix a bug or update a schema, remember to delete the previous (incorrect) output streams first to avoid having duplicate records in your downstream system.


### Common Issues


*   **No Streams Processed**:
    Check that your Processor Filter's criteria (Feed, Stream Type, Status) exactly match the meta-data of your source data.
    Ensure the filter is `Enabled`.
*   **Empty Output**:
    This often means your XSLT didn't match any elements in the input XML.
    Check your XSLT logic and namespaces.
*   **Fatal Errors**:
    These often relate to system issues like "Disk Full" on an appender or "Network Timeout" on an HTTP appender.
*   **Incorrect XML**:
    * If using {{< stroom-doc "PlanB" >}} there must be a `<map>name</map>` element within the `<referenceData>` XML to ensure that the data is written to the right data store.
    * Ensure that the schema of the data is correct for the destination. 
      You can use a {{< pipe-elm "SchemaFilter" >}} pipeline element during development to ensure that the format is correct.


{{% note %}}
If a Processor Filter has already scanned all available data and found nothing, it may not automatically pick up "old" data that existed before the filter was created.
In some cases, you may need to **Delete and Recreate** the Processor Filter to force a fresh scan of the data store from the beginning.
{{% /note %}}


---

| Navigation          |                           |                   |
|:--------------------|---------------------------|------------------:|
| {{< prev-page >}}   | [Up]({{< relref "./" >}}) | {{< next-page >}} |