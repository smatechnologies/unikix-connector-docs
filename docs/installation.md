---
title: Installation
description: "Installation and configuration reference for the SMA OpCon Unikix Connector, including prerequisites and the parameters in the connector INI file."
sidebar_label: "Installation"
tags:
  - Procedural
  - Reference
  - System Administrator
  - Installation
---

# Installation

This page describes how to install the SMA OpCon Unikix Connector and configure its INI file.

## What is it?

The SMA OpCon Unikix Connector is a Java-based connector delivered as a stand-alone JAR file. It runs on the Unix or Linux system that hosts Unikix BPE and integrates OpCon with Unikix BPE for job submission and return code management. For a broader description of what the connector does, see [Overview](./overview.md).

## Installation overview

To install the Unikix Connector, complete the following steps:

1. Confirm the [prerequisites](#before-you-begin) are met.
2. [Install the OpCon Unix Agent](#step-1-install-the-opcon-unix-agent) on the same system as Unikix BPE.
3. [Configure the INI file](#step-2-configure-the-ini-file) for the connector.

## Before you begin

Confirm the following before starting the installation:

| Requirement | Detail |
|---|---|
| Operating system | Supported Unix or Linux system that hosts Unikix BPE. |
| Unikix BPE | Installed and running on the target system. |
| OpCon Unix Agent | Installed on the same system as Unikix BPE. |
| Java runtime | Compatible Java distribution. As of release 21.0, the connector is compiled with Java 1.8, so an OpenJDK 1.8 distribution is required. |

:::note
The connector and the OpCon Unix Agent must run on the same system as Unikix BPE.
:::

## Step 1: Install the OpCon Unix Agent

The Unikix Connector relies on the OpCon Unix Agent to communicate with the OpCon server. The Unix Agent installation procedure is documented in the OpCon Unix Agent documentation.

## Step 2: Configure the INI file

The connector reads its configuration from an INI file in the same directory as the JAR file. The INI file groups parameters into the following categories:

| Category | Purpose |
|---|---|
| [Workspace directories](#workspace-directories) | Locations for the daily JCL, run-time copy, and procedures. |
| [Unikix log directory](#unikix-log-directory) | Location of the Unikix BPE job log files. |
| [Unikix history directory](#unikix-history-directory) | Locations where the connector archives job log files by class. |
| [Language](#language) | Output language for connector log messages. |
| [Logger](#logger) | Connector log level and log file behavior. |

:::tip File name and location
The INI file is named `unikix_connector.properties`. Place it in the same directory as the connector JAR file.
:::

### Workspace directories

The workspace section defines where the connector reads daily JCL files, where it copies them to run, and where it looks for procedures referenced from those JCL files.

`DAILY_DIR`, `EXEC_DIR`, and `PROC_DIR` are relative to `WORKSPACE`. For example, if `WORKSPACE` is `/appl/unikix` and `DAILY_DIR` is `daily`, the daily JCL scripts are located in `/appl/unikix/daily`.

| Parameter | Meaning | Default value |
|---|---|---|
| `WORKSPACE` | Workspace directory. All other folders are relative to this one. | - |
| `DAILY_DIR` | Directory where the daily JCL scripts are located. | - |
| `EXEC_DIR` | Directory where the daily JCL files are copied so they can be run. | - |
| `PROC_DIR` | Directory where the procedures called from the JCL files are located. | - |

### Unikix log directory

The Unikix log directory tells the connector where Unikix BPE writes its raw job log files. The connector reads from this location after each job and copies the matching file into the [history directory](#unikix-history-directory).

| Parameter | Meaning | Default value |
|---|---|---|
| `UNIKIX_LOG_DIR` | Directory where Unikix writes the job log file. | - |

### Unikix history directory

The history section defines where the connector archives Unikix log files for completed jobs, organized into one subdirectory per JCL class.

The class subdirectories are relative to `HISTORY_DIR`. For example, if `HISTORY_DIR` is `/opt/Sched` and `CLASSE_A` is `historyA`, the log files for class A jobs are stored in `/opt/Sched/historyA`.

| Parameter | Meaning | Default value |
|---|---|---|
| `HISTORY_DIR` | Directory where the log files are copied. | - |
| `CLASSE_A` | Log directory for class A jobs. | - |
| `CLASSE_B` | Log directory for class B jobs. | - |
| `CLASSE_C` | Log directory for class C jobs. | - |
| `CLASSE_F` | Log directory for class F jobs. | - |
| `CLASSE_I` | Log directory for class I jobs. | - |
| `CLASSE_T` | Log directory for class T jobs. | - |

### Language

The connector can output log messages in English or in Italian.

:::note
Most log messages are not translated into Italian. Debug messages are not translated and are always written in English.
:::

| Parameter | Meaning | Default value |
|---|---|---|
| `LANG` | Language for output messages. Values: `en` or `it`. | `en` |

### Logger

The logger section controls the verbosity of the connector log and whether the connector writes a dedicated log file.

| Parameter | Meaning | Default value |
|---|---|---|
| `LOG_LEVEL` | Level of the log messages. | `debug` |
| `LOG_FILE` | If `true`, write a file called `unikix_connector.log`. | `true` |

## Example INI file

The example below shows a complete INI file with one value for every parameter. Use it as a starting point for your own configuration.

```ini
# execution options
WORKSPACE=/etc/prod/
DAILY_DIR=daily
EXEC_DIR=eshell
PROC_DIR=procshp

# directory where Unikix logs are
UNIKIX_LOG_DIR=/etc/log/batch

# directory where this connector stores the history files
HISTORY_DIR=/etc/sched
CLASSE_A=histA
CLASSE_B=histB
CLASSE_C=histC
CLASSE_F=histF
CLASSE_I=histD
CLASSE_T=histE

# logging options
LOG_LEVEL=debug
LOG_FILE=true

# language
LANG=en
```

## FAQs

### Where does the INI file need to be placed?

The INI file is named `unikix_connector.properties` and must be placed in the same directory as the connector JAR file.

### Are the workspace subdirectories absolute or relative paths?

`DAILY_DIR`, `EXEC_DIR`, and `PROC_DIR` are relative to `WORKSPACE`. The class subdirectories (`CLASSE_A` through `CLASSE_T`) are relative to `HISTORY_DIR`.

### Which logging level should I use?

The default is `debug`. Adjust it based on the volume of detail you need in the connector log.

### Where are the connector log files written?

When `LOG_FILE` is `true`, the connector writes a file named `unikix_connector.log` alongside the JAR file. Job-specific Unikix log files are written by Unikix BPE to `UNIKIX_LOG_DIR` and are then moved by the connector into the matching class subdirectory under `HISTORY_DIR`.

## Glossary

| Term | Definition |
|---|---|
| Unikix BPE | The Batch Processing Environment within the Unikix solution that runs JCL files. |
| Class | Also called Unikix activity class. Identifies the class of a Unikix job. The connector accepts only `a`, `b`, `c`, `f`, `i`, and `t`. Class names are case sensitive. |
| Master repository | The folder on the Unikix server that holds the source JCL files. |
| Daily repository | A dated copy of the master repository that the connector uses for the day's runs. The folder name ends with `_YY_MM_DD`. |
| INI file | The connector's configuration file (`unikix_connector.properties`), placed alongside the JAR. |

**Related topics:**

- [Overview](./overview.md)
- [Operation](./operation.md)
- [Release Notes](./release-notes.md)
