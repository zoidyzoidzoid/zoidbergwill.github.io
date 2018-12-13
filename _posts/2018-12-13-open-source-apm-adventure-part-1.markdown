---
layout: post
title:  "Open Source APM Adventure - Part 1"
excerpt: >
  Getting started with Census
date:   2018-10-22 17:05:06 +0200
categories:
- APM
- distributed tracing
- ops
---

# Getting started with Census and Django

So I have a simple Django project [here](https://github.com/zoidbergwill/tracing-example/tree/master/projects/frontend), that I wanna instrument with [Census](https://opencensus.io/), which is an exciting looking project announced by Google in [January 2018](https://opensource.googleblog.com/2018/01/opencensus.html). It's been adopted by other companies, like [Microsoft](https://cloudblogs.microsoft.com/opensource/2018/06/13/microsoft-joins-the-opencensus-project/), since then too.

It promises to solve everything from logs and metrics, to the more rarely solved pieces, distributed tracing. Nike Engineering has a great [post](https://medium.com/nikeengineering/hit-the-ground-running-with-distributed-tracing-core-concepts-ff5ad47c7058) on describing the core concepts of distributed tracing, so I'll just mention a simple overview of distributed tracing and why it's useful. Distributed tracing is a term popularised by Google's [Dapper](https://ai.google/research/pubs/pub36356) paper, where you instrument requests coming into your complex architecture to trace what services it touches during a request, which is useful when debugging complex systems. In our case, we're gonna use one of the oldest and most popular open source implementations of this, called [Zipkin](https://zipkin.io/).

Census's Python client has support for a bunch of frameworks, including Flask and Django, so I started off by just copy'ing what the [README](https://github.com/census-instrumentation/opencensus-python/blob/master/README.rst#django) says.

The usual Django framework integration stuff, adding it to `INSTALLED_APPS` and `MIDDLEWARE_CLASSES`, to make sure it's initialised by Django, and that the middleware is called on each request.

```python
INSTALLED_APPS = [
    # ...
    'opencensus.trace.ext.django',
]

MIDDLEWARE_CLASSES = [
    'opencensus.trace.ext.django.middleware.OpencensusMiddleware',
    # ...
]
```

Then we start getting to the more exciting configurable bits:

```python
OPENCENSUS_TRACE = {
    'SAMPLER': 'opencensus.trace.samplers.probability.ProbabilitySampler',
    'EXPORTER': 'opencensus.trace.exporters.print_exporter.PrintExporter',
    'PROPAGATOR': 'opencensus.trace.propagation.google_cloud_format.GoogleCloudFormatPropagator',
}

OPENCENSUS_TRACE_PARAMS = {
    'BLACKLIST_PATHS': ['/_ah/health'],
    'GCP_EXPORTER_PROJECT': None,
    'SAMPLING_RATE': 0.5,
    'SERVICE_NAME': 'my_service',
    'ZIPKIN_EXPORTER_HOST_NAME': 'localhost',
    'ZIPKIN_EXPORTER_PORT': 9411,
    'ZIPKIN_EXPORTER_PROTOCOL': 'http',
}
```

We don't want the print exporter, because that just prints our distributed traces to a terminal, and we'd like to see them in Zipkin's [UI](https://zipkin.io/). So we're gonna export them to Zipkin, using the same settings as the default example for now.

I don't want people to get scared off thinking this is Google Cloud specific, since I am doing this for a simple example all on my laptop for now, so I am gonna instead use the [W3C trace-context](https://github.com/w3c/trace-context/blob/master/spec/10-overview.md) format for propagating distributed traces. Zipkin and W3C's trace-context both can use HTTP Headers to propagate a trace ID, so requests into other services can continue the same trace, and pass it onto other services.

We're also gonna change a few other things:

- Drop the `GCP_EXPORTER_PROJECT`, since we're not using GCP in this example
- Bump the `SAMPLING_RATE` to 1.0, which configures the probality sampler to sample 100% of the traffic for now, this will allow us to easily drop the amount of traces we create when we ship to production.
- Change our `SERVICE_NAME` to something more descriptive, in this case `frontend`, since we're starting by instrumenting the service that users most frequently are gonna interact with.

```python
OPENCENSUS_TRACE = {
    'SAMPLER': 'opencensus.trace.samplers.probability.ProbabilitySampler',
    'EXPORTER': 'opencensus.trace.exporters.zipkin_exporter.ZipkinExporter',
    'PROPAGATOR': 'opencensus.trace.propagation.trace_context_http_header_format.TraceContextPropagator',
}
```

```python
OPENCENSUS_TRACE_PARAMS = {
    'BLACKLIST_PATHS': ['/_ah/health'],
    'SAMPLING_RATE': 1.0,
    'SERVICE_NAME': 'frontend',
    'ZIPKIN_EXPORTER_HOST_NAME': 'localhost',
    'ZIPKIN_EXPORTER_PORT': 9411,
    'ZIPKIN_EXPORTER_PROTOCOL': 'http',
    'TRANSPORT': 'opencensus.trace.exporters.transports.background_thread.BackgroundThreadTransport',
}
```

```python
from opencensus.trace import config_integration
config_integration.trace_integrations(['requests'])
```

<div class="mermaid">
graph TD
A[Frontend]
</div>

<div class="mermaid">
graph TD
A[Frontend] --> B[Weather]
</div>

<div class="mermaid">
graph TD
A[Frontend] --> B[Weather]
A[Frontend] --> H[Frontend Cache]
A[Frontend] --> C[Frontend DB]
A[Frontend] --> D[Frontend RabbitMQ]
A[Frontend] --> E[Frontend Kafka]

D --> F[Frontend Celery]
G[Frontend Celery Scheduler] --> D
</div>
