# SIEM Homelab with Elastic Stack

This repository contains a small, self-contained SIEM homelab designed for security analysis and log monitoring practice.
The environment is built using Docker Compose and includes:

- A Linux endpoint generating authentication logs
- Filebeat for log forwarding
- Elasticsearch for indexing and storage
- Kibana for dashboards and investigations
- An attacker container used to simulate SSH brute force attempts and other activities

The lab reproduces a simplified log ingestion pipeline to experiment with detection, log analysis, and incident investigation workflows.

---

## Overview

This setup is intentionally minimal. It focuses on the essential flow: endpoint → forwarder → SIEM.

Typical flow:

1. Endpoint writes events to `/var/log/auth.log`
2. Filebeat monitors the log file and forwards entries to Elasticsearch
3. Elasticsearch indexes the events
4. Kibana provides a UI to query and visualize the data
5. Attacker container generates activity (e.g., failed SSH logins) to populate logs

---

## Architecture

```mermaid
flowchart LR

  %% Layout & styles
  classDef infra fill:#f3f4f6,stroke:#333,stroke-width:1px;

  %% Groups / subgraphs
  subgraph Attacker ["Attacker"]
    A[Attacker Kali]
  end

  subgraph Monitored ["Monitored Endpoint"]
    E[Endpoint Ubuntu]
  end

  subgraph Forwarder ["Log Forwarder"]
    F[Filebeat]
  end

  subgraph Storage ["SIEM / Storage"]
    ES[Elasticsearch]
    K[Kibana]
  end

  %% Flows
  A -->|SSH brute-force / scans| E
  E -->|writes /var/log/auth.log| F
  F -->|forwards events| ES
  ES -->|queries / dashboards| K
  E --- F

  %% Apply style to nodes
  class A,E,F,ES,K infra;
  ```
---

## Components

**Elasticsearch & Kibana:** Elastic Stack acts as the SIEM backend. Containers expose:

- Elasticsearch: `http://localhost:9200`
- Kibana: `http://localhost:5601`

Security features are disabled by default for simplicity — do not use this configuration in production.

**Endpoint:** Lightweight Ubuntu-based container with:

- SSH enabled
- Demo user to exercise authentication
- Rsyslog/syslog to write authentication logs to a shared volume

**Filebeat:** Reads logs from the endpoint via a shared volume and forwards to Elasticsearch.

**Attacker:** Kali container used to generate test events (hydra, ssh, nmap).

---

## Quickstart

Clone and start the lab:

```bash
git clone <repo-url>
cd SIEM-Homelab
docker-compose build
docker-compose up -d
```

Check containers:

```bash
docker ps
```

Open Kibana: `http://localhost:5601` and create a data view for `filebeat-*` if prompted.

---

## Generating Logs / Tests

Enter the attacker container:

```bash
docker exec -it attacker bash
```

Connection test and simple SSH:

```bash
ssh demo@endpoint1
```

Example brute force (for testing only):

```bash
hydra -l demo -P /usr/share/wordlists/rockyou.txt ssh://endpoint1
nmap -sV endpoint1
```

These actions generate authentication events visible in Kibana → Discover.

---

## What You Can Practice

- Detecting SSH brute force patterns
- Event correlation and pivoting
- Building dashboards and visualizations
- Basic threat hunting in Kibana

---

## Future Improvements

- Add Winlogbeat + Windows endpoint
- Add Suricata for network traffic monitoring
- Integrate Wazuh manager or other EDR telemetry
- Prebuilt Kibana dashboards and detection rules

---

## License

This project is provided for educational and research purposes.


