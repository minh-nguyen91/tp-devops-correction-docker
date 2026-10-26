# Image used by the CD workflow to run the playbook
FROM python:3.12-slim

# ssh client for Ansible to connect to the server
RUN apt-get update \
    && apt-get install -y --no-install-recommends openssh-client \
    && rm -rf /var/lib/apt/lists/*

# the "ansible" package includes the community.docker collection
RUN pip install --no-cache-dir ansible==14.5.0

WORKDIR /ansible
