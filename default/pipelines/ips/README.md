# Pipeline to process product=IPS data

## Field dictionary

### Remove fields
|Field Spec|Why|
|----------|---|
### Fields to aggregate
Need to be listed !fieldname in `5. Serialize` and with agg-function in `8. Aggregations`.
If aggregation function is not listed, `last` is used to capture the most recent value.

|Aggregation|Field|Reason|
|-----------|-----|------|
|            min|sequencenum||
|            max|sequencenum||
|            values|unrecognized_fields||
|            |action||
|            |attack||
|            |attack_info||
|            |confidence_level||
|            |dst||
|            |ifdir||
|            |ifname||
|            |origin||
|            |originsicname||
|            |performance_impact||
|            |product||
|            |protection_id||
|            |protection_name||
|            |protection_type||
|            |proto||
|            |rule||
|            |rule_name||
|            |rule_uid||
|            |s_port||
|            |logid||
|            |match_id||
|            |message_info||
|            |origin||
|            |originsicname||
|            |outzone||
|            |product||
|            |proto||
|            |protocol||
|            |service||
|            |severity||
|            |smartdefense_profile||
|            |src||
|            |sub_policy_name||
|            |version||
|            |src||
|            min|start_time||
|            min|time||
|            max|time||
|            |src_uo_name||
|            values|hll_key||
