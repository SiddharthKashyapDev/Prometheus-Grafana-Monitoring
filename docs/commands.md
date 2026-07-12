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
