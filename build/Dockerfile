FROM almalinux:latest

# Install systemd and SSH server
RUN dnf update -y && \
    dnf install -y systemd openssh-server && \
    ssh-keygen -A && \
    systemctl enable sshd.service

# Copy SSH keys for root user
RUN mkdir /root/.ssh && \
  chmod 700 /root/.ssh && \
  ssh-keygen -f /root/.ssh/id_rsa -N '' && \
  cp /root/.ssh/id_rsa.pub /root/.ssh/authorized_keys && \
  chmod 600 /root/.ssh/authorized_keys

# Expose ports for SSH
EXPOSE 22 6123 8081

CMD ["/usr/lib/systemd/systemd"]
