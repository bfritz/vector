---
id: splunk_hec_to_clickhouse
title: Writing Splunk HEC Events to Clickhouse
description: Learn how to send Splunk HEC events to Clickhouse with optional enrichments.
keywords: ["splunk hec","clickhouse"]
---

In this guide we'll be consuming `splunk_hec` logs and
writing them to a `clickhouse` sink.

<!--truncate-->

<!--
      THIS FILE IS AUTOGENERATED!

      To make changes please edit the template located at: scripts/generate/templates/guides/guide.md.erb
-->

## Setup

If you haven't already, install Vector. Here's a script for the lazy:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.vector.dev | sh
```

Or [choose your preferred installation method][docs.installation].

## Configure a Source

Next, create a config file in a local directory (called `config.toml`) and add a
`splunk_hec` source by pasting in this snippet:

```toml
[sources.my-source-id]
  # REQUIRED
  type = "splunk_hec" # must be: "splunk_hec"

  # OPTIONAL
  address = "0.0.0.0:8088" # default
  token = "A94A8FE5CCB19BA61C4C08" # example, no default
```

## (Optional) Parse Events

If our logs consumed from Splunk HEC are structured then we should
parse them. We can do that with a range of [transforms][docs.transforms], in
this example we will use the `json_parser` transform:

```toml
[transforms.parser]
  # REQUIRED
  type = "json_parser" # must be: "json_parser"
  inputs = ["my-source-id"] # example
  drop_invalid = true # example

  # OPTIONAL
  drop_field = true # default
  field = "message" # default
```

Note that for the `inputs` field we specify our `splunk_hec` source by
its name `my-source-id`.

This step is optional, if you choose to skip it then remember to set the
`inputs` of the next component to the previous component in the topology
(`my-source-id`).

## (Optional) Enrich Events

We can also choose to enrich our events with [transforms][docs.transforms]. In
this example we're going to add a `aws_ec2_metadata` transform:

```toml
[transforms.enricher]
  # REQUIRED
  type = "aws_ec2_metadata" # must be: "aws_ec2_metadata"
  inputs = ["parser"] # example

  # OPTIONAL
  fields = ["instance-id", "local-hostname", "local-ipv4", "public-hostname", "public-ipv4", "ami-id", "availability-zone", "vpc-id", "subnet-id", "region"] # default
  host = "http://169.254.169.254" # default
  namespace = "" # default
  refresh_interval_secs = 10 # default
```

Note that for the `inputs` field we specify `parser`.

This step is optional, if you choose to skip it then remember to set the
`inputs` of the next component to the previous component in the topology
(`parser`).

## Configure a Sink

Now we configure our sink, making sure to set the input to `enricher`:

```toml
[sinks.my-sink-id]
  # REQUIRED
  type = "clickhouse" # must be: "clickhouse"
  inputs = ["enricher"] # example
  host = "http://localhost:8123" # example
  table = "mytable" # example

  # OPTIONAL
  database = "mydatabase" # example, no default
```

## Run It

That's it, we're ready to execute our pipeline. You can run it locally with:

```sh
vector -c ./config.toml
```

Vector is very flexible and can be deployed in a way that suits your target
environment. For guidance check out our [deployment documentation][docs.deployment].


[docs.deployment]: /docs/setup/deployment/
[docs.installation]: /docs/setup/installation/
[docs.transforms]: /docs/reference/transforms/