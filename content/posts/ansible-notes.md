---
author: alex
category:
  - devnet
  - study-material
date: "2024-05-19T13:49:53+00:00"
guid: https://netwithalex.blog/?p=704
title: Ansible Notes
url: /ansible-notes/

---
### Introduction

[_This post is part of DEVNET DevOps (300-910) – Blueprint_](/study-devops-blueprint/)

Ansible is an open-source automation tool used for configuration management, application deployment and task automation. Although it can be used for a lot of purposes, we are particularly interested on network automation related tasks.

There are a lot of resources to learn Ansible online, these notes should serve only as guidance, but you are encouraged to go deeper at your own pace.

### Key Features

- **Agentless:** Ansible doesn't require agents to be installed on managed nodes, making it lightweight and easy to deploy.
- **YAML-based:** Ansible playbooks and configuration files are written in YAML, human-readable and easy to understand.
- **Idempotent:** Ansible ensures that the desired state of the system is maintained, making it safe to run multiple times.

### Components

- **Control Node**: The machine where Ansible is installed and from which automation is initiated.
- **Managed Nodes**: The machines managed by Ansible. Ansible connects to managed nodes via SSH (Linux) or WinRM (Windows) to execute tasks.
- **Inventory**: A list of managed nodes that Ansible will manage.
- **Playbooks**: YAML files containing a set of tasks to be executed on managed nodes.
- **Modules**: Units of code that Ansible executes on managed nodes to perform tasks.

### Requirements to Use Ansible

- **Control Node requirements**
  - Ansible requires python to be installed
  - Can be installed with yum, apt or pip
  - Most commonly installed on Linux or macOS

### Inventory File

- Text file containing a list of managed nodes that Ansible can connect to and manage in this case our network devices.
- It can be in INI or YAML format
- You can group nodes together and define variables. See multiple examples [here](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html)

### Playbook

YAML files that define a set of tasks to be executed on managed nodes. They are the heart of Ansible automation.

#### Structure

- **Hosts**: Specifies the target hosts or groups (inventory)
- **Tasks**: Contains a list of tasks to be executed on managed nodes.
- **Variables**: Defines variables used within the playbook.

Playbooks use modules to interact with the managed modes . I recommend getting familiar with some [Cisco modules](https://docs.ansible.com/ansible/latest/collections/cisco/index.html)

### Practical Examples

You can find some practical examples on the following [GitHub](https://github.com/aruiz-p/NetWithAlex-Ansible/tree/main) repository.

### [Back to BluePrint](/study-devops-blueprint/)
