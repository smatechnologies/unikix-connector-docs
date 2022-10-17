# Installation

Unikix BPE (Batch Processing Environment) is a Mainframe rehosting solution by Clerity Solutions (now Dell) that “provides a complete environment for the administration, execution, and management of batch workloads on open systems servers” (source ClerityWhitePaper Rehosting Mainframe Workloads).

## Installation
The installation process consists of the following steps:

- OpCon Unix Agent Installation.
- Unikix ini file.
 
### OpCon Unix Agent Installation
The Unikix Connector requires the installation of a Unix Agent on the same system as the Unikix BPE.

#### INI file content

All the parameters of the INI file are described.

**Workspace directories**

The last three directories are relative to the first one. For instance, if WORKSPACE is ***/appl/unikix*** and DAILY_DIR is ***daily***, it means that the daily JCL scripts are located in ***appl/unikix/daily***.

Parameter |	Meaning	| Default value
--------- | ----------------------------------------------------------------------- | ---------
WORKSPACE |	Workspace directory. All other folders are relative to this one.        | -
DAILY_DIR |	Directory where the daily JCL scripts are located.                      | -
EXEC_DIR  |	Directory where the daily JCL are copied in order to be executed.       | -
PROC_DIR  |	Directory where the procedures executed from the JCL files are located.	| -

**Unikix log directory**

Parameter |	Meaning	| Default value
-------------- | ----------------------------------------------------------------------- | ---------
UNIKIX_LOG_DIR | Directory where Unikix writes the job execution log file.	             | -

**Unikix history directory**

The last five directories are relative to the first one. 
For instance, if ***HISTORY_DIR*** is ***/opt/Sched*** and ***CLASSE_A*** is ***historyA***, it means that the log files for class A jobs will be stored in ***/opt/Sched/historyA***.

Parameter |	Meaning	| Default value
----------- | ----------------------------------------------------------------------- | ---------
HISTORY_DIR	| Directory where the log files should be copied.                         | -
CLASSE_A    | Log directory for A class jobs                                          | -
CLASSE_B    | Log directory for B class jobs                                          | -
CLASSE_C    | Log directory for C class jobs                                          | -
CLASSE_F    | Log directory for F class jobs                                          | -
CLASSE_I    | Log directory for I class jobs                                          | -
CLASSE_T    | Log directory for T class jobs                                          | -

**Lang**

The connector can output log messages in English or in Italian. For now, most messages are not translated into Italian.
The debug messages will not be translated and are always in English.

Parameter |	Meaning	| Default value
--------- | ----------------------------------------------------------------------- | ---------
LANG      | Language for output messages. values en or it                           | en

**Logger**
Parameter |	Meaning	| Default value
--------- | ----------------------------------------------------------------------- | ---------
LOG_LEVEL | Level of the log messages                                               | debug
LOG_FILE  | If true, write a file called unikix_connector.log                       | true

Exampl INI file:

```
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













