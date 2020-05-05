:tocdepth: 1

.. Please do not modify tocdepth; will be fixed when a new Sphinx theme is shipped.

.. sectnum::

Introduction
============

This document describes monitoring for the IT deployments at Cerro Pachon and La Serena.

Infrastructure overview
=======================

Monitoring services
-------------------

A standard monitoring implementation has the following services.

Metrics collection
^^^^^^^^^^^^^^^^^^

Metrics are derived from a variety sources, and have a range of collectors.

Some systems and devices only expose a metrics interface - for example,
switches and HVAC equipment expose their state via SNMP but do not export that
information. A metrics collector such as Telegraf must be configured to walk
and export the SNMP entries.

Other systems manage their own metrics. For instance, some web applications
have a built in capacity to export metrics directly to a time series database
as part of processing a request.

Examples of metrics collectors include Telegraf, Diamond, CollectD, and the
Prometheus scraper job.

Metrics storage
^^^^^^^^^^^^^^^

A metrics database tracks numeric values over time, providing insight into
system operation, load, and health over time. Metrics storage provide
infrastructure for both metrics visualization and alerting.

Some metrics storage systems (such as Prometheus) implement both metrics
collection and metrics storage.

Examples of metrics storage systems include Graphite, InfluxDB, Prometheus, and
TimescaleDB.

Metrics visualization
^^^^^^^^^^^^^^^^^^^^^

Metrics visualization converts time series measurements stored in a metrics
database into graphs, charts, dashboards, and other visual representation. A
metrics visualization provides both high level insight into the monitored
infrastructure, information about the state of specific systems at specific
times, and history/trending of system properties.

Examples of metrics visualizations include Grafana and Chronograf.

Alerting
^^^^^^^^

Alerting can be separated into two subcategories - white box alerting, and
black box alerting.

White box alerting is performed by monitoring a metric and generating an alert
when the metric leaves a desired range. One example of white box alerting is
for website requests per second, where alerts may be generated when no requests
are being received (indicating traffic is failing to reach the service) or a
large number of requests are received (indicating that the server may become
saturated and could fail). Another example is host memory, where memory usage
greater than 90% indicates that the host may become overloaded.

Black box alerting is performed by monitoring a boolean condition and
generating an alert when that condition becomes false. Examples of black box
monitoring include host reachability (alerting when a host cannot be pinged) or
a service being offline (indicating that service may have crashed).

Examples of white box alerting include Kapacitor, Grafana Alerts, and
Prometheus Alertmanager.

Examples of black box alerting include Icinga.

Requirements
============

Overall requirements
--------------------

Monitoring infrastructure has the following overall requirements.

1. Reliable - monitoring is the heart of modern SRE practices. If monitoring
   is down engineers are effectively blind, so the system must be robust and
   able to operate even if there is infrastructure degradation.

1. Maintainable - the monitoring stack will be maintained by the observatory
   IT team. Engineers should be able to deploy, modify, maintain, and repair
   monitoring services.

1. Automatable/Repeatable - Monitoring infrastructure must be provisioned in an
   automated manner that ensures that the infrastructure operates in a known
   state, is well understood and characterized, can be upgraded and rolled back
   in a well defined manner, and is automated in a manner that can be operated
   by all of IT.

1. Accessible - system metrics should be visible to both the IT team and IT
   infrastructure users. Infrastructure users should be able to view metrics,
   understand system health and usage, and make effective decisions based on
   that information.

Monitoring component requirements
---------------------------------

The monitoring components have the following requirements.

Metrics collection
^^^^^^^^^^^^^^^^^^

Metrics must be collected at least once a minute. Ideally metrics should be
collected every 15 seconds.

Metrics collection on observatory owned systems (such as CSC hosts) must not
affect system performance. Metrics collection should be throttled to use a
minimum amount of CPU and RAM; it is better to collect fewer metrics than to
interfere with system operations.

Metrics storage
^^^^^^^^^^^^^^^

The metrics storage system must be capable of ingesting metrics from all hosts
and all services once a minute on a sustained basis. The metrics storage system
should be capable of ingesting metrics from some hosts and services every 15
seconds.

The metrics storage system must be able to search and store a minimum of 30
days of metrics, and should be able to store and search 90 days of metrics.

The metrics storage system must support a programmatic interface (REST or
other) for fetching and querying metrics.

Metrics visualization
^^^^^^^^^^^^^^^^^^^^^

The metrics visualization console must be able to store metrics dashboards.
Metrics dashboards must support serialization so that dashboards can be saved
and re-created.

The metrics visualization console should support a browse or explore function
to enumerate available metrics.

The metrics visualization console should (but is not required to) support a
programmatic interface for generating visualizations.

The metrics visualization console must support LDAP authentication. The metrics
visualization may support access control but access control is not required.
