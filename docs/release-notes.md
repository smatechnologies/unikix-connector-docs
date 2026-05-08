---
sidebar_label: "Release notes"
title: Unikix Connector release notes
description: "Version history and change details for the SMA OpCon Unikix Connector, including new features, improvements, and bug fixes."
tags:
  - Reference
  - System Administrator
  - Automation Engineer
---

# Unikix Connector release notes

## 21

### 21.0

### What's new

:eight_spoked_asterisk: Replaced log4j with slf4j and logback as the logging component.

:eight_spoked_asterisk: Adopted a new installer format. Files are now extracted from the zip file into the desired directory.

:eight_spoked_asterisk: The configuration file has been renamed from `Agent.config` to `Connector.config`.

### Migration considerations

The connector has been compiled with Java 1.8. An appropriate OpenJDK 1.8 distribution must be installed on the target Unix or Linux system before upgrading.

The configuration file has been renamed from `Agent.config` to `Connector.config`. Carry over your existing settings to the new file before starting the connector.

### Bug fixes

:white_check_mark: **CONNUTIL-539**: Updated the Unikix Connector to address CVE-2021-44228 by removing log4j and replacing it with slf4j and logback.

:white_check_mark: **CONNUTIL-545**: Updated the Unikix Connector documentation.
