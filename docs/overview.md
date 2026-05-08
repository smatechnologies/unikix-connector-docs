---
title: Overview
description: "Overview of the SMA OpCon Unikix Connector, including what it does, the additional capabilities it provides, and how it integrates with Unikix BPE."
sidebar_label: "Unikix Connector"
tags:
  - Conceptual
  - System Administrator
  - Automation Engineer
  - Getting Started
---

# Overview

The SMA OpCon Unikix Connector integrates OpCon with Unikix BPE (Batch Processing Environment) so OpCon can submit JCL jobs on Unikix and manage their return codes.

## What is it?

The SMA OpCon Unikix Connector is a Java-based connector delivered as a stand-alone JAR file. It runs on the same Unix or Linux system as Unikix BPE and acts as the bridge between OpCon and Unikix components.

Unikix BPE (Batch Processing Environment) is a mainframe rehosting solution by Clerity Solutions (now Dell) that provides an environment for the administration, execution, and management of batch workloads on open systems servers (source: ClerityWhitePaper Rehosting Mainframe Workloads).

The connector implements the interaction between OpCon and Unikix components to support:

- Job submission on Unikix BPE
- Return code management

## Additional capabilities

The SMA OpCon Unikix Connector also includes capabilities that address specific customer needs:

- JCL master/daily repository management
- Pre-processing of the JCL file to be run by Unikix BPE, with JCL tailoring
- Step discovery and management, including restart on step from Enterprise Manager

## How it works with OpCon

The Unikix Connector is invoked from the OpCon Unix Agent on the Unikix server. OpCon submits a job that runs the connector JAR, the connector pre-processes and tailors the daily JCL, runs it through Unikix BPE, reports any steps back to OpCon, and returns a status code that OpCon uses to mark the job as successful or failed.

For details on configuring the connector and the INI file, see [Installation](./installation.md). For details on the connector workflow and JCL tailoring rules, see [Operation](./operation.md).

## Glossary

| Term | Definition |
|---|---|
| Unikix | A mainframe rehosting solution provided by Clerity Solutions (now Dell). |
| Unikix BPE | Batch Processing Environment. The batch platform within the Unikix solution that runs JCL files. |
| Class | Also called Unikix activity class. Identifies the class of a Unikix job. The connector accepts only `a`, `b`, `c`, `f`, `i`, and `t`, and the class name is case sensitive. |
| JID | Also called SCHID. An integer that determines which sections of the JCL file are run. |
| Procedure | A program run by a JCL file in Unikix. |
| JCL master repository | The folder on the Unikix server that holds the source JCL files. |
| JCL daily repository | A dated copy of the master repository that the connector uses for the day's runs. The folder name ends with `_YY_MM_DD`. |

## FAQs

### Which platforms does the connector run on?

The connector is written in Java and is delivered as a stand-alone JAR file. It runs on the Unix or Linux system that hosts Unikix BPE.

### What does OpCon need to use the connector?

The connector requires the OpCon Unix Agent to be installed on the same system as Unikix BPE. See [Installation](./installation.md) for the full prerequisites.

### Which JCL classes does the connector support?

The connector accepts only the classes `a`, `b`, `c`, `f`, `i`, and `t`. Class names are case sensitive.

### Can a job be restarted on a specific step?

Yes. A job can be restarted on a step in two ways: from the OpCon Enterprise Manager **Restart on Step** action on a job that has already run at least once in the daily, or by passing the `-R` option with the step name on the connector command line. See [Operation](./operation.md) for details.

### How are job steps reported back to OpCon?

When the connector runs a JCL file, it reports each step to OpCon using the `sma_job_step` program included with the OpCon Unix Agent. Up to 100 steps are reported per job. See [Operation](./operation.md) for details.

**Related topics:**

- [Installation](./installation.md)
- [Operation](./operation.md)
- [Release Notes](./release-notes.md)
