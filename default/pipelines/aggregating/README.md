# Example Pipeline for aggregating checkpoint data

## Field dictionary

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
