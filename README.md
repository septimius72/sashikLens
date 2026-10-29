[threatlens_github_readme.md](https://github.com/user-attachments/files/34151235/threatlens_github_readme.md)
# ThreatLens // Open-Source CTI Aggregator & Daily Threat Briefing

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![Focus: FinTech & LegalTech](https://img.shields.io/badge/Focus-FinTech%20%7C%20LegalTech-00f2fe.svg)](#threat-profile)

**ThreatLens** is an open-source, lightweight Cyber Threat Intelligence (CTI) dashboard and daily briefing aggregator built for enterprise Threat Hunters, Detection Engineers, and SOC Analysts operating in high-risk verticals (**FinTech** and **LegalTech**). 

It consolidates high-fidelity feeds from the top 10 open-source repositories, correlates MITRE ATT&CK techniques, extracts Indicators of Compromise (IoCs), and provides ready-to-run **Osquery** and **Wazuh** hunting queries.

---

## Key Features

- **Daily Top-10 Ingestion:** Aggregates feeds from CISA KEV, The DFIR Report, abuse.ch (ThreatFox / URLhaus), AlienVault OTX, vx-underground, SigmaHQ, Malpedia, Shadowserver, Mandiant/Unit 42, and MISP communities.
- **Sector-Specific Risk Indices:** Real-time threat posture scoring specifically calibrated for Financial Services (BEC, SWIFT fraud, banking Trojans) and LegalTech (deal-room exfiltration, M&A insider threats, credential harvesting).
- **Adversary TTP & IoC Inspector:** Modal drawer parsing MITRE ATT&CK techniques with one-click clipboard copying for single or bulk IoCs.
- **Hunting Logic Out-of-the-Box:** Includes executable queries for:
  - **Osquery** (Endpoint state & file integrity)
  - **Wazuh** (Sysmon & SIEM rules)
  - **Sigma CLI / PySigma** (Vendor-agnostic detection translation)
- **One-Click Daily Briefing Export:** Export filtered operational digests directly into Markdown (for Slack/Teams briefings) or raw structured JSON (for ticketing/SOAR ingestion).
- **Zero-Budget & Zero-Dependency:** Runs client-side as a single responsive HTML application, or integrates into headless container pipelines.

---

## 🛰️ Monitored Sources

| Source | Category | Value to FinTech / LegalTech Hunts |
|---|---|---|
| **CISA KEV** | Exploit Catalog | Zero-day tracking on exposed perimeter devices & corporate VPNs. |
| **The DFIR Report** | Intrusion Playbooks | Real-world intrusion timelines, lateral movement, and dwell time. |
| **abuse.ch (ThreatFox/URLhaus)** | Malware C2 | Confirmed malicious botnet infrastructure, payloads, and hashes. |
| **SigmaHQ** | Universal Detections | Portable, community-tested behavioral detection logic. |
| **AlienVault OTX** | Community Pulses | Crowdsourced threat clusters, novel domain names, and ASNs. |
| **vx-underground** | Malware Advisories | Fast-breaking samples, source code leaks, and advisory drops. |
| **Malpedia** | Curated Signatures | YARA rules mapped to specific APT and eCrime families. |
| **Shadowserver** | Attack Surface | Exposure scans across external subnets, Redis, and message queues. |
| **Mandiant / Unit 42** | Threat Research | Deep-dive research on advanced nation-state espionage groups. |
| **MISP / FS-ISAC** | Financial Indicators | Sector-tailored indicator sharing and banking trojan TTPs. |

---

## Quick Start

### Option 1: Standalone Browser Usage (Zero Install)

1. Clone or download this repository:
   ```bash
   git clone https://github.com/your-org/threatlens.git
   cd threatlens
   ```
2. Open `threatlens_cti_dashboard.html` directly in any modern browser:
   ```bash
   # On macOS
   open threatlens_cti_dashboard.html

   # On Linux
   xdg-open threatlens_cti_dashboard.html

   # On Windows
   start threatlens_cti_dashboard.html
   ```

### Option 2: Run via Local Lightweight Web Server

```bash
# Using Python 3
python3 -m http.server 8080

# Using Node.js
npx serve .
```
Navigate to `http://localhost:8080/threatlens_cti_dashboard.html`.

---

## Repository Structure

```text
├── threatlens_cti_dashboard.html   # Complete, single-file interactive CTI application
├── scripts/
│   ├── fetch_cisa_kev.py           # Python fetcher for real-time CISA KEV JSON
│   ├── fetch_rss_feeds.py          # Script to pull RSS from DFIR Report & Unit 42
│   └── export_slack_digest.py      # Automated webhook poster for daily Slack briefing
├── rules/
│   ├── osquery_hunts.sql           # Curated Osquery detection hypotheses
│   └── wazuh_rules.xml             # Custom Wazuh decoders for FIN7 and Lumma
├── .github/
│   └── workflows/
│       └── daily_brief_bot.yml     # GitHub Action running headless briefing at 07:00 UTC
├── LICENSE
└── README.md
```

---

## Automating Live RSS & API Integration

To bypass browser CORS restrictions when fetching live feeds directly from third-party sources (e.g., CISA KEV JSON, DFIR Report RSS), use the included Python script or set up an open-source proxy like **CORS Anywhere** or a Cloudflare Worker:

### Automated Fetcher Script (`scripts/fetch_cisa_kev.py`)

```python
import json
import urllib.request

KEV_URL = "https://www.cisa.gov/sites/default/files/feeds/known_exploited_vulnerabilities.json"

def get_latest_vulnerabilities(limit=5):
    req = urllib.request.Request(KEV_URL, headers={"User-Agent": "ThreatLens-Collector/1.0"})
    with urllib.request.urlopen(req) as response:
        data = json.loads(response.read().decode())
        vulnerabilities = data.get("vulnerabilities", [])
        return vulnerabilities[:limit]

if __name__ == "__main__":
    latest = get_latest_vulnerabilities()
    print(json.dumps(latest, indent=2))
```

---

## Threat Hunting Queries Included

### 1. Osquery: Browser Credential Vault Access (Lumma Stealer Detection)
```sql
SELECT p.pid, p.name, p.path, f.path AS accessed_path
FROM processes p
JOIN process_open_files f ON p.pid = f.pid
WHERE f.path LIKE '%AppData%Google%Chrome%User Data%Default%Login Data%'
  AND p.name NOT IN ('chrome.exe', 'msedge.exe');
```

### 2. Wazuh / Sysmon Rule: FIN7 Domain Reconnaissance
```xml
<rule id="100340" level="12">
  <if_group>sysmon_event_1</if_group>
  <field name="win.eventdata.image">powershell.exe</field>
  <field name="win.eventdata.commandLine">.*Get-ADGroupMember.*Domain Admins.*</field>
  <description>Suspicious Active Directory Group Enumeration (FIN7 TTP)</description>
</rule>
```

---

## Contributing

Contributions are welcome! Please read `CONTRIBUTING.md` before submitting pull requests:
1. Fork the repository.
2. Create your feature branch (`git checkout -b feature/sigma-enhancements`).
3. Commit your changes (`git commit -m 'Add new BEC detection logic'`).
4. Push to the branch (`git push origin feature/sigma-enhancements`).
5. Open a Pull Request.

---

## License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.
