# Kusto Snippets

This repository is a collection of usefull query snippets for Kusto.
The focus is on hunting and detection, but there are also some general usefull snippets.

## Timeframes

### Today Backwards

Assuming Timestamp is the Time Field you're interested in.
In Sentinel it is often TimeGenerated instead.

This simple query will return all records from the last 24 hours.

```kusto
| where Timestamp > ago(1d)
```

Alternatively you can also use the between function like this.
It will return exactly the same values, but is a bit more human readable.

```kusto
| where Timestamp between (ago(1d) .. now())
```

### Time Between

If you want to specify a range, you can do so like this:

```kusto
| where Timestamp between (datetime(2024-12-31 00:00) .. datetime(2024-12-31 23:59))
```

## Variables

### Define a String

Simple String Variables can be used set like this
```kusto
let MyVariable = "MyValue;
```
And for example used in a where clause like this
```kusto
| where Field == MyVariable
```

### Array of Strings

Arrays are defined like this

```kusto
let sha256_whitelist = dynamic(["8a31d82daf77c252d9cd71ba69a24044495696d7c16c95e9505f235af0554f99",
                                "04dc71b15e42e515226deeebc95b6754b8cc64cc999e098bbb47d7a15b2041d0"]);
```

And then for example used in a where clause like this.
In this case not() was used to negate the result, since it's a whitelist.

```kusto
| where not(SHA256 has_any(sha256_whitelist))
```

### Loading External Data (CSV)
You can load external data from CSV files into Kusto using the `externaldata` function and then referencing it in your queries.
Here's an example of how to load a CSV file from a GitHub repository.

```kusto
let indicators = (externaldata(created_at:string,
                                entity_type:string,
                                objectLabel:string,
                                objectMarking:string,
                                observable_value:string,
                                updated_at:string,
                                value:string,
                                x_opencti_description:string,
                                x_opencti_score:int)
[@"https://raw.githubusercontent.com/WildDogOne/CTI/main/domain-name_24h.csv"] with (format="csv", ignoreFirstRecord=true));
```

And then you can use the variable in your query for example with an inner join.

```kusto
| join kind=inner indicators on $left.dnsquery == $right.value
```


### Result of a Subquery
You can also set a variable to the result of a subquery
```kusto
let test=DeviceProcessEvents | where Timestamp > ago(1d)| where FileName contains "certutil";
```
And then for example used in an antijoin like this.
The Usecases are varied, anti joins are usefull to exclude somethins found in a subquery.
Inner joins are used to enrich data with information from another table.
```kusto
DeviceProcessEvents
| join kind=anti (test) on DeviceName
```

## Expands / Extends

### extend
Expands are used to extract information from a field.
For example if you have a field that contains a path and you want to extract the filename.
Or if you have JSON content in a field and you want to extract a specific value.
```kusto
| extend FileName = tostring(split(FileName, "\\")[-1])
```

### MV Expand
Multivalue expands are extremely usefull if you have a field that contains multiple values,
and you want to extract them into separate records.
This will give you the ability to then search for individual values of that field, retaining the context of the original record.

```kusto
| mv-expand nameserver = split(RegistryValueData, " ")
```

## Aggregations / Grouping / Summarizing

### Count
Count is used to count the number of records.
```kusto
| summarize count = count() by nameserver
```

### Make Set
Make_set is used to create a list of unique values.
This is exeptionally useful for creating lists of unique values from a field that contains multiple values.

```kusto
| summarize nameservers = make_set(nameserver) by DeviceName
```

## Invoke

Invoke is one of the most powerful functions in Kusto.
However it is also not very easy to figure out what exists, and what is usefull.
So here some examples

### File Profile
File Profile is a function that will give you a lot of information about a file based on a hash.
FileProfile(x, y) where x is the hash and y is the file amount you want to enrich.
Default is 100, and max is 1000.
Hence in a hunting context it makes sense to always use 1000, and of course make sure that there are not more than 1000 different hash values.
Often it makes sense to summarize the data before invoking the function.

```kusto
| invoke FileProfile(InitiatingProcessSHA1, 1000)
```

Expected Resulting Fields:
- SHA1
- SHA256
- MD5
- FileSize
- GlobalPrevalence
- GlobalFirstSeen
- GlobalLastSeen
- Signer
- Issuer
- SignerHash
- IsCertificateValid
- IsRootSignerMicrosoft
- SignatureState
- IsExecutable
- SoftwareName
- ProfileAvailability

GlobalPrevalence for example is very usefull to see how common a file is, and might be an indicator of a false positive.
IsCertificateValid is usefull to see if a file is signed by a valid certificate, which is an indicator of trustworthyness.


## Geolocation

### IP to Location

If you have an IP Address and you want to know where it is located, you can use the geo_info_from_ip_address function.
```kusto
| extend location = geo_info_from_ip_address(RemoteIP)
```
