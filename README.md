 ### ELK Project
 
 Automated ELK Stack Deployment

This document contains the following details:
- Description of the Topology
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build
- Access Policies

### Description of the Topology
This repository includes code defining the infrastructure below. 

![](https://github.com/asolis8649/ELK-Project/blob/main/Images/Cloud%20Security%20Diagram.png)

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the "D*mn Vulnerable Web Application"

Load balancing ensures that the application will be highly available, in addition to restricting inbound access to the network. The load balancer ensures that work to process incoming traffic will be shared by both vulnerable web servers. Access controls will ensure that only authorized users will be able to connect in the first place.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the file systems of the VMs on the network, as well as watch system metrics, such as CPU usage; attempted SSH logins; `sudo` escalation failures; etc.

The configuration details of each machine may be found below.

| Name     |   Function  | IP Address | Operating System |
|----------|-------------|------------|------------------|
| Jump Box | Gateway     | 40.83.180.149   | Linux            |
| WEB-1   | Web Server  | 10.0.0.15   | Linux            |
| WEB-2   | Web Server  | 10.0.0.16  | Linux            |
| WEB-3    | Web Server  | 10.0.0.18   | Linux            |
| ELK      | Monitoring  | 10.2.0.4   | Linux  |

In addition to the above, Azure has provisioned a load balancer in front of all machines except for the jump box. The load balancer's targets are organized into the following availability zones:
- **Availability Set: RedTeam-AS**: WEB-1, WEB-2, WEB-3

## ELK Server Configuration
The ELK VM exposes an Elastic Stack instance. Docker is used to download and manage an ELK container.

Rather than configure ELK manually, we opted to develop a reusable Ansible Playbook to accomplish the task. This playbook is duplicated below.


To use this playbook, one must log into the Jump Box, then issue: ansible-playbook install_elk.yml elk. This runs the install_elk.yml playbook on the ELK host.

### Access Policies
The machines on the internal network are _not_ exposed to the public Internet. 

Only the **Jump Box** machine can accept connections from the Internet. Access to this machine is only allowed from the IP address 47.229.62.35.

Machines within the network can only be accessed by each other. The WEB-1, WEB-2, and WEB-3 VMs send traffic to the ELK server.

A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible | Allowed IP Addresses |
|----------|---------------------|----------------------|
| Jump Box | Yes                 | 47.229.62.35         |
| ELK      | No                  | 10.0.0.1-254         |
| WEB-1   | No                  | 10.0.0.1-254         |
| WEB-2   | No                  | 10.0.0.1-254         |
| WEB-3   | No                | 10.0.0.1-254           |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because it allows for quick and accurate configuration of one or many machines at one time, minimizing the risk for human error.

The playbook implements the following tasks:
- Install Docker
- Install Python3
- Install docker module
- Increase Virtual memory
- Increase virtual memory on restart
- Download and launch a docker elk container
- Enable service docker on boot


The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

  ![](https://github.com/asolis8649/ELK-Project/blob/main/Images/ELK%20docker%20ps.PNG)



The playbook is duplicated below.

```yaml
---
# install_elk.yml
- name: Configure Elk VM with Docker
  hosts: elkservers
  remote_user: elk
  become: true
  tasks:
    # Use apt module
    - name: Install docker.io
      apt:
        update_cache: yes
        name: docker.io
        state: present

      # Use apt module
    - name: Install pip3
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present

      # Use pip module
    - name: Install Docker python module
      pip:
        name: docker
        state: present

      # Use command module
    - name: Increase virtual memory
      command: sysctl -w vm.max_map_count=262144

      # Use sysctl module
    - name: Use more memory
      sysctl:
        name: vm.max_map_count
        value: "262144"
        state: present
        reload: yes

      # Use docker_container module
    - name: download and launch a docker elk container
      docker_container:
        name: elk
        image: sebp/elk:761
        state: started
        restart_policy: always
        published_ports:
          - 5601:5601
          - 9200:9200
          - 5044:5044
```

### Target Machines & Beats
This ELK server is configured to monitor the WEB-1, WEB-2, WEB-3, at `10.0.0.15`, `10.0.0.16` and `10.0.0.18`, respectively.

We have installed the following Beats on these machines:
- Filebeat
- Metricbeat

These Beats allow us to collect the following information from each machine:
- **Filebeat**: Filebeat detects changes to the filesystem. Specifically, we use it to collect Apache logs.
- **Metricbeat**: Metricbeat detects changes in system metrics, such as CPU usage. We use it to detect SSH login attempts, failed `sudo` escalations, and CPU/RAM statistics.

The playbook below installs filebeat on the target hosts. The playbook for installing metricbeat is not included, but looks essentially identical — simply replace `filebeat` with `metricbeat`, and it will work as expected.

```yaml
---
- name: installing and launching filebeat
  hosts: webservers
  become: yes
  tasks:

  - name: download filebeat deb
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.6.1-amd64.deb

  - name: install filebeat deb
    command: dpkg -i filebeat-7.6.1-amd64.deb

  - name: drop in filebeat.yml
    copy:
      src: /etc/ansible/filebeat-config.yml
      dest: /etc/filebeat/filebeat.yml

  - name: enable and configure system module
    command: filebeat modules enable system

  - name: setup filebeat
    command: filebeat setup

  - name: start filebeat service
    command: service filebeat start

  - name: enable service filebeat on boot
    systemd:
      name: filebeat
      enabled: yes
---
```

### Using the Playbooks
In order to use the playbook, you will need to do the following:
1. SSH into Jump-Box-Provisioner from local host
2. From Jump-Box start and attach to docker container with the following command:`sudo docker container start vigilant_villani && sudo docker container attach vigilant_villani`
3. Change directory to /etc/ansible and this is where playbooks will be created and kept

Next, you must create a `hosts` file to specify which VMs to run each playbook on. Run the commands below:

```bash
$ cd /etc/ansible
$ nano hosts
# This is the default ansible 'hosts' file.
#
# It should live in /etc/ansible/hosts
#
#   - Comments begin with the '#' character
#   - Blank lines are ignored
#   - Groups of hosts are delimited by [header] elements
#   - You can enter hostnames or ip addresses
#   - A hostname/ip can be a member of multiple groups

# Ex 1: Ungrouped hosts, specify before any group headers.

## green.example.com
## blue.example.com
## 192.168.100.1
## 192.168.100.10

# Ex 2: A collection of hosts belonging to the 'webservers' group

 **[webservers]**
**10.0.0.15 ansible_python_interpreter=/usr/bin/python3**
**10.0.0.16 ansible_python_interpreter=/usr/bin/python3**
**10.0.0.18 ansible_python_interpreter=/usr/bin/python3**

 **[elk]**
**10.2.0.4 ansible_python_interpreter=/usr/bin/python3**

# If you have multiple hosts following a pattern you can specify
# them like this:

## www[001:006].example.com

# Ex 3: A collection of database servers in the 'dbservers' group

## [dbservers]
##
## db01.intranet.mydomain.net
## db02.intranet.mydomain.net
## 10.25.1.56
## 10.25.1.57

# Here's another example of host ranges, this time there are no
# leading 0s:

## db-[99:101]-node.example.com
```

After this, the commands below run the playbook:

 ```bash
 $ cd /etc/ansible
 $ ansible-playbook install_elk.yml 
 $ ansible-playbook install_filebeat.yml 
 $ ansible-playbook install_metricbeat.yml 
 ```

To verify success, wait five minutes to give ELK time to start up. 

Then, run: `curl http://10.2.0.4:5601`. This is the address of Kibana. If the installation succeeded, this command should print HTML to the console.


---


