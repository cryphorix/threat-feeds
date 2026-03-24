# Threat Feeds

Daily threat intelligence feeds — newly registered domains, DNS zone metadata, and zone transfer (AXFR) detection. Updated automatically.

Powered by **ICANN CZDS zone files** (831 TLDs), **Certificate Transparency logs**, **passive DNS**, and **community threat intel sources**.

## Current Stats

| Metric | Value |
|--------|-------|
| Unique domains in database | **63,082,097** |
| Domain-NS mappings | **130,903,636** |
| Unique nameservers | **733,357** |
| TLD zone files parsed | **831** |
| DNSSEC DS records | **2,315,823** |
| DNSSEC coverage | **4.24%** |
| Glue records | **892,779** |
| AXFR nameservers tested | **27,337** |
| Open zone transfers | **513** (1.88%) |
| CT log domains (24h) | **2,212,953** |

Feeds are updated and pushed to this repo daily.

## Feeds

### NRD (Newly Registered Domains)

| File | Domains | Description |
|------|---------|-------------|
| `nrd-today-*.txt` | 63,065,909 | Domains ingested in the last 24 hours |
| `nrd-week-*.txt` | 63,071,709 | Domains from the last 7 days |
| `czds-nrd-*.txt` | 58,543,252 | CZDS zone file new domains (last 24h) |
| `ct-domains.txt` | 2,212,953 | CT log discoveries (last 24h) |
| `nrd-full-*.txt` | 63,082,097 | Complete master list (all domains in DB) |

Large feeds are split into parts (e.g. `nrd-full-part1.txt` through `part15.txt`) to stay within GitHub's 100MB file size limit.

### DNS Zone Intelligence

| File | Records | Description |
|------|---------|-------------|
| `zone-nameservers.csv` | 1,000 | Top nameservers ranked by domain count |
| `zone-dnssec-stats.csv` | 831 | DNSSEC adoption per TLD (domains with DS / total) |
| `zone-ns-providers.csv` | 15 | DNS provider market share (Cloudflare, GoDaddy, Google, etc.) |

### AXFR Zone Transfer Detection

| File | Records | Description |
|------|---------|-------------|
| `axfr-open.csv` | 513 | Domains with open zone transfers — nameserver, record count |
| `axfr-summary.txt` | — | Scan summary: 27,337 NS tested, 1.88% open rate |

### Full Zone Dataset (Parquet)

The complete 130.9M-record zone dataset is available as Parquet files in [GitHub Releases](https://github.com/cryphorix/threat-feeds/releases). Hive-partitioned by TLD, ZSTD compressed — queryable with DuckDB, Python, Polars, or Spark.

| Asset | Records | Size |
|-------|---------|------|
| `domain_ns.tar.gz` | 130,903,636 | 628 MB |
| `domain_ds.tar.gz` | 2,315,823 | 100 MB |
| `glue_records.tar.gz` | 892,779 | 8.4 MB |
| `nameservers.parquet` | 733,357 | 4.7 MB |
| `axfr_results.parquet` | 27,337 | 391 KB |

```bash
# Download
gh release download v2026.03.24-zones -R cryphorix/threat-feeds
tar xzf domain_ns.tar.gz

# Query with DuckDB
duckdb -c "
SELECT tld, COUNT(*) as domains
FROM read_parquet('domain_ns/*/*.parquet', hive_partitioning=true)
GROUP BY tld ORDER BY domains DESC LIMIT 20;
"

# Python
import duckdb
df = duckdb.sql("SELECT * FROM read_parquet('domain_ns/*/*.parquet', hive_partitioning=true)").df()
```

## Why These Feeds?

### NRD Lists

Most NRD lists on GitHub are just mirrors of a single source, typically whoisds.com, with no additional data or validation.

- **First-party zone file access** — We diff ICANN CZDS zone files directly across 831 TLDs. No middlemen, no scraping, no third-party dependency.
- **Certificate Transparency monitoring** — Domains captured the moment certificates are issued via real-time CT log streaming. Many NRDs appear in CT logs hours before they show up in any WHOIS-based feed.
- **Multi-source aggregation** — CZDS diffs, CT logs, passive DNS, and curated threat intel feeds merged and deduplicated.
- **No whoisds.com dependency** — Our data pipeline is fully independent. No blind spots from limited TLD coverage or third-party outages.

### Zone Intelligence

No other public GitHub feed provides parsed DNS zone metadata at this scale:

- **Nameserver rankings** — See which DNS providers host the most domains, track market share shifts
- **DNSSEC coverage** — Per-TLD DNSSEC adoption rates from actual DS record presence in zone files (4.24% overall)
- **Infrastructure mapping** — Map domains to nameservers, detect shared hosting clusters, identify bulletproof hosting

### AXFR Detection

Open zone transfers are a well-known DNS misconfiguration that exposes all records in a domain's zone. We test 27,337 unique nameservers and found 513 (1.88%) allow unauthorized AXFR. Results are published for defensive use.

### Comparison

| Source | Domains | Zone files | Zone metadata | AXFR | Parquet |
|--------|---------|-----------|---------------|------|---------|
| **cryphorix/threat-feeds** | **63M** | **831 TLDs** | **130.9M records** | **513 open** | **Yes** |
| cenk/nrd | ~3M | — | — | — | — |
| shreshta-labs/newly-registered-domains | Unknown | — | — | — | — |
| cbuijs/nrd | Unknown | — | — | — | — |

## Usage

Raw URLs:

```
https://raw.githubusercontent.com/cryphorix/threat-feeds/main/nrd-full-part1.txt
https://raw.githubusercontent.com/cryphorix/threat-feeds/main/axfr-open.csv
https://raw.githubusercontent.com/cryphorix/threat-feeds/main/zone-nameservers.csv
```

## Format

- `.txt` files: one domain per line
- `.csv` files: comma-separated with header row
- `.parquet` files: Apache Parquet with ZSTD compression (in Releases)

## License

This data is provided as-is for security research and threat intelligence purposes.
