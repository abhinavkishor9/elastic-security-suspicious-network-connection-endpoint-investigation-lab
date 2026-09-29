# Timeline 

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

