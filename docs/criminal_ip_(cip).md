# Criminal IP(CIP)
Criminal IP is a Cyber Threat Intelligence (CTI) search engine by AI SPERA. It gives threat data collected by its own scanners: inbound and outbound risk scores for IP addresses, open ports and banners, vulnerabilities linked to public exploit code, VPN / Tor / proxy detection, and domain scan reports.

- Website: https://www.criminalip.io
- API documentation: https://search.criminalip.io/developer/api/get-asset-ip-report

## Prerequisites
- A Criminal IP account and API key (My Information page on criminalip.io)
- For the Threat Intelligence feed actions: a separate TI feed license key

## Authentication
1. In Shuffle, open **Apps**, search for **Criminal IP(CIP)** and activate it.
2. Add the app to a workflow and click **Authenticate**.
3. Enter your Criminal IP API key in **apikey**. The URL is `https://api.criminalip.io`.

### Second authentication for the TI feed
The Threat Intelligence feed actions (`Get threat intelligence feed`, `Get threat intelligence feed status`) use a different host: `https://dl.criminalip.io`.
You do not need a second app. Add a second authentication in the same app:

1. Click **Authenticate** again and create a new authentication (for example named `CIP(TI)`).
2. Set the URL to `https://dl.criminalip.io`.
3. On the two TI feed actions, select this authentication. Enter the TI feed license key in the `key` parameter.

The host of a request comes from the URL of the selected authentication, not from the app's base URL.
If the request still goes to `api.criminalip.io` after setting the URL, save the URL again in the App Authentication section of your account's Admin settings and run the workflow again.

## Actions
| Action | Endpoint | Main parameters | What it returns |
|---|---|---|---|
| Get IP report | GET /v1/asset/ip/report | ip (full: optional) | Full IP report: inbound/outbound score, open ports and banners, SSL, vulnerabilities, VPN/Tor/proxy, connected domains |
| Get IP report summary | GET /v1/asset/ip/report/summary | ip | Short version of the IP report |
| Get IP summary | GET /v1/asset/ip/summary | ip | Location, ISP and basic IP data |
| Get IP malicious info | GET /v2/feature/ip/malicious-info | ip | Whether the IP is malicious, with the reasons |
| Get IP suspicious info | GET /v2/feature/ip/suspicious-info | ip | Data suspected to be malicious for the IP |
| Get IP privacy threat | GET /v1/feature/ip/privacy-threat | ip | Privacy threat data for the IP |
| Get IP hosting check | GET /v1/ip/hosting | ip (full: optional) | Whether the IP belongs to a hosting provider |
| Get IP VPN and proxy check | GET /v1/ip/vpn | ip | Whether the IP is a VPN, Tor or proxy |
| Get safe DNS server check | GET /v1/feature/ip/is_safe_dns_server | ip | Whether the IP is a known safe DNS server |
| Get asset search results | GET /v1/asset/search | Queries: `query=<keyword>&offset=0` | Assets that match a search filter |
| Get banner search results | GET /v1/banner/search | Queries: `query=<keyword>&offset=0` | Banners that match a search filter |
| Get banner statistics | GET /v1/banner/stats | Queries: `query=<keyword>` | Statistics of banner search results |
| Get exploit search results | GET /v1/exploit/search | Queries: `query=cve_id:<CVE>&offset=0` | Exploits for a CVE or keyword |
| Get domain reputation | GET /v1/domain/quick/hash/view | domain | Quick verdict: `result` is `safe`, `unknown` or `malicious` |
| Get malicious domain check | GET /v1/domain/quick/malicious/view | domain | Whether the domain is malicious |
| Get trusted domain check | GET /v1/domain/quick/trusted/view | domain | Whether the domain is trusted |
| Get lite scan ID | GET /v1/domain/lite/scan | Queries: `query=<domain>` | Starts a lite domain scan (2-5 seconds), returns scan_id |
| Get lite scan progress | GET /v1/domain/lite/progress | scan_id | Progress of a lite scan |
| Get lite scan report | GET /v1/domain/lite/report/{id} | id (scan_id) | Lite scan report |
| Get lite scan report list | GET /v1/domain/lite/reports | offset (Queries: `query=<domain>`) | List of lite scan reports |
| Get domain scan ID | POST /v1/domain/scan | body: `query=<domain>` (form) | Starts a full domain scan, returns scan_id |
| Get private domain scan ID | POST /v1/domain/scan/private | body: `query=<domain>` (form) | Starts a private (not public) full domain scan |
| Get domain scan status | GET /v1/domain/status/{id} | id (scan_id) | Scan progress in percent |
| Get domain report | GET /v2/domain/report/{id} | id (scan_id) | Full domain scan report |
| Get domain report list | GET /v1/domain/reports | offset (Queries: `query=<domain>`) | List of domain reports |
| Get my domain report list | GET /v1/domain/reports/personal | offset, show_public, show_private, scan_type | Domain reports you scanned |
| Get user info | POST /v1/user/me | none (leave body empty) | Your account and remaining quota |
| Get threat intelligence feed | GET /ti/v1/feed/{direction} | key, direction (`inbound` / `outbound`), profile (`block` / `monitor`), confidence, format, limit, type | TI feed download (needs TI license key, once per hour per feed) |
| Get threat intelligence feed status | GET /ti/v1/feed/status | key | TI feed status |

### How to enter parameters
- **IP and domain lookups (GET)**: fill in the fields shown in the action (for example `ip`).
- **Search actions (GET)**: put the search keyword in the action's **Queries** field as `key=value&key2=value2`, for example `query=ssh&offset=0`. Use the Queries field, not separate fields.
- **Domain scan (POST)**: put the domain in **body** as form data: `query=example.com`. Leave Queries empty.
  The header `Content-Type: application/x-www-form-urlencoded` is required. With a JSON body (`{"query": "..."}`) Criminal IP returns `412 Missing Parameter`.
- **TI feed**: `direction` is part of the path (`/ti/v1/feed/inbound`). Do not put it in Queries.
- **Get user info (POST)**: no parameters. Leave the body empty.

## Example: check an IP
Action **Get IP report** with `ip` = `1.1.1.1`.

Example response (shortened):
```json
{
  "ip": "1.1.1.1",
  "score": { "inbound": "Safe", "outbound": "Safe" },
  "issues": { "is_vpn": false, "is_tor": false, "is_proxy": false, "is_scanner": false },
  "port": { "count": 2, "data": [ { "open_port_no": 443, "app_name": "...", "banner": "..." } ] },
  "status": 200
}
```

Score values are `Safe`, `Low`, `Moderate`, `Dangerous` and `Critical`.

## Example: domain full scan
1. **Get domain scan ID** with body `query=example.com` (form data) - returns `scan_id`.
2. Wait about 50 seconds (for example a Python step with `time.sleep(50)`). Most domains are done by then.
3. **Get domain scan status** with `id` = `scan_id` (from step 1) - `scan_percentage` is 100 when done.
4. **Get domain report** with `id` = `scan_id` (from step 1, not from the status step).

Tip: run **Get domain report list** with Queries `query=example.com&offset=0` first. If `count` is above 0, a public report already exists and you do not need a new scan.

The lite scan works the same way (Get lite scan ID → Get lite scan progress → Get lite scan report) and takes about 2-5 seconds.

## Example: check a CVE for public exploits
**Get exploit search results** with Queries `query=cve_id:CVE-2024-6387&offset=0`. `data.count` is the number of public exploits.

## Use in Shuffle
A common use is enrichment and blocking: an alert from a SIEM (for example Splunk) sends IPs to a webhook, **Get IP report** checks each IP, and a Python step decides whether to block it on the firewall and notify a channel (for example Slack).

## Notes
- Quota and rate limits depend on your Criminal IP plan. Check them with **Get user info**.
- The TI feed can be downloaded once per hour per feed. Use **Get threat intelligence feed status** first and download only when `feed_version` changed.
- Always set `limit` on the TI feed. Without it the feed returns up to 50,000 records, which is too large for later Python steps.

## References
- Criminal IP: https://www.criminalip.io
- Criminal IP API documentation: https://search.criminalip.io/developer/api/post-user-me
