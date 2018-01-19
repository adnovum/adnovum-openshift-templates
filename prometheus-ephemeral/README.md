# prometheus-ephemeral - a small Prometheus instance in every OpenShift project

## Motivation

It seems that the currently most favored approach to deploy Prometheus in a
Kubernetes/OpenShift context, is to deploy a big Prometheus instance inside of
the cluster and giving it "cluster-admin" access, so that it can discover and
scrape everything in the cluster.

That might be good to give ops metrics to cluster administrators, but we were
more interested in giving metrics to openshift users that aren't cluster
admins, especially for application metrics. Also, the approach of deploying one
central Prometheus server for the whole cluster has various issues:

- It doesn't scale.

- It is in contradiction with the important principle that you should protect
  different areas of responsibility from high-cardinality issues, by deploying
  separate Prometheus servers for each group of people.

- It's not possible (without a proxy doing some magic) to restrict the access
  of the metrics only to the project users.

- Prometheus needs fast storage to perform well. That's why we prefer to store
  the metrics using a Prometheus instance outside of OpenShift.

So we evaluated the possibility of deploying small Prometheus instances inside
of every project.

## General Concept

The basic idea is to have a Prometheus instance inside of projects, which only
stores data for a couple of hours, and doesn't use persistent storage. Using
Prometheus' federation, the data can then be transfered to a Prometheus outside
of the cluster that has fast storage. It basically means using a
Prometheus instance inside the project as a "proxy" to get access to all
endpoints that are only available from within the OpenShift project network.

This concept makes the deployment of the small Prometheus instance inside of
the projects very simple, because you probably don't need to configure
anything. No need for backup, recording rules, or alert configurations.

On the other hand, with this you solve service-discovery and scraping of all
your applications inside of the OpenShift project. Also, with the help of
blackbox-exporter and kube-state-metrics, you can gather all these metrics:

- Application metrics on pods (discovered with annotations)
- Application metrics on services (discovered with annotations)
- State metrics (# of pods running for what deploymentconfig, for example)
- Probe metrics (blackbox exporter) for annotated services

What you currently can't get from inside of the projects, unfortunately, are
the container system metrics provided by cadvisor (CPU, memory, etc.).

## Proof of Concept

The ([prometheus-ephemeral.yml](prometheus-ephemeral.yml)) template is a proof
of concept that demonstrate how you can deploy a Pod inside of your projects
with the following containers:

- prometheus
- blackbox\_exporter
- kube-state-metrics

It requires no configuration and exposes a route, so that you can immediately
look at the metrics in Prometheus at the following URL:

  http://prometheus-PROJECT.OPENSHIFT\_DOMAIN/

You are then supposed to configure a Prometheus server outside of OpenShift
to pull metrics using the federation protocol. You can then in that outside
instance implement alerting, etc.
