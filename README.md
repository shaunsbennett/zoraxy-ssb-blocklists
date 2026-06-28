# SSB Blocklists

Zoraxy router plugin for importing URL-based IP/CIDR blocklists and blocking
matching client IPs before requests reach the upstream service.

## Features

- Multiple URL sources
- Per-source refresh interval in hours
- Formats: `auto`, `plain`, `ndjson`, `json`
- Keeps the last good cache for each source
- Deduplicates enabled source entries into one active matcher
- Blocks matching client IPs with HTTP 403
- Simple in-memory stats for inspected, passed, dropped, unknown IP, and errors
- Per-source cached entry viewer

## Source Formats

- `plain` accepts one IP/CIDR per line, comments, and whitespace-separated tokens.
- `ndjson` and `json` read `cidr`, `ip`, or `network` fields.
- `auto` tries JSON/NDJSON first, then falls back to plain text.

The default seed source points at Spamhaus DROP:

```text
https://www.spamhaus.org/drop/drop.txt
```

## Enable In Zoraxy

Router plugins are attached by proxy tags:

```text
HTTP proxy rule tag -> plugin assigned to that tag -> plugin inspects requests
```

To enable this plugin for a proxy host:

1. Open the Zoraxy admin UI.
2. Go to **HTTP Proxy**.
3. Edit the proxy host you want to protect.
4. Add a tag, for example:

```text
ssb-blocklists
```

5. Save the proxy host.
6. Go to **Plugins**.
7. In the tag/plugin assignment area, select the same tag.
8. Add **SSB Blocklists** to that tag.

After this, requests to any proxy host with that tag should increment the
plugin's `Inspected` counter.

## Operation

Use the plugin UI to add sources, enable/disable them, set refresh intervals,
save changes, refresh sources immediately, and view cached source entries.

The plugin stores:

```text
config.json
config.json.sample
cache/<source-id>.cidr
```

If `config.json` does not exist, the plugin creates an empty config on startup.
Use `config.json.sample` as an optional starter config. The sample includes
Spamhaus DROP:

```text
https://www.spamhaus.org/drop/drop.txt
```

The active matcher is rebuilt from enabled source caches. Duplicate CIDRs across
sources are removed from the active set. Individual source caches are kept
separate so the **View** button can show the normalized entries for that source.

Stats are in memory and reset when the plugin or Zoraxy restarts.

## Logs

Blocked requests are logged through Zoraxy's plugin manager output. Search with:

```bash
docker logs zoraxy 2>&1 | rg "blocked sniff|SSB Blocklists|/d_block"
```

Example blocked decision log:

```text
blocked sniff match ip=1.10.16.1 host=example.test uri=/path
```

## Install Notes

Restart Zoraxy after installing the plugin so it can discover the executable.
