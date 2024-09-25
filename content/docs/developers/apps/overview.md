---
weight: 311
title: "Overview"
description: "A quick overview on the way Apps work within Home Cloud"
icon: "article"
draft: false
toc: true
---

## What is a Home Cloud app?

Fundamentally a Home Cloud app is no different than any other hostable service. As long as an application can be delivered as a Docker container it can be hosted by Home Cloud. What's special about Home Cloud is that because the platform is based on [Kubernetes](https://kubernetes.io/) it can provide many application dependencies without the app developer having to worry about deploying or maintaining them.

For example, if your app requires a PostgreSQL database, a reverse proxy for its web API, and some persistent storage all you need to do is ask for these resources from Home Cloud and they will be automatically provisioned for your app during installation! This means you can focus more time on building and improving your app and less time on keeping miscelaneous dependencies up to date.

## Digging deeper

We mentioned earlier that Home Cloud is build on top of [Kubernetes](https://kubernetes.io/), but don't worry, you don't need to know Kubernetes to release your app on Home Cloud. Home Cloud apps are configured as [Helm Charts](https://helm.sh/) and Home Cloud itself provides a simple API defined in the chart to create common resources like databases, routing rules, and persistent storage.

Let's try it out with [an example]({{% relref "/docs/developers/apps/example" %}})!
