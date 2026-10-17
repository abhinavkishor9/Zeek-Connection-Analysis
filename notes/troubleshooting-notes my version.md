# Troubleshooting Notes

## 1. Environment

The lab was conducted using Zeek 8.0.10 in Ubuntu 24.04 on WSL.

Working directory:

    ~/zeek-labs/lab02

Zeek executable:

    /opt/zeek/bin/zeek

Zeek field-extraction utility:

    /opt/zeek/bin/zeek-cut

## 2. Preparing the Capture

The capture was copied from Lab 1 to keep the working directories separate.

    mkdir -p ~/zeek-labs/lab02
    cd ~/zeek-labs/lab02
    cp ~/zeek-labs/lab01/quickstart.pcap .

Verify the file:

    ls -lh quickstart.pcap
    file quickstart.pcap

The capture was approximately 2.4 KB and identified as a PCAP version 2.4 Ethernet file.

### If the File Cannot Be Found

Check the current directory:

    pwd

Check whether the original capture exists:

    ls -lh ~/zeek-labs/lab01/quickstart.pcap

If the path is incorrect, locate the actual file before copying it. Do not continue until the expected capture is available.

## 3. Generating Zeek Logs

The following command completed without a reported error:

    /opt/zeek/bin/zeek -r quickstart.pcap

List the generated logs:

    ls -lh *.log

The observed output included `conn.log`, `http.log`, `files.log`, `weird.log`, and `packet_filter.log`.

### If No Logs Appear

1. Confirm that the terminal is in the directory containing the PCAP.
2. Verify the capture filename and format.
3. Check the Zeek installation:

       /opt/zeek/bin/zeek --version

4. Run Zeek again and inspect any error messages.
5. Check the current directory for output files.

Do not assume every possible Zeek log will be generated for every capture. Log availability depends on the observed traffic and the logging behavior enabled.

## 4. Counting Connection Records

Command used:

    grep -vc '^#' conn.log

Observed output:

    2

Zeek log headers begin with `#`. This command counts the non-header lines in `conn.log`.

For this lab, the result confirmed that two connection records were available for analysis.

## 5. Extracting Fields with zeek-cut

Example command:

    /opt/zeek/bin/zeek-cut id.orig_h id.orig_p id.resp_h id.resp_p proto service duration conn_state < conn.log

This produces a compact view of selected connection fields.

### If a Field Is Not Recognized

- Check the field name against the `#fields` line in the log.
- Use the correct log file for the field.
- Preserve the exact spelling, including underscores.
- Confirm that the file is a Zeek log.

Example for HTTP records:

    /opt/zeek/bin/zeek-cut uid method host uri user_agent status_code status_msg < http.log

## 6. Filtering by Destination Port

Command used:

    /opt/zeek/bin/zeek-cut id.orig_h id.resp_h id.resp_p proto service conn_state < conn.log | awk '$3 == 80'

The extracted columns are:

1. Originator IP
2. Responder IP
3. Responder port
4. Transport protocol
5. Detected service
6. Connection state

The condition `$3 == 80` checks the third output column, which contains the responder port.

If the selected fields change, verify the column positions before reusing the `awk` condition.

## 7. Sorting Numeric Values

Sort by duration:

    /opt/zeek/bin/zeek-cut ts id.orig_h id.resp_h duration orig_bytes resp_bytes < conn.log | sort -t $'\t' -k4,4nr

Sort by originator bytes:

    /opt/zeek/bin/zeek-cut ts id.orig_h id.resp_h duration orig_bytes resp_bytes < conn.log | sort -t $'\t' -k5,5nr

The `-n` option enables numeric sorting, and `-r` reverses the order. The selected key must match the relevant column in the extracted output.

## 8. Counting Hosts and Ports

Count originator addresses:

    /opt/zeek/bin/zeek-cut id.orig_h < conn.log | sort | uniq -c | sort -nr

Count responder addresses:

    /opt/zeek/bin/zeek-cut id.resp_h < conn.log | sort | uniq -c | sort -nr

Count responder ports:

    /opt/zeek/bin/zeek-cut id.resp_p < conn.log | sort | uniq -c | sort -nr

These commands summarize repeated values. They do not determine whether the traffic is malicious.

## 9. Correlating Records by UID

Extract connection details:

    /opt/zeek/bin/zeek-cut uid id.orig_h id.resp_h id.resp_p service conn_state < conn.log

Extract HTTP details:

    /opt/zeek/bin/zeek-cut uid method host uri user_agent status_code status_msg < http.log

The `uid` field helps link records associated with the same connection.

In this lab, the connection to `192.0.78.150:80` correlated with an HTTP transaction whose method value was `WEIRD`.

## 10. Investigating unknown_HTTP_method

Review the anomaly log:

    cat weird.log

The observed event was `unknown_HTTP_method`, associated with the second connection.

### Interpretation

- Confirmed: Zeek recorded an unknown HTTP method.
- Not confirmed: That the request was an exploit or a malicious action.
- Recommended next step: Inspect the relevant packets and request bytes, if available, to establish the method's context.

An anomaly record is an investigation lead, not a final incident verdict.

## 11. Reviewing File Metadata

Command used:

    cat files.log

The inspected records reported `text/html`, 162 bytes observed, and 162 total bytes for each HTTP-related file entry.

The displayed records did not contain populated hashes or extracted file paths. Do not invent filenames, hashes, or file contents when those fields are unavailable.

## 12. Logs Not Fully Validated

`packet_filter.log` was generated, but its detailed contents were not established in the recorded output.

The observed generated-log listing did not include `dns.log` or `ssl.log`. This does not independently prove that DNS or TLS traffic was absent from the original environment.

When a log is missing:

1. Check whether the relevant traffic exists in the capture.
2. Confirm whether Zeek generated the relevant log.
3. Review the capture and configuration if necessary.
4. Document what was actually checked.

