---
layout: post
title:  "Open Source APM Adventure - Part 3"
excerpt: >
  The landscape
date:   2018-10-22 17:05:06 +0200
categories:
- APM
- distributed tracing
- ops
---

# Open Source APM Adventure - Part 3

## The landscape

Whether you wanna call it observability or monitoring, or care most about
specific metrics or want more general tracing, I think the fact that they're
all related, and they all have value, means I'd like to solve them all in a way
that's easy to integrate with whatever languages we're using.

A couple times I've put some effort into researching what'd it take to put
together some existing observability and performance monitoring tools, to get
a similar experience to what you get with paid propietary solutions.

The open source landscape has progressed a lot in the last few years, when it
comes to a lot of the more fancy features you expect from an out of the box
solution. Census and Elastic APM are some of the projects I'm heard people
talk about the most, and are some of the ones I'm most excited about, so
this series will hopefully cover some of what they offer.

First I'll talk about what I'd like to get from an open source project or set
of projects that promises to make life easier for me as an application
developer. Then we can get into the interesting *Why*, and the most challening
*How*.

## What I'd Like To See

### Logging

This is probably the closest to a solved problem, when it comes to existing
solutions. Fluentd, logstash, and Elastic's products going into ElasticSearch
is the most common solution I've witnessed, and heard from people. Otherwise
people are probably using something that's provided by their cloud provider or
another SaaS, like Loggly or Splunk.

### Metrics

Most places I've worked, and people I've spoken to, either use Graphite or
Prometheus or both. They'll probably make an apperance later. Though if you're
going all in on Elastic's products, I assume you'll need to store all your
logs, but then you could probably aggregate them there.

### Distributed Tracing

Zipkin is the most stable and mature open source project when it comes to
distributed tracing, but there are a few more modern projects.

In the propitary space, there are a bunch of awesome SaaS products like
LightStep and HoneyComb, which offer really simple SDKs to easily integrate
most programming languages, and easily get everything you need SDK wise.

### Alerting

Prometheus's AlertManger and Grafana's built in alerting are both possible
solutions in this space. Elastic seems to have some alerting support too.

Potentially these go into PagerDuty or OpsGenie.

## Why?

Whether it's the load balancer, nginx, your application code, or the database,
you probably wanna know when something's going wrong and where. Logs, metrics,
and distributed tracing, all offer different valuable features.

Once your site starts getting a decent amount of traffic, or if you're
trying to keep your storage costs down. You're probably going to want to keep
logs and traces for a shorter period of time than metrics. However your metrics
will probably be aggregated in a way that keeps costs much cheaper for storing
longer history.

SaaS and cloud provider solutions like New Relic and StackDriver offer a lot of
out of the box features that require a lot more effort to get out of free
solutions.

## How?

## Links

https://www.honeycomb.io/blog/lies-my-parents-told-me-about-logs/
