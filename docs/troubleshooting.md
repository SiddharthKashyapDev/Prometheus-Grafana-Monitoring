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
