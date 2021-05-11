## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![Diagram of the Network](https://github.com/zwang21/Elk-Stack-Project/blob/main/Diagrams/Diagram_of_the_Network.png)
(Images/Diagram_of_the_Network.png)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the playbook file may be used to install only certain pieces of it, such as Filebeat.

Playbook file 1: pentest.yml

```yaml

- name: Config Web VM with Docker
  hosts: webservers
  become: true
  tasks:
    - name: docker.io
      apt:
        update_cache: yes
        name: docker.io
        state: present

    - name: Install pip3
      apt:
        name: python3-pip
        state: present

    - name: Install Docker python module
      pip:
        name: docker
        state: present

    - name: download and launch a docker web container
      docker_container:
        name: dvwa
        image: cyberxsecurity/dvwa
        state: started
        restart_policy: always
        published_ports: 80:80

    - name: Enable docker service
      systemd:
        name: docker
        enabled: yes

```

Playbook file 2: install-elk.yml 

```yaml
- name: Configure Elk VM with Docker
  hosts: elk
  remote_user: ubuntu
  become: true
  tasks:
    # Use apt module
    - name: Install docker.io
      apt:
        update_cache: yes
        force_apt_get: yes
        name: docker.io
        state: present

      # Use apt module
    - name: Install python3-pip
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present

      # Use pip module (It will default to pip3)
    - name: Install Docker module
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
        value: 262144
        state: present
        reload: yes

      # Use docker_container module
    - name: download and launch a docker elk container
      docker_container:
        name: elk
        image: sebp/elk:761
        state: started
        restart_policy: always
        # Please list the ports that ELK runs on
        published_ports:
          -  5601:5601
          -  9200:9200
          -  5044:5044

      # Use systemd module
    - name: Enable service docker on boot
      systemd:
        name: docker
        enabled: yes
```

Playbook file 3: filebeat-playbook.yml

```yaml
---
- name: installing and launching filebeat
  hosts: webservers
  become: yes
  tasks:
  - name: download filebeat deb
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.6.1-amd64.deb

  - name: install filebeat deb
    command: sudo dpkg -i filebeat-7.6.1-amd64.deb

  - name: drop in filebeat.yml
    copy:
      src: /etc/ansible/files/filebeat-config.yml
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

```

Playbook file 4: metricbeat-playbook.yml

```yaml
---
- name: installing and launching filebeat
  hosts: webservers
  become: yes
  tasks:

  - name: download filebeat deb
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.6.1-amd64.deb

  - name: install filebeat deb
    command: sudo dpkg -i filebeat-7.6.1-amd64.deb

  - name: drop in filebeat.yml
    copy:
      src: /etc/ansible/files/filebeat-config.yml
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
```

Playbook file 5: metricbeat-playbook-elk.yml

```yaml
---
- name: Install metric beat
  hosts: elk
  become: true
  tasks:
    # Use command module
  - name: Download metricbeat
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.6.1-amd64.deb

    # Use command module
  - name: install metricbeat
    command: dpkg -i metricbeat-7.6.1-amd64.deb

    # Use copy module
  - name: drop in metricbeat config
    copy:
      src: /etc/ansible/files/metricbeat-config.yml
      dest: /etc/metricbeat/metricbeat.yml

    # Use command module
  - name: enable and configure docker module for metric beat
    command: metricbeat modules enable docker

    # Use command module
  - name: setup metric beat
    command: metricbeat setup

    # Use command module
  - name: start metric beat
    command: service metricbeat start

    # Use systemd module
  - name: enable service metricbeat on boot
    systemd:
      name: metricbeat
      enabled: yes
```

This document contains the following details:

- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build

### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the Damn Vulnerable Web Application.

Load balancing ensures that the application will be highly reliable and capable, in addition to restricting access to the network.

What aspect of security do load balancers protect? 

- Mitigates DoS attacks.

What is the advantage of a jump box?

- Increase web security;
- Easily install, update, maintenance, and monitor through single VM gateway.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the network traffic and system files.

What does Filebeat watch for?

- Collects data about the file system.
 
What does Metricbeat record?

- Collects machine metrics which is simply a measurement about an aspect of a system that tell analysts how "healthy" it is.

The configuration details of each machine may be found below.

|Name      | Function |IP Address| Operating System |
|----------|----------|----------|------------------|
|Jump Box  | Gateway  | 10.0.0.7 |Linux ubuntu 18.04|
|  Web-1   |Web server| 10.0.0.5 |Linux ubuntu 18.04|
|  Web-2   |Web server| 10.0.0.6 |Linux ubuntu 18.04|
|  Web-3   |Web server| 10.0.0.9 |Linux ubuntu 18.04|
|Elk server|Monitoring| 10.1.0.4 |Linux ubuntu 18.04|

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the Jump Box machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:

- Add whitelisted IP addresses: 76.209.224.121

Machines within the network can only be accessed by Jump Box.

Which machine did you allow to access your ELK VM? What was its IP address? 

- Allow jump box private IP SSH through port 22
- Its IP address is 10.0.0.7

A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible |Allowed IP Addresses|
|:---------|:--------------------|:------------------:|
| Jump Box | Yes (SSH p22)       | 76.209.224.121     |
| Jump Box | Yes (RDP p3389)     | 76.209.224.121     |
| Web-1    | Yes (HTTP p80)      | 76.209.224.121     |
| Web-2    | Yes (HTTP p80)      | 76.209.224.121     |
| Web-3    | Yes (HTTP p80)      | 76.209.224.121     |
|Elk server| Yes (HTTP p5601)    | 76.209.224.121     |
|Elk Server| Yes (SSH p22)       | 76.209.224.121     |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because:

- Free: Ansible is an open-source tool.
- Very simple to set up and use: Automation simplifies complex tasks. No special coding skills are necessary to use Ansible's playbooks.
- Powerful: Ansible lets you model even highly complex IT workflows.
- Flexible: You can orchestrate the entire application environment no matter where it's deployed.
- Agentless and Efficient: You don’t need to install any other software or firewall ports on the client systems you want to automate.

The playbook implements the following tasks:

- Use the Ansible `apt` module to install `docker.io` and `python3-pip`;
- Use the Ansible `pip` module to install `docker`;
- Use the Ansible `command` module to increase virtual memory;
- Use the Ansible `sysctl` module to use more virtual memory;
- Use the Ansible `docker_container` module to download and launch Elk container;
- Use the Ansible `systemd` module to enable service docker on boot.

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

![screenshot displays the result of running `docker ps`](https://github.com/zwang21/Elk-Stack-Project/blob/main/Diagrams/docker_ps_output.png)
(Image/Screenshot of docker_ps_output.png)

### Target Machines & Beats

This ELK server is configured to monitor the following machines:
 
- Web-1: 10.0.0.5
- Web-2: 10.0.0.6
- Web-3: 10.0.0.9

We have installed the following Beats on these machines:

- `Filebeat`
- `Matricbeat`

These Beats allow us to collect the following information from each machine:

- `Filebeat` collects data about the file system. Filebeat enables analysts to monitor files for suspicious changes. Such as `system Filebeat` module collects and parses logs wrriten by the system logging service of common Unix/Linux based distributions.
- `Metricbeat` collects machine metrics, such as uptime.

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the configuration file to Ansible container.
- Update the the configuration file to include Elk server's IP address, which is 10.1.0.4 in my project.
- Run the playbook, and navigate to the bottom of [http://Elk-server-ip:5601/app/kibana](http://<Elk-server-ip>:5601/app/kibana) and click on **Verify Incoming Data** to check that the installation worked as expected.

Answer the following questions to fill in the blanks:
- Which file is the playbook? 
	- filebeat-playbook.yml and metricbeat-playbook.yml 

- Where do you copy it? 
	- I copied into /etc/ansible/files/

- Which file do you update to make Ansible run the playbook on a specific machine? 
	- filebeat-config.yml, metricbeat-config.yml

- How do I specify which machine to install the ELK server on versus which to install Filebeat on?
	- Edit in beat configuration file templates 

```
    output.elasticsearch:
    hosts: ["10.1.0.4:9200"]
    username: "elastic"
    password: "changeme"
    
    ...
    
    setup.kibana:
    host: "10.1.0.4:5601"
```

- Which URL do you navigate to in order to check that the ELK server is running?
	- http://52.188.106.10:5601/app/kibana

*As a **Bonus**, provide the specific commands the user will need to run to download the playbook, update the files, etc.*

To download files from GitHub, you should navigate to the top level of the project (**Elk-Stack-Project** in this case) and then a green "Code" download button will be visible on the right. Choose the Download ZIP option from the Code pull-down menu. That ZIP file will contain the entire repository content, including the area you wanted. That's the best way to download from GitHub, then update files on any machine.
