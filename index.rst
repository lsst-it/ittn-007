:tocdepth: 1

.. Please do not modify tocdepth; will be fixed when a new Sphinx theme is shipped.

.. sectnum::

Introduction
============

This document describes monitoring for the IT infrastructure at Cerro Pachon,
reviews monitoring components and deployment mechanisms for managing monitoring
services, enumerates monitoring and deployment requirements, discusses
potential software and deployment options, and makes recommendations for a new
monitoring implementation.

Motivation
==========

At the time that this document is being written the Cerro Pachon IT
infrastructure is not being actively maintained. Monitoring services have been
manually configured and rolling out small changes have had unexpected impacts,
making further maintenance difficult and risky. These services have had less
than desired availability and have failed unexpectedly. In spite of these
issues these services are in production and thus cannot tolerate outages, which
further increases the challenge of maintaining the existing infrastructure.

The unknown configuration of these services, high risk associated with
maintenance, lack of reliability, and production status of the present
infrastructure drive the need for a replacement monitoring implementation.

Infrastructure overview
=======================

Monitoring services
-------------------

A standard monitoring implementation has the following services.

Centralized logging
^^^^^^^^^^^^^^^^^^^

Central logging aggregates log entries from devices, systems, and applications
and stores them in a single location. Logs can then be queried by users,
processed into metrics and sent to a time series database, or translated into
alerts.

Centralized logging provides visibility into systems operations and errors, and
simplifies the process of tracing data and events through a system.

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

Deployment system
-----------------

The deployment system is the mechanism that deploys, upgrades, manages, and
maintains the monitoring services.

It is assumed that host provisioning is implemented by an outside service and
that the monitoring deployment system will operate on systems accessible via
SSH or another management protocol.

The deployment service is responsible for the following functions.

Service installation and upgrades
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The deployment service is responsible for installing and upgrading the
monitoring software.

Installation and upgrading can be done with system packages such as RPMs, or by
fetching and launching containerized applications.

Service management
^^^^^^^^^^^^^^^^^^

The deployment service ensures that all services are up and running. In case
the underlying infrastructure is rebooted the deployment system will start the
appropriate services. If monitoring systems crash the deployment system will
restart services as needed.

Service maintenance
^^^^^^^^^^^^^^^^^^^

The deployment service is responsible for running regular system maintenance.
This includes tasks like cleaning up and compacting databases, or pruning stale
information.

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

Centralized logging
^^^^^^^^^^^^^^^^^^^

The logging storage system must be able to store and search a minimum of 30
days of logs, and should be able to store and search 90 days of logs.

The logging storage system should be able to sort groups of logs into
categories. For example, AuxTel users should be able to look at logs related to
AuxTel systems without requiring them to write and manage their own filters.

The centralized logging console must support LDAP authentication. As system
logs may contain sensitive information the logging console should be able to
limit access to different groups of logs.

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
days of logs, and should be able to store and search 90 days of logs.

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

Deployment requirements
-----------------------

The deployment system operating and managing monitoring services has the
following requirements.

Automated/Idempotent
^^^^^^^^^^^^^^^^^^^^

The deployment system must be an automated system that can provision systems
from the ground up. Manual configuration must be kept to an absolute minimum if
tolerated at all, and must be limited to trivial initialization steps that are
easy to document and reproduce. The automation system shall be the entry point
for operating the monitoring systems.

Immutable deployment
^^^^^^^^^^^^^^^^^^^^

The deployment system must deploy immutable infrastructure that is
provisioned once and minimally modified once in production. The deployment
system should discourage (if not outright forbid) manual configuration.

Some mutability is required - databases are inherently stateful and not all
services can be fully configured through automation, but state mutation after
initial provisioning shall be treated as a bug, not a requirement of operating
the system.

Atomic upgrades/rollbacks
^^^^^^^^^^^^^^^^^^^^^^^^^

Services should support atomic upgrades and rollbacks to ensure that
availability requirements can be met. Upgrades should be performed by
provisioning new systems, rolling forward and verifying functionality, and
rolling back to previous infrastructure in case of error.

In-place upgrades should be avoided; new systems should be provisioned so that
uprade failures can be resolved by rolling back to an already functioning
service.

Service isolation
^^^^^^^^^^^^^^^^^

All monitoring services must run independently. Databases and other supporting
services must run in separate execution environments (containers or hosts) so
that independent components can be maintained or replaced without impacting
their dependent services.

Approachable
^^^^^^^^^^^^

The deployment system should not require extensive knowledge in order to
upgrade applications, maintain services, debug issues, and perform other common
maintenance tasks. Common tasks shall be documented, and the overall deployment
infrastructure shall be built with simplicity, reliability, and clarity in mind.
