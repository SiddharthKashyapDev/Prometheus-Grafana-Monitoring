# Interview and viva guide

Use these answers as a guide, but explain them in your own words. The strongest answer connects the concept to something you personally configured or verified in this project.

## 60-second project explanation

I built a Linux monitoring lab on an Ubuntu 26.04 LTS VMware virtual machine. Node Exporter runs as a dedicated system service and exposes host metrics such as CPU, memory, disk, filesystem, and network data on port 9100. Prometheus runs as another native systemd service, scrapes those metrics every 15 seconds, stores them as time-series data, and evaluates recording and alert rules. Grafana connects to Prometheus on the same VM and visualizes the data through a Linux Server Monitoring dashboard on port 3000. I documented the configuration, systemd units, alert rules, dashboard export, screenshots, and troubleshooting steps in GitHub. I deliberately used native Linux installation and systemd rather than Docker so I could learn permissions, services, networking, and logs directly.

## Architecture and monitoring concepts

### 1. What problem does this project solve?

It makes the health and resource usage of a Linux host observable. Instead of logging into the server to check CPU, memory, disk, or uptime manually, an administrator can view current and historical metrics in Grafana and identify problems with Prometheus queries and rules.

### 2. What is Node Exporter?

Node Exporter is a Prometheus exporter for host-level Linux metrics. It reads operating-system statistics and exposes them in Prometheus text format at `/metrics`. In this project it listens on port `9100` and runs as the restricted `node_exporter` user.

### 3. What is Prometheus?

Prometheus is a monitoring system and time-series database. It pulls metrics from configured targets at regular intervals, stores each value with a timestamp and labels, and provides PromQL for querying those metrics. In this lab it listens on port `9090` and scrapes every 15 seconds.

### 4. What is Grafana?

Grafana is the visualization layer. It does not collect the Node Exporter metrics itself. Instead, it sends PromQL queries to Prometheus and turns the returned data into dashboard panels, graphs, gauges, and tables. It listens on port `3000` in this lab.

### 5. Explain the complete data flow.

Node Exporter reads host statistics and exposes them at `localhost:9100/metrics`. Prometheus scrapes that endpoint every 15 seconds and stores the results. Grafana connects to Prometheus at `localhost:9090`, sends PromQL queries, and displays the results in the browser. Prometheus also evaluates the configured rules every 15 seconds.

### 6. Why does Prometheus use a pull model?

Prometheus normally pulls metrics from known HTTP endpoints. This gives the monitoring server control over scrape frequency and lets it detect a failed target through the `up` metric. It also makes target configuration explicit in `prometheus.yml`.

### 7. What does the `up` metric mean?

Prometheus creates `up` for every scrape target. A value of `1` means the most recent scrape succeeded, while `0` means Prometheus could not scrape that target. I verified `up=1` for both `prometheus` and `node_exporter`, proving that the full scrape path works.

### 8. What are labels in Prometheus?

Labels are key-value metadata attached to a time series. They allow the same metric name to represent different machines, jobs, environments, CPUs, or filesystems. This project uses labels such as `job=node_exporter`, `instance=localhost:9100`, and `environment=lab`.

## Linux, systemd, and security

### 9. Why did you use systemd services?

`systemd` starts the monitoring services automatically at boot, keeps their lifecycle consistent, exposes service status, and centralizes logs through `journalctl`. It is more reliable than leaving commands running in a terminal session.

### 10. How do you check whether a service is healthy?

I use `systemctl is-active <service>` to check whether it is running and `systemctl is-enabled <service>` to check whether it starts at boot. I also check the actual endpoints: Prometheus `/-/ready`, Node Exporter `/metrics`, and Grafana `/api/health`.

### 11. Why use dedicated service accounts?

Prometheus and Node Exporter do not need to run as my interactive Linux user or as root. Dedicated non-login accounts reduce the permissions available if a service is compromised and make file ownership clearer.

### 12. What is the difference between `restart` and `reload` for Prometheus?

A restart stops and starts the Prometheus process. A reload asks the running process to read its configuration again after validation. For a safe configuration-only change, I run `promtool check` first and then use `systemctl reload prometheus`.

### 13. Where would you look if a service failed to start?

First I would run `systemctl status <service>`. Then I would inspect recent logs with `journalctl -u <service> -n 50 --no-pager`. For Prometheus configuration changes, I would validate the configuration and rule file with `promtool` before restarting or reloading.

## Prometheus configuration and rules

### 14. What is a scrape interval?

It is the frequency at which Prometheus requests metrics from a target. This lab uses 15 seconds, balancing reasonably fresh lab data with low overhead. A shorter interval creates more data and load; a longer interval can miss short events.

### 15. What is PromQL?

PromQL is Prometheus Query Language. It selects, filters, aggregates, and calculates over time-series data. For example, `up{job="node_exporter"}` checks whether the Node Exporter target is reachable.

### 16. What is a recording rule?

A recording rule periodically calculates and stores the result of a PromQL expression as a new metric. I created `instance:node_cpu_utilisation:ratio`, which stores non-idle CPU utilization as a ratio. This simplifies repeated use of the calculation in queries or dashboards.

### 17. What is an alerting rule?

An alerting rule defines a condition that becomes pending and then firing when it remains true for a configured duration. This project includes `NodeExporterDown` and `HostHighCPUUsage`. They are evaluated by Prometheus and visible in its UI.

### 18. Why are alerts not sending email or messages in this project?

Prometheus evaluates the alert conditions, but Alertmanager is not configured. Alertmanager is the component responsible for grouping, routing, silencing, and delivering notifications to email, Slack, or other receivers. This is a deliberate lab boundary documented in the repository.

### 19. How would you safely change `prometheus.yml`?

I would edit the source-controlled copy, update the active file carefully, validate it with `sudo -u prometheus promtool check config /etc/prometheus/prometheus.yml`, validate any changed rules with `promtool check rules`, then reload Prometheus. Finally, I would check the Targets and Rule health pages and commit the validated configuration to Git.

## Grafana and dashboard questions

### 20. Why is the Grafana data source URL `http://localhost:9090`?

Grafana and Prometheus run on the same Ubuntu VM. The connection is made by the Grafana server, so its own `localhost` refers to that VM. A browser on Windows uses the VM's address only to reach the Grafana web interface.

### 21. Which dashboard did you use and how did you adapt it?

I imported dashboard ID `1860`, Node Exporter Full, then selected my Prometheus data source and set the dashboard variables to `job=node_exporter` and `instance=localhost:9100`. I saved it as Linux Server Monitoring and exported the JSON to the repository for reproducibility.

### 22. Why can some imported dashboard panels show `N/A`?

Community dashboards often include panels for optional Node Exporter collectors or host features such as swap. If that collector or feature is unavailable, the corresponding panel can show `N/A` even when the core exporter, Prometheus target, and main CPU, memory, disk, and network metrics are healthy.

## Git, reproducibility, and next steps

### 23. Why keep monitoring configuration in Git?

Git creates a reviewable history of the Prometheus configuration, rule files, service units, dashboard export, and documentation. This makes the lab reproducible and lets me explain what changed and why. Runtime metric data is excluded because it is generated, large, and not the configuration needed to rebuild the system.

### 24. What did you avoid committing, and why?

I avoided credentials, Grafana passwords, API keys, tokens, logs, downloaded archives, and Prometheus runtime data. These can be sensitive, machine-specific, or unnecessarily large. The repository contains sanitized screenshots and the exported dashboard instead.

### 25. How would you scale this design to multiple servers?

I would install Node Exporter on each monitored Linux host and add each host's `host:9100` endpoint to Prometheus scrape configuration, ideally using service discovery in a larger environment. Prometheus and Grafana would normally run on separate monitoring infrastructure, with firewall restrictions, HTTPS, authentication, retention planning, and Alertmanager for notifications.

## Commands worth remembering

```bash
# Service state and boot configuration
systemctl is-active prometheus node_exporter grafana-server
systemctl is-enabled prometheus node_exporter grafana-server

# Local endpoint checks
wget -qO- http://localhost:9090/-/ready
wget -qO- http://localhost:9100/metrics | head
wget -qO- http://localhost:3000/api/health

# Prometheus target health
wget -qO- 'http://localhost:9090/api/v1/query?query=up'

# Logs and safe configuration validation
journalctl -u prometheus -n 50 --no-pager
sudo -u prometheus promtool check config /etc/prometheus/prometheus.yml
sudo -u prometheus promtool check rules /etc/prometheus/rules/node-alerts.yml
sudo systemctl reload prometheus
```

## Final self-test

Before an interview, practise answering these without reading:

1. Draw the flow from host metric to Grafana panel.
2. Explain why `up=1` is stronger evidence than only seeing a running Node Exporter process.
3. Explain the difference between Prometheus, Node Exporter, Grafana, and Alertmanager.
4. Describe the safe workflow for changing a Prometheus rule.
5. State one security decision and one scaling improvement for this lab.
