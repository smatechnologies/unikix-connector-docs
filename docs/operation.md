# Operation

Unikix BPE (Batch Processing Environment) is a Mainframe rehosting solution by Clerity Solutions (now Dell) that “provides a complete environment for the administration, execution, and management of batch workloads on open systems servers” (source ClerityWhitePaper Rehosting Mainframe Workloads).

## definitions

Name | Description
-------------- | -----------
**Unikix**     | Unikix is a Mainframe rehosting solution provided by Clerity Solutions (now Dell).
**Unikix BPE** | (Batch Processing Environment) Batch platform used to execute JCL files. It is part of the Unikix solution.
**Class**      | (or Unikix activity class) Class of the Unikix job.
**JID**        | (or SCHID) number of a job Integer that determines which parts of the JCL file should be executed.
**Procedures** | (in Unikix) A procedure is a program run by a JCL file.

## Specifications

### JCL Master/Daily source repository management
On the Unikix server, the JCL files are in a specific folder considered as the Master Repository of the source JCLs.

Each day, a job in OpCon (that must be set up separately) copies all the JCL files from the Master Repository to a Daily Repository. The name of the Daily Repository must end with **_YY_MM_DD** (YY_MM_DD being the day’s date).

This allows daily changes to the source file without affecting the source in the Master Repository.
When a job is executed, the JCL file is copied into the dedicated folder for the submission and then pre-processed. This copy is then executed on Unikix BPE.

All these paths are configurable in the property file of the connector.

###	Connector workflow
The following paragraph explains the sequence of actions performed by the SMA Unikix Connector. 
Some of these steps are dedicated to some specific customer needs.

#### Connector’s Command line parsing
The following is the expected command line:

```
unikix_connector.jar jobname -c <arg> [-d <arg>] [-j <arg>] [-R <arg>]
-c,--class <arg> Class of the job (a, b, c, f, i or t)
-d,--date <arg> Date ; format: yy-mm-dd (default: today)
-j,--jid <arg> JID number
-R,--restart-step <arg> Starting step of the job

```

The job name is also the name of the JCL to be executed.

The only JCL classes accepted by the connector are a, b, c, f, i and t. The class name is case sensitive.

A job can be restarted at a specific step by two means:
- by using the OpCon **Restart on Step** feature, available from the Enterprise Manager (EM) on a job that has already been executed at least one in the daily
- by using the -R option with the step name

#### Read property file
The program requires a property file called **unikix_connector.properties**. This file should be placed in the same directory than the JAR file.
The description of the different parameters and an example can be found in installation section.

#### Preprocessing and JCL Tailoring Management
The JCL file is read from the DAILY_DIR directory and copied into the EXEC_DIR directory. Depending on the JID number, some parts of the file can be omitted.

In the JCL file, some special statements indicate JID specific sections.
- #JI,ID=1 : beginning of a section for JID = 1
- #JI,JEND : end of a section

All the lines between these statements should be written in the output file if the JID number equals 1.

The statements (#JI and #JEND) should be removed.

Simple example:

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

If JID=1:

```
BEGINJOB mode='MVS'
EBMSYSCMD << !
echo "SCHID 1" >> /tmp/OPCON_test.log
!
ENDJOB
```
If JID=2:

```
BEGINJOB mode='MVS'
EBMSYSCMD << !
echo "SCHID 2" >> /tmp/ OPCON_test.log
!
ENDJOB
```

If JID does not equal 1 and does not equal 2:

```
BEGINJOB mode='MVS'

ENDJOB
```

The #JI statement can have the following syntax:
- #JI,ID=N     : match only integer N
- #JI,ID=(N-M) : match integers between N and M included (parenthesis are optional)
- #JI,ID=(N,M) : match integer N and integer M only (parenthesis are optional)

#### Report steps back to OpCon
JCL file can have steps. The steps are reported into OpCon during the job execution thanks to the program sma_job_step This program is included in the bin folder of the Unix LSAM. The daily job can then be restarted on a specific step from the Enterprise Manager.
The connector can report at most 100 steps to the SAM. This limitation was introduced in version 2.1.0 so that the limit of the 32000 bytes of message from LSAM to SAM is not exceeded.

To report step number 3 called **STEP030**, the following command line is executed by the connector:

```
$SMA_BINDIR/sma_job_step 3 STEP030
```

In the JCL file, the steps are indicated with the following statement:

```
LABEL name=STEP030
```

A JCL file can call procedures, which can contain steps. Those steps are also reported into OpCon. A procedure is called with the following statement:

```
EXECPROC procname='TESTPROC' stepname='STEP030'
```
For example, with the following JCL file:

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
And procedure TESTPROC:

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

The following steps are reported into OpCon:
- STEP010
- STEP020
- STEP020.STEP010P
- STEP020.STEP020P
- STEP030

#### Launch Job on Unikix BPE
Once the JCL file is tailored and copied into the EXEC_DIR directory, and once the steps are reported into OpCon, the tailored JCL can be executed.

The Unikix command executed to start a job is:

```
unikixjob jobname -c class –w -j -R stepname
```
where:

argument | Description
--------------- | -----------
**-c class**    | is the class of the job
**-w**          | waits for the job to complete before returning,
**-j**          | directs the job number to the standard output, so it can be caught by the connector,
**-R stepname** | stepname restarts the job at the specified step.

The Unikix job number is derived from the standard output of the unikixjob command. To find the job number in the standard output, the following regular expression is used:

```
JOB (\d+):\s+Ready
```
The job number is used later on to get the Unikix job return code.

#### Copy history files
Unikix writes log files into UNIKIX_LOG_DIR directory. These files are copied to another location by the connector. This location is configurable in the INI file (**HISTORY_DIR**). There is one sub-directory per JCL class, also configurable in the INI file.

To get the history file corresponding to the job that has just been launched, the connector:
- browses the UNIKIX_LOG_DIR directory and looks for files whose name starts with the job name
- moves the last modified file into one of HISTORY_DIR sub-directories.

#### Get job’s return code
If the unikixjob command returned 0, then the connector will return 0 to OpCon.
Otherwise, the connector will return 10 and the name of the step that failed will be written in the standard output.
To get the step that failed in a Unikix job, a Unikix command must be executed. For example, for job number 512:

```
lststs –j512 –c
```
A list of messages are returned and parsed. Three different regular expressions are tested against the messages. 
If one of the messages:
- Match regular expression **was executed.+COND-CODE=[^0][0-9]*$** and the 3rd word of this message is **MA2522**
- Match regular expression **EBMSYSCMD.+Job Status : ABORT$** and the 3rd word of the message is **EBMSYSCMD**
- Match regular expression **ASSGNDD.+Job Status : ABORT$** and the 3rd word of the message is **ASSGNDD**,

Then the 2nd word of this message is the name of the step that failed. For example, in the following output, the 2nd message matches the first condition, so step STEP01 failed.

```
PDRIC600(58) STEP01 MA2517 (I) starting step STEP01
PDRIC600(58) STEP01 MA2522 (I) was executed RETURN-CODE=9999 , COND-CODE=255
```
```