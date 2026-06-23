# Asterisk Metrics Exporter

A Prometheus exporter for [Asterisk](https://www.asterisk.org/) PBX. It exposes
PJSIP and SIP channel, peer, QoS, and system metrics over HTTP so they can be
scraped by Prometheus and visualized in Grafana.

By default the exporter collects **PJSIP** metrics. SIP (`chan_sip`) collection
can be enabled with a flag. Each collector group can be toggled individually,
and the exporter supports optional basic auth and white-labeling for custom or
rebranded PBX deployments.

---

## Features

- PJSIP (`chan_pjsip`) metrics: channel status, contact RTT, endpoint channels, and QoS
- SIP (`chan_sip`) metrics: peers, QoS, and registry (opt-in)
- Core channel and current-call statistics
- Asterisk version, uptime, and reload info
- Optional HTTP basic authentication
- Configurable port, metrics prefix, and systemd service name
- White-label support to rebrand the `asterisk` command/label

---

## Requirements

- Linux (amd64 build provided: `asterisk_exporter_amd64`)
- A running Asterisk instance reachable by the exporter (via the local Asterisk
  CLI / systemd service)
- Prometheus, to scrape the exporter

---

## Installation

Download the binary, make it executable, and run it:

```bash
chmod +x asterisk_exporter_amd64
./asterisk_exporter_amd64
```

The exporter listens on port `9110` by default and serves metrics at `/metrics`.

```bash
curl http://localhost:9110/metrics
```

> Note: the metrics path is `/metrics` by Prometheus convention. Adjust if your
> build differs.

---

## Usage

```
./asterisk_exporter_amd64 [options]
```

### Examples

PJSIP only (default):

```bash
./asterisk_exporter_amd64
```

With basic auth:

```bash
./asterisk_exporter_amd64 -authfile /etc/asterisk-exporter/auth.txt
```

SIP only (PJSIP disabled):

```bash
./asterisk_exporter_amd64 -trunkfile trunk_names.txt -collect-sip=true -collect-pjsip=false
```

White-label (custom PBX command):

```bash
./asterisk_exporter_amd64 -label mypbx -service mypbx
```

---

## Configuration Options

### General

| Flag | Default | Description |
|------|---------|-------------|
| `-port int` | `9110` | Port for the HTTP server |
| `-prefix string` | `asterisk` | Metrics prefix |
| `-label string` | `asterisk` | Custom label to replace the PBX command |
| `-service string` | `asterisk` | Systemd service name |
| `-authfile string` | _(none)_ | File with basic auth credentials (format: `username:password`) |
| `-trunkfile string` | _(none)_ | File containing trunk names (required for SIP peers) |
| `-version` | | Show version and exit |

### Core collectors

| Flag | Default | Description |
|------|---------|-------------|
| `-collect-channelsdetail` | `true` | Detailed channel metrics (core metric) |
| `-collect-currentcallschannelstat` | `true` | Current calls channel statistics |
| `-collect-uptimereloadinfo` | `true` | Uptime and reload info metrics |
| `-collect-version` | `true` | Asterisk version metrics |

### PJSIP collectors

| Flag | Default | Description |
|------|---------|-------------|
| `-collect-pjsip` | `true` | PJSIP (`chan_pjsip`) metrics collection |
| `-collect-pjsip-channelstatus` | `true` | PJSIP channel status (requires `-collect-pjsip`) |
| `-collect-pjsip-contactrtt` | `true` | PJSIP contact RTT (requires `-collect-pjsip`) |
| `-collect-pjsip-endpointchannels` | `true` | PJSIP endpoint channels (requires `-collect-pjsip`) |
| `-collect-pjsip-qos` | `true` | PJSIP QoS metrics (requires `-collect-pjsip`) |

### SIP collectors

| Flag | Default | Description |
|------|---------|-------------|
| `-collect-sip` | `false` | SIP (`chan_sip`) metrics collection |
| `-collect-sip-peers` | `true` | SIP peers metrics (requires `-collect-sip`) |
| `-collect-sip-qos` | `true` | SIP QoS metrics (requires `-collect-sip`) |
| `-collect-sip-registry` | `false` | SIP registry metrics (requires `-collect-sip`) |

---

## Basic Authentication

Create an auth file containing a single `username:password` line:

```bash
echo "prometheus:s3cr3t" > /etc/asterisk-exporter/auth.txt
chmod 600 /etc/asterisk-exporter/auth.txt
```

Start the exporter pointing at it:

```bash
./asterisk_exporter_amd64 -authfile /etc/asterisk-exporter/auth.txt
```

Configure the matching credentials in your Prometheus scrape job (see below).

---

## SIP Peers and the Trunk File

SIP peers collection (`-collect-sip-peers`) requires a trunk file passed via
`-trunkfile`. The file lists trunk names, one per line:

```
trunk_provider_a
trunk_provider_b
trunk_backup
```

```bash
./asterisk_exporter_amd64 -collect-sip=true -trunkfile /etc/asterisk-exporter/trunk_names.txt
```

---

## Running as a systemd Service

Create `/etc/systemd/system/asterisk-exporter.service`:

```ini
[Unit]
Description=Asterisk Metrics Exporter
After=network.target asterisk.service

[Service]
ExecStart=/usr/local/bin/asterisk_exporter_amd64 -port 9110
Restart=on-failure
User=root

[Install]
WantedBy=multi-user.target
```

Enable and start it:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now asterisk-exporter
sudo systemctl status asterisk-exporter
```

---

## Prometheus Scrape Configuration

Add a job to `prometheus.yml`:

```yaml
scrape_configs:
  - job_name: asterisk
    static_configs:
      - targets: ["localhost:9110"]
```

With basic auth enabled:

```yaml
scrape_configs:
  - job_name: asterisk
    basic_auth:
      username: prometheus
      password: s3cr3t
    static_configs:
      - targets: ["localhost:9110"]
```

---

## White-Labeling

For rebranded or custom PBX deployments, override the label and systemd service
name so metrics and service references match your product:

```bash
./asterisk_exporter_amd64 -label mypbx -service mypbx -prefix mypbx
```

This replaces the default `asterisk` command/label and uses your prefix on
exported metric names.

---

## Version

```bash
./asterisk_exporter_amd64 -version
```

---

## License

Specify your license here (e.g. MIT). Add a `LICENSE` file to the repository.

## Contributing

Issues and pull requests are welcome. Please open an issue to discuss
significant changes before submitting a PR.
