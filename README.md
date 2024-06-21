# Learning DevOps

### Here is the the overview of the my journey(ongoing) of learning DevOps. Here are some of the key points that I have shared and also provided all the resourses and code refernces I'm following.


## Basics

- DevOps is a culture or a practice or process of increasing or improving any organization’s ability to deliver applications.
    - It ensures CI/CD, automation, quality, continuous monitoring, continuous testing.

### SDLC

- Plan: Gather requirements
- Define: SRS
- Design: High Level Design (HLD), LLD
- Develop
- **Build(dev): Push the code in git**
- **Test(QE: quality ensure):**
- **Deploy: push into production**

Bold parts are the main parts for DevOps engineers to automate.

### VMs

- Logical partitions of the physical machines
- E.g. ec2 instance is a VM
- VMs are created using hypervisor like VMWARE, XEN, VirtualBox etc.

### Automate AWS VMs by scripting

AWS CLI

- Install AWS CLI: Using this we can use any AWS service from our local terminal
- Add to path
- Authenticate with the AWS account
    - Go to security credentials in the profile section in top right corner
    - Create new access key
    - Enter `aws configure` in terminal

AWS CFT

Boto3 (python module)

https://www.youtube.com/watch?v=cN4pt5KQ9eA&list=PLdpzxOOAlwvIKMhk8WhzN1pYoJ1YU8Csa&index=6

### Linux & Shell Scripting

https://www.youtube.com/watch?v=9jw9F6mcQDo&list=PLdpzxOOAlwvIKMhk8WhzN1pYoJ1YU8Csa&index=7

OS

- Kernel: It is primarily responsible for
    1. Device management
    2. Memory management
    3. Process managements 
    4. Handling System calls

Shell

- It is a way to talk to the OS

Some important CMDs to remember ( for details type `$man <cmd_name>`)

- cd 
- ls -ltr
- pwd
- free -g (memory)
- nproc (cpus)
- df -h (disk size)
- top (for above 3 in one place)

Scripting

- create a file with .sh extension e.g. test.sh
- Make it executable using chmod (4 read, 2 write, 1, execute) usr grp everyone then provide the file name
- To run use `./[test.sh](http://test.sh)` or `sh test.sh`
- The syntaxs are:

```bash
 #!/bin/bash (#! is called shebang, bash is the executable which executes the script)
 #!/bin/sh is now linked with #!/bin/dash so with need to write bash instead of bash as they have different syntax
 
 echo "Hello world" # it's the print statement
 
```

VIM

:q! quits

:wq! write then quit

i to insert

### Imp AWS services for DeOvps

1. **EC2** ← Elastic Compute Cloud (VMs)
2. **VPC** ← Virtual Private cloud (for security)
3. **EBS** ← Volume
4. **S3** ← Storage (encrypted by default)
5. **IAM** ← Identity and Access Management
6. **CloudWatch** ← takes care of monitoring and observability  (guardrail service)
7. **Lambda** ← serverless compute (we can directly run a program to use it without even creating any virtual instances, AWS will create an instance as per needs and perform the task accordingly, it is used for short actions or functionalities for which we don’t want an entire dedicated instance running 24x7, unlike ec2 instances which we use to deploy an entire application)
8. **CloudTrail** ← records API logs and preserve it for specific time
9. **Cloud Build Services** ← AWS Code Pipeline, AWS Code Build, AWS Code Deploy
10. **AWS configuration** ← Used for monitoring, one can write and lookup configurations and specific actions or remedies for any specific tasks (guardrail service)
11. **Billing and Costing**
12. **AWS KMS** ← Key Management Service
13. **AWS EKS** ← AWS Elastic Kubernetes Service (It is a managed service from Kubernetes by AWS)
14. **Fargate (container service), ECS(Elastic Container Service** ←  it is also a containerization service but it is a AWS proprietary service)
15. **ELK (ElasticSearch Logstash Kibana stack)** ← logging search mechanism (splunk is also similar)

### Configuration Management with Ansible

Configuration management (CM) in DevOps is the process of managing and controlling the configuration of software systems throughout their lifecycle. It involves tracking and maintaining the consistency and integrity of software and hardware components, settings, and dependencies. CM is a key part of the DevOps lifecycle and is necessary for managing complex software systems. It helps engineering teams build robust and stable systems, and helps businesses scale.

Advantages

- It is a push model (puppet has master/slave model)
- It is an agentless model
- Dynamic Inventory
- Support for windows(winRM) and Linux(ssh) both
- Simple (uses .yaml instead of any new language)
- We can write our own Ansible modules in python and share using Ansible galaxy
- For Ansible the cloud providers not matters like if it is AWS or Azure or GCP

Disadvantages

- For windows advanced ansible is a little bit tricky
- Not good debugging
- Not very good performance in terms of parallel executions

Project

prerequisite

- Install anisible
- setup password less authentication using ansible
    - Create public-key using ssh-keygen
    - It creates a private key and public key (with .pub extension). Private key should not be shared its only for logging into the current server.
    - setup the public key as authorized key in the target machine in “~/.ssh/authorized_keys”
- Inventory file is something which stores the IP addresses of the target servers.
    
    ```bash
    [grp name]
    IP address list
    ```
    
- ansible adhoc commands ([All modules](https://docs.ansible.com/ansible/2.9/modules/list_of_all_modules.html))
    
    ```bash
    ansible -i “inventory_filename” * -m “module_name” -a “cmd_we_want_to_execute_in_the_target_machine”
    ```
    
    **“*” ← `all` for every ips, `specific ip` for a specific ip address, `grp_name` for all the ips in a specific group.**
    

When we want to perform multiple tasks we use playbooks

```bash
ansible-playbook -i inventory “playbook_name.yml”
```

Example of a playbook installing node in all instances of test-grp

```yaml
---
- name: Install Node.js using NVM
  hosts: test
  become: yes
  tasks:
    - name: Update and upgrade apt packages
      apt:
        update_cache: yes
        upgrade: yes
        cache_valid_time: 3600

    - name: Install dependencies
      apt:
        name:
          - build-essential
          - libssl-dev
          - curl
        state: present

    - name: Clone NVM repository
      git:
        repo: 'https://github.com/nvm-sh/nvm.git'
        dest: ~/.nvm
        version: v0.39.7
        update: no

    - name: Add NVM to bash profile
      shell: |
        echo 'export NVM_DIR="$HOME/.nvm"' >> ~/.bashrc
        echo '[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"' >> ~/.bashrc
        echo '[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"' >> ~/.bashrc
      args:
        executable: /bin/bash

    - name: Source bash profile
      shell: source ~/.bashrc
      args:
        executable: /bin/bash

    - name: Install Node.js
      shell: |
        . ~/.nvm/nvm.sh
        nvm install node
      args:
        executable: /bin/bash

    - name: Verify Node.js installation
      shell: |
        . ~/.nvm/nvm.sh
        node -v
        npm -v
      register: node_version
      args:
        executable: /bin/bash

    - name: Display Node.js version
      debug:
        msg: "Node.js version: {{ node_version.stdout_lines }}"
```

**Ansible Roles**

> Used to structure complex playbooks
> 

> `ansible-galaxy role init “role-name”`
> 

> role-name is a newly created folder which will contain all of the tasks (in a folder named tasks)
> 

```yaml
---
- hosts: test

  roles: 
    - role-name
```

Reference

https://github.com/ansible/ansible-examples

### Infrastructure As Code (IAC)

> **IAC ← We automate our infrastructure with some code. For example, AWS CFT or AWS CLI could be used to automate the process of multiple ec2 instances at once with some scripts. Same thing we can think for Azure or GCP.**
> 

> **Terraform ← It is an API as code tool, where we don’t have to learn multiple tools (for IAC) for different cloud providers.**
> 
- **Migration between one cloud to another is very smooth in terraform.**
- **Also terraform is good for hybrid cloud infrastructure**
