# Simplifying Homelab Management: Automate with Ansible

This repo serves to work with my blog post on using Ansible to automate your homelab. Instructions to use are as follows and the blog post is below

### NOTE: This was built against an ubuntu install. RHEL9 doesn't officially support docker (thanks podman) so the docker install playbook currenlty doesn't work on RHEL machines

### Pre Requisites

- You must have the target machines setup and have ssh key based authentication setup. If you don't have that completed yet you can follow along with the blog post below.

- You must have ansible installed on your local machine. You do not need Ansible installed on the target hosts.

### To Use These Playbooks
Simply clone this repository and follow the blog post below to get started

### File structure:

``````
├── ansible.cfg
├── inventory.yml
└── playbooks
    ├── docker_status.yml
    ├── docker_update_containers.yml
    ├── fresh_install.yml
    ├── install_docker.yml
    ├── install_vim.yml
    ├── os_family_discovery.yml
    ├── ping_test.yml
    └── update_upgrade.yml
``````

#### ansible.cfg
This is the ansible configuration file. It tells ansible basic information about where to find certain files and how to run
#### inventory.yml
This file contains the inventory for all of our hosts and information about those hosts. We will go into detail later.
#### playbooks
This directory contains all of our plalybooks, which we will touch on later in the blog post when we get to them.

## Introduction

Welcome back to our Homelab posts! Today, we're venturing into the realm of automation with Ansible. Ansible is an open-source automation tool that makes it easier to configure and manage computers and servers. It uses a simple, human-readable language to automate the setup and management of your systems, eliminating repetitive tasks and ensuring consistency across your environment. Whether you're managing a few machines or a large-scale network, Ansible's versatility allows for efficient scaling and control, making it an indispensable tool in any homelab setup.

In this post, we dive into Ansible to simplify and streamline your homelab operations. We'll begin by discussing the basics of setting up SSH key-based authentication and creating an Ansible inventory file. From there, we'll progress through various playbooks, covering everything from OS family discovery to Docker container management. By automating these essential tasks, your homelab will become more efficient, secure, and easier to maintain.

## Section 1: Setting Up SSH Key-Based Authentication

Before jumping into Ansible playbooks, it's crucial to establish a secure and efficient way to connect to your remote machines. SSH key-based authentication offers a more secure alternative to traditional password-based methods. I've detailed the setup process in a previous post, which you can find [here](https://medium.com/@jonezy7173_88832/using-ssh-key-based-authentication-remote-machines-and-github-f7fe6d142be8). This setup is essential for seamless Ansible operations.

## Section 2: Crafting Your Ansible Inventory File

An inventory file in Ansible is where you define the hosts and groups for your automation tasks. Let's create an inventory.yml file that categorizes your machines into groups. Fill in your host information, ip address, and user name & key file in the `inventory.yml` file. If you're unsure how to group your machines, just put them all in the same group for now and we will regroup them in the next section.

## Section 3: Discovering OS Families with Ansible

Now that you have your inventory set, let's start with the `os_family_discovery.yml`` playbook. This playbook will help you identify the operating system family of your hosts, which is crucial for tailoring further automation tasks to specific OS types.

Run this playbook against all of your hosts to get information on which family they belong to.

`ansible-playbook playbooks/os_family_discovery.yml`

Now that you know what family each host is, I recommend going back to the `inventory.yml` and grouping the hosts based on family (Debian, RedHat, etc.). If you want to furthur subdivide your hosts you can have a host in multiple groups as well.

## Section 4: The Fresh Install - Setting Up New Machines

Next up is the `fresh_install.yml` playbook. This playbook is designed to install a suite of essential utilities on new machines, whether they're running Debian/Ubuntu or RedHat/CentOS. Notably, this playbook also imports the `install_docker.yml` playbook, automating Docker installation as part of the setup process.

`ansible-playbook playbooks/fresh_install.yml --ask-become`

This will prompt you to input the BECOME password, which is the sudo password for your target machine

## Section 5: Monitoring Docker Containers

After setting up your machines, let's focus on Docker with the `docker_status.yml` playbook. This playbook checks the status of your Docker containers, ensuring they are running as expected.

If you have hosts already running docker, you will want to add them to the `inventory.yml` file in a group named `Docker`.

If you don't have any hosts running docker, pick a host that you just ran the fresh install script on and add it to the `Docker` group in you `inventory.yml`. Then execute the `first_container` playbook which will stand up your first Docker container.

`ansible-playbook playbooks/first_container.yml --ask-become`

Now add the following to your `compose_file_paths` for the docekr host in your `inventory.yml` file:

`~/firstContainer/docker-compose.yml`

You can now run `docker_status.yml` against your docker hosts to check the status of your containers. This playbook will return all green if your containers are all good, and it will fail if any container is in status "exited"

`ansible-playbook playbooks/docker_status.yml`

## Section 6: Keeping Docker Containers Up-to-Date

Lastly, we have the `docker_update_containers.yml` playbook. This playbook is crucial for updating your Docker containers with the latest images. It also re-imports the docker_status.yml playbook to check the status of containers after the update.

At this point you should have all of your hosts running docker containers in the `Docker` group of your `inventory.yml` file.

To update your containers this playbook brings your containers down, deletes their images, and brings them back up to pull the updated images. 

**YOU MUST HAVE PERSISTANT STORAGE SETUP! IF YOU DON'T THEN THIS PLAYBOOK WILL <u>DELETE ALL OF YOUR DATA FROM YOUR CONTAINERS**</u> - consider yourself warned.

`ansible-playbook playbooks/docker_update_containers.yml`

## Conclusion:

Throughout this post, we've explored a series of Ansible playbooks designed to automate various aspects of your homelab. From initial setup with SSH key-based authentication and inventory creation to managing and updating Docker containers, these playbooks are the building blocks for a highly efficient homelab environment. Remember, Ansible's flexibility allows you to expand and customize these automations to suit your specific needs. Dive in, experiment, and watch your homelab thrive with the power of automation!