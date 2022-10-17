---
slug: '/'
sidebar_label: 'Unikix Connector'
---

# SMA Unikix Connector

Unikix BPE (Batch Processing Environment) is a Mainframe rehosting solution by Clerity Solutions (now Dell) that “provides a complete environment for the administration, execution, and management of batch workloads on open systems servers” (source ClerityWhitePaper Rehosting Mainframe Workloads).

The SMA OpCon Unikix Connector implements the interaction between OpCon and some Unikix components to allow:
-	the Job submission on Unikix BPE 
-	the return code management 

The SMA OpCon Unikix Connector contains additional capability, dedicated to some specific customer needs that allows to:
-	JCL master/daily repository management
-	pre-process of the JCL file to be executed by Unikix BPE (with JCL tailoring capability)
-	step discovery and management (restart on step from EM)

The connector is written in Java. It is delivered as a stand-alone JAR file.

---