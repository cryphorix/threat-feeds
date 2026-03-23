# NRD Lists

Daily newly registered domain (NRD) feeds, updated automatically.

Aggregated from **ICANN CZDS zone files**, **Certificate Transparency logs**, and **community NRD sources**. Deduplicated and sorted.

## Why This Feed?

Most NRD lists on GitHub are just mirrors of a single source — typically whoisds.com — with no additional data or validation. This feed is different.

### What makes us unique

- **First-party zone file access** — We diff ICANN CZDS zone files directly across 1,000+ TLDs. No middlemen, no rate-limited scraping, no dependency on third-party scrapers that go down without notice.
- **Certificate Transparency monitoring** — We capture domains the moment certificates are issued via real-time CT log streaming (Gungnir). Many NRDs appear in CT logs hours before they show up in any WHOIS-based feed.
- **Multi-source aggregation** — We merge and deduplicate across CZDS diffs, CT logs, passive DNS, and curated threat intel feeds into a single clean output.
- **No whoisds.com dependency** — We do not pull from whoisds.com. Feeds that rely on it inherit its blind spots (limited TLD coverage, delayed updates, and periodic outages). Our data pipeline is fully independent.
- **Threat-enriched** — Our pipeline also ingests phishing feeds (abuse.ch, CERT.PL, Phishing.Database) and cross-references them, so our lists naturally capture malicious NRDs that pure registration feeds miss entirely.

### Comparison with other GitHub NRD sources

| Source | Domains/day | Data origin | Independent? | Limitations |
|--------|------------|-------------|--------------|-------------|
| **cryphorix/nrd-lists** (this repo) | **10,000–100,000+** | CZDS zone diffs, CT logs, passive DNS, threat feeds | Yes | — |
| cenk/nrd | ~100,000 | whoisds.com mirror | No | Single source, breaks when whoisds goes down |
| shreshta-labs/newly-registered-domains | ~10,000 (free tier) | Proprietary passive DNS | Yes | Free feed is a capped sample; full data requires paid subscription |
| cbuijs/nrd | Thousands (filtered) | Likely whoisds + DNS query filter | No | Filtered to "queried" domains only, misses net-new malicious NRDs |
| Phishing.Database | N/A | Community reports | Yes | Not an NRD feed — phishing blocklist only, no registration data |

### Key differences

- **cenk/nrd** mirrors whoisds.com raw data — high volume but single point of failure, no CT or zone file coverage, no threat enrichment.
- **shreshta-labs** has good methodology but gates their full feed behind a paywall. The free GitHub feed is a 10k sample out of 50–150k.
- **cbuijs/nrd** filters NRDs against DNS popularity lists, which sounds useful but means it drops domains that haven't been queried yet — exactly the ones that matter for early threat detection.
- **This feed** combines multiple independent pipelines (CZDS, CT, community) with no whoisds dependency, giving broader TLD coverage, faster detection of newly issued domains, and a single deduplicated output.

## Feeds

| File | Description | Update Frequency |
|------|-------------|-----------------|
| `nrd-today.txt` | Domains registered today (all sources) | Daily |
| `nrd-week.txt` | Domains from the last 7 days | Daily |
| `czds-nrd.txt` | CZDS zone file new domains (last 24h) | Daily |
| `ct-domains.txt` | CT log discoveries (last 24h) | Daily |
| `nrd-full.txt` | Complete master list (all domains in DB) | Daily |

## Usage

Raw URLs for direct consumption:

```
https://raw.githubusercontent.com/cryphorix/nrd-lists/main/nrd-today.txt
https://raw.githubusercontent.com/cryphorix/nrd-lists/main/nrd-week.txt
https://raw.githubusercontent.com/cryphorix/nrd-lists/main/czds-nrd.txt
https://raw.githubusercontent.com/cryphorix/nrd-lists/main/ct-domains.txt
https://raw.githubusercontent.com/cryphorix/nrd-lists/main/nrd-full.txt
```

## Format

Plain text, one domain per line. Lines starting with `#` are comments containing metadata (date, count, source).

## License

This data is provided as-is for security research and threat intelligence purposes.
