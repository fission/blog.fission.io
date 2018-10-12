---
title: "Using Fission's Prometheus Metrics"
date: 2018-10-06T01:46:30-07:00
draft: false
---
intro blurb

### Prometheus Introduction

Prometheus is a popular tool for aggregating, storing, and querying
metrics.  There are a few crucial concepts to learn with Prometheus.

The Prometheus [Data
model](https://prometheus.io/docs/concepts/data_model/) is built
around time series.  Each time series has a name and a set of
key-value pairs (called labels).  You can query time series by these
labels.

Prometheus operates on a "pull" model -- Fission exposes a set of
metrics, and Prometheus samples these metrics every minute or so.

## Using Fission's Prometheus Metrics

Fission exposes metrics from its components that Prometheus then
scrapes.  Prometheus offers a web-based querying UI that you can use
to query and graph Fission metrics.

Below we'll show a few sample queries for common things you may want
to monitor from your deployment.  First let's make sure your
Prometheus is up and running, and that you can access it.

### Setting up Prometheus

Starting in Fission 0.11, an installation of Prometheus is included by
default with Fission.  With older versions, you can install Prometheus
with `helm install stable/prometheus`.

### Accessing your Prometheus server

The Prometheus server has a web-based UI.  To access it, you'll need
to setup a local port forwarded to the prometheus service on your
cluster:

```bash

# Forward local port 9090 to the Prometheus server pod
$ kubectl -n fission port-forward $(kubectl -n fission get pod -l app=prometheus,component=server -o name) 9090

```

To check that this works, visit http://localhost:9090 in your
browser.  You should see a page with an empty query window.

Now, let's see what kind of metrics we can query!

### Number of function invocations, and their rate

For a start, try querying the number of function calls, entering into
the query box:

```
fission_function_calls_total
```

If this query returns no data points, make sure you have at least one
function and have invoked it at least once.

Click on the `Graph` tab.  You'll see one or more line graphs showing
the number of invocations.

You can narrow down this query further, for example by function name:

```
fission_function_calls_total{name="hello"}
```

This will give you metrics for a function named `hello`.  There still
may be multiple lines, for example from different router instances.
You can calculate the sum of all these using a `sum` query:

```
sum(fission_function_calls_total{name="hello"}) without (cached,job,instance,code,kubernetes_pod_name,pod_template_hash,path,method,application,svc)
```

This somewhat long query sums over all of the labels that we don't
want to distinguish between.

Note that this metric is a counter.  It's the total number of function
calls, ever.  That means it only ever goes up, never down.  To find
the rate of function calls, you can use the `rate` function:

```
rate(fission_function_calls_total{name="func-v1"}[5m]) 
```

### Function success/error rate

The previous section showed the total number of function calls.  We
also count the number of function calls that resulted in an error.
For example, here is the function error metric for a function named
"hello":

```
fission_function_errors_total{name="hello"}
```

### Function duration percentiles

Fission also tracks how long your functions took to execute.  You can
query these by quantiles: 50%le (i.e. the median), 90th percentile,
and 99-th percentile.

For example, the 90-th percentile duration in seconds for a function
"hello":

```
fission_function_duration_seconds{name="hello",quantile="0.9"}
```

Note that this will return two results, one with `cached="true"` and
the other with `cached="false"`.  The result with `cached="false"`
measures the 90%le cold-start time.  The result with `cache="true"`
shows the function duration when the function was already running.
This is a very close measurement of your function's execution time,
since Fission's overheads in this case are close to zero.

## Alerting

Prometheus comes with an alerting service called AlertManager.  You
can configure Alertmanager to generate alerts for various conditions.
For example, if a function's rate of errors goes beyond a certain
threshold, you could send a slack message or a pagerduty page.

For more on this, consult the [Alertmanager
documentation](https://prometheus.io/docs/alerting/alertmanager/).

## Dashboards

You can use Grafana or other tools for creating dashboards from
Prometheus data.  Install Grafana on your cluster, and follow
instructions to add a Prometheus data source.  You can then add graphs
based on Prometheus queries.

## Conclusion

Fission's Prometheus metrics integration allows you to query the
function call rate, error rate and duration quantiles without writing
any Prometheus integration yourself.  Using Prometheus' Alertmanager
service, you can also set up alerting for cases where you need it.

Check out the [Fission installation guide](https://docs.fission.io/installation/) to get started with
Fission.  Join us on the [Fission Slack](http://slack.fission.io) to
chat, or follow us on Twitter at @fissionio.
