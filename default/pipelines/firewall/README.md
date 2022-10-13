# Pipeline to process product=Firewall data

## Samples
|Sample name|Use|
|-----------|---|
|cp_fw_onesession|Log records received for a single loguid.  Demonstrates the field reductions and aggregations|

## Field dictionary

### Remove fields
|Field Spec|Why|
|----------|---|
|bytes|Use client_*_bytes instead|
|client_inbound_interface|name of fw network interface where packets from server->client show up.  not important|
|client_outbound_interface|name of fw network interface where packets from client->server show up.  not important|
|cribl_breaker|processing artifact|
|dst_uo_icon|icon to display for destination|
|elapsed|how long a session has been in place.  not important|
|ifname|name of a firewall network interface.  not important|
|lastupdatetime|last time this loguid's records were updated|
|layer_uuid|not useful unless deep in the weeds of firewall troubleshooting, use names instead of uuids|
|log_delay|how long the update sat in the log exporter|
|nat_addtlnl_rulenum|see next|
|nat_rulenum|not useful unless deep in the weeds, essentially a unique id for nat_rule|
|needs_browse_time|not sure what this field represents|
|packets|total packets on this session.  use client_*_packets instead|
|parent_rule|the name that called the current rule.  not important|
|rule_uid|Use names instead of uuids|
|segment_time|does not seem to vary from other timestamps|
|server_inbound_*|Use client_* instead|
|server_outbound_*|Use client_* instead|
|src_uo_icon|icon to display for source, useless|
|version|a constant|

### Fields to aggregate
Need to be listed !fieldname in `5. Serialize` and with agg-function in `8. Aggregations`.
If aggregation function is not listed, `last` is used to capture the most recent value.

|Aggregation|Field|Reason|
|-----------|-----|------|
|            min|sequencenum|
|            max|sequencenum|
|            values|unrecognized_fields|
|            |action|
|            |alert|
|            |client_inbound_bytes|
|            |client_inbound_packets|
|            |client_outbound_bytes|
|            |client_outbound_packets|
|            |conn_direction|
|            |context_num|
|            |contextnum|
|            |dst|
|            |dst_uo_name|
|            |fw_message|
|            values|hll_key|
|            |host|
|            |hostname|
|            |icmp|
|            |icmp_code|
|            |icmp_type|
|            |ifdir|
|            |inzone|
|            |layer_name|
|            |logid|
|            |match_id|
|            |message_info|
|            |origin|
|            |originsicname|
|            |outzone|
|            |product|
|            |proto|
|            |protocol|
|            |reason|
|            |rule_action|
|            |time|
|            values|rule_uid|
|            values|rule_name|
|            |s_port|
|            |service|
|            |service_id|
|            values|sig_id|
|            |source_object|
|            |src|
|            min|start_time|
|            min|time|
|            max|time|
|            |src_uo_name|
|            |xlatedport|
|            |xlatedst|
|            |xlatesport|
|            |xlatesrc|
|            values|unrecognized_fields|
