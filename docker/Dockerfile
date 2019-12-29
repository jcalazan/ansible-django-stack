# Use the official Ubuntu 18.04 base image from the Docker repository
FROM ubuntu:bionic

# Allow processes to detect that they are being run in a container
ENV container oci

# Install packages for sudo and OpenSSH Server
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y sudo openssh-server

# Add a "vagrant" user, and disable password-based login
RUN adduser --disabled-password --gecos "" vagrant

# Allow the "vagrant" user to login via SSH using the insecure keypair
RUN su -c "URL=https://github.com/hashicorp/vagrant/raw/master/keys/%s.pub ssh-import-id vagrant" vagrant

# Grant password-less sudo privileges to the "vagrant" user
RUN echo "vagrant ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/vagrant

# Start the OpenSSH Server when the container is run
RUN systemctl enable ssh

# Indicate the port number of the listening socket for SSH connections
EXPOSE 22

# Execute the init system (systemd) when the container is run
ENTRYPOINT ["/sbin/init"]

# Send the init process SIGRTMIN+3 when stopping the container
STOPSIGNAL SIGRTMIN+3
