FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive

# Basic packages: SSH, rsyslog, tools
RUN apt-get update && \
    apt-get install -y wget gnupg2 ca-certificates lsb-release apt-transport-https \
                       openssh-server rsyslog && \
    mkdir -p /var/run/sshd && \
    rm -rf /var/lib/apt/lists/*

# Install Elastic APT repo + Filebeat
RUN wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | gpg --dearmor -o /usr/share/keyrings/elastic-keyring.gpg && \
    echo "deb [signed-by=/usr/share/keyrings/elastic-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" > /etc/apt/sources.list.d/elastic-8.x.list && \
    apt-get update && \
    apt-get install -y filebeat && \
    rm -rf /var/lib/apt/lists/*

# Demo user for SSH
RUN useradd -ms /bin/bash demo && echo "demo:Password123!" | chpasswd

# Enable password authentication for SSH (lab only)
RUN sed -i 's/^#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config && \
    sed -i 's/^PasswordAuthentication no/#PasswordAuthentication no/' /etc/ssh/sshd_config

# Copy Filebeat config from build context
COPY filebeat.yml /etc/filebeat/filebeat.yml

# Expose SSH inside Docker network (no need to map to host)
EXPOSE 22

# Start rsyslog, then Filebeat, then sshd
CMD ["bash", "-c", "service rsyslog start && filebeat -e & /usr/sbin/sshd -D"]
