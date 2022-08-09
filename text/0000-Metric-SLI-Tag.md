# Enable adding Metrics Metadata definition

This proposal is to enable capability in OTel APIs to allow adding metadata while adding a metrics definition.

## Motivation

OpenTelemetry provides API specifications to build language specific SDK to instrument telemetry data. It also have OTel Collector concepts to process n publish telemetry data to multiple different Observability backends. However, current specifications lacks the ability to define additional context while adding a metrics definition. This additional metrics context will be very useful for Telemetry Data Management and Analytics.

### Data Management

We're leveraging OTel .Net SDK to instrument telemetry data specifically for metrics data across many services in Azure. And then publishing this data to one or more Observability backends. Given the data volumn and scale, it's being very challenging for us to manage data access or policies. Any simple change like adding/updating data visibility or life-span becomes very time consuming and tedious process. Same goes for privacy and compliance policies.

This is a generic problem, there are countless telemetry data generated every moment and how to automatically manage this data is a big challenge. By adding additional context to the telemetry data (metrics in this case), we can allow users to specify the visibility, life-span, and other information about the telemetry data ahead of time. Users will also be able to specify privacy and compliance tags to indicate what telemetry data is being emitted and how to handle it.

### Data Analysis and Consumption
We also faced significant challenges to consume our telemetry data for analytics purpose due to lack of knowing how to consume this data. Since data authors and data consumers can and mostly are different personas so consumers always needs some guidance to be able to consume it.

To achieve end-to-end data analysis, data authors usually write some additional tool or service to specify how to transform and consume specific types of telemetry metrics data. We could leverage the metrics metadata (added as additional metrics context) to provide an out-of-box solution for analytics which allows users to attach essential information when they are instrumenting. For example, some service health analytics scenarios can leverage metrics metadata to figure out if a given metrics is SLI metrics or not and if it's then what SLI category like 'Availability' or 'Latency' it represents.

## Explanation

From a user perspective, giving the ability of adding metadata to metrics will be much helpful when they construct data pipeline, do data analysis, and data management.

For instance, users could register a metric with some sli definition which could help user to consume metric data easily for analytics purpose:

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

One alternative way is to add those additional metrics context as a set of record when users are instrumenting. This is a workable solution with the following trade-offs:

- Hard to distinguish between the metrics metadata and the actual metric data.
- No high level abstraction means users need to repeat the logics over and over again.

## Open questions

N/A

## Future possibilities

Apply this idea to Traces and Logs as well.
