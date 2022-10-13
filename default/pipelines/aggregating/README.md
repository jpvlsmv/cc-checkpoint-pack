# Example Pipeline for aggregating checkpoint data

## Field dictionary
To get a list of all the different field names in a sample, you can download it as ndjson and run
```shell
jq -sr '.[]._raw' < yoursample.json | tr \| \\n | cut -d= -f 1| sort | uniq -c | sort -n
```
This can give you an idea of which fields should be "recognized" first.

To see the variations in a particular field, so that you can assess what different values you
might be aggregating from,
```shell
jq -sr '.[]._raw' <~/cp_ips.log.json | tr \| \\n | sort | uniq -c | sort -n | grep yourfield
```


### Remove fields
|Field Spec|Why|
|----------|---|

### Fields to aggregate
Need to be listed !fieldname in `5. Serialize` and with agg-function in `8. Aggregations`.
If aggregation function is not listed, `last` is used to capture the most recent value.

|Aggregation|Field|Reason|
|-----------|-----|------|
|            min|sequencenum|
|            max|sequencenum|
|            values|unrecognized_fields|
