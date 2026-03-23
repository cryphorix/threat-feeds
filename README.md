# Threat Feeds

Daily threat intelligence feeds - newly registered domains, DNS zone metadata, and zone transfer (AXFR) detection. Updated automatically.

Powered by **ICANN CZDS zone files** (1,000+ TLDs), **Certificate Transparency logs**, **passive DNS**, and **community threat intel sources**.

## Current Stats

- **63+ million** unique domains in the database
- **2+ million** new domains ingested daily on average
- **825 TLD zone files** parsed daily with full NS, DS, and glue records
- AXFR zone transfer scanning across all indexed nameservers
- Feeds are updated and pushed to this repo every day at 06:00 UTC

## Feeds

### NRD (Newly Registered Domains)

| File | Description | Update Frequency |
|------|-------------|-----------------|
| `nrd-today.txt` | Domains ingested in the last 24 hours | Daily |
| `nrd-week.txt` | Domains from the last 7 days | Daily |
| `czds-nrd.txt` | CZDS zone file new domains (last 24h) | Daily |
| `ct-domains.txt` | CT log discoveries (last 24h) | Daily |
| `nrd-full.txt` | Complete master list (all domains in DB) | Daily |

### DNS Zone Intelligence

| File | Description | Update Frequency |
|------|-------------|-----------------|
| `zone-nameservers.csv` | Top nameservers ranked by domain count | Daily |
| `zone-dnssec-stats.txt` | DNSSEC adoption statistics per TLD | Daily |
| `zone-ns-providers.csv` | Nameserver provider market share | Daily |

### AXFR Zone Transfer Detection

| File | Description | Update Frequency |
|------|-------------|-----------------|
| `axfr-open.csv` | Domains with open zone transfers (AXFR) | Daily |
| `axfr-summary.txt` | AXFR scan statistics and summary | Daily |

Large feeds are automatically split into parts (e.g. `nrd-full-part1.txt`, `nrd-full-part2.txt`) to stay within GitHub's file size limits.

## Why These Feeds?

### NRD Lists

Most NRD lists on GitHub are just mirrors of a single source, typically whoisds.com, with no additional data or validation.

- **First-party zone file access** - We diff ICANN CZDS zone files directly across 1,000+ TLDs. No middlemen, no scraping, no third-party dependency.
- **Certificate Transparency monitoring** - Domains captured the moment certificates are issued via real-time CT log streaming. Many NRDs appear in CT logs hours before they show up in any WHOIS-based feed.
- **Multi-source aggregation** - CZDS diffs, CT logs, passive DNS, and curated threat intel feeds merged and deduplicated.
- **No whoisds.com dependency** - Our data pipeline is fully independent. No blind spots from limited TLD coverage or third-party outages.
- **Threat-enriched** - Cross-referenced with phishing feeds (abuse.ch, CERT.PL, Phishing.Database).

### Zone Intelligence

No other public GitHub feed provides parsed DNS zone metadata at this scale:

- **Nameserver rankings** - See which DNS providers host the most domains, track market share shifts
- **DNSSEC coverage** - Per-TLD DNSSEC adoption rates from actual DS record presence in zone files
- **Infrastructure mapping** - Map domains to nameservers, detect shared hosting clusters, identify bulletproof hosting

### AXFR Detection

Open zone transfers are a well-known DNS misconfiguration that exposes all records in a domain's zone. Research shows ~8% of nameservers allow AXFR, affecting ~0.4% of domains. We scan for this at scale and publish results for defensive use.

### Comparison with other GitHub sources

| Source | Domains/day | Total domains | Data origin | Zone metadata? | AXFR? |
|--------|------------|---------------|-------------|----------------|-------|
| **cryphorix/threat-feeds** (this repo) | **2,000,000+** | **63,000,000+** | CZDS, CT logs, passive DNS, threat feeds | Yes | Yes |
| cenk/nrd | ~100,000 | ~3,000,000 | whoisds.com mirror | No | No |
| shreshta-labs/newly-registered-domains | ~10,000 (free) | Unknown | Proprietary passive DNS | No | No |
| cbuijs/nrd | Thousands | Unknown | Likely whoisds + DNS query filter | No | No |

We ingest over 20x more domains per day than the next largest free GitHub NRD source, and are the only feed providing zone metadata and AXFR detection.

## Usage

Raw URLs (GitHub will redirect from the old repo name):

```
https://raw.githubusercontent.com/cryphorix/threat-feeds/main/nrd-full-part1.txt
https://raw.githubusercontent.com/cryphorix/threat-feeds/main/axfr-open.csv
https://raw.githubusercontent.com/cryphorix/threat-feeds/main/zone-nameservers.csv
```

## Format

- `.txt` files: one domain per line. Lines starting with `#` are comments.
- `.csv` files: comma-separated with header row.

## License

This data is provided as-is for security research and threat intelligence purposes.
