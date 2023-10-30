# First component(s) and the Standard Library

_This section covers the basics of the River language and the standard library. It then introduces a basic pipeline that collects metrics from the host and sends them to Prometheus._

## River basics

### Recommended Reading

- [Flow Configuration Language](https://grafana.com/docs/agent/latest/flow/config-language/)
- [Grafana Agent Flow Configuration Language](https://grafana.com/docs/agent/latest/flow/concepts/configuration_language/)

[River](https://github.com/grafana/river) is an HCL-inspired configuration language, and is what is used to configure the Grafana Agent in Flow mode. A River file is mainly comprised of three things:

1. **Attributes**: `key = value` pairs used to configure individual settings.

    ```river
    url = "http://localhost:9090"
    ```

2. **Expressions**: Expressions are used to compute values. They can be constant values (e.g. `"localhost:9090"`) or they can be more complex (e.g. referencing a component's export: `prometheus.exporter.unix.targets`, a mathematical expression: `(1 + 2) * 3`, or a standard library function call: `env("HOME")`). We will cover more available expressions later.

3. **Blocks**: Blocks are used to configure components with groups of attributes or nested blocks. The following example block can be used to configure the logging output of a Grafana Agent in Flow mode:

    ```river
    logging {
        level  = "debug"
        format = "json"
    }
    ```

    **Note:** The default log level is `info` and the default log format is `logfmt`.

    Try pasting this into `config.river` and running `/path/to/agent run config.river` to see what happens!

    Congratulations, you've just written your first River file! You've also just written your first Grafana Agent Flow configuration file. This configuration won't do anything, so let's add some components to it.

**Pro tip**: _Comments in River are prefixed with `//` and are single-line only._

```river
// This is a comment
```

## Components

### Recommended Reading

- [Components](https://grafana.com/docs/agent/latest/flow/concepts/components/)
- [Component Controller](https://grafana.com/docs/agent/latest/flow/concepts/component_controller/)

Components are the building blocks of a Grafana Agent Flow configuration. They are configured and linked together in order to create pipelines that collects, processes, and outputs your telemetry data. Components are configured with `Arguments` and have `Exports` that may be referenced by other components.

Let's look at a simple example pipeline:

```river
local.file "example" {
    path = env("HOME") + "secret.txt"
}

prometheus.remote_write "local_prom" {
    endpoint {
        url = "http://localhost:9090/api/v1/write"

        basic_auth {
            username = "admin"
            password = local.file.example.content
        }
    }
}
```

**Pro tip**: _A list of all available components can be found in the [Component Reference](https://grafana.com/docs/agent/latest/flow/reference/components/). Each component has a link to its documentation, which contains a description of what the component does, its arguments, its exports, and Example(s)._

This pipeline has two components: `local.file` and `prometheus.remote_write`. The `local.file` component is configured with a single argument, `path`, which is set by calling the [env](https://grafana.com/docs/agent/latest/flow/reference/stdlib/env/) standard library function to retrieve the value of the `HOME` environment variable and concatenating it with the string `"secret.txt"`. The `local.file` component has a single export, `content`, which contains the contents of the file.

The `prometheus.remote_write` component is configured with an `endpoint` block, which contains the `url` attribute and a `basic_auth` block. The `url` attribute is set to the URL of the Prometheus remote write endpoint. The `basic_auth` block contains the `username` and `password` attributes, which are set to the string `"admin"` and the `content` export of the `local.file` component, respectively. Note that the `content` export is referenced by using the syntax `local.file.example.content`, where `local.file.example` is the fully qualified name of the component (the component's type + it's label) and `content` is the name of the export.

**Pro tip**: _The `local.file` component's label is set to `"example"`, so the fully qualified name of the component is `local.file.example`. The `prometheus.remote_write` component's label is set to `"local_prom"`, so the fully qualified name of the component is `prometheus.remote_write.local_prom`._

This example pipeline still doesn't do anything, so let's add some more components to it.

## Shipping our first metrics

### Recommended Reading

- Optional: [prometheus.exporter.unix](https://grafana.com/docs/agent/latest/flow/reference/components/prometheus.exporter.unix/)
- Optional: [prometheus.scrape](https://grafana.com/docs/agent/latest/flow/reference/components/prometheus.scrape/)
- Optional: [prometheus.remote_write](https://grafana.com/docs/agent/latest/flow/reference/components/prometheus.remote_write/)

Let's make a simple pipeline with a `prometheus.exporter.unix` component, a `prometheus.scrape` component to scrape it, and a `prometheus.remote_write` component to send the scraped metrics to Prometheus.

```river
prometheus.exporter.unix "localhost" { 
    // This component exposes a lot of metrics by default, so we will keep all of the default arguments.
}

prometheus.scrape "default" {
    // Setting the scrape interval lower to make it faster to be able to see the metrics
    scrape_interval = "10s"

    targets    = prometheus.exporter.unix.localhost.targets
    forward_to = [
        prometheus.remote_write.local_prom.receiver,
    ]
}

prometheus.remote_write "local_prom" {
    endpoint {
        url = "http://localhost:9090/api/v1/write"
    }
}
```

Run the agent with:

```bash
/path/to/agent run config.river
```

And navigate to [http://localhost:3000/explore](http://localhost:3000/explore) in your browser. After ~15-20 seconds you should be able to see the metrics from the `prometheus.exporter.unix` component! Try querying for `node_memory_Active_bytes` to see the active memory of your host.

![Memory usage](mem_usage.png)

## Exercise for the reader

### Recommended Reading

- Optional: [prometheus.exporter.redis](https://grafana.com/docs/agent/latest/flow/reference/components/prometheus.exporter.redis/)

Let's start a container running Redis and configure the agent to scrape metrics from it.

```bash
docker container run -d --name flow-redis -p 6379:6379 redis
```

Try modifying the above pipeline to scrape metrics from the Redis exporter. You can find the documentation for the `prometheus.exporter.redis` component [here](https://grafana.com/docs/agent/latest/flow/reference/components/prometheus.exporter.redis/).

If you get stuck, you can find the solution in [full-config.river](full-config.river).

## Finishing up and next steps

You might have noticed that running the agent with the above configs created a directory called `data-agent` in the current directory. This directory is where components can store data, such as the `prometheus.exporter.unix` component storing its WAL (Write Ahead Log). If you look in the directory, do you notice anything interesting? The directory for each component is the fully-qualified name!

If you'd like to store the data elsewhere, you can specify a different directory by supplying the `--storage.path` flag to the agent's run command, e.g. `/path/to/agent run config.river --storage.path /etc/grafana-agent`. Generally you will want to use a persistent directory for this, as some components may use the data stored in this directory to perform their function.

In the next section we will look at how to configure the agent to collect logs from a file and send them to Loki. We will also look at using different components to process metrics and logs before sending them.
