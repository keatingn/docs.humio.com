---
title: "Docker"
---

There are two key steps to getting logs from Docker containers into Humio:

1. Shipping the container logs to Humio
2. Parsing the logs

{{% notice note %}}
In this guide, we assume that you use Docker in the standard way, where
logs are captured from `stdout` and `stderr`.
{{% /notice %}}

## 1. Shipping container logs to Humio

The easiest way to get logs from Docker containers is using the
[`docker2humio`](https://hub.docker.com/r/pmech/docker2humio/)
container.

With `docker2humio`, you configure and run a shipper container on each
Docker host. Then, you hook up all the containers for which you want
logs using the fluentd log-driver.

{{% notice tip %}}
***Log Types***

You should set the log types for your containers so Humio can parse the logs.

Humio can accept logs even when it does not know their type. So just start sending logs to Humio, and then create and enhance the relevant parsers afterwards.
{{% /notice %}}

Go to the [`docker2humio` container page](https://hub.docker.com/r/pmech/docker2humio/)
for further documentation on running the container.


## 2. Parsing the logs

Since Docker just handles log lines from `stdout` as text blobs, you must parse
the lines to get the full value from them.

To do this, you can either use a built-in parser, or create new ones for your log
types.  For more details on creating parsers, see the [parsing page](/sending_logs_to_humio/parsers/parsing/).

{{% notice tip %}}
In terms of log management, Docker is just a transport layer.

Before writing a custom parser, see the [built in parsers](/sending_logs_to_humio/parsers/built_in_parsers/) page to see if Humio already supports your log type.
{{% /notice %}}

## 3. Metrics

To get standard host level metrics for your docker containers, use
[Metricbeat](https://www.elastic.co/guide/en/beats/metricbeat/current/index.html).
It includes a [docker
module](https://www.elastic.co/guide/en/beats/metricbeat/current/metricbeat-module-docker.html).

### Example Metricbeat Configuration

``` yaml
metricbeat.modules:
  - module: docker
    metricsets: ["cpu", "info", "memory", "network", "diskio", "container"]
    hosts: ["unix:///var/run/docker.sock"]
    enabled: true
    period: 10s

output.elasticsearch:
  hosts: ["https://<humio-host>:443/api/v1/dataspaces/<dataspace>/ingest/elasticsearch"]
  username: <ingest-token>
```

Where:

* `<humio-host>` - is the name of your Humio server
* `<dataspace>` - is the name of your dataspace on your server
* `<ingest-token>` - is the [ingest token](/sending_logs_to_humio/ingest_tokens/) for your dataspace

See also the page on [Beats](/sending_logs_to_humio/log_shippers/beats/) for more
information.
