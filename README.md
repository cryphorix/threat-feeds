# Threat Feeds

Daily threat intelligence feeds — newly registered domains, DNS zone metadata, and zone transfer (AXFR) detection.

Powered by **ICANN CZDS zone files** (831 TLDs), **Certificate Transparency logs**, **passive DNS**, and **community threat intel sources**. Updated and pushed automatically every day.

## Current Stats

<!-- These values are updated automatically by the publish pipeline -->

| Metric | Value |
|--------|-------|
| Unique domains in database | **73,995,047** |
| Domain-NS mappings | **130,903,636** |
| Unique nameservers | **733,357** |
| TLD zone files parsed | **831** |
| DNSSEC DS records | **2,315,823** |
| DNSSEC coverage | **4.24%** |
| Glue records | **892,779** |
| AXFR nameservers tested | **27,337** |
| Open zone transfers | **513** (1.88%) |
| CT log domains (24h) | **1,460,224** |

## Feeds

### NRD (Newly Registered Domains)

| File | Description |
|------|-------------|
| `nrd-today.txt` | Domains registered/discovered in the last 24 hours |
| `nrd-week.txt` | Domains from the last 7 days |
| `czds-nrd.txt` | New domains found by diffing CZDS zone files (last 24h) |
| `ct-domains.txt` | Domains discovered from Certificate Transparency logs (last 24h) |
| `nrd-full.txt` | Complete master domain list (all sources, all time) |

> Files over 90MB are automatically split into numbered parts (e.g. `nrd-full-part1.txt`, `nrd-full-part2.txt`). If the file is small enough, it stays as a single file.

**Format:** One domain per line. Lines starting with `#` are comments (header, source info, total count).

```
# Newly Registered Domains - 2026-03-25
# Source: domain-scanner (aggregated NRD feeds)
# https://github.com/cryphorix/threat-feeds

0-0.sbs
0-1-2-3-4-5-6-7-8-9.ai
example-domain.xyz
...

# Total: 2000895 domains
```

### DNS Zone Intelligence

| File | Format | Description |
|------|--------|-------------|
| `zone-nameservers.csv` | CSV | Top 1,000 nameservers ranked by number of domains they serve |
| `zone-dnssec-stats.csv` | CSV | DNSSEC adoption per TLD — domains with DS records vs total |
| `zone-ns-providers.csv` | CSV | DNS provider market share (Cloudflare, GoDaddy, AWS, Google, etc.) |

All CSV files have a header row. Example (`zone-nameservers.csv`):

```csv
nameserver,domain_count
ns5.afternic.com,3192989
ns6.afternic.com,3192889
dns2.registrar-servers.com,2398979
```

### AXFR Zone Transfer Detection

AXFR (zone transfer) is a DNS mechanism that replicates an entire zone file from a nameserver. When misconfigured, it allows anyone to download all DNS records for a domain — exposing subdomains, internal hostnames, and infrastructure details.

| File | Format | Description |
|------|--------|-------------|
| `axfr-open.csv` | CSV | Every domain/nameserver pair where AXFR is open, with record count |
| `axfr-summary.txt` | Text | Scan statistics — how many nameservers tested, open rate, top offenders |

Example (`axfr-open.csv`):

```csv
domain,nameserver,record_count,tested_at
1box.link,ns2.crm-onebox.com,352,2026-03-24 22:27:31
0conf.org,ns1.bolo.net,321,2026-03-24 22:27:59
```

### Full Zone Dataset (Parquet)

The complete zone metadata dataset is published weekly as Parquet files in [GitHub Releases](https://github.com/cryphorix/threat-feeds/releases). These contain the full domain→nameserver mappings, DNSSEC records, and glue records across all 831 TLDs.

| Asset | Description | Size |
|-------|-------------|------|
| `domain_ns.tar.gz` | Domain → Nameserver mappings (Hive-partitioned by TLD) | ~628 MB |
| `domain_ds.tar.gz` | DNSSEC DS records (Hive-partitioned by TLD) | ~100 MB |
| `glue_records.tar.gz` | Glue A/AAAA records (Hive-partitioned by TLD) | ~8 MB |
| `nameservers.parquet` | All unique nameserver hostnames | ~5 MB |
| `axfr_results.parquet` | Full AXFR scan results | ~400 KB |

Parquet files use ZSTD compression and are Hive-partitioned by TLD (e.g. `domain_ns/tld=com/data_0.parquet`).

## Usage

### Download raw feeds

```bash
# curl (single file)
curl -sL https://raw.githubusercontent.com/cryphorix/threat-feeds/main/nrd-today.txt -o nrd-today.txt

# wget (all NRD files)
wget https://raw.githubusercontent.com/cryphorix/threat-feeds/main/nrd-full.txt

# If files are split into parts:
for i in $(seq 1 15); do
  wget -q "https://raw.githubusercontent.com/cryphorix/threat-feeds/main/nrd-full-part${i}.txt" 2>/dev/null
done
```

### Grep / filter domains

```bash
# Find all .bank domains
grep '\.bank$' nrd-today.txt

# Find domains containing "paypal" (potential phishing)
grep -i 'paypal' nrd-today.txt

# Strip comments and get just domains
grep -v '^#' nrd-full.txt | grep -v '^$' > domains-only.txt
```

### Use in a SIEM / blocklist

```bash
# Convert to hosts file format (block list)
grep -v '^#' nrd-today.txt | grep -v '^$' | awk '{print "0.0.0.0 " $1}' > blocklist-hosts.txt

# Convert to Suricata/Snort domain list
grep -v '^#' nrd-today.txt | grep -v '^$' | awk '{print "." $1}' > suricata-domains.txt

# Pi-hole / AdGuard format
grep -v '^#' nrd-today.txt | grep -v '^$' | awk '{print "||" $1 "^"}' > adguard-blocklist.txt
```

### Query Parquet data with DuckDB

```bash
# Download latest Parquet release
gh release download --repo cryphorix/threat-feeds --pattern '*.tar.gz' --pattern '*.parquet'
tar xzf domain_ns.tar.gz

# Install DuckDB: https://duckdb.org/docs/installation
# Count domains per TLD
duckdb -c "
  SELECT tld, COUNT(*) as domains
  FROM read_parquet('domain_ns/*/*.parquet', hive_partitioning=true)
  GROUP BY tld ORDER BY domains DESC LIMIT 20;
"

# Find all nameservers for a specific domain
duckdb -c "
  SELECT nameserver
  FROM read_parquet('domain_ns/*/*.parquet', hive_partitioning=true)
  WHERE domain = 'example.com';
"

# DNSSEC adoption across all TLDs
duckdb -c "
  SELECT tld, domains_with_dnssec, total_domains, coverage_pct
  FROM read_csv('zone-dnssec-stats.csv')
  ORDER BY total_domains DESC LIMIT 20;
"
```

### Query Parquet data with Python

```python
import duckdb

# Load all domain-NS mappings
df = duckdb.sql("""
    SELECT * FROM read_parquet('domain_ns/*/*.parquet', hive_partitioning=true)
""").df()

# Top nameservers
print(df.groupby('nameserver').size().sort_values(ascending=False).head(20))

# Domains on a specific nameserver
suspicious = df[df['nameserver'] == 'ns1.suspicious-host.com']
```

## Data Sources

| Source | What it provides | Update frequency |
|--------|-----------------|------------------|
| [ICANN CZDS](https://czds.icann.org/) | Zone files for 831 gTLDs — NS, DS, glue records | Daily (07:00 UTC) |
| Certificate Transparency logs | Domains from newly issued TLS certificates | Real-time (streaming) |
| Passive DNS | DNS queries observed on the wire | Real-time |
| CZDS zone diffs | New domains by comparing today's vs yesterday's zone files | Daily |

## Comparison

| Source | Total domains | Zone metadata | AXFR detection | Parquet export |
|--------|--------------|---------------|----------------|----------------|
| **cryphorix/threat-feeds** | **73,995,047+** (growing daily) | **130.9M NS records, 831 TLDs** | **Yes** | **Yes** |
| cenk/nrd | ~3M | No | No | No |
| shreshta-labs/newly-registered-domains | Unknown | No | No | No |
| cbuijs/nrd | Unknown | No | No | No |

## File Format Reference

| Extension | Format | Details |
|-----------|--------|---------|
| `.txt` | Plain text | One domain per line. `#` lines are comments. |
| `.csv` | CSV | Comma-separated, first row is header. |
| `.parquet` | Apache Parquet | ZSTD compressed, Hive-partitioned by TLD. Available in Releases. |

## License

This data is provided as-is for security research and threat intelligence purposes.
