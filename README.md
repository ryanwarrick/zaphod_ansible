![license](https://img.shields.io/github/license/ryanwarrick/zaphod_ansible)
![GitHub last commit](https://img.shields.io/github/last-commit/ryanwarrick/zaphod_ansible)
[![GitHub issues](https://img.shields.io/github/issues/ryanwarrick/zaphod_ansible)](https://github.com/ryanwarrick/zaphod_ansible/issues)
[![](https://tokei.rs/b1/github/ryanwarrick/zaphod_ansible)](https://github.com/ryanwarrick/zaphod_ansible)
[![](https://tokei.rs/b1/github/ryanwarrick/zaphod_ansible?category=files)](https://github.com/ryanwarrick/zaphod_ansible)
[![code with hearth by ryanwarrick](https://img.shields.io/badge/%3C%2F%3E%20with%20%E2%99%A5%20by-ryanwarrick-ff1414.svg?style=flat-square)](https://github.com/ryanwarrick)

# Summary
Zaphod_Ansible is a configuration and deployment automation tool. It is designed as a companion to [Zaphod](https://github.com/ryanwarrick/zaphod), a website I built with Python Flask.

## Initial Problem
Like most apps, Zaphod requires system dependencies and specific environment configurations to run in production. For example, Zaphod requires dependencies like Flask and Nginx while also requiring the configuration of services and file permissions.

While performing the initial install and config process to deploy my Zaphod website to a fresh OS image, it became clear that the finicky manual steps of the procedure are untenable in a mock DRP scenario. Not only is the admin time a burden, but redeploying from scratch wouldn't satisfy my self-defined RTO (recovery time objective). IT automation tools like Ansible are designed to help address these concerns.

## Ansible to the Rescue
Informed by a background in IT compliance, I deeply appreciate the importance of configuration management and validation processes. Throughout my experience in cybersecurity administration, I have observed a general need for compliance and efficiency improvements.

Ansible, Red Hat's open source IT automation tool, promises to provide help with these troublesome functions of IT. Consequently, I chose to learn about Ansible and how to use it to build this deployment automation tool.

## Solution
To consistently and repeatably create an appropriate production environment and deploy my Zaphod website, I chose to use Ansible to build a majority of my DevOps stack. As a result, I can execute this ansible-playbook against a fresh install of Linux in a cloud VM to reliably deploy the Zaphod website from a package file.

So far, Ansible has helped me with the deployment and administration of my Zaphod website by helping to:
- increase productivity by handling rote deployment and management tasks
- minimize misconfiguration risks by reducing the need for manual configuration tasks and, consequently, the opportunity for human error.
- provide a level of auditability otherwise impractical via manual server administration methods

# Control / Remote Node Requirements:

## Virtual Environment (Control Node):
It's strongly suggested that you install Python dependencies (i.e. Ansible, as seen below) in a virtual environment on your control node. See the python docs on [Virtual Environments](https://docs.python.org/3/tutorial/venv.html) for more info.

## Software Dependencies (Control Node)
These dependencies must be installed 
| Dependency | Version (developed & tested with) |
| ---------- | --------------------------------- |
| Python     | 3.8.x                             |
| Ansible*   | 4.4.x                             |
\* Installed via pip *(inside the virtual environment created below)*

Note: YMMV, I haven't explored or tested with other versions of these dependencies. Newer Python and Ansible versions may work just fine.

## Ansible Dependencies (Control Node):
The following Ansible role and collection dependencies must be installed to the Ansible instance within the control node's virtual environment. See 'Installing Ansible Dependencies' for instructions.

| Dependency                | Type       | Source | Version      |
| ------------------------- | ---------- | ------ | ------------ |
| nginxinc.nginx            | Collection | Galaxy | 0.20.0       |
| community.crypto          | Collection | Galaxy | 0.20.0       |
| geerlingguy.certbot       | Role       | Galaxy | 4.1.0        |
| geerlingguy.pip           | Role       | Galaxy | 2.1.0        |
| geerlingguy.git           | Role       | Galaxy | 3.0.0        |
| ansible-role-nginx-config | Role       | GitHub | 0.4.0 (main) |

Note: YMMV, I haven't explored or tested with other versions of these dependencies (for all except ansible-role-nginx-config, which does currently specifically require the built from source version available via GitHub).

### Installing Ansible Dependencies
From the project root directory AND with the aforementioned control node's virtual environment activated, execute the following commands to install the required dependencies:
1. Install required roles: `ansible-galaxy role install -r requirements.yml`
2. Install required collections: `ansible-galaxy collection install -r requirements.yml`

## SSH Private/Public Keys (Control and Remote Node):
Ansible communicates with remote nodes to perform automation tasks using SSH. To allow Ansible to establish SSH connections between your control and remote nodes, set up an SSH trust relationship between them using the following steps:
1. Create a key pair on the control node.
2. Transfer public key to remote node's authorized_keys file of X user.

To not reinvent the wheel, I'll link a well-made tutorial article on the topic published by Digital Ocean: [How To Set Up SSH Keys](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-2)

# Ansible Configuration:

To provide flexibility for ease of use with any environment and use case, this project is created in a modular fashion so that one can 'bring their own'  (hosts) and  (host_vars).

Please see the provided files as examples and edit as needed for your environment:
- [inventory listing](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html): project_root/hosts
  - Note: listed hosts must match host_var file names (see below)
- [host variables](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#assigning-a-variable-to-one-machine-host-variables): project_root/host_vars/{host1,host2,...}
  - ansible-specific variables: python interpreter, ansible_host, ansible_user
  - deployment-specific variables: server IP, domain name, etc.
- [group variables](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#assigning-a-variable-to-many-machines-group-variables): project_root/group_vars/webservers.yml
  - path to SSH private key from pair previously created and used in the 'SSH Private/Public Keys (Control and Remote Node)' section above
  - app name

# Usage
Once dependencies are satisfied and inventory/host_vars/group_vars are defined per the sections above, then you can run the Ansible setup.

From within the project root directory, execute the following command associated with the desired action
- Basic:
  - `ansible-playbook -i hosts site.yml`
- Limit Host Scope:
  - `ansible-playbook -i hosts --limit host1 site.yml`

# Files Not in Change Management (Manual Backups Required)
Some files within projects are not controlled by change management as specified by the .gitignore config because they contain sensitive data, therefore it's not appropriate to sync them to a change management repository.

Sensitive files:
* <project_root>/host_vars/<hostname>.yml (host-specific variable files for each deployment target host)
* <project_root>/my_hosts (custom inventory file to not clash with the example 'hosts' inventory file)

To avoid data loss of these files that aren't Git tracked, make sure to backup these files to a different location, as appropriate.

# TODO - Possible Future Improvements:
Below are some, but not all, possible future improvements to be developed for the project.
- [ ] Research more about certbot (letsencrypt) so that I can properly manage letsencrypt accounts and credentials.
- [ ] Test certbot's renew cert functionality (both running manually and via the configured cron job)
- [ ] Add functionality to create a new 'service account user'. Then, configure and deploy a remote node with the new service account (instead of current method of performing all functions using standard user account).
- [ ] improve nginx configurations to align with security best practices (security headers, server-side configs, etc.)
- [ ] improve file system security by implementing least privilege on server-related files/dirs.
- [ ] complete the modularization of the 'app name' so that others can substitute 'zaphod' for 'foo'.
- [ ] Learn about and integrate Ansible's cloud provider modules to build out automation capabilities such as managing cloud platform resources (VMs, VPCs, etc.).
- [ ] increase testing & improve support across various distros