# Investigation of a Suspected Credential Phishing Domain Targeting Google Accounts

## Executive Summary
I analysed the domain seorx.vu, a domain crafted to impersonate Google's account login flow. Using only passive, publicly available tools (urlscan.io and VirusTotal), I confirmed the domain is hosted behind Cloudflare infrastructure and has been independently flagged as malicious by 18 of 92 security vendors on VirusTotal, as well as by urlscan.io and Google Safe Browsing.
The path structure (/accounts.google/) is a classic path-based brand impersonation technique, designed so that a victim, glancing at the URL bar, sees the string "accounts.google" and assumes it is legitimate. The real registered domain is seorx.vu, which has no relationship to Google.
This report documents the indicators, the analytical reasoning, and the conclusions drawn, entirely from passive sources. At no point was the suspected phishing page directly interacted with.

## Scope and Objectives

The investigation focused on the URL seorx.vu/accounts.google/, its underlying domain seorx.vu, the hosting infrastructure,  the TLS certificate, and its reputation across public threat intelligence sources.
This investigation was conducted as part of my ongoing OSINT learning. The goal was to practice a complete passive triage workflow against a real-world suspicious URL, moving from a single indicator (a URL) to a defensible verdict (phishing / not phishing) using only publicly available sources, and to produce a write-up that mirrors the structure of a real threat-intel report.
Boundaries: 
No direct browser visit to the suspect URL.
No credential submission, no sandbox detonation.
All findings were derived from third-party scan artefacts.

## Methodology

All analysis was conducted from passive, publicly available sources. No traffic was sent directly to the suspect domain from my environment.

#### Tools and sources:

Tool	| Purpose
---| ---| 
urlscan.io | Historical scan data, screenshots, DOM, HTTP transactions, infrastructure
VirusTotal | Multi-vendor reputation aggregation, detection labels
Google Safe | Browsing	Google's own malicious-site classification
WHOIS / DNS enrichment | Surface registration and resolution data (via urlscan.io)

#### Technique:

1. Parsed the URL structure to identify the true registered domain.
2. Pulled the most recent urlscan.io report for seorx.vu/accounts.google/ to gather infrastructure, screenshots, and behavioural data.
3. Cross-referenced the URL on VirusTotal to gauge multi-vendor consensus and review the specific detection labels.
4. Checked Google Safe Browsing status as a third independent signal.
5. Compared findings against common phishing tradecraft patterns to assess intent.


## Findings
#### Domain and URL structure
The full URL looks Google-related, but parsing the URL correctly:
- Registered domain: seorx.vu
- Path: /accounts.google/
The string accounts.google appears in the path, not the hostname.

This is a deliberate visual trick: many users scan a URL left to right and stop at familiar words. The legitimate Google sign-in page is https://accounts.google.com/. Note that .com is the TLD and accounts is a subdomain. Here, .vu is the TLD, and "google" is just a folder name.
The .vu ccTLD belongs to Vanuatu. While it has legitimate uses, it is uncommon outside of niche use cases (notably URL shorteners) and is occasionally abused for phishing because of relatively light registration scrutiny.
#### Hosting and infrastructure
DNS resolves seorx.vu to three Cloudflare edge IPs (188.114.96.3, 104.18.94.41, 104.18.95.41), all in AS13335 (CLOUDFLARENET). This means the true origin server is hidden behind Cloudflare's reverse proxy, a common operational choice for phishing operators because it:
- Masks the actual hosting provider.
- Provides free TLS certificates.
- Slows down takedown by abstracting the origin.
The TLS certificate is issued by Let's Encrypt (intermediate E7), valid for 90 days, a normal pattern for both legitimate sites and phishing kits, which often automate Let's Encrypt issuance.
#### Reputation signals
Three independent reputation sources converge on a malicious verdict:
- urlscan.io: "Potentially Malicious"; the scan summary explicitly notes the site has been scanned 5 times.
- VirusTotal: 18 of 92 vendors flagged the URL. Detections include BitDefender, ESET, Fortinet, Sophos, Emsisoft, G-Data, Forcepoint ThreatSeeker, Netcraft, Phishtank, alphaMountain.ai, Cluster25, CyRadar, Gridinsoft, LevelBlue, Lionic, SOCRadar, a diverse mix of commercial AV, reputation feeds, and specialist phishing trackers. Nearly every flag uses the label "Phishing" specifically, rather than generic "malicious".
- Google Safe Browsing: classified as malicious for seorx.vu.
The consistency across unrelated vendors, particularly specialist phishing trackers like Netcraft and Phishtank, is strong corroboration that this is a credential phishing page rather than, for example, a malware dropper or scam ad page.
#### Page content at scan time
The urlscan.io screenshot does not show a Google login clone, it shows a Cloudflare interstitial reading "Warning: Suspected Phishing. This website has been reported for potential phishing". This indicates Cloudflare itself has placed the domain behind its phishing warning page, which fires before the underlying content is served.
This is a useful analytical detail: the original phishing payload (likely a fake Google sign-in form) is no longer being served to scanners, but the infrastructure remains live. Operators can lift the warning by changing accounts, or pivot the same kit to a new domain. The "Screenshot now" panel returns no image, suggesting the page is either further degraded or rate-limiting scrapers.
The HTTP status of 403 returned to VirusTotal's most recent fetch is consistent with this: Cloudflare is gating access rather than the origin being offline.
#### Behavioural observations
From the urlscan report:
- 8 HTTP transactions across 2 domains and 2 subdomains.
- 75% HTTPS, 0% IPv6.
- 1 cookie set, ~121 kB page size, 43 kB transferred.
- All requests resolve within Cloudflare's ASN.
The small page size and single-cookie footprint are consistent with a lightweight static phishing kit or, currently, the Cloudflare warning page itself, not a fully featured legitimate application.


## Analysis
The evidence supports a high-confidence assessment that seorx.vu/accounts.google/ is a credential phishing page impersonating Google account sign-in. Several patterns reinforce this conclusion:

* Intent is visible in the URL itself. A domain with no legitimate relationship to Google has no benign reason to use /accounts.google/ as a path. The string exists purely to deceive users who don't parse URLs carefully — particularly on mobile, where address bars truncate aggressively.

* Multi-vendor consensus across unrelated detection methodologies. 18 vendors flag this URL, including specialist phishing trackers (Netcraft, Phishtank), commercial AV (BitDefender, ESET, Sophos), and threat-intel platforms (Cluster25, SOCRadar). When reputation signals converge across vendors with different data sources and detection logic, the risk of false positives drops sharply.

* Cloudflare’s interstitial indicates the domain has been reported and/or flagged by its abuse detection systems.

Tradecraft fits a recognisable pattern. The combination of:
* A non-mainstream ccTLD (.vu).
* Cloudflare-fronted hosting hiding the origin.
* Automated Let's Encrypt TLS.
* Path-based brand impersonation.

…is a recognisable signature of low-cost, kit-based credential phishing operations. None of these elements is malicious in isolation, but their combination is consistent with the pattern.

What cannot be determined from passive OSINT alone:
* The specific phishing kit family.
* The credential exfiltration channel (Telegram bot, gate.php, SMTP, etc.).
* The distribution vector (email, SMS, malvertising).
* Victim count or geography.
* Whether sibling domains exist using the same kit.

These would require sandbox detonation, paid threat-intel pivots, or DOM analysis of historical scans.


## Conclusion

Based on available passive OSINT indicators, seorx.vu/accounts.google/ is assessed with high confidence to be associated with credential phishing activity targeting Google account users.

Key takeaways for defenders:

* Block seorx.vu at DNS / proxy layers.
* Search mail gateway logs for any inbound message referencing seorx.vu in the last 60 days.
* Hunt for the path fragment /accounts.google/ across proxy logs — the same kit may resurface on sibling domains.
* Reinforce user awareness: the registered domain is what matters, not the path. Legitimate Google sign-in is always accounts.google.com.

Key takeaways for me as an analyst-in-training:

* Reading URLs correctly is foundational. The trick here is almost embarrassingly simple but extremely effective on a tired user.
* Passive OSINT alone produced a defensible verdict in under ten minutes — direct interaction with the suspect site was never necessary.
* Next time I want to pivot on the TLS certificate fingerprint and try urlscan.io's similarity search to discover sibling phishing pages.


  ## Appendix

#### Indicators of Compromise (Defanged)
* URLs: hxxps://seorx[.]vu/accounts[.]google/
* Domains :seorx[.]vu
* IP Addresses (Cloudflare Edge / CDN):
188[.]114[.]96[.]3
104[.]18[.]94[.]41
104[.]18[.]95[.]41
* Autonomous System:
AS13335 (CLOUDFLARENET)

Note: IPs belong to Cloudflare edge infrastructure and do not represent the origin server.

TLS Certificate
Issuer: Let’s Encrypt (E7)
Issued: 2026-04-27
Validity: 90 days (standard automated certificate lifecycle)

#### Timeline

Timestamp (UTC)	| Event
---| ---|
2026-04-27 | TLS certificate issued by Let's Encrypt (E7)
~2026-05-03	| VirusTotal last analysis (26 days prior to report)
2026-05-29 09:03:06	| urlscan.io scan submitted from GB, scanned from UK
2026-05-29 | Report compiled
