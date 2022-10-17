# Release Notes Unikix 21.0

## General

The release removes log4j and replaces it with slj4j and logback.

## Migration Considerations

This release includes the new format installer where the files are extracted from the zip file into the desired directory. 
The software has been updated and compiled with Java version 1.8 so an appropriate OpenJava 1.8 version must be downloaded 
for the target Unix / Linux system.

The configuration file has been changed from Agent.config to Connector.config.

### New Features

### Fixes

**CONNUTIL-539**    
                    Update Unikix Connector for CVE-2021-44228 (remove log4j as the logging component and replace with slf4j and logback).
**CONNUTIL-545**    
                    Update Unikix Connector documentation.	


			
