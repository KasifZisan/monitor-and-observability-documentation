# Introduction to Prometheus

## What is Promethues
Protmetheus is an open-source monitoring tool, mostly written in the 'Go' language by SoundCloud. It is specifically made for short-term monitoring. It has multi-dimensional data with time-series which is identified by Metric Name that are represented in Key/Value Pairs (Key:Value). It primarily pulls metric from an endpoint but there is also a Push gateway involved in some deployments.

- Prometheus has a strong suite in Machine Centric Monitoring
- Even a stronger suite in Dynamic service-oriented architectures (microservices)
- It is designed for reliability. Standalone and Independent of the larger architecture, totally decoupled from everything

## What isn't Prometheus
- It is not a long term metric storage solution
- Does not have a 100% accuracy in metrics
- Not meant for deep, detailed metrics. Instead, it is good at -
    - Summarizing Metrics
- It is not a push oriented metrics system as its main strength is pulling metrics

## Features Overview

#### Core Set
- Time series Data Identification
    - Metric name
    - Key Value Pairs
- Functional Query Language; built on a specific, custom, PromQL tool. PromQL performs aggregation in real-time. These can be graphed and consumed by external systems via HTTP API.
- Pre-configured rules can speed up longer queries.
- It also provides flexible graphing and dashboarding using PromDash; a GUI based builder having an SQL backend.
- HTTP API, which is involved in the prometheus server and exists as ```/api/v1```

#### PromQL Rules
- In the configuration file -recording rules are referred to via a file containing their configuration. So we have ```rule_files``` :
    - ```prometheus.rules.yaml```
```yaml
groups:
  # name of the rule
- name: 
  rules:
    # the rules by recording
  - record: 
    # the expressions we want to evaluate them against
    expr: 
```
- Prometheus is non-reliant on a distributed storage system. As a result, it gets high througput using the on-node storage.
- Non-meant as a long term storage solution. Ideally for 3-6 months of storage. And then engineers might want to upload the data to Thanos.
- Prometheus collects its metrics using HTTP Pulls (one URI per exporter)
- Prometheus scrapes these endpoints and the endpoints can either be defined or discovered

#### Defined & Discovered 
- Prometheus collects metrics over HTTP Pulls, which can either be -
    - Defined: Statically configured in the YAML file
    - Discovered: Automatically discovered by Prometheus using its' DNS discovery service

``` yaml
# Defined 
- job name: prometheus
  honor_timestamps: true
  scrape_interval: 5s
  scrap_timeout: 5s
  metrics_path: /metrics
  scheme: http
  static_configs:
  - targets:
    - localhost: 9090
```

#### Pushing
- Pushing is available through the PushGateway. It is used for non-scrapable components and for short-lived jobs.
- It can also be used to present metrics to prometheus for pulling. You can think of this as an intermediary server that prometheus can pull from.

## Components
#### PushGateway
PushGateway is best suited for service-level jobs because when we have an intermediary pushing the metrics, it can also be a single point of failure for those metrics. If we build our critical layers on system level information and our PushGateway failed, so would our alerting. This is why PushGateway needs to act for only service short-lived jobs.

The native health monitoring that Prometheus generates is not generated on PushGateway instances. That's because the inteneded purpose of this metrics is jobs to push and then exit

Metrics pushed to the PushGateway are not destroyed which Prometheus usually does. So they persist even after the jobs are exited. So they will be exposed to Prometheus forever until they are manually deleted via the PushGateway's API. 

#### Prometheus Server
The prometheus server may look monolithic in nature but it has several components.

- Retrieval Worker - Goes out and scrapes the endpoints
- The Storage Database
    - Connects to local Node Storage
    - Time Series Database
- HTTP Server 
    - Dashboarding
    - API - Responses are in JSON
    - Plug into our Alertmanager

But how is Prometheus Configured? Let's look at a Prometheus config file. We will divide the prometheus config file into several blocks -

```yaml
global:
    scrape_interval: 15s
    scrape_timeout: 10s
    evaluation_interval: 1m
    external_labels:
        monitor: Cloudcademy-Prom-Demo

rule_files:
- prometheus.rules.yaml

scrape_configs:
# we can mention specific jobs here 
# this job is monitoring prometheus after every 5s from the /metrics path 
# and it is hosted on 9090 port
- job name: prometheus
  honor_timestamps: true
  scrape_interval: 5s
  scrap_timeout: 5s
  metrics_path: /metrics
  scheme: http
  static_configs:
  - targets:
    - localhost: 9090

- job_name: Prom1-node
  honor_timestamps: true
  scrape_interval: 5s
  scrap_timeout: 5s
  metrics_path: /metrics
  scheme: http
  static_configs:
  - targets:
    - localhost: 8080
    - localhost: 8081
    labels:
      group: CloudAcademy-Prod
  - targets:
    - localhost: 8082
    labels:
      group: CloudAcademy-Dev
```

#### Alertmanager
Alertmanager handles alerts sent by client applications, sends them along their way and routes them to the correct receiver, whether that be - Email, PagerDuty etc. 

**Grouping:** It also has ways to group alerts into a single notification. So you are not alerted every single time an instance maybe failing. 

**Inhibition:** It also has the ability to prevent alerts from failing if another alert is going on. For example - If alert X is alerting, don't turn trigger alert Y. This could be useful in situations for example if a whole cluster is down, we don't need alerts for every instances within that cluster and their alerts. 

**Silences:** There is also ability to mute alerts for given period of time. 

**High-Availability:** There is high availability for multi-cluster support available through a command-line plug

#### Exporters
Exporters are open-source generally and they are tailored for Prometheus. They - 

- Grab metrics for Prometheus
- Alter those metrics into a Prometheus format
    - Using a client library such as - GO, Java or Scala, Python, Ruby
- After this they typically start a web server for themselves to be scraped upon