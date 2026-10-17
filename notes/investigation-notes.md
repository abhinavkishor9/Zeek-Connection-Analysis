# Zeek Lab 02 — Connection Analysis

## 1. Environment

- Tool: Zeek 8.0.10
- Operating environment: Ubuntu 24.04 on WSL
- Analysis mode: Offline PCAP
- Capture filename: `quickstart.pcap`
- Working directory: `~/zeek-labs/lab02`
- Capture size: Approximately 2.4 KB
- Connection records: 2

## 2. Evidence Reviewed

- `conn.log` — two connection records inspected.
- `http.log` — two HTTP transaction records inspected.
- `weird.log` — `unknown_HTTP_method` event inspected.
- `files.log` — two HTTP-related file records inspected.
- `packet_filter.log` — generated, but detailed contents were not established in the recorded output.
- DNS/TLS logs — not present in the observed generated-log listing.

## 3. Findings

### Connection Overview

Two TCP connections originated from `192.168.1.8` and targeted different destination IP addresses on port `80`.

| Field | Connection 1 | Connection 2 |
|---|---|---|
| UID | `CT9flN127BQj0Utuga` | `CSgp6l4zrX00LAX6Hi` |
| Source IP | `192.168.1.8` | `192.168.1.8` |
| Source port | `52917` | `52918` |
| Destination IP | `192.0.78.212` | `192.0.78.150` |
| Destination port | `80` | `80` |
| Protocol | TCP | TCP |
| Detected service | HTTP | HTTP |
| Duration | `0.098478` seconds | `0.100107` seconds |
| Originator bytes | `71` | `73` |
| Responder bytes | `377` | `377` |
| Connection state | `SF` | `SF` |
| Missed bytes | `0` | `0` |

Both connection records have state `SF` and history `ShADadFf`. Both reported zero missed bytes.

### HTTP Activity

The HTTP records were correlated with the connection records using their UIDs.

| Field | Connection 1 | Connection 2 |
|---|---|---|
| UID | `CT9flN127BQj0Utuga` | `CSgp6l4zrX00LAX6Hi` |
| Method | `GET` | `WEIRD` |
| Host | `zeek.org` | `zeek.org` |
| URI | `/` | `/` |
| User agent | `curl/8.7.1` | `curl/8.7.1` |
| Status code | `301` | `301` |
| Status message | `Moved Permanently` | `Moved Permanently` |

Both transactions reference the same HTTP host and URI and report the same user agent and response status.

The actual redirect destinations were not established from the inspected fields.

### Protocol Anomaly

The inspected `weird.log` record contained the following details:

- Connection UID: `CSgp6l4zrX00LAX6Hi`
- Source: `192.168.1.8:52918`
- Destination: `192.0.78.150:80`
- Event name: `unknown_HTTP_method`
- Additional information: `-`

The correlated HTTP record reports the method value `WEIRD`.

This confirms that Zeek recorded an unknown HTTP method for the second transaction. The reason for the method value has not been established.

### File-Analysis Metadata

Two records were observed in `files.log`. Both were associated with the HTTP connections and reported:

- MIME type: `text/html`
- Bytes observed: `162`
- Total bytes: `162`
- Missing bytes: `0`
- Extracted file path: not populated
- File hashes: not populated in the displayed records

The metadata indicates that Zeek recorded HTML-related content. It does not establish whether the content was benign or malicious.

### Communication Patterns

- The source host `192.168.1.8` initiated two connections.
- Each connection targeted a different destination IP.
- Both connections used TCP destination port `80`.
- Both connections were identified as HTTP.
- Each destination appeared once in the connection records.
- The small capture does not establish a broader communication baseline.

## 4. Assessment

### Confirmed Observations

- Two connection records were present in `conn.log`.
- Both connections originated from `192.168.1.8`.
- Both connections targeted destination port `80`.
- Both records reported state `SF` and zero missed bytes.
- Both HTTP transactions referenced `zeek.org` and returned status `301 Moved Permanently`.
- The second HTTP transaction contained method value `WEIRD`.
- `weird.log` recorded `unknown_HTTP_method` for the second connection.
- `files.log` contained two HTTP-related `text/html` records with 162 bytes observed per record.

### Potential Concerns

The unknown HTTP method is the primary anomaly identified in this capture. It warrants examination of the raw request and relevant packets to establish its context.

The use of two destination IP addresses for the same HTTP host is another observation to contextualize. It is not sufficient, by itself, to classify the activity as suspicious or malicious.

### Evidence Limitations

- The capture contains only two connection records.
- The reason for the unknown HTTP method has not been established.
- The actual redirect destinations have not been confirmed.
- The HTML response contents have not been independently examined.
- File hashes were not populated in the displayed `files.log` records.
- The detailed contents of `packet_filter.log` were not established in the recorded output.
- The absence of DNS or TLS logs from the generated-file listing does not prove that such traffic was absent from the original environment.
- The available evidence does not establish whether the observed activity was malicious.

### Recommended Follow-up

1. Inspect the relevant packets to understand the exact HTTP request and method.
2. Examine response headers and determine the actual `Location` value, if available.
3. Inspect the HTML response content if the capture contains sufficient data.
4. Review `packet_filter.log` for capture-processing context.
5. Compare the observed method with expected client behavior.
6. Avoid assigning a malicious classification unless additional evidence supports it.

## Conclusion

The capture shows two short-lived TCP/HTTP connections from `192.168.1.8` to two different destination IP addresses on port `80`. Both HTTP transactions reference `zeek.org` and return `301 Moved Permanently`.

The second transaction contains an unknown HTTP method, and Zeek records an `unknown_HTTP_method` event. The available evidence does not establish why it occurred or whether the activity was malicious. Further packet-level analysis is required before reaching a stronger conclusion.
