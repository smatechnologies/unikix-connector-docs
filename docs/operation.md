---
title: Operation
description: "Reference for the SMA OpCon Unikix Connector workflow, including command-line arguments, JCL pre-processing rules, step reporting, and return code handling."
sidebar_label: "Operation"
tags:
  - Conceptual
  - Reference
  - System Administrator
  - Automation Engineer
---

# Operation

This page describes how the SMA OpCon Unikix Connector runs Unikix BPE jobs, how JCL files are pre-processed, how steps are reported back to OpCon, and how return codes are handled.

## What is it?

The Unikix Connector is the runtime component that OpCon invokes to submit a JCL job to Unikix BPE. The connector reads a property file, copies and pre-processes the daily JCL, reports JCL steps back to OpCon, runs the JCL through Unikix BPE, and returns a status to OpCon based on the Unikix job result.

For a higher-level description of the connector, see [Overview](./overview.md). For installation and INI file details, see [Installation](./installation.md).

## Workflow at a glance

When OpCon submits a job to the connector, the connector runs the following phases in order:

| # | Phase | Purpose |
|---|---|---|
| 1 | [Parse the command line](#1-parse-the-command-line) | Read the job name, class, date, JID, and optional restart step. |
| 2 | [Read the property file](#2-read-the-property-file) | Load directory paths and logging settings from `unikix_connector.properties`. |
| 3 | [Pre-process and tailor the JCL](#3-pre-process-and-tailor-the-jcl) | Copy the daily JCL into `EXEC_DIR` and keep only the sections that match the JID. |
| 4 | [Report steps to OpCon](#4-report-steps-to-opcon) | Send up to 100 step names to OpCon so the job can be restarted on step. |
| 5 | [Run the job on Unikix BPE](#5-run-the-job-on-unikix-bpe) | Submit the tailored JCL using `unikixjob`. |
| 6 | [Copy history files](#6-copy-history-files) | Move the matching Unikix log file into the class subdirectory under `HISTORY_DIR`. |
| 7 | [Return the result](#7-return-the-result) | Return 0 on success, or 10 with the failed step name on failure. |

## JCL master and daily repository management

The Unikix server holds the source JCL files in a folder that acts as the master repository. Each day, an OpCon job that is set up separately copies all of the JCL files from the master repository to a daily repository.

:::note Daily repository naming
The name of the daily repository ends with `_YY_MM_DD`, where `YY_MM_DD` is the date of the run.
:::

This pattern allows daily changes to the source JCL without affecting the master repository. When a job runs, the JCL file is copied into the directory used for submission, pre-processed, and then run on Unikix BPE.

All paths involved are configurable in the connector property file. See [Installation](./installation.md) for the parameter list.

## Connector workflow

The following sections describe each phase of the connector workflow in detail. Some phases are dedicated to specific customer needs.

### 1. Parse the command line

The connector is invoked with the following command line:

```bash
unikix_connector.jar jobname -c <arg> [-d <arg>] [-j <arg>] [-R <arg>]
```

The job name is also the name of the JCL file to be run.

| Argument | Required | Description |
|---|---|---|
| `-c, --class <arg>` | Yes | Class of the job. Allowed values: `a`, `b`, `c`, `f`, `i`, `t`. |
| `-d, --date <arg>` | No | Date in `yy-mm-dd` format. Defaults to today. |
| `-j, --jid <arg>` | No | JID number used to select sections of the JCL file. |
| `-R, --restart-step <arg>` | No | Starting step of the job. |

:::caution Class names are case sensitive
Use the lowercase letters `a`, `b`, `c`, `f`, `i`, or `t` for the `-c` argument. The connector rejects any other class name.
:::

#### Restart options

A job can be restarted on a specific step in two ways:

- Use the OpCon **Restart on Step** action available in Enterprise Manager on a job that has already run at least once in the daily.
- Pass the `-R` option with the step name on the connector command line.

### 2. Read the property file

The connector requires a property file named `unikix_connector.properties`. The file must be placed in the same directory as the JAR file.

For the parameter descriptions and an example, see [Installation](./installation.md).

### 3. Pre-process and tailor the JCL

The connector reads the JCL file from `DAILY_DIR` and copies it into `EXEC_DIR`. Depending on the JID number, some sections of the file can be omitted.

JID-specific sections are marked with the following statements:

| Statement | Meaning |
|---|---|
| `#JI,ID=1` | Beginning of a section for `JID = 1`. |
| `#JI,JEND` | End of a JID-specific section. |

When the JID number matches, the lines between the markers are written to the output file. The `#JI` and `#JEND` statements are removed from the output.

#### Supported `#JI` syntax

| Syntax | Match |
|---|---|
| `#JI,ID=N` | Matches integer `N` only. |
| `#JI,ID=(N-M)` | Matches integers between `N` and `M` inclusive. Parentheses are optional. |
| `#JI,ID=(N,M)` | Matches integer `N` and integer `M` only. Parentheses are optional. |

#### Example: tailoring by JID

Given the input JCL below, the connector produces a different output for each JID value.

**Input JCL:**

```
BEGINJOB mode='MVS'
#JI,ID=1
EBMSYSCMD << !
echo "SCHID 1" >> /tmp/OPCON_test.log
!
#JI,ID=2
EBMSYSCMD << !
echo "SCHID 2" >> /tmp/ OPCON_test.log
!
#JI,JEND
ENDJOB
```

**Output when `JID=1`:**

```
BEGINJOB mode='MVS'
EBMSYSCMD << !
echo "SCHID 1" >> /tmp/OPCON_test.log
!
ENDJOB
```

**Output when `JID=2`:**

```
BEGINJOB mode='MVS'
EBMSYSCMD << !
echo "SCHID 2" >> /tmp/ OPCON_test.log
!
ENDJOB
```

**Output when `JID` does not equal 1 or 2:**

```
BEGINJOB mode='MVS'

ENDJOB
```

### 4. Report steps to OpCon

A JCL file can contain steps. The connector reports each step back to OpCon during the job run using the `sma_job_step` program included in the `bin` folder of the OpCon Unix Agent. The daily job can then be restarted on a specific step from Enterprise Manager.

:::caution 100-step limit per job
The connector reports a maximum of 100 steps per job. This limit was introduced in version 2.1.0 to stay within the 32,000-byte message limit between the OpCon Unix Agent and the OpCon SAM.
:::

To report step number 3 named `STEP030`, the connector runs:

```bash
$SMA_BINDIR/sma_job_step 3 STEP030
```

#### How steps are identified in JCL

| Statement | Purpose |
|---|---|
| `LABEL name=STEPxxx` | Marks a step in the JCL file. |
| `EXECPROC procname='PROC' stepname='STEPxxx'` | Calls a procedure. Steps inside the procedure are also reported. |

#### Example: nested steps from a procedure call

Given the following JCL file:

```
BEGINJOB mode='MVS'
LABEL name=STEP010
EBMSYSCMD << !
echo "Begin job"
!
LABEL name=STEP020
EXECPROC procname='TESTPROC' stepname='STEP020'
LABEL name=STEP030
EBMSYSCMD << !
echo "End job"
!
ENDJOB
```

And procedure `TESTPROC`:

```
BEGINPROC procname='TESTPROC'
LABEL name=STEP010P
EBMSYSCMD << !
echo "Hello world"
!
LABEL name=STEP020P
ASSGNDD ddname='SYSUT1' filename='fileinput.txt' disp='i'
ASSGNDD ddname='SYSUT2' filename='fileoutput.txt' disp='o'
EXECPGM pgmname='IEBGENER' stepname='STEP020P'
ENDPROC
```

The following steps are reported to OpCon:

- `STEP010`
- `STEP020`
- `STEP020.STEP010P`
- `STEP020.STEP020P`
- `STEP030`

### 5. Run the job on Unikix BPE

After the JCL file is tailored, copied into `EXEC_DIR`, and the steps have been reported to OpCon, the tailored JCL is run on Unikix BPE.

The Unikix command used to start the job is:

```bash
unikixjob jobname -c class -w -j -R stepname
```

| Argument | Description |
|---|---|
| `-c class` | Class of the job. |
| `-w` | Waits for the job to complete before returning. |
| `-j` | Directs the job number to standard output so the connector can read it. |
| `-R stepname` | Restarts the job at the specified step. |

The Unikix job number is captured from standard output of the `unikixjob` command. The connector matches the following regular expression to find the job number:

```regex
JOB (\d+):\s+Ready
```

The job number is then used to retrieve the Unikix job return code.

### 6. Copy history files

Unikix writes log files into `UNIKIX_LOG_DIR`. The connector copies these files to another location, configurable in the INI file as `HISTORY_DIR`, with one subdirectory per JCL class.

To find and move the history file for the job that was just run, the connector:

1. Browses the `UNIKIX_LOG_DIR` directory and looks for files whose names start with the job name.
2. Moves the last modified matching file into the `HISTORY_DIR` subdirectory for the job's class.

### 7. Return the result

The connector return code tells OpCon whether the Unikix job succeeded.

| Connector return code | Meaning |
|---|---|
| `0` | `unikixjob` returned 0. The job succeeded. |
| `10` | `unikixjob` did not return 0. The connector also writes the name of the failed step to standard output. |

#### Identifying the failed step

To find the failed step in a Unikix job, the connector runs the Unikix `lststs` command. For example, for job number 512:

```bash
lststs -j512 -c
```

The connector parses each message in the response against three regular expressions. If a message matches one of the following conditions, the second word of that message is the name of the step that failed:

| Match condition | Required third word |
|---|---|
| Message matches `was executed.+COND-CODE=[^0][0-9]*$` | `MA2522` |
| Message matches `EBMSYSCMD.+Job Status : ABORT$` | `EBMSYSCMD` |
| Message matches `ASSGNDD.+Job Status : ABORT$` | `ASSGNDD` |

**Example:** in the output below, the second message matches the first condition, so step `STEP01` is reported as the failed step:

```
PDRIC600(58) STEP01 MA2517 (I) starting step STEP01
PDRIC600(58) STEP01 MA2522 (I) was executed RETURN-CODE=9999 , COND-CODE=255
```

## FAQs

### Which JCL classes does the connector accept?

The connector accepts only the classes `a`, `b`, `c`, `f`, `i`, and `t`. Class names are case sensitive.

### How are JID-specific sections of a JCL file handled?

The connector reads `#JI` markers in the JCL file and writes only the sections whose ID matches the JID passed on the command line. The `#JI` and `#JEND` markers are removed from the output JCL.

### How many steps can the connector report per job?

The connector reports a maximum of 100 steps per job. This limit keeps the message between the OpCon Unix Agent and the SAM within the 32,000-byte limit. The 100-step limit was introduced in version 2.1.0.

### How can a job be restarted on a specific step?

Use the OpCon **Restart on Step** action in Enterprise Manager for a job that has already run at least once in the daily, or pass the `-R <stepname>` option to the connector on the command line.

### What return codes does the connector send to OpCon?

The connector returns `0` when the underlying `unikixjob` command returns 0, and `10` when it does not. When the connector returns 10, it writes the name of the failed step to standard output.

## Glossary

| Term | Definition |
|---|---|
| Unikix | A mainframe rehosting solution provided by Clerity Solutions (now Dell). |
| Unikix BPE | Batch Processing Environment. The batch platform within the Unikix solution that runs JCL files. |
| Class | Also called Unikix activity class. Identifies the class of a Unikix job. The connector accepts only `a`, `b`, `c`, `f`, `i`, and `t`, and the class name is case sensitive. |
| JID | Also called SCHID. An integer that determines which sections of the JCL file are run. |
| Procedure | A program run by a JCL file in Unikix. |
| Master repository | The folder on the Unikix server that holds the source JCL files. |
| Daily repository | A dated copy of the master repository that the connector uses for the day's runs. The folder name ends with `_YY_MM_DD`. |

**Related topics:**

- [Overview](./overview.md)
- [Installation](./installation.md)
- [Release Notes](./release-notes.md)
