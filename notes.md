
# Ansible

# Shell Script Execution

There are two common ways to execute a shell script:

```bash
sh <filename.sh>       # Executes using 'sh' shell
./<filename.sh>        # Executes directly (file must be executable)
```

## Disadvantages of Shell Scripting

- No proper error handling
- Difficult to manage across multiple servers (not scalable)
- Assumes a homogeneous environment
- Syntax is sometimes hard to understand
- Password security is not built-in
- Scripts are not idempotent (re-running may cause issues)

---

# Configuration Management

Configuration Management is the process of provisioning and configuring servers for application hosting. It involves:

- Installing required packages
- Setting up programming languages
- Creating folders and users
- Downloading the code
- Installing dependencies
- Creating systemd/systemctl services
- Starting the service/server

### Flow:

1. Provision server
2. Configure (as described above)
3. Deploy application:
   - Remove old versions
   - Download new version
   - Restart services

---

# Pull vs Push Model in Configuration Management

## Pull-Based Model (e.g., Chef)

- Configuration scripts are available on the **CM Server**.
- Nodes (agents) check the CM server every 30 minutes for new configurations.
- If new configurations are found, they are downloaded and executed on the node.

### Architecture:

```plaintext
Control Server ----> CM Server (Configurations) <----- Node 1 (Agent)
                                                        Node 2 (Agent)
                                                        Node 3 (Agent)
```

### Disadvantages:

1. Unnecessary traffic (nodes poll every 30 minutes even if no changes)
2. Wasted system resources
3. Increased power usage
4. Requires agent software
5. Protocols developed for agent communication

---

## Push-Based Model (e.g., Ansible)

- Ansible Server contains both the configurations and acts as the control/master node.
- Ansible uses **SSH protocol** to push configurations directly to nodes.

### Architecture:

```plaintext
Ansible Server (Config) ----SSH----> Node 1
                                    Node 2
                                    Node 3
```

### Advantages:

1. Uses secure SSH protocol
2. No agent software required
3. No unnecessary resource usage
4. Easier to manage and scale

---


# Ansible Remote Login and Basic Usage

## Overview

Ansible allows remote login to another server from a control server (e.g., server1 to server2) using SSH. This method allows operations to be performed on remote nodes from the control machine.

### In the Background:
1. An SSH connection is established.
2. A file with the desired content or instructions is created.
3. The connection is closed after execution.

## Architecture

- **Control Node (server1)**: The Ansible server where Ansible is installed.
- **Managed Nodes (server2 and others)**: Target machines where tasks are performed via SSH.
- **Note**: No need to install anything on the node side.

Ansible can connect to **multiple servers simultaneously** and execute commands.

---

## Installing Ansible on the Control Server

```bash
sudo dnf install ansible -y
```

---

## Ansible Syntax

General structure:

```bash
<command> <options> <inputs>
```

---

## Check Connection to Node

```bash
ansible all -i  <privare_io>, -e ansible_user=ec2-user -e ansible_password=DevOps321 -m ping
```

---

## Install and Start NGINX on Remote Node

### Install NGINX

```bash
ansible all -i 172.31.80.5, -e ansible_user=ec2-user -e ansible_password=DevOps321 -b -m dnf -a "name=nginx state=installed"
```

### Start NGINX Service

```bash
ansible all -i ,PRVIATEIP>, -e ansible_user=ec2-user -e ansible_password=DevOps321 -b -m service -a "name=nginx state=started"

```

---

## Notes

- `-i` specifies the inventory (in this case, a single IP).
- `-e` is used to pass extra variables like username and password.
- `-b` allows privilege escalation (e.g., sudo).
- `-m` specifies the module (e.g., `ping`, `dnf`, `service`).
- `-a` allows you to pass arguments to the module.

## variables

variable holds the values we can use whever we want if we change one place it will reflect to all places where you are reffreing

## variables prefrerences

1.arrguemnts/command line
# ansible-playbook -i inventory.ini 12-vars-preference.yaml -e "GREETING=hellofromarrgs"
2. task
3. file
4. prompt
5. play
6. inventory

## data types
int,float,double,string,bool,list,map......

## conditions


module:
========
A module is a small tool that ansible uses to do spefic task (install,copy, run commands)
we have 2 modules
1. shell
2. command

Command:
=========
Runs command directly on system
Doesn't use shell
Cannot use things like |, > , Environment variables ($Home)

Shell:
=======
Runs a command throught a shell
can use shell features |, > >> , varibales, operators