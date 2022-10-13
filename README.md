# Checkpoint pack
This pack filters and reduces log data from Checkpoint Log Exporter.  The output is compatible with the 
[Checkpoint App for Splunk](https://splunkbase.splunk.com/app/4293) while eliminating much of the redundant
event data.

## Quick Start
1. Install the pack
2. Configure checkpoint to export logs to workers
3. Create a global route that filters for the checkpoint data: Set the pipeline to cc-checkpoint-pack and specify 
   the Data Destination to your SIEM
   

## Data that is processed by the pack
| Event Type | pipeline | Sample Name      |
|------------|----------|------------------|
| Firewall   | [Firewall](default/pipelines/firewall/README.md) | cp_fw_onesession |
| IPS   | [IPS](default/pipelines/ips/README.md) | cp_ips |

   
## Your own fields
1. Have a sample of the data you want to aggregate
2. Clone the `aggregating` pipeline to your own pipeline name and Preview your sample
3. Note the field names embedded in the `unrecognized_fields` column (turn off step `14[F] Eval`)
4. Decide what to do with it.  Add it to `3. Eval Explicitly remove fields...` or to both `5. Serialize` (as
!fieldname) and `8. Aggregations` with the appropriate aggregation function.
5. Create a route in the pack to direct the appropriate data to your new pipeline

## Requirements
Checkpoint Log Exporter is configured as per <link> to send data in Splunk format and Semi-Unified mode.  See 
[Checkpoint KB sk122323](https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk122323) 
for more information on how to configure the log exporter

The pack is tested with log exporter configured to send via UDP and port 514.  While port 514 is usually used for 
syslog messages, the data sent is not formatted that way.  If the worker data source is also used for syslog messages,
the checkpoint data needs to be distinguishable.   If using the cribl-syslog-input pack, the CriblSyslogPreProcessing 
pipeline should not be configured to "Drop anything that arrived on a syslog port, but that isn't really syslog".  
Instead, the option to "Just ... do nothing" should be enabled.

The `parse_splunkformat` pipeline can be used as a Data Source preprocessing pipeline, or it can be enabled in the 
routing defined in the pack.

## Motivation
Traffic data across firewalls is a key component of network intrusion detection, but can be a large data set.

When configured as documented, however, the Checkpoint log export sends a lot of redundant data that can be filtered 
out without losing fidelity to the underlying traffic flow.  This is because the events that are exported represent a 
point in time from a long-running stream of activity that needs to be logged.  Every stream of activity is given 
a unique identifier (`loguid`) and a series of events is sent with an increasing `sequencenum` value.  

When configured in *semi-unified* read-mode, the log exporter will occasionally send an event that includes all of
the history for that stream.  In this mode, a SIEM can discard all but the very last record for each loguid, since 
the last update will include data from all previous.  Unfortunately, determining what is the last update requires 
knowledge of the future, which most SIEM platforms have difficulty with, not to mention the difficulty of 
discarding the prior events.  Each intermediate event that would be discarded later still consumes resources in the 
SIEM -- storage and processing, often with significant impact on licensing cost.

To visualize this, if you look at the `cp_fw_onesession` sample (firewall pipeline, IN mode, show as columnar)
, you will see that there are multiple events with the same `loguid` which build upon each other. In this example 
if you observe the `bytes` field, we first see a log with 4568 bytes, then 1 second later we get an entire new log 
event updating it to 27897 bytes.  At some point between these two, the client sent 44 packets out to the external
web server, and the web server returned 20 or so.  As you can also see, nearly all of the fields in these records
are identical.  The OUT view of this sample shows the single event that would be passed along to the output, after
a 91% reduction in _raw length.

This pack can place Cribl Stream between the log exporter and the SIEM.  The Stream worker will act as a short-term 
cache for the semi-unified log records, and if an update is received for a specific loguid, will retain only the most 
recent event.  This results in a significant reduction in SIEM volume, since most connections will be updated several 
times over their lifetime.  

In one real-world practice, this pack has shown a 20:1 reduction in SIEM volume sent to the Splunk instance, from 
700GB/day to under 35GB/day.

## Data Flow diagram
```mermaid
flowchart TD
syslog("Checkpoint<br>Log Exporter<br><pre>cp_log_export add name cribl target-server cribl-worker target-port cribl-port protocol udp format splunk read-mode semi-unified</pre>") -->|network| f[Data Source];
subgraph pf[Optional preprocessing pipeline]
    direction LR
    sfsu[Splunk semi-unified parser] --> pd[Parsed fields]
    c["(Attach to data source or enable below)"]
end
f --> pf[Optional preprocessing<br>pipeline] ;
subgraph pack[My New Pack]
    direction LR
    opt("Optional preprocessing pipeline<br>(disable if using at input stage)")
    datatype>Structured <br>input Data]-->data_filter(["product==Firewall"])--> 
    pipe[Firewall fields<br>reduction and aggregation] --> o[structured output data] -->
    res[reserialize to _raw];
    datatype2>Structured <br>input Data]-->data_filter2(["product== filter"])--> 
    pipe2[per-product pipeline<br>and aggregation] --> o2[structured output data] -->
    res2[reserialize to _raw];
    datatype3>Structured <br>input Data]-->data_filter3(["product== filter"])--> 
    pipe3[per-product pipeline<br>and aggregation] --> o3[structured output data] -->
    res3[reserialize to _raw];
end
pf --> rt[Global Routing] --> pack;
pack --> out[Output routing]
```

## Caveats
It is worth metioning a few caveats to the use of this pack:
* The Checkpoint log exporter still sends all of the data across its network.  That can be a large volume, especially 
    if one log export station is serving multiple busy firewall instances.  When using UDP as its transport protocol, 
    there is no feedback to the exporter if data is being lost or dropped.  The Cribl Stream worker group must be 
    configured to be large enough to support peak workloads.
    
* The Cribl worker's aggregation functions (used to implement the short-term cache) operate only on the data available 
    to a single worker thread on a single worker instance.  This means that if updates arrive to a different worker 
    (server) or worker process, it will be kept as a separate aggregate on that worker.  Both events will eventually 
    be passed to the SIEM.  This can impose a limit on how horizontally-scaled worker groups will reduce the data volume.
    
* The parse_splunkformat pipeline can run as a preprocessing pipeline, or it can be called as part of the pack routing.
    The other pipelines assume that the raw data has been split and parsed and will fail without that parsing pipeline 
    being run at least once.  Enabling it more than once is probably not harmful, but would be a waste of resources.

## Contributing to the Pack
Suggestions, comments, issues, and *especially* pull requests are welcome at https://github.com/jpvlsmv/cc-checkpoint-pack

## Contact
- Joe Moore [Github](https://github.com/jpvlsmv)
- JP Bourget [Twitter](https://twitter.com/jp_bourget) | [Github](https://github.com/punkrokk)

## License
This pack is released under the Apache-2 license, see LICENSE for details.
