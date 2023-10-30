# flow-by-example

## Who is this for?

This repository contains a collection of examples that build on each other to demonstrate how to configure and use the [Grafana Agent](https://grafana.com/docs/agent/latest/) in [Flow mode](https://grafana.com/docs/agent/latest/flow/). It assumes you have a basic understanding of what the Agent is and telemetry collection in general. It also assumes a base level of familiarity with Prometheus and PromQL, Loki and LogQL, and basic Grafana navigation. It assumes no knowledge of Flow or River concepts.

## What is Flow?

Flow is a new way to configure the Grafana Agent. It is a declarative configuration language that allows you to define a pipeline of telemetry collection, processing, and output. It is built on top of the [River](https://github.com/grafana/river) configuration language, which is designed to be fast, simple, and debuggable.

## What do I need to get started?

You will need a Linux or Unix environment with Docker installed. The examples are designed to be run on a single host, so you can run them on your laptop or in a VM. You are encouraged to follow along with the examples using a `config.river` file and experiment with the examples yourself.

To run the examples, you should have a Grafana Agent binary available. You can follow the instructions on how to [Install Grafana Agent as a Standalone Binary](https://grafana.com/docs/agent/latest/flow/setup/install/binary/#install-grafana-agent-in-flow-mode-as-a-standalone-binary) to get a binary.

## How should I follow along?

The `docker-compose.yml` file in the root of this repository will start Prometheus, Loki, and Grafana containers to use with the examples. You can start these containers with `docker compose up` and stop them with `docker compose down`. The examples are designed to work with the default configuration of these containers, so you should not need to modify them.

After running `docker-compose up`, open [http://localhost:3000](http://localhost:3000) in your browser to view the Grafana UI.

The examples in this repository are designed to be followed in order. Examples generally build on each other, and each example contains a README that explains what it does and how it works. The examples are designed to be run locally, so you can follow along and experiment with them yourself.

Each example's README will contain a list of Recommended Reading. These are links to documentation that will help you understand the concepts used in the example, and should be read in order. Some information may be included in the README as well, but the Recommended Reading will provide more detail.
