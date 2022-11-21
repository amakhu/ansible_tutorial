# ANSIBLE TUTORIAL

## Ansible Installation

**Controller node requirements**:
- Python 3.x
- An SSH client
- Access to an Ansible Repository
- A dedicated user account that is configured with SSH and sudo permissions on managed hosts

**Using redhat repository to install Ansible**:
- subscription-manager repos --list
- subscription-manager repos --enable=ansible-2-for-rhel-8-x86_64-rpms
- yum install ansible
- ansible --version

**Using python-pip to install Ansible**:
- yum install -y python3-pip
- alternatives --set python /usr/bin/python3
- pip3 install ansible --user
- ansible --version

**Configuring Managed Hosts**:
- systemctl status sshd
- rpm -qa | grep python
- firewall-cmd --list-all

**Configuring the Ansible User**:
- The user must be able to SSH into the managed machines
- The user must be enabled to run tasks as root on the managed machines

**On control node**:
- ssh-keygen
- ssh-copy-id <managed_node_ip/fqdn>
- ssh <managed_node_ip/fqdn>

**Setting up privilege escalation on managed nodes**:
- echo "ansible ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/ansible
