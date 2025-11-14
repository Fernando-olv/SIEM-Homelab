FROM kalilinux/kali-rolling

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get install -y openssh-client hydra nmap iputils-ping && \
    rm -rf /var/lib/apt/lists/*

CMD ["/bin/bash"]