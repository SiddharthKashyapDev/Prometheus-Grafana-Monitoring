# Commands reference

## Phase 1 commands

| Command | Purpose | Changes files? |
| --- | --- | --- |
| `hostnamectl` | Shows hostname, OS, kernel, and virtualization information. | No |
| `cat /etc/os-release` | Displays the Ubuntu release metadata. | No |
| `ip -brief address` | Shows interfaces and their assigned addresses. | No |
| `ip route` | Shows the route table and default gateway. | No |
| `ping -c 4 1.1.1.1` | Tests IP-level connectivity. | No |
| `getent hosts archive.ubuntu.com` | Tests DNS name resolution. | No |
| `sudo apt update` | Refreshes the package index. | Yes, APT lists under `/var/lib/apt/lists/` |
| `sudo apt upgrade --dry-run` | Previews upgrades without installing them. | No |
| `sudo apt upgrade` | Installs available package updates. | Yes, installed packages and APT/DPKG state |
| `uname -r` | Shows the running kernel version. | No |
| `uptime` | Shows boot duration and load averages. | No |
| `id` | Shows user and group IDs. | No |
| `ls -ld ~` | Shows ownership and mode of the home directory. | No |
| `chmod 640 <file>` | Sets owner read/write, group read, and no other access. | Yes, file mode only |
| `systemctl status <unit> --no-pager` | Shows a service’s status and recent log lines. | No |
| `systemctl --failed` | Lists failed systemd units. | No |

## Essential service lifecycle commands

These will be used in later phases for Prometheus, Node Exporter, and Grafana.

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now SERVICE_NAME
sudo systemctl status SERVICE_NAME --no-pager
sudo journalctl -u SERVICE_NAME -n 50 --no-pager
```

- `daemon-reload` makes systemd re-read changed unit files.
- `enable --now` configures startup at boot and starts the service now.
- `status` gives a quick health view.
- `journalctl -u` reads logs for one named unit.

## Prometheus commands

| Command | Purpose | Changes files? |
| --- | --- | --- |
| `prometheus --version` | Shows installed Prometheus build information. | No |
| `promtool check config /etc/prometheus/prometheus.yml` | Validates Prometheus YAML and configuration semantics. | No |
| `sudo systemd-analyze verify /etc/systemd/system/prometheus.service` | Validates the systemd unit file. | No |
| `sudo systemctl enable --now prometheus` | Enables Prometheus at boot and starts it immediately. | Yes, systemd state and runtime data |
| `systemctl is-active prometheus` | Confirms that the service is running. | No |
| `systemctl is-enabled prometheus` | Confirms that startup at boot is configured. | No |
| `wget -qO- http://localhost:9090/-/ready` | Checks the local Prometheus readiness endpoint. | No |

## First PromQL query

```promql
up
```

`up` is created for every scrape target. A value of `1` means the latest scrape succeeded; `0` means the target could not be scraped. This is the first health query to use when a target appears down.

## Node Exporter commands

| Command | Purpose | Changes files? |
| --- | --- | --- |
| `node_exporter --version` | Shows the Node Exporter version and platform. | No |
| `wget -qO- http://localhost:9100/metrics` | Reads the local host-metrics endpoint. | No |
| `systemctl is-active node_exporter` | Confirms the exporter is running. | No |
| `systemctl is-enabled node_exporter` | Confirms automatic startup is configured. | No |
| `sudo systemctl reload prometheus` | Reloads validated scrape configuration without stopping Prometheus. | Yes, runtime configuration only |

## Useful host-metric queries

```promql
up{job="node_exporter"}
node_memory_MemAvailable_bytes
rate(node_cpu_seconds_total{mode!="idle"}[5m])
```

- `up{job="node_exporter"}` should return `1` when Prometheus can scrape Node Exporter.
- `node_memory_MemAvailable_bytes` is currently available RAM in bytes.
- The `rate(...)` query calculates per-second non-idle CPU time over five minutes.
