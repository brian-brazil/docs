---
title: Examples
sort_rank: 4
---

# Query Examples

## Simple time series selection

Return all time series with the metric `http_requests_total`:

    http_requests_total

Return all time series with the metric `http_requests_total` and the given
`job` and `handler` labels:

    http_requests_total{job="apiserver", handler="/api/comments"}

Return a whole range of time (in this case 5 minutes) for the same vector,
making it a range vector:

    http_requests_total{job="apiserver", handler="/api/comments"}[5m]

Note that an expression resulting in a range vector cannot be graphed directly,
but viewed in the tabular view of the expression browser.

Using regular expressions, you could select time series only for jobs whose
name match a certain pattern, in this case, all jobs that end with `server`.
Note that this does a substring match, not a full string match:

    http_requests_total{job=~"server$"}

To select all HTTP status codes except 4xx ones, you could run:

    http_requests_total{status!~"^4..$"}

## Using Functions, Operators, etc.

Return the per-second rate for all time series with the `http_requests_total`
metric name, as measured over the last 5 minutes:

    rate(http_requests_total[5m])

Assuming that the `http_requests_total` time series all have the labels `job`
(fanout by job name) and `instance` (fanout by instance of the job), we might
want to sum over the rate of all instances, so we get fewer output time series,
but still preserve the `job` dimension:

    sum by (job)(rate(http_requests_total[5m]))

If we have two different metrics with the same dimensional labels, we can apply
binary operators to them and elements on both sides with the same label set
will get matched and propagated to the output. For example, this expression
returns the unused memory in MB for every instance (on a fictional cluster
scheduler exposing these metrics about the instances it runs):

    (instance_memory_limit_bytes - instance_memory_usage_bytes) / 1024 / 1024

The same expression, but summed by application, could be written like this:

    sum by (app, proc)(
      instance_memory_limit_bytes - instance_memory_usage_bytes
    ) / 1024 / 1024

If the same fictional cluster scheduler exposed CPU usage metrics like the
following for every instance:

    instance_cpu_time_ns{app="lion", proc="web", rev="34d0f99", env="prod", job="cluster-manager"}
    instance_cpu_time_ns{app="elephant", proc="worker", rev="34d0f99", env="prod", job="cluster-manager"}
    instance_cpu_time_ns{app="turtle", proc="api", rev="4d3a513", env="prod", job="cluster-manager"}
    instance_cpu_time_ns{app="fox", proc="widget", rev="4d3a513", env="prod", job="cluster-manager"}
    ...

...we could get the top 3 CPU users grouped by application (`app`) and process
type (`proctype`) like this:

    topk(3, sum by (app, proc)(rate(instance_cpu_time_ns[5m])))

Assuming this metric contains one time series per running instance, you could
count the number of running instances per application like this:

    count by (app)(instance_cpu_time_ns)
