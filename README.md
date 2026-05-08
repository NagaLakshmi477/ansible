
# Ansible

# Shell Script Execution

There are two common ways to execute a shell script:

```bash
sh <filename.sh>       # Executes using 'sh' shell
./<filename.sh>        # Executes directly (file must be executable)
```

## Disadvantages of Shell Scripting

- No proper error handling : If something goes wrong, it doesn’t clearly tell you what happened or why  
- Difficult to manage across multiple servers (not scalable) : It works fine on one or two machines, but when you have 10, 100, or 1000 servers, it becomes diffcult  
- Assumes a homogeneous environment  
- Syntax is sometimes hard to understand  
- Password security is not built-in  
- Scripts are not idempotent (re-running may cause issues)  

**💡 Idempotent**  

If you run the same command, action, or script many times,  
➡️ the final result stays the same every time.  

**⚠️ Not Idempotent**  

If you run the same command multiple times,  
➡️ the result keeps changing — it might add duplicates, throw errors, or mess things up.

---

# Configuration Management

Configuration Management is the process of **setting up and managing servers automatically**.  
It means we can run our applications without having to manually set up everything ourselves.

It usually involves:

- Installing required packages  
- Setting up programming languages  
- Creating folders and users  
- Downloading the code  
- Installing dependencies  
- Creating `systemd` / `systemctl` services  
- Starting the service or server


### Flow:

1. Provision server: This just means getting a new server ready to use
2. Configure server: Set up the server so it can run your app.
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

```
Control Server  --->  CM Server (Stores Configurations / Playbooks)
                           ^
                           |
                     -----------------
                     |       |       |
                  Node 1  Node 2  Node 3
                  (Agent)(Agent) (Agent)

```
# Architecture Overview

## 1. Control Server (e.g., GitHub)
- Acts as the **source of truth** for all Ansible files (playbooks, roles, inventories).

## 2. CM Server (Configuration Management Server)
- Intermediary repository between Control Server and nodes.
- **Gets updated files from the Control Server** (e.g., GitHub).
- Nodes **fetch configuration files** from the CM Server.

## 3. Nodes (Agents)
- Each node has an **agent installed**.
- Agent **polls the CM Server every 30 minutes**.
- Downloads and executes updates if found.



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

server 1:

ssh ec2-user@<server2 Ip>
ssh ec2-user@54.85.119.91 'echo "Hello Ansible " >/tmp/ansible.txt'
We can see the file in server2


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
ansible all -i  <node_privare_ip>, -e ansible_user=ec2-user -e ansible_password=DevOps321 -m ping
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
4. play
5. inventory

## data types
int,float,double,string,bool,list,map......

## conditions

using when condition 
when: <condition>

refer session 24---> notes

loops:

item is reserved keyword


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








if there are no modules avaialble
1.develop your own module
2.use shell or command line

shell vs Command:
Shell module: If your are using shell module your logging inside the server and ruinning the tasks. It conatin redirects,variabiles...

command module:
executing command from ouside. you will not get any linux full environement redirections piping and variables will not work here


write shell-command.yaml

- name: shell vs command
  hosts: frontend
  tasks:
  - name: redirects ls output to a file
    #ansible.builtin.command: "ls -ltr > /tmp/output.txt"
    ansible.builtin.shell: "ls -ltr > /tmp/output.txt"
    register: command_result
  
  - name: print the output
    ansible.builtin.debug:
      msg: "{{ command_result }}"


Write `inventory.ini` file

We can have any number of nodes.

```ini
[frontend]
<node1-privateIP> ansible_user=ec2-user ansible_password=DevOps321
<node2-privateIP> 

[backend]
<node1-privateIP>
<node2-privateIP>

[local:vars]

[database]
<node1-privateIP>
<node2-privateIP>



Initial Setup on Ansible-server
Bash:

git clone https://github.com/lakshmireddy15/ansible1.git

DELL@DESKTOP-LNC7QP1 MINGW64 ~/Documents/DevOps/DevOps_Projects/repos
$ cd ansible1

DELL@DESKTOP-LNC7QP1 MINGW64 ~/Documents/DevOps/DevOps_Projects/repos/ansible1 (main)
$ git add . ; git commit -m "ansible" ; git push origin main
[main 67aa0f5] ansible
 3 files changed, 34 insertions(+), 52 deletions(-)
Enumerating objects: 9, done.
Counting objects: 100% (9/9), done.
Delta compression using up to 4 threads
Compressing objects: 100% (5/5), done.
Writing objects: 100% (5/5), 755 bytes | 755.00 KiB/s, done.
Total 5 (delta 1), reused 0 (delta 0), pack-reused 0 (from 0)
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
To https://github.com/lakshmireddy15/ansible1.git
   dbd2cae..67aa0f5  main -> main


Mobaxterm (on Ansible-server):

sudo dnf install ansible -y 
git clone https://github.com/lakshmireddy15/ansible1.git


Output:

git clone https://github.com/lakshmireddy15/ansible1.git
Cloning into 'ansible1'...
remote: Enumerating objects: 47, done.
remote: Counting objects: 100% (47/47), done.
remote: Compressing objects: 100% (36/36), done.
remote: Total 47 (delta 21), reused 36 (delta 10), pack-reused 0 (from 0)
Receiving objects: 100% (47/47), 4.64 KiB | 2.32 MiB/s, done.
Resolving deltas: 100% (21/21), done.
```54.210.227.178 | 172.31.22.186 | t2.micro | null`
`[ ec2-user@ip-172-31-22-186 ~ ]$ cd ansible1`

`54.210.227.178 | 172.31.22.186 | t2.micro | https://github.com/lakshmireddy15/ansible1.git`
`[ ec2-user@ip-172-31-22-186 ~/ansible1 ]$`

[ ec2-user@ip-172-31-93-59 ~/ansible1 ]$ git pull
remote: Enumerating objects: 9, done.
remote: Counting objects: 100% (9/9), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 5 (delta 3), reused 5 (delta 3), pack-reused 0 (from 0)
Unpacking objects: 100% (5/5), 715 bytes | 715.00 KiB/s, done.
From https://github.com/lakshmireddy15/ansible1
   cac1c26..5e5b843  main       -> origin/main
Updating cac1c26..5e5b843
Fast-forward
 03-install.yaml |  9 ++++-----
 inventory.ini   |  2 +-
 notes.md        | 91 +++++++++----------------------------------------------------------------------------
 3 files changed, 14 insertions(+), 88 deletions(-)
 
Errors: 
beacause command line redirections are not working

[ ec2-user@ip-172-31-93-59 ~/ansible1 ]$ ansible-playbook -i inventory.ini 21-shell-command.yaml

PLAY [shell vs command] ***************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************
ok: [172.31.85.223]

TASK [redirects ls output to a file] **************************************************************************************************
fatal: [172.31.85.223]: FAILED! => {"changed": true, "cmd": ["ls", "-ltr", ">", "/tmp/output.txt"], "delta": "0:00:00.007431", "end": "2025-06-21 17:10:11.738259", "msg": "non-zero return code", "rc": 2, "start": "2025-06-21 17:10:11.730828", "stderr": "ls: cannot access '>': No such file or directory\nls: cannot access '/tmp/output.txt': No such file or directory", "stderr_lines": ["ls: cannot access '>': No such file or directory", "ls: cannot access '/tmp/output.txt': No such file or directory"], "stdout": "", "stdout_lines": []}

PLAY RECAP ****************************************************************************************************************************
172.31.85.223              : ok=1    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0

solution:
[ ec2-user@ip-172-31-93-59 ~/ansible1 ]$ ansible-playbook -i inventory.ini 21-shell-command.yaml

PLAY [shell vs command] ***************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************
ok: [172.31.85.223]

TASK [redirects ls output to a file] **************************************************************************************************
changed: [172.31.85.223]

PLAY RECAP ****************************************************************************************************************************
172.31.85.223              : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

register: It is used to store the ouitput 

[ ec2-user@ip-172-31-93-59 ~/ansible1 ]$ ansible-playbook -i inventory.ini 21-shell-command.yaml

PLAY [shell vs command] ***************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************
ok: [172.31.85.223]

TASK [redirects ls output to a file] **************************************************************************************************
changed: [172.31.85.223]

TASK [print the output] ***************************************************************************************************************
ok: [172.31.85.223] => {
    "msg": {
        "changed": true,
        "cmd": "ls -ltr > /tmp/output.txt",
        "delta": "0:00:00.006311",
        "end": "2025-06-21 17:22:07.634085",
        "failed": false,
        "msg": "",
        "rc": 0,
        "start": "2025-06-21 17:22:07.627774",
        "stderr": "",
        "stderr_lines": [],
        "stdout": "",
        "stdout_lines": []
    }
}












