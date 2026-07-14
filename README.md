# Prometheus & Grafana Linux Monitoring

A hands-on DevOps project that monitors an Ubuntu VMware virtual machine with Prometheus, Node Exporter, and Grafana. Everything is installed manually with native Linux packages and `systemd` services— and no Docker, Kubernetes, or Compose had been used

## What this project demonstrates

- Linux administration, users, groups, ownership, permissions, and services
- Networking fundamentals: IP addresses, ports, listeners, and browser access
- Prometheus collection, storage, targets, and PromQL queries
- Node Exporter host metrics
- Grafana data sources, dashboards, panels, and dashboard export
- Careful Git documentation and repeatable troubleshooting

## Target architecture

```mermaid
flowchart LR
    U["Administrator browser"] -->|"3000 / HTTP"| G["Grafana"]
    G -->|"PromQL / 9090"| P["Prometheus"]
    P -->|"scrapes / 9100"| N["Node Exporter"]
    N -->|"reads"| L["Ubuntu VM host metrics"]
```

All three services will initially run on the same Ubuntu VM. The design still uses their real network endpoints, so it can later be expanded to monitor additional servers acording to the need

## Planned repository layout

```text
Prometheus-Grafana-Monitoring/
├── README.md
├── LICENSE
├── .gitignore
├── prometheus.yml                 # Version-controlled Prometheus configuration
├── docs/
│   ├── phase-0-planning.md
│   ├── installation.md
│   ├── architecture.md
│   ├── troubleshooting.md
│   ├── interview-questions.md
│   └── commands.md
├── screenshots/                    # PNG/JPG
├── dashboards/
│   └── linux-server-monitoring.json # Exported Grafana dashboard JSON
└── systemd/
    ├── prometheus.service            # Version-controlled service unit
    └── node_exporter.service         # Version-controlled service unit
```

## Technology choices

| Component                  | Role                                             | Default port |
| -------------------------- | ------------------------------------------------ | ------------ |
| Ubuntu 26.04 LTS VMware VM | Linux system being monitored                     | —            |
| Node Exporter              | Exposes host metrics in Prometheus format        | `9100`       |
| Prometheus                 | Scrapes and stores time-series metrics           | `9090`       |
| Grafana                    | Visualizes metrics with dashboards               | `3000`       |
| systemd                    | Starts, stops, restarts, and supervises services | —            |

## Phased learning roadmap

1. **Phase 0 — Planning and Git foundation**: repository, scope, VM plan, and Git workflow.
2. **Phase 1 — Ubuntu foundation**: installation, network checks, updates, Linux basics, permissions, and systemd. **Complete.**
3. **Phase 2 — Prometheus**: native installation, least-privilege user, directories, configuration, and service. **Complete.**
4. **Phase 3 — Node Exporter**: install it as a system service and validate host metrics. **Complete.**
5. **Phase 4 — Scraping**: configure and reload Prometheus, inspect targets, and introduce alert-rule concepts. **Complete.**
6. **Phase 5 — Grafana**: native installation, secure first login, data source, dashboard, panels, and export. **Complete.**
7. **Phase 6 — Portfolio polish**: screenshots, verified architecture, troubleshooting, and repository cleanup. **Complete.**
8. **Phase 7 — Interview practice**: explain every component and practise viva questions. **Complete.**

## Getting started

Read [the Phase 0 plan](docs/phase-0-planning.md) for the idea of the project and also to get where to start it.

For a deep project explanation , read the [interview and viva guide](docs/interview-questions.md).

## License

This repository uses the MIT [LICENSE](LICENSE).
