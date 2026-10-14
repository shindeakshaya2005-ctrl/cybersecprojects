# Network Traffic Analysis Report

## Setup

- Tool: Wireshark
- Host: 10.0.2.15 (client), local DNS resolver 192.168.1.1
- Capture A (DNS + TCP experiments): interface `any`, 54,268 packets total
- Capture B (TLS experiment): interface `eth0`, 18,838 packets total (`wireshark_eth0B7HOW3.pcapng`)
- Final saved capture: `network_capture.pcapng`

## Observations

| Observation | What I recorded |
|---|---|
| **DNS query** | `cdn.ndtv.com`, type **A** (packet 10651, ID `0x555b`, 10.0.2.15 → 192.168.1.1). The same host was also asked as **AAAA** (packet 10654). Other queries seen: `fonts.googleapis.com` (A, AAAA, HTTPS), `fonts.gstatic.com` (A, AAAA), `drop.ndtv.com` (A, AAAA, HTTPS), `c.ndtvimg.com` (A, AAAA, HTTPS). |
| **DNS response** | Packet 10657 (192.168.1.1:53 → 10.0.2.15:46711), flags `0x8180` (no error), 3 answers, 6.39 ms after the request. `cdn.ndtv.com` → CNAME `ipv6.ndtv.com.edgekey.net` → CNAME `e9448.dsca.akamaiedge.net` → **A 23.37.85.131**. Others: `fonts.googleapis.com` → 142.251.220.42 (AAAA 2404:6800:4009:80b::200a); `fonts.gstatic.com` → 192.178.174.94 (AAAA 2404:6800:4000:101f::5e). HTTPS-type queries returned a CNAME chain plus an SOA instead of an address. |
| **TCP connection** | Stream 43: **10.0.2.15:59256 → 142.251.220.42:443**. Three-way handshake: SYN (pkt 10659, Seq=0, Win=64240, MSS=1460, SACK_PERM, WS=512), SYN-ACK (pkt 10674, 443 → 59256, Seq=0 Ack=1, Win=65535, MSS=1460), ACK (pkt 10680, Seq=1 Ack=1). SYN to SYN-ACK took about 12 ms. Several parallel connections to the same server used source ports 59268, 59284, 59294, 59304 and 59308. A separate connection went to 23.37.85.131:443 from port 58518. |
| **TLS traffic** | Client Hello with SNI `fonts.googleapis.com` (TLSv1.3, 1961 bytes, to 142.251.220.42). In Capture B, a TLS 1.2 handshake with `firefox.settings.services.mozilla.com` (10.0.2.15 → 151.101.209.91): Client Hello (pkt 18643, 276 bytes), Server Hello, Certificate, Server Key Exchange + Server Hello Done, Client Key Exchange + Change Cipher Spec + Encrypted Handshake Message (pkt 18652), New Session Ticket, then Application Data. Visible metadata: SNI, cipher suite list, and ALPN (`h2`, `http/1.1`). A TLS 1.3 session to 34.107.243.93 (SNI `push.services.mozilla.com`) and QUIC Initial packets were also present. |
| **Overall traffic** | In Capture A, TCP made up 58.2% of packets (31,558 of 54,268) and DNS 3.3% (1,771). In Capture B, TLS made up 8.8% (1,665 of 18,838). Other protocols seen: QUIC. *(Add the top protocols from Statistics → Protocol Hierarchy.)* |

## Filters used

| Filter | Purpose |
|---|---|
| `dns` | Show all DNS queries and responses |
| `tcp` | Show all TCP packets, including handshakes |
| `tls` | Show TLS handshake and application data records |

Useful refinements to try: `dns.qry.name contains "ndtv"`, `tcp.flags.syn == 1`, `tcp.stream eq 43`, `tls.handshake.type == 1` (Client Hello), `tls.handshake.extensions_server_name`.

## What each experiment taught me

### DNS
Before any connection to a site, the host asks the resolver for its address. One page load triggered A, AAAA and HTTPS queries for each hostname. Responses can contain CNAME chains (here from ndtv.com through Akamai's edgekey.net to akamaiedge.net) before the final IP. Transaction IDs match each response to its request. DNS is unencrypted UDP, so anyone on the path can see which hostnames I look up.

### TCP
After DNS returns an IP, the client opens a TCP connection to port 443 with a three-way handshake: SYN, SYN-ACK, ACK. Sequence and acknowledgment numbers start at 0 in Wireshark's relative view, and the ACK number is the next byte expected. The SYN carried options (MSS, SACK, timestamps, window scale), while the SYN-ACK only offered MSS. Browsers open many parallel connections, each with a different ephemeral source port.

### TLS
Right after the TCP handshake, the client sends a Client Hello. The server name (SNI) is visible in plaintext, so an observer can tell which site I'm visiting even though the content is encrypted. In TLS 1.2 the handshake is visible step by step (Server Hello, Certificate, key exchange, Change Cipher Spec); after that only Application Data appears, and the payload can't be read.

## Screenshots

- `screenshots/dns-analysis.png`
- `screenshots/tcp-analysis.png`
- `screenshots/tls-analysis.png`
