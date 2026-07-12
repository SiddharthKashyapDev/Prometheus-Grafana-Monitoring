# Phase 0 — Planning, repository, and Git foundation

## Objective

Create a clean, version-controlled home for the project before changing the Ubuntu VM. This phase establishes the scope, evidence standard, network plan, and Git workflow that make the rest of the project reproducible and credible in an interview.

## Concept explanation

Monitoring has four distinct jobs:

1. **Exporter** — exposes measurements in a format Prometheus can read. Node Exporter is the exporter in this project.
2. **Collector** — requests those measurements at fixed intervals and stores them. Prometheus is the collector.
3. **Time-series database** — stores values together with timestamps and labels. Prometheus includes one.
4. **Visualization** — turns queries over stored data into graphs and tables. Grafana provides this layer.

The first deployment uses one Ubuntu VM. Keeping the ports and service boundaries real is important: in a larger environment, Prometheus can scrape Node Exporter on other hosts with the same model.

## Scope and decisions

| Decision | Choice | Why |
| --- | --- | --- |
| Installation style | Manual/native Linux installation | Matches the learning goal and exposes `systemd`, permissions, paths, and logs. |
| Operating system | Ubuntu Server in VMware | A stable, common Linux server environment. |
| Service manager | `systemd` | Ubuntu’s standard manager for starting services at boot and observing failures. |
| Initial topology | All services on one VM | Keeps the first build understandable and inexpensive. |
| Version control | Git plus GitHub | Creates an auditable portfolio and recovery guide. |
| Secrets policy | Never commit credentials or keys | A public repository must not expose reusable access. |

## Prerequisites checklist

Complete this before starting Phase 1:

- [ ] VMware is installed and you can create or use an Ubuntu Server VM.
- [ ] The VM has at least 2 vCPUs, 2 GB RAM, and 20 GB disk for a comfortable learning setup.
- [ ] You have an Ubuntu Server ISO available (use an LTS release).
- [ ] You know whether VMware networking is NAT or bridged.
- [ ] You have a GitHub account and Git installed on the computer where this repository lives.
- [ ] You can copy commands from this repository into the Ubuntu VM.

### Why the network mode matters

The VM needs network access for updates and package downloads. NAT gives outbound access with less exposure; bridged networking makes the VM appear as another device on the local network. Either works for a single-VM lab. We will identify the VM IP and test access deliberately in Phase 1 rather than assuming a port is reachable.

## Files created in this phase

| File or directory | Purpose |
| --- | --- |
| `README.md` | Public portfolio overview, architecture, technology choices, and roadmap. |
| `.gitignore` | Prevents common secrets, runtime data, downloads, logs, and editor files from being committed. |
| `LICENSE` | Declares how others may use the repository; replace its holder placeholder before publishing. |
| `docs/phase-0-planning.md` | This plan and the learning notes for this phase. |
| `screenshots/` | Reserved for sanitized visual evidence. |
| `dashboards/` | Reserved for exported Grafana dashboard JSON. |

`prometheus.yml` is intentionally not created yet. It is a live service configuration file, so it belongs to the Prometheus configuration phase after we know the installed paths and users.

## Git concepts

Git stores changes in three places:

```text
working directory  →  staging area  →  local commit  →  GitHub remote
your edited files     selected files    saved history     shared backup
```

- The **working directory** is the project folder you edit.
- The **staging area** is your deliberate list of changes for the next commit.
- A **commit** is a named snapshot in your local repository.
- A **remote** is a hosted copy, such as the GitHub repository.

## Commands to run for this milestone

The repository has been initialized locally. Review the files first, then run these commands from the repository root:

```bash
git status
git add README.md LICENSE .gitignore docs/phase-0-planning.md screenshots/.gitkeep dashboards/.gitkeep
git commit -m "Initial project setup"
```

### What each command does

| Command | Explanation | Expected result |
| --- | --- | --- |
| `git status` | Shows files Git sees as new, modified, or staged. | The Phase 0 files appear as untracked before `git add`. |
| `git add …` | Puts only the listed Phase 0 files into the staging area. | `git status` lists them under “Changes to be committed.” |
| `git commit -m "Initial project setup"` | Saves the staged snapshot with an interview-friendly message. | Git reports one new root commit and the file count. |

If Git asks who you are, configure your identity once on this computer:

```bash
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
```

`user.name` and `user.email` are stored in Git configuration and identify the author of future commits. Use the email associated with your GitHub account if you want the commits linked to it. Do not copy the example email.

## GitHub setup after the first commit

Create an empty GitHub repository named `Prometheus-Grafana-Monitoring`. Do **not** let GitHub generate a README, `.gitignore`, or license, because this local repository already has them.

Then replace the placeholder URL and run:

```bash
git branch -M main
git remote add origin https://github.com/YOUR-USERNAME/Prometheus-Grafana-Monitoring.git
git push -u origin main
```

| Command | Explanation | Verification |
| --- | --- | --- |
| `git branch -M main` | Renames the current branch to the conventional primary branch name. | `git branch --show-current` prints `main`. |
| `git remote add origin …` | Records the GitHub repository URL under the conventional name `origin`. | `git remote -v` prints matching fetch and push URLs. |
| `git push -u origin main` | Uploads the commit and remembers `origin/main` as the default upstream. | Refresh the GitHub repository page and see the files. |

### Common mistakes and troubleshooting

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| `Author identity unknown` | Git name/email is not configured. | Run the two `git config --global` commands above with your own details, then commit again. |
| `remote origin already exists` | A remote was previously added. | Inspect it with `git remote -v`; if it is wrong, use `git remote set-url origin <correct-url>`. |
| Push is rejected because the remote has commits | GitHub initialized files remotely. | Clone/merge only after reviewing the difference, or create a new empty remote. Avoid force-pushing while learning. |
| A secret was staged | `.gitignore` does not remove a file already staged. | Remove it from staging with `git restore --staged <file>`, then verify with `git status`. Rotate any secret that was pushed. |

## Verification checklist

- [ ] `git status` shows this project directory is a Git repository.
- [ ] The planned directories and documentation files exist.
- [ ] The `README.md` renders the architecture diagram on GitHub.
- [ ] `LICENSE` contains your actual name or GitHub username before public release.
- [ ] After committing, `git log --oneline -1` shows `Initial project setup`.
- [ ] After pushing, the GitHub repository contains no credentials, archives, or runtime data.

## What you should be able to explain in an interview

1. Why is Node Exporter separate from Prometheus?  
   Node Exporter reads host-level data and exposes it; Prometheus collects it on a schedule and stores the history.
2. Why use a dedicated service manager?  
   `systemd` can start a service at boot, restart it after certain failures, run it as a restricted user, and centralize logs and status.
3. Why keep configuration and dashboards in Git?  
   They become reviewable, reproducible infrastructure documentation and can be restored on a fresh VM.
4. Why should Prometheus data not go into Git?  
   It is generated runtime data that changes constantly, grows quickly, and is not the source configuration needed to rebuild the system.

## Phase 0 summary

You now have a project scope, a realistic architecture, deliberate repository hygiene, and a repeatable Git milestone. The next phase starts only after this documentation is reviewed, committed, and—if you choose—pushed to GitHub.
