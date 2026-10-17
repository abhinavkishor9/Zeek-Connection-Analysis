# Zeek Lab 02 — Investigation Timeline

## 1. Lab Execution Timeline

### Phase 1 — Environment Setup

**Date:** October 17, 2026

- Created the working directory `~/zeek-labs/lab02`.
- Copied `quickstart.pcap` from the Lab 1 directory.
- Verified the capture size and format using `ls -lh` and `file`.
- Confirmed that the capture was a PCAP version 2.4 Ethernet file, approximately 2.4 KB in size.

### Phase 2 — Log Generation

**Date:** October 17, 2026

- Executed Zeek in offline analysis mode:

      /opt/zeek/bin/zeek -r quickstart.pcap

- Confirmed the presence of `conn.log`, `http.log`, `files.log`, `packet_filter.log`, and `weird.log`.
- Began analysis of the generated connection metadata.

### Phase 3 — Connection Analysis

- Counted non-header records in `conn.log`.
- Confirmed that two connection records were present.
- Extracted source and destination addresses, ports, protocol, service, duration, and connection state.
- Filtered records by destination port `80`.
- Compared duration and byte counts.
- Summarized originator IPs, responder IPs, and destination ports.

### Phase 4 — HTTP Correlation

- Extracted connection UIDs from `conn.log`.
- Extracted HTTP methods, hostnames, URIs, user agents, and response status codes from `http.log`.
- Correlated both HTTP transactions with their connection records.
- Observed that both transactions referenced `zeek.org`, URI `/`, user agent `curl/8.7.1`, and response `301 Moved Permanently`.

### Phase 5 — Anomaly and File-Metadata Review

- Reviewed `weird.log`.
- Identified an `unknown_HTTP_method` event associated with the connection to `192.0.78.150:80`.
- Correlated the event with the HTTP method value `WEIRD`.
- Reviewed `files.log` and observed two HTTP-related `text/html` entries, each reporting 162 bytes observed.
- Recorded that the detailed contents of `packet_filter.log` were not established in the captured output.

### Phase 6 — Assessment and Documentation

- Consolidated connection, HTTP, anomaly, and file-metadata observations.
- Separated confirmed observations from potential concerns.
- Documented that the reason for the unusual HTTP method and the actual redirect destinations remain unconfirmed.
- Recorded packet-level analysis as a recommended follow-up.

## 2. PCAP Event Timeline

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

## 3. Investigation Summary

The analysis identified two short-lived TCP/HTTP connections from `192.168.1.8` to different destination IP addresses on port `80`.

Both associated HTTP transactions referenced `zeek.org` and returned `301 Moved Permanently`. The second transaction contained the method value `WEIRD`, and Zeek recorded an `unknown_HTTP_method` event.

The observed records establish the connection pattern and protocol anomaly. They do not establish the actual redirect destinations, the cause of the unusual method, or malicious intent.

## 4. Follow-Up Actions

- Inspect the relevant packets to determine the actual HTTP request and method.
- Examine response headers to establish redirect destinations, if available.
- Inspect the HTML response content if sufficient data is present.
- Review `packet_filter.log` for additional capture-processing context.
- Update the timeline and investigation notes if further evidence changes the assessment.
