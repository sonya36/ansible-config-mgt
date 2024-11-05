# ansible-config-mgt
testing jjjjjj



[nginx]
172.31.44.244 ansible_ssh_user=ubuntu

[tooling]
172.31.43.239 ansible_ssh_user=ec2-user
172.31.35.14 ansible_ssh_user=ec2-user

[database]
172.31.33.99 ansible_ssh_user=ubuntu



---
- hosts: all
  name: Include dynamic variables
  become: yes
  tasks:
    - include_tasks: ../dynamic-assignments/env-vars.yml
      tags:
        - always

- import_playbook: ../static-assignments/common.yml

- import_playbook: ../static-assignments/uat_webservers.yml

- import_playbook: ../static-assignments/loadbalancers.yml

- import_playbook: ../static-assignments/db-servers.yml