---
layout: post
title:  "Windows Forensics 101 - Event Logs"
date:   2025-02-08 14:53:00 +0100
categories: ["Windows Forensics 101"]
tags : [forensics,chainsaw,fulleventlogview,evtx]
---

Windows Event Logs 

## What are event logs? 

Event logs are detailed records of system, security, and application events that occur on a Windows operating system. These logs are stored in files with the `.evtx` extension and can be viewed using the System Event Viewer tool. They are crucial for troubleshooting issues, monitoring system health, and conducting forensic investigations.

## Where are they stored? 

Event logs are stored in location: 

```text
 C:\Windows\System32\winevt\Logs\ 
```

![img-description](/assets/img/windows-evtx_logs.png)
_Events in file system_


## How data can be analyzed

These files can be parsed using system tool called Event Viewer, or using third party software like for instance [**NirSoft FullEventLogView**][nfulleventlogview]. 
This allows filtering events or serching with certain EventIDs.

![img-description](/assets/img/forensics-windows_eventlogs.png)
_Events loaded into FullEventLogView_

*.evtx files can be also ripped using [**Chainsaw**][chainsaw_link] which is tool developed by WithSecure Countercept.

![img-description](/assets/img/events-chainsaw.png)
_Chainsaw_


[chainsaw_link]:<https://github.com/WithSecureLabs/chainsaw>
[nfulleventlogview]: <https://www.nirsoft.net/utils/full_event_log_view.html>