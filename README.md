# Checkpoint pack
This pack filters and reduces log data from Checkpoint Log Exporter.

## Requirements
Checkpoint Log Exporter is configured as per <link> to send data in Splunk format and Semi-Unified mode.

## Release Notes
I have never released this pack before

## Data Flow diagram
```mermaid
flowchart TD
subgraph pack[My New Pack]
    direction LR
    datatype>Data]-->data_filter(["true"])-->destination_data
end
syslog{{Data Source}}-->f;
f(["the filter to select relevant data "])-->pack;
pack-->output{{"Output#40;s#41;"}}
```

## Contributing to the Pack
Suggestions, comments, issues, and especially pull requests are welcome at https://github.com/jpvlsmv/cc-checkpoint-pack

## Contact
I can be reached via jpvlsmv@gmail.com

## License
This pack is released under the Apache-2 license, see LICENSE for details.
