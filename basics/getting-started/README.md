---
description: >-
  This section contains quick start guides to help you get up and running with
  Pinot.
---

# Getting Started

## Running Pinot

We want your experience getting started with Pinot to be both low effort and high reward. Here you'll find a collection of quick start guides that contain starter distributions of the Pinot platform.

### Bootstrapping a cluster

{% content-ref url="running-pinot-locally.md" %}
[running-pinot-locally.md](running-pinot-locally.md)
{% endcontent-ref %}

{% content-ref url="running-pinot-in-docker.md" %}
[running-pinot-in-docker.md](running-pinot-in-docker.md)
{% endcontent-ref %}

{% content-ref url="kubernetes-quickstart.md" %}
[kubernetes-quickstart.md](kubernetes-quickstart.md)
{% endcontent-ref %}

### Deploy to a public cloud

{% content-ref url="public-cloud-examples/azure-quickstart.md" %}
[azure-quickstart.md](public-cloud-examples/azure-quickstart.md)
{% endcontent-ref %}

{% content-ref url="public-cloud-examples/gcp-quickstart.md" %}
[gcp-quickstart.md](public-cloud-examples/gcp-quickstart.md)
{% endcontent-ref %}

{% content-ref url="public-cloud-examples/aws-quickstart.md" %}
[aws-quickstart.md](public-cloud-examples/aws-quickstart.md)
{% endcontent-ref %}

## How to setup a Pinot cluster

This video will show you a step-by-step walk through for launching the individual components of Pinot and scaling them to multiple instances. This is an excellent resource for developers and operators that want to understand setting up each component and debugging a cluster.

{% hint style="info" %}
You can find the commands that are shown in this video on GitHub\
[https://github.com/npawar/pinot-tutorial](https://github.com/npawar/pinot-tutorial)
{% endhint %}

{% embed url="https://www.youtube.com/watch?v=cNnwMF0pOJ8" %}
Neha Pawar from the Apache Pinot team shows you how to setup a Pinot cluster
{% endembed %}

We also have a step-by-step guide for manually setting up a Pinot cluster using Docker or shell scripts.

{% content-ref url="advanced-pinot-setup.md" %}
[advanced-pinot-setup.md](advanced-pinot-setup.md)
{% endcontent-ref %}

## Data import examples

Getting data into Pinot is easy. Take a look at these two quick start guides which will help you get up and running with sample data for offline and real-time [tables](../components/table.md).

{% content-ref url="pushing-your-data-to-pinot.md" %}
[pushing-your-data-to-pinot.md](pushing-your-data-to-pinot.md)
{% endcontent-ref %}

{% content-ref url="pushing-your-streaming-data-to-pinot.md" %}
[pushing-your-streaming-data-to-pinot.md](pushing-your-streaming-data-to-pinot.md)
{% endcontent-ref %}
