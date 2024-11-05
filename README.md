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
  become: true
  vars:
    # Define the absolute path to the env-vars directory
    env_vars_dir: "/var/lib/jenkins/workspace/ansible-config-mgt_main/env-vars"

  tasks:
    - include_tasks: ../dynamic-assignments/env-vars.yml
      tags:
        - always

- import_playbook: ../static-assignments/uat-webservers.yml

- import_playbook: ../static-assignments/loadbalancers.yml

- import_playbook: ../static-assignments/db-servers.yml
 




 # This file is for including dynamic environment variables

- name: Check if environment-specific files exist
  stat:
    path: "{{ env_vars_dir }}/{{ item }}"
  register: env_file
  loop:
    - dev.yml
    - stage.yml
    - pod.yml  # Replaced 'prod.yml' with 'pod.yml' as per your directory
    - uat.yml

- name: Set existing_files with existing variable files
  set_fact:
    existing_files: "{{ env_file.results | selectattr('stat.exists', 'eq', true) | map(attribute='item') | list }}"

- name: Debug existing_files
  debug:
    msg: "Existing files: {{ existing_files }}"

- name: Collate variables from existing environment-specific files
  include_vars:
    file: "{{ env_vars_dir }}/{{ item }}"
  loop: "{{ existing_files }}"
  when: existing_files | length > 0
  tags:
    - always

- name: Looping through included variable files
  debug:
    msg: "Processing file: {{ env_vars_dir }}/{{ item }}"
  loop: "{{ existing_files }}"

- name: Display completion message for each host
  debug:
    msg: "Completed processing for host: {{ inventory_hostname }} with included files: {{ existing_files }}"