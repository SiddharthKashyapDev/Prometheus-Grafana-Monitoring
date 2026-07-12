# Troubleshooting guide

## APT reports an unavailable Grafana signing key

### Symptom

```text
NO_PUBKEY 963FA27710458545
The repository 'https://packages.grafana.com/oss/deb stable InRelease' is not signed.
```

### Cause

A Grafana APT source existed before the Grafana installation phase, but its trusted signing key was absent. APT blocks unsigned or unverifiable repositories to prevent unauthenticated package installation.

### Safe response

Do not use `--allow-unauthenticated` and do not disable APT signature checks. Disable the stale source file temporarily, run `sudo apt update` again, then add Grafana later using its official current key procedure.

## `sudo reboot` is inhibited by GNOME

### Symptom

```text
Operation inhibited by "sid" ... reason is "user session inhibited"
```

### Cause

The graphical GNOME session protects an active user session from an unexpected reboot.

### Safe response

Save work and use the graphical restart option. If there is no unsaved work, `sudo systemctl reboot -i` restarts while ignoring that session inhibitor.

## A service does not start

Start with:

```bash
sudo systemctl status SERVICE_NAME --no-pager
sudo journalctl -u SERVICE_NAME -n 50 --no-pager
```

Check the service account, executable path, file ownership, listening port, and configuration syntax before repeatedly restarting the service.

## Prometheus does not start

### Checks

```bash
sudo systemd-analyze verify /etc/systemd/system/prometheus.service
sudo -u prometheus promtool check config /etc/prometheus/prometheus.yml
sudo journalctl -u prometheus -n 50 --no-pager
```

### Common causes

- YAML indentation error or tabs in `prometheus.yml`.
- The `prometheus` account cannot read the configuration or write `/var/lib/prometheus`.
- Port `9090` is already occupied.
- The binary path in `ExecStart` is incorrect.

Correct the specific error, validate again, then run `sudo systemctl restart prometheus` once.

## Prometheus UI does not open from Windows

Confirm the service is active and listening on port `9090`, then check the current VM address with `ip -4 -brief address show ens33`. Use `http://<current-VM-IP>:9090` from the Windows host. VMware NAT is normally sufficient for host-to-guest access.

## Node Exporter target is down

### Checks

```bash
systemctl is-active node_exporter
sudo ss -ltnp 'sport = :9100'
wget -qO- http://localhost:9100/metrics | head
```

Then check that Prometheus has a target of `localhost:9100`, validate the configuration with `promtool`, and reload Prometheus.

### Common causes

- Node Exporter service is inactive or port `9100` is in use.
- Target address or port is incorrect.
- Prometheus configuration was changed but not reloaded.
- YAML indentation is invalid.

Do not run two exporters on port `9100`; audit an existing installation and use a controlled replacement or upgrade.

## Prometheus rules do not appear or show an error

### Checks

```bash
sudo -u prometheus promtool check rules /etc/prometheus/rules/node-alerts.yml
sudo -u prometheus promtool check config /etc/prometheus/prometheus.yml
sudo systemctl reload prometheus
sudo journalctl -u prometheus -n 50 --no-pager
```

### Common causes

- The `rule_files` path does not match the rule file.
- The service account cannot traverse `/etc/prometheus/rules` or read the file.
- Invalid YAML indentation or tabs.
- A PromQL expression uses an absent metric or invalid label matcher.

Validate both the individual rule file and the main configuration before every reload.
