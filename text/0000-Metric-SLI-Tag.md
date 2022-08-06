# Add Metrics Metadata tag definition

Add a standard metadata tag to Metric data structure.

## Motivation

OpenTelemetry provides a standard data format and model to process telemetry data with different types of backends. However, in the current standard format there is no existing space for adding metadata. By adding metadata tag to metrics, we can get benefits from the following scenarios:

### Data Analysis and Consumption

To achieve end-to-end data analysis, users need to manually write some configurations to specify how to transform and consume specific types of metrics data. We could leverage the metadata tag to provide an out-of-box solution which allows users to attach essential information when they are instrumenting. In details, users could attach a `Latency` tag to a metric which denotes this is a latency related metric. And in the consuming pipeline, we could read this tag and automatically compute the latency statistics.

### Data Management

There will be countless telemetry data generated every moment and how to automatically manage the data is also a challenge. By attaching metadata tag to the telemetry data(metrics in this case), we can allow users to specify the visibility, life-span, and other information about the telemetry data ahead of time.

## Explanation

From a user perspective, giving the ability of adding metadata to metrics will be much helpful when they construct data pipeline, do data analysis, and data management.

For instance, users could register a metric with some sli definition which could help user to consume metric data easily in the following pipeline:

```cs
metricsBuilder.AddMeter("SLIMeter", {
    {"Category", "Latency"},
    {"Window", "PT5M"},
    {"Percentile", "999"},
});
```

In this use case, users could attach SLI definition(Category, Observation Window, and Percentile) to the metric data. And in the later stage, users could utilize those information to construct a data pipeline.

Apart from SLI definition, user could also utilize the metadata to manage the data policy and data governance of telemetry data. For instance, users could use the following code:

```cs
metricsBuilder.AddMeter("MeterName", {
    {"Visibility", "Public"},
    {"LifeSpan", "Permanent"},
});
```

With the above metadata attached in the metric data, users could set data policy at the instrumenting stage and directly connect OpenTelemetry to the data management system.

## Internal details

Implementations will first require changes on the `OpenTelemetry Proto`. In this level, we will need to add a set of new field to describe the metadata tag.

Then, we need to update the Otel Proto at the collector and sdk side to support the new field.

At last, we will need to add a set of new entry at the `Metric` class level at SDK and adding new APIs to support setting those fields.

## Trade-offs and mitigations

We only introduce new tags in the OpenTelemetry data, which will potentially increase the size of transmitted data. However, this increase will be trivial compared to the size of the data.

## Prior art and alternatives

One alternative way is to add those metadata tags as a set of record when users are instrumenting. This is a workable solution with the following trade-offs:

- Hard to distinguish between the metadata tags and the actual metric data.
- No high level abstraction means users need to repeat the logics over and over again.

## Open questions

N/A

## Future possibilities

Apply this idea to Traces and Logs as well.
