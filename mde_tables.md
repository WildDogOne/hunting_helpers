# MDE Tables

This Document describes some interesting locations for hunting in Defender for Endpoint

## DeviceEvents

| ActionType        | Description                                       | Parse                                                                               |
| ----------------- | ------------------------------------------------- | ----------------------------------------------------------------------------------- |
| PowerShellCommand | Has Powershell command blocks in AdditionalFields | extend Command = tostring(parse_json(AdditionalFields)["Command"])                  |
| DnsQueryResponse  | Has DNS Query Results                             | extend DnsQueryResults = parse_json(parse_json(AdditionalFields)["DnsQueryResult"]) |
| ScriptContent     | Contents of Scriptfiles                           |

## DeviceNetworkEvents

| ActionType | Description | Parse |
| ---------- | ----------- | ----- |
