# Checkpoint pack
This pack filters and reduces log data from Checkpoint Log Exporter.

## Requirements
Checkpoint Log Exporter is configured as per <link> to send data in Splunk format and Semi-Unified mode.  See
https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk122323 for more
information on how to configure the log exporter

## Release Notes
I have never released this pack before

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

## Contributing to the Pack
Suggestions, comments, issues, and *especially* pull requests are welcome at https://github.com/jpvlsmv/cc-checkpoint-pack

## Contact
I can be reached via jpvlsmv@gmail.com

## License
This pack is released under the Apache-2 license, see LICENSE for details.
