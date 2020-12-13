# IAC - Infrastructure As Code

- This is the process of managing and provisioning computers through machine-readable files rather through physical hardware configuration (ie manually doing as you've done before)

<br>

## Overview
- There are two methods of IAC:
    1. Push
        - The controlling server pushes the configuration to the destination systems
    2. Pull
        - The server to be configured will pull its configuration from the controlling server

<br>

## Configuration Management

**Tools**
- Broadly speaking, any framework or tool that performs changes or configures infrastructure programatically can be considered IAC

- Configuration tools commonly used are:
    - Push-based
        1. Chef
        2. Puppet
    
    - Pull-based
        1. Terraform
        2. Ansible

## Orchestration
- Once we have an AMI, we use orchestration tools to deploy these into more complex environments. 
    - Focused on networking and architecture rather than individualistic configurations

- Tools are:
    - Terraform
    - Ansible
    - Cloud Formation