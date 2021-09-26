# Ansible
 
 |**Index**
|:---
|[Introduction](#Intro)|
|[Requitements](#Requitements)|
|[Install](#Install)|
|[AddHost](#Addhost)|
|[DelHost](#Delhost)|
|[Reference](#Reference)|

## Intro

This is a set of ansible playbooks which helps you to automate some stuff of your environment.

- Cluster server shutudown;
- Copy important files;
- Install important services
- Ensure the services are running and enabled

## Requitements

- Operating system: 

```CentOS Linux release 7.9.2009 (Core)```

- Ansible packages:

```ansible-2.9.21-1.el7.noarch```

- For Zabbix API interaction:

```pip install zabbix-api```
```ansible-galaxy collection install community.zabbix```

## Install

In order to rollout the zabbix agent installation and setup the required files you need to do the below:

** Download zabbix packages **

- Create a new directory for saving zabbix agent files

```mkdir ansible```

```cd ansible```

- Download the files to new ansible directory

```wget https://repo.zabbix.com/zabbix/3.4/rhel/7/x86_64/zabbix-agent-3.4.9-1.el7.x86_64.rpm```

```wget https://repo.zabbix.com/zabbix/3.4/rhel/7/x86_64/zabbix-get-3.4.9-1.el7.x86_64.rpm```

```wget https://repo.zabbix.com/zabbix/3.4/rhel/7/x86_64/zabbix-sender-3.4.9-1.el7.x86_64.rpm```

** Writing the Zabbix install PlayBook **

```

---
  - name: "Zabbix Copy agent Files"
    hosts: cluster
    serial: 2
    tasks:

      - name: Copy zabbix agent installation files
        ansible.builtin.copy:
          src: /etc/ansible/Playbook/zabbix/zabbix-get-3.4.9-1.el7.x86_64.rpm
          dest: /tmp/zabbix-get-3.4.9-1.el7.x86_64.rpm
          mode: '0744'

      - name: Copy zabbix agent installation files
        ansible.builtin.copy:
          src: /etc/ansible/Playbook/zabbix/zabbix-sender-3.4.9-1.el7.x86_64.rpm
          dest: /tmp/zabbix-sender-3.4.9-1.el7.x86_64.rpm
          mode: '0744'


      - name: Copy zabbix agent installation files
        ansible.builtin.copy:
          src: /etc/ansible/Playbook/zabbix/zabbix-agent-3.4.9-1.el7.x86_64.rpm
          dest: /tmp/zabbix-agent-3.4.9-1.el7.x86_64.rpm
          mode: '0744'

      - name: Install Zabbix Agent
        yum:
          name: /tmp/zabbix-agent-3.4.9-1.el7.x86_64.rpm
          state: present

      - name: Install Zabbix Sender
        yum:
          name: /tmp/zabbix-sender-3.4.9-1.el7.x86_64.rpm
          state: present

      - name: Install Zabbix Get
        yum:
          name: /tmp/zabbix-get-3.4.9-1.el7.x86_64.rpm
          state: present


      - name: Copy zabbix agent configuration file
        ansible.builtin.copy:
          src: /etc/ansible/Playbook/zabbix/zabbix_agentd.conf
          dest: /etc/zabbix/zabbix_agentd.conf
          mode: '0744'

      - name: Make sure zabbix agent is running
        ansible.builtin.systemd:
          state: started
          enabled: yes
          name: zabbix-agent

```

** Result:

![image](https://user-images.githubusercontent.com/81998668/132993091-e791a897-03a4-4eab-a797-36c00cf7ff46.png)

## Addhost

If you have successfully installed the zabbix agent on your target servers, now its the time to add them on zabbix console.
Lucky, the zabbix server interacts through an API which makes possible to you automate this step as below described:

** Adding a host to your Zabbix Server with Ansible

-- Writing the required playbook

```

---
  - name: Create host on zabbix server
    hosts: dbclus1
    vars_prompt:

    - name: hostname
      prompt: Please define a hostname
      private: no

    - name: description
      prompt: Please provide a host description
      private: no

    - name: login_user
      prompt: Enter zabbix server login
      private: no

    - name: password
      prompt: Enter zabbix server password

    - name: ip
      prompt: Please provide host to be added ip address
      private: no


    tasks:

      - name: Create a new host or update an existing host's info
        local_action:
          module: community.zabbix.zabbix_host
          server_url: http://dbclus2/zabbix
          login_user: "{{ login_user }}"
          login_password: "{{ password }}"
          host_name: "{{ hostname }}"
          visible_name: "{{ hostname }}"
          description: "{{ description }}"
          host_groups:
            - Linux servers
            - Linux servers
          link_templates:
            - Template OS Linux by Zabbix agent
          status: enabled
          state: present
          interfaces:
            - type: 1
              main: 1
              useip: 1
              ip:  "{{ ip }}"
              dns: ""
              port: "10050"

```

** Result:

![image](https://user-images.githubusercontent.com/81998668/132993215-b938f2de-0881-4615-b0fe-bc2f932ea9c6.png)

## Delhost

Ops, you wrongly add a host and now you need to rollback it and remove from the zabbix server.
No panic, you can make it easier with Ansible

** Deleting a host to your Zabbix Server with Ansible

-- Writing the required playbook

```

---
  - name: Remove host on zabbix server
    hosts: dbclus1
    vars_prompt:

    - name: hostname
      prompt: Please specify hostname to be deleted
      private: no

    - name: login_user
      prompt: Enter zabbix server login
      private: no

    - name: password
      prompt: Enter zabbix server password

    tasks:
      - name: Create a new host or update an existing host's info
        local_action:
          module: community.zabbix.zabbix_host
          server_url: http://dbclus2/zabbix
          login_user: "{{ login_user }}"
          login_password: "{{ password }}"
          host_name: "{{ hostname }}"
          status: enabled
          state: absent
```

** Result:

![image](https://user-images.githubusercontent.com/81998668/132993058-dd3b8bc4-edfc-4a72-8406-c63fad85af92.png)


## Reference

```https://docs.ansible.com/ansible/latest/collections/ansible/builtin/yum_module.html```

```https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html```

```https://repo.zabbix.com/zabbix/3.4/rhel/7/x86_64/```

```https://docs.ansible.com/ansible/latest/collections/ansible/builtin/systemd_module.html```

```https://docs.ansible.com/ansible/2.5/modules/systemd_module.html```

```https://docs.ansible.com/ansible/latest/collections/community/zabbix/zabbix_host_module.html#requirements```

```https://docs.ansible.com/ansible/latest/user_guide/playbooks_prompts.html```

```https://github.com/ansible-collections/community.zabbix#installation```

