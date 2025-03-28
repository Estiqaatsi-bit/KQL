# Identify Hosts Not Sending Logs in Sentinel 

## Description
This query helps you discover hosts that haven't sent logs in a while, the last log type and the days elapsed since the last log was sent. You can you use the query to look for specific hosts or any hosts that it is not sending logs. 

## Search for specific hosts
```
let devices = dynamic(['device1', 'device2']); 
let timeFrame = timespan(14d); 
// This function calculates the days since the last log was sent. 
let timeDiffDays = (startDate:datetime, endDate:datetime) {
    toint((endDate - startDate) / 1d)
}; 
union isfuzzy=true Device*
| where TimeGenerated > ago(timeFrame)
| where Type !in ('DeviceInfo', 'DeviceNetworkInfo')
| where DeviceName in (devices)
| summarize LastLogTimestamp = arg_max(TimeGenerated, Type) by Host
| extend ElapsedDays = timeDiffDays(LastLogTimestamp, now())
| project DeviceName, LastLogTimestamp, LastLogType = Type, ElapsedDays
```
## Search for any hosts
```
let timeFrame = timespan(14d); 
// This function calculates the days since the last log was sent. 
let timeDiffDays = (startDate:datetime, endDate:datetime) {
    toint((endDate - startDate) / 1d)
}; 
union isfuzzy=true Device*
| where TimeGenerated > ago(timeFrame)
| where Type !in ('DeviceInfo', 'DeviceNetworkInfo')
| summarize LastLogTimestamp = arg_max(TimeGenerated, Type) by DeviceName
| extend ElapsedDays = timeDiffDays(LastLogTimestamp, now())
| where ElapsedDays > 0
| project DeviceName, LastLogTimestamp, LastLogType = Type, ElapsedDays
```
| Version | Date     | Comments        |
|---------|----------|-----------------|
| 1.0     | 28/03/25 | Initial Publish |
