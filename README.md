# SIEM Homelab with Elastic Stack

This repository contains a small, self-contained SIEM homelab designed for security analysis and log monitoring practice.
The environment is built using Docker Compose and includes:

- A Linux endpoint generating authentication logs
- Filebeat for log forwarding
- Elasticsearch for indexing and storage
- Kibana for dashboards and investigations
- An attacker container used to simulate SSH brute force attempts and other activities

The goal of this lab is to replicate a simplified version of a real log ingestion pipeline and provide a practical environment for experimenting with detection, log analysis, and incident investigation workflows.

---

## 1. Overview

This setup is intentionally minimalistic. It focuses only on the essentials needed to demonstrate how endpoint logs travel through a log forwarder and into a SIEM for analysis.
No complex orchestration, external dependencies, or unnecessary components.

Typical workflow:

1. The Linux endpoint writes events to /var/log/auth.log
2. Filebeat monitors the log file and forwards new entries to Elasticsearch
3. Elasticsearch indexes the events
4. Kibana provides a UI to query and visualize the data
5. The attacker container can generate activity such as failed SSH logins to populate meaningful logs

This architecture mirrors what is commonly found in enterprise environments, just scaled down.

---

## 2. Architecture Diagram

+----------------+        +-------------------+        +----------------+
|   Attacker     | -----> |    Endpoint        | -----> |    Filebeat    |
| (SSH Brute)    |        | (SSH + Auth Logs)  |        |  (Log Forward) |
+----------------+        +-------------------+        +--------+-------+
                                                                  |
                                                                  v
                                                        +------------------+
                                                        |  Elasticsearch   |
                                                        |  (Log Storage)   |
                                                        +--------+---------+
                                                                  |
                                                                  v
                                                        +------------------+
                                                        |     Kibana       |
                                                        | (UI for Analysis)|
                                                        +------------------+

---

## 3. Components

### Elasticsearch & Kibana
Elastic Stack is used as the SIEM backend.
Both services run as containers and are reachable locally on the following ports:

- Elasticsearch: http://localhost:9200
- Kibana: http://localhost:5601

Security features are disabled for simplicity.

---

### Endpoint
A lightweight Ubuntu-based container configured with:

- SSH service enabled
- A demo user for authentication attempts
- Rsyslog writing authentication logs to a mounted directory

This represents a monitored server in a real environment.

---

### Filebeat
Filebeat runs as a separate container and reads logs from the endpoint via a shared volume.

It forwards events to Elasticsearch in near real-time.

---

### Attacker
A Kali-based container used only to generate test data.
Contains tools like:

- Hydra
- SSH client
- Nmap

Example brute force command:

hydra -l demo -P /usr/share/wordlists/rockyou.txt ssh://endpoint1

---

## 4. Running the Lab

Clone the repository and run:

docker-compose build
docker-compose up -d

Verify everything is running:

docker ps

Access Kibana:

http://localhost:5601

If this is your first run, create a data view pointing to:

filebeat-*

Kibana → Stack Management → Index Patterns → Create.

---

## 5. Testing & Generating Logs

Enter the attacker container:

docker exec -it attacker bash

Basic connection test:

ssh demo@endpoint1

Brute force attempts:

hydra -l demo -P /usr/share/wordlists/rockyou.txt ssh://endpoint1

Port scan:

nmap -sV endpoint1

All these actions will produce events visible in Kibana’s Discover tab.

---

## 6. What You Can Practice

- SSH brute force detection patterns
- Event correlation
- Creating dashboards
- Basic threat hunting
- Writing simple detections in Kibana
- Understanding endpoint → forwarder → SIEM flow

This environment is intentionally small so you can focus on log behavior rather than infrastructure.

---

## 7. Future Improvements

Some extensions planned:

- Add Winlogbeat + Windows endpoint
- Suricata network traffic monitoring
- Wazuh manager side-by-side with Elastic
- Prebuilt Kibana dashboards
- Automated attack sequences

---

## 8. License

This project is provided for educational and research purposes.

