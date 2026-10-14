# network-traffic-analyzer

A hands-on packet analysis project using Wireshark. I captured live traffic from a browsing session and examined how DNS, TCP and TLS work together when a browser loads a website.

## What this project covers

| Experiment | Filter | What I looked at |
|---|---|---|
| DNS | `dns` | A, AAAA and HTTPS queries, CNAME chains, resolver responses |
| TCP | `tcp` | Three-way handshake (SYN, SYN-ACK, ACK), ports, sequence numbers, options |
| TLS | `tls` | Client Hello, SNI, TLS 1.2 handshake steps, encrypted application data |

## Key findings

- A single page load triggers many DNS lookups, often for both IPv4 (A) and IPv6 (AAAA) addresses.
- `cdn.ndtv.com` resolved through a CNAME chain to an Akamai edge server (23.37.85.131).
- Browsers open several parallel TCP connections to the same server on port 443, each from a different source port.
- The server name (SNI) in the TLS Client Hello is visible in plaintext, while the actual content is encrypted.

## Repository structure

```
network-traffic-analyzer/
├── README.md
├── analysis-report.md
├── filters.md
├── network_capture.pcapng
└── screenshots/
    ├── dns-analysis.png
    ├── tcp-analysis.png
    └── tls-analysis.png
```

## How to reproduce

1. Install Wireshark and start a capture on your main network interface.
2. Open a few websites in your browser, then stop the capture.
3. Apply the filters `dns`, `tcp` and `tls` one at a time.
4. Click individual packets and inspect the details pane. Use `tcp.stream eq <n>` to follow a single connection.
5. Save the capture as `network_capture.pcapng`.

## Files

- [`analysis-report.md`](analysis-report.md): observation table, filters and lessons learned
- [`filters.md`](filters.md): display filters used
- [`screenshots/`](screenshots/): Wireshark screenshots for each experiment

## Tools

- Wireshark
- Firefox (traffic source)

## Note

The capture contains traffic from my own machine only. Don't commit captures that include credentials, cookies or other personal data.
