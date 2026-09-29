# Timeline — Suspicious Network Connection from Endpoint

## Investigation Timeline

| Time | Activity | Process / Artifact | Evidence / Notes |
|---|---|---|---|
| Before request | DNS resolution | `example.com` | Resolved to `172.66.147.243`, `104.20.23.154`, and an IPv6 address |
| 04:13:21.732 | Controlled process execution | `curl.exe` PID `12968` | Parent `pwsh.exe` PID `34288`, user `Dell` |
| 04:13:21.732 | Command-line correlation | `curl.exe` | Command line referenced `https://example.com` |
| 04:13:21.865 | Controlled network connection | `curl.exe` PID `12968` | Destination `172.66.147.243:443`, TCP |
| 04:18:25.756 | Existing HTTPS activity | `Photos.exe` | Destination `13.107.5.93:443` |
| 04:19:09.510 | Existing HTTPS activity | `chrome.exe` | Destination `69.173.158.64:443` |
| 04:19:13.204 | Existing HTTPS activity | `chrome.exe` | Destination `64.239.123.1:443` |
| 04:20:53.286 | Existing HTTPS activity | `chrome.exe` | Destination `64.239.123.1:443` |
| 04:22:17.887 | Existing HTTPS activity | `chrome.exe` | Destination `104.18.32.47:443` |
| 04:22:26.801 | Existing HTTPS activity | `chrome.exe` | Destination `172.64.155.209:443` |
| Investigation | Local TCP validation | Chrome / other processes | Multiple established TCP/443 connections observed |

## DNS Resolution

The controlled destination was resolved using:

```powershell
Resolve-DnsName example.com
```

Observed:

```text
172.66.147.243
104.20.23.154
```

and:

```text
2606:4700:9765:72db:f2a5:0:ef6b:ff98
```

The Elastic network event later referenced:

```text
172.66.147.243
```

## 04:13:21.732 — Controlled Process Execution

Elastic recorded:

```text
Process: curl.exe
PID: 12968
Parent: pwsh.exe
Parent PID: 34288
User: Dell
Executable: C:\Windows\System32\curl.exe
```

Command line referenced:

```text
https://example.com
```

This established:

```text
pwsh.exe
    |
    +-- curl.exe
```

## 04:13:21.865 — Controlled Network Event

Elastic recorded:

```text
Process: curl.exe
PID: 12968
User: Dell
Destination: 172.66.147.243
Destination Port: 443
Transport: tcp
```

This established:

```text
curl.exe
    |
    +-- TCP
          |
          +-- 172.66.147.243:443
```

## HTTP Response

The controlled request:

```powershell
curl.exe -I https://example.com
```

returned:

```text
HTTP/1.1 200 OK
```

The response contained normal HTTP headers.

No payload was downloaded or executed.

## Existing HTTPS Activity

Broader network telemetry showed additional HTTPS activity.

Examples included:

```text
chrome.exe → 172.64.155.209:443
chrome.exe → 104.18.32.47:443
chrome.exe → 64.239.123.1:443
Photos.exe → 13.107.5.93:443
```

This demonstrated that TCP/443 was common on the endpoint.

## Local TCP Validation

The endpoint's established connections were inspected using:

```powershell
Get-NetTCPConnection -State Established |
Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, State, OwningProcess
```

The results showed multiple applications with established TCP/443 connections.

Chrome PID `24112`, for example, had connections to:

```text
76.223.31.44:443
104.18.39.21:443
```

This reinforced the need to associate network activity with the owning process.

## Final Assessment

```text
DNS Resolution
      ↓
example.com
      ↓
curl.exe execution
      ↓
Process correlation
      ↓
172.66.147.243:443
      ↓
TCP connection
      ↓
HTTP 200 response
      ↓
Network baseline comparison
      ↓
Benign controlled activity
```

The controlled connection was consistent with the intended HTTPS request to `example.com`.

No command-and-control, malware communication, data exfiltration, credential theft, persistence, or confirmed compromise was demonstrated.

The main investigation lesson is that **network connections should be analyzed together with the process, user, command line, destination, and surrounding endpoint activity rather than by port number alone**.
