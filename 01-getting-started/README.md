# 1. Getting started

Open a new terminal window, go to this directory and run

```
prometheus
```

You will see something like this:

```
level=info ts=2018-01-08T09:02:06.815824474Z caller=main.go:215 msg="Starting Prometheus" version="(version=2.0.0, branch=non-git, revision=non-git)"
level=info ts=2018-01-08T09:02:06.815920703Z caller=main.go:216 build_context="(go=go1.9.2, user=brew@Sierra-2.local, date=20171111-16:10:49)"
level=info ts=2018-01-08T09:02:06.815949333Z caller=main.go:217 host_details=(darwin)
level=info ts=2018-01-08T09:02:06.818468303Z caller=main.go:314 msg="Starting TSDB"
level=info ts=2018-01-08T09:02:06.818475681Z caller=web.go:380 component=web msg="Start listening for connections" address=0.0.0.0:9091
level=info ts=2018-01-08T09:02:06.818487984Z caller=targetmanager.go:71 component="target manager" msg="Starting target manager..."
level=info ts=2018-01-08T09:02:06.828397549Z caller=main.go:326 msg="TSDB started"
level=info ts=2018-01-08T09:02:06.828457152Z caller=main.go:394 msg="Loading configuration file" filename=prometheus.yml
level=info ts=2018-01-08T09:02:06.828600133Z caller=main.go:371 msg="Server is ready to receive requests."
```

Prometheus automatically looks for a configuration file `prometheus.yml` in the current directory and creates a `data` directory in the same place for its time series database.

If your config file has a different name or location, run Prometheus with the `--config.file=<filename>` instead.

Open up your browser at `http://localhost:9090` for the Prometheus web interface:

![Prometheus main query interface](./_images/prometheus_graph.png)

This is the main page where we can query information from the time series database in the next chapters.


> ### What Prometheus doesn't include
>
> The Prometheus team focuses on what's most important: Providing a highly performant and reliable way for metrics collection. So they avoid bundling stuff  that has been already solved for ages by other tools or which highly depends on your specific environment:
>
> - No fancy interface (we'll come to that in [chapter 6](../06-dashboards/))
> - Default protocol for the web-interface is HTTP (HTTPS is supported in Prometheus, node_exporter, etc. but not in all exporters. You can use a reverse proxy like HAProxy, nginx, etc. instead.)
> - No authentication/authorization (use OAuth, LDAP, etc.)


For now, let's head to _Status > Configuration_. This is the most basic config (and some defaults) we've just loaded:

![Prometheus configuration](./_images/prometheus_config.png)

By default, Prometheus will check each target every minute for new data (`scrape_interval`).

For a better target overview, go to _Status > Targets_:

![Prometheus scraping targets](./_images/prometheus_targets.png)

So there is one target called 'prometheus', the data is scraped from `http://localhost:9090/metrics`, successfully last time 2.37 seconds ago. The simple metric `up` is a quick indicator to check if a scraping target is available.

I encourage you to look at the raw metrics format exposed at the URL above, because that's really the cool thing about Prometheus: **Every system or application** which can expose a website with this plain text metric format can be processed by Prometheus, **no matter which programming language is used** to produce the result.

There is already a long list of [exporters](https://prometheus.io/docs/instrumenting/exporters/) available for a wide variety of systems. Also [client libraries](https://prometheus.io/docs/instrumenting/clientlibs/) for over 15 popular programming languages (and you can even write your own).


## Metric types

There are four types of metrics Prometheus supports:

- **Counter**: A simple numeric value that only goes up (monotonically increasing) and is set to zero on restart. Used for metrics like number of requests served, tasks completed, or errors.
- **Gauge**: Similar to a counter, but can increase in decrease in value. For temperatures, current requests served but also for storage, CPU or memory usage. 
- **[Histogram](https://en.wikipedia.org/wiki/Histogram)**: Observations (usually things like request durations or response sizes) counted in configurable buckets. It also provides a sum of all observed values.
- **Summary**: Similar to a histogram, but calculates configurable quantiles. It calculates configurable quantiles over a sliding time window.

There is a great [explanation of histograms and summaries](https://prometheus.io/docs/practices/histograms/) on the Prometheus website.


## What a metric looks like

Let's take the basic `up` metric we've already seen above in more detail:

```

metric name                                      value
???                                                ???
up{instance="localhost:9090",job="prometheus"}	 1
   ???
   labels
```

In the following parts, we will use a mixture of metric names, labels and functions to retrieve information from Prometheus.
