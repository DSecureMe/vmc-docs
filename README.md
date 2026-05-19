# OWASP Vulnerability Management Center — Documentation

> [!WARNING]
> **This software is NOT a release candidate and is NOT production-ready.**
>
> The code in this repository (and its dependencies) may contain
> vulnerabilities. The maintainers provide no warranty of any kind and
> accept no liability for any defect, vulnerability, data loss, security
> incident or other damage resulting from its use.
>
> You install, run, evaluate and operate this software **at your own
> risk**. Do not deploy it in a production environment, and do not
> expose it to untrusted networks without an independent security
> review.

## Introduction

This document presents the OWASP Vulnerability Management Center (VMC)
platform — its design, the requirements for running it, the installation
options, the architecture and the administration panel.

## VMC platform

Vulnerability Management Center (VMC) is a tool that facilitates and supports
the process of managing vulnerabilities in an organisation. It integrates
with infrastructure elements such as vulnerability scanners, publicly
available knowledge bases on published vulnerabilities, IT asset inventory
systems and reporting platforms. With its data-normalisation capabilities and
a transparent interface, VMC presents threat information in an accessible way
and alerts on deviations and anomalies.

The main innovation factor of VMC is *organisation-level* vulnerability
analysis — as opposed to the common per-host vulnerability assessment that
ignores environmental conditions. Thanks to the environmental score computed
by VMC, the cybersecurity team can focus on the most exposed elements of the
infrastructure, prioritising remediation based on the tool's recommendations.

The architecture allows VMC to run in any environment — a public cloud, a
private cloud, a physical server or a virtual machine.

## Requirements

### Hardware

The minimum hardware footprint required for the platform to run correctly:

| Resource | Minimum                          |
|----------|----------------------------------|
| RAM      | 8 GB                             |
| Storage  | 30 GB per machine                |
| CPU      | 4 cores                          |

### Software

| Name           | Minimum version                                                                | Description                                                                |
|----------------|--------------------------------------------------------------------------------|----------------------------------------------------------------------------|
| Linux OS       | Debian 12 (Bookworm) / RHEL 9 / Ubuntu 22.04 (any modern Linux with Docker)    | Host OS.                                                                   |
| Python 3       | 3.10                                                                           | Application runtime.                                                       |
| Nginx          | any current release                                                            | <https://www.nginx.com/> — proxy in front of the VMC admin.                |
| Redis          | 6.2                                                                            | <https://redis.io/>                                                        |
| RabbitMQ       | 3.12                                                                           | <https://www.rabbitmq.com/>                                                |
| Elasticsearch  | 7.17                                                                           | <https://www.elastic.co/elasticsearch/>                                    |
| Kibana         | 7.17                                                                           | <https://www.elastic.co/kibana/>                                           |
| PostgreSQL     | 15                                                                             | <https://www.postgresql.org/>                                              |

### Supported scanners

The following vulnerability scanners are supported (optional for installation):

| Name    | Version                       |
|---------|-------------------------------|
| Nessus  | 7.x                           |
| Nessus  | 8.x                           |
| OpenVAS | GVM protocol version 9.0      |
| OpenVAS | OMP protocol version 7.0      |

## Installation

### On a virtual machine

#### From source

```bash
git clone https://github.com/DSecureMe/vmc.git
cd vmc
pip3 install .
```

#### From PyPI

```bash
pip3 install vmcenter
```

> [!NOTE]
> The `vmcenter` release on PyPI may lag the latest source by one or more
> cycles. For the most recent code (post-modernisation), prefer the git
> checkout above.

### Using Docker

If you want to build your own image with a specific VMC version:

```bash
git clone https://github.com/DSecureMe/vmc-docker
cd vmc-docker
make build
```

The official VMC image is available on
[Docker Hub](https://hub.docker.com/repository/docker/dsecureme/vmc):

```bash
docker pull dsecureme/vmc:tagname
```

For a turnkey demo (admin + scheduler + worker + Postgres + Elasticsearch +
Kibana + Ralph + TheHive + ElastAlert), use the orchestrator at
[`vmc-dev-toolkit`](https://github.com/DSecureMe/vmc-dev-toolkit):

```bash
git clone --recurse-submodules https://github.com/DSecureMe/vmc-dev-toolkit.git
cd vmc-dev-toolkit
make up all=y
make demodata
```

After `make demodata` succeeds, the admin panel is available at
<http://localhost:8080/admin/> (credentials: `admin` / `admin`).

## Table of contents

- [Configuration](./Configuration/README.md)
  - [Installation on a virtual machine](./Configuration/README.md#installation-on-a-virtual-machine)
  - [Installation using Docker](./Configuration/README.md#installation-using-docker)
- [Architecture](./Architecture/README.md)
  - [Knowledge Collector](./Architecture/README.md#knowledge-collector)
  - [Asset Collector](./Architecture/README.md#asset-collector)
  - [Vulnerability Collector](./Architecture/README.md#vulnerability-collector)
  - [Processing Module](./Architecture/README.md#processing-module)
- [Administration](./Admin-Panel/README.md)
  - [Adding vulnerability scanners](./Admin-Panel/README.md#adding-vulnerability-scanners)
  - [Importing data from the scanner](./Admin-Panel/README.md#importing-data-from-the-scanner)
  - [Disabling and enabling scanner configurations](./Admin-Panel/README.md#disabling-and-enabling-scanner-configurations)
  - [Adding an IT resource management database](./Admin-Panel/README.md#adding-an-it-resource-management-database)
  - [Data import from an IT resource management database](./Admin-Panel/README.md#data-import-from-an-it-resource-management-database)
  - [Creation of API tokens](./Admin-Panel/README.md#creation-of-api-tokens)
  - [Support for multiple organisations or groups](./Admin-Panel/README.md#support-for-multiple-organisations-or-groups)
- [API](./API/README.md)
- [Integration with IT asset management](./AM-integration/README.md)
- [Integration with TheHive](./HIVE-Integration/README.md)
  - [ElastAlert configuration](./HIVE-Integration/README.md#elastalert-configuration)
  - [Creating alerts](./HIVE-Integration/README.md#creating-alerts)
  - [Responder configuration](./HIVE-Integration/README.md#responder-configuration)
  - [TheHive webhooks configuration](./HIVE-Integration/README.md#thehive-webhooks-configuration)
- [Introduction to KPI](./KPI/README.md)
