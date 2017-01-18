[![slack in](https://slackin-pypmyuhqds.now.sh/badge.svg)](https://slackin-pypmyuhqds.now.sh/)

# 3Blades On-Premise Installation Instructions

## Table of Contents

- [Overview](#overview)
- [Preparation](#preparation)
- [Installation](#installation)
- [First Login](#first-login)
- [Trouble Shooting](#trouble-shooting)

## Overview

These instructions are to allow developers and customers install 3Blades on an AWS EC2 instance. A few notes about how this installation works:

- Uses ansible playbook to download, install and configure artifacts, such as docker engine.
- Docker images are pulled from Docker Hub.
- All images communicate with each other using docker links.
- All services and data stores run in separate containers.

You could do all of this with docker-compose. As a matter of fact, the ansible script does use docker-compose to launch and link services. However, we chose to use ansible since it provides more customization options.

## Preparation

1. Launch EC2 instance within your AWS account using the these requirements.

  - Ubuntu 16.04 AWS AMI (ami-a9d276c9)
  - t2.Medium (2 CPU and 4 GB RAM)
  - Security group with the following in-bound rules:
	- 22 and 80
  - SSH key
  - 30 GB of disk space

1. Install ansible on your local computer:

    `pip install ansible`

    > It is recommended to install ansible from `virtualenv` environment

    > If installing ansible with Linux (Debian or Ubuntu), using Vagrant (VirtualBox) or other virutal machine you may have to install openssl dependencies: `sudo apt-get install build-essential libssl-dev libffi-dev python-dev`

1. Clone deployment repo:

    `git clone https://github.com/3Blades/onpremise.git`

1. Change directory into project folder:

    `cd onpremise`

1. Copy / Paste SSH key into folder and add permissions to read SSH file:

    `chmod 600 SSH_KEY`

1. Test your SSH access into EC2 instance with external ip address:

    `ssh -i key-name ubuntu@[public_ip_address]`

    > Both external and internal IP addresses can be obtained from AWS console

1. Edit the inventory file called `hosts` by changing your EC2 public IP address: 
    
    `hubserver ansible_ssh_host=[public_ip_address] ansible_ssh_port=22`

    > Change the ssh port if it is different in your case

1. Edit the file `ansible.cfg` to change the `private_key_file` value and use your SSH-key file name. The file should look like this:
  
    ```
    [defaults]
    hostfile = hosts
    remote_user = ubuntu
    private_key_file = SSH_KEY.pem
    host_key_checking = False
    roles_path = roles
    ```
    
## Installation

1. From the current folder, run ansible playbook:

    ```
    ansible-playbook -s -i hosts full-stack-deployment.yml \
    --e "force_pull_images=yes smtp_server=SMTP_HOST smtp_username=SMTP_USERNAME \
    smtp_password=SMTP_PASSWORD app_secret_key=APP_SECRET_KEY SMTP_PORT=587"
    ```

Please, consider this:

- `APP_SECRET_KEY` should be a securely generated random string.
- `SMTP_HOST`, `SMTP_PORT`, `SMTP_USERNAME` and `SMTP_PASSWORD` values are used to authenticate to SMTP service. Them should be replaced with your respective SMTP configurations and are necessary to enable system notifications, including password reset emails.

  > The `force_pull_images` argument is only useful if there is a new version of our images and we want to update them. In that case we have to run
  > the ansible-plabook as before

You'll have to wait several minutes for the playbook to complete.

## First login

1. Once the playbook completes successfully, you'll be able to loign to the application. Use the following credentials

     admin@admin.com
     admin
     > Make sure you select `Remember me` option

1. You'll be asked to change your password immediately. Provide the old password and a new one, then click `Change password`.

1. You'll be logged out of the application and you'll be required to login with your new password.

     > Make sure you select `Remember me` option

The ansible script creates some environment types that are the servers that you can launch with two alternatives in memory. If you'd like your users to launch environments with more memory, you can create new items, just changing the `Memory` parameter.

## Trouble shooting

- To re run ansible playbook, log into host and run:

    sudo docker-compose -f 3blades/deployment/docker-compose.yml down

- Then, run ansible playbook as you did before with new settings.
