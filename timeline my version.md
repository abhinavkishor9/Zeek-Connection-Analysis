# Investigation Timeline


## PCAP Event Timeline

The timestamps below are recorded network-event timestamps from the Zeek logs. They are separate from the date and time when the lab commands were executed or the logs were generated.

| PCAP event timestamp | Connection UID | Source | Destination | Observed event |
|---|---|---|---|---|
| `1747147647.668533` | `CT9flN127BQj0Utuga` | `192.168.1.8:52917` | `192.0.78.212:80` | First TCP connection record; HTTP service detected |
| `1747147647.702181` | `CT9flN127BQj0Utuga` | `192.168.1.8:52917` | `192.0.78.212:80` | HTTP transaction with method `GET` and status `301` |
| `1747147647.735035` | `CT9flN127BQj0Utuga` | `192.168.1.8:52917` | `192.0.78.212:80` | HTTP-related file metadata recorded |
| `1747147654.275660` | `CSgp6l4zrX00LAX6Hi` | `192.168.1.8:52918` | `192.0.78.150:80` | Second TCP connection record; HTTP service detected |
| `1747147654.311012` | `CSgp6l4zrX00LAX6Hi` | `192.168.1.8:52918` | `192.0.78.150:80` | HTTP transaction with method `WEIRD` and status `301` |
| `1747147654.311012` | `CSgp6l4zrX00LAX6Hi` | `192.168.1.8:52918` | `192.0.78.150:80` | `unknown_HTTP_method` event recorded in `weird.log` |
| `1747147654.341780` | `CSgp6l4zrX00LAX6Hi` | `192.168.1.8:52918` | `192.0.78.150:80` | HTTP-related file metadata recorded |

