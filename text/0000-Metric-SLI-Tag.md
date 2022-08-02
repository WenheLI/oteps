# Add Metrics Metadata tag definition

Add a standard metadata tag to Metric data structure.

## Motivation

By adding metadata tag to metrics, we can get benefits from the following scenarios:

### End to End Metrics data consuming

To enable an end-to-end metrics consuming and computation pipeline, user needs to manually write some configurations to specify how to transform and consume specific types of metrics data. We could leverage the metadata tag to provide an out-of-box solution which allows users to attach essential information when they are instrumenting. In details, users could attach a `Latency` tag to a metric which denotes this is a latency related metric. And in the consuming pipeline, we could read this tag and automatically compute the latency statistics.

### Data Management

There will be countless telemetry data generated every moment and how to automatically manage the data is also a challenge. By attaching metadata tag to the telemetry data(metrics in this case), we can allow users to specify the visibility, life-span, and other information about the telemetry data ahead of time.

## Explanation

From a user perspective, giving the ability of adding metadata as a tag to metrics will be much helpful when they construct data pipeline. For instance, users could register a metric with some sli definition:
```csharp
metricsBuilder.AddMeter("MeterName", {
    {"Category", "Latency"},
    {"Visibility", "Public"},
    {"LifeSpan", "Permanent"},
})
```
And then those metadata will be attached to the metric data and sent to the data pipeline. In data pipeline, it can read those metadata and react with correct behavior with the given metadata.

## Internal details

Implementations will first require changes on the `OpenTelemetry Proto`. In this repo, we will need to add a set of new field to describe the metadata tag.

Then, we need to update the Otel Proto at the collector and sdk side to support the new field.

At last, we will need to add a set of new entry at the `Metric` class level at SDK and adding new APIs to support setting those fields.

## Trade-offs and mitigations

We only introduce new tags in the OpenTelemetry data, which will potentially increase the size of transmitted data. However, this increase will be trivial compared to the size of the data.

## Prior art and alternatives

N/A

## Open questions

N/A

## Future possibilities

Apply this idea to Traces and Logs as well.
