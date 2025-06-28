Role Name
=========

A brief description of the role goes here.

Requirements
------------

Any pre-requisites that may not be covered by Ansible itself or the role should be mentioned here. For instance, if the role uses the EC2 module, it may be a good idea to mention in this section that the boto package is required.

Role Variables
--------------

A description of the settable variables for this role should go here, including any variables that are in defaults/main.yml, vars/main.yml, and any variables that can/should be set via parameters to the role. Any variables that are read from other roles and/or the global scope (ie. hostvars, group vars, etc.) should be mentioned here as well.

Dependencies
------------

A list of other roles hosted on Galaxy should go here, plus any details in regards to parameters that may need to be set for other roles, or variables that are used from other roles.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - { role: username.rolename, x: 42 }

License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
# Ansible Role: Hugo Site Deployment

This repository contains a reusable **Ansible Role** that fully automates the setup of a secure, production-ready server for deploying a [Hugo](https://gohugo.io/) static site. It implements an **Infrastructure as Code (IaC)** solution, building a complete environment with a Git-based CI/CD pipeline from scratch.

This setup powers my personal blog, which can be viewed live at [https://wuweb.westeurope.cloudapp.azure.com/](https://wuweb.westeurope.cloudapp.azure.com/). The source code for the Hugo site itself is located at [My-Blog](https://github.com/wusshit/My-Blog.git).

## Features

*   **Automated Server Setup:** Configures a fresh Ubuntu VM from scratch.
*   **Software Installation:** Installs Nginx, Git, Hugo, and Certbot.
*   **Secure Web Server:** Configures Nginx with a secure HTTPS setup using a free SSL certificate from Let's Encrypt.
*   **Git-based CI/CD:** Creates a server-side Git hook that automatically builds and deploys your Hugo site on `git push`.
*   **Reusable & Configurable:** Packaged as an Ansible Role, making it easy to reuse and customize for different projects.

## How to Use This Repository

There are two distinct workflows: the one-time server setup (using Ansible) and the ongoing content deployment (using Git).

**1. Server Provisioning (Using Ansible):**
    a. Create a new Ubuntu 20.04/22.04 VM on Azure (or any cloud provider).
    b. **From your local machine**, run the main Ansible playbook (`my-site.yml`) included in this repository.
    c. Ansible connects to the VM and uses the `hugo_deploy` role to perform all configuration steps automatically.
    d. **Result:** In minutes, the server is fully configured and ready for content deployment.

**2. Content Deployment (Using Git):**
    a. Make changes to your Hugo site's content on your local machine.
    b. Push your changes to a specific Git remote (`vps-deploy`) that points to the configured Azure VM.
    c. The server-side `post-receive` hook automatically triggers, builds the Hugo site, and publishes it to the live web directory.

### Requirements

*   Ansible installed on your local machine.
*   A fresh Ubuntu VM with SSH access using a key pair.
*   A DNS A record pointing your domain name to the VM's public IP address.

### Local Setup and Execution

1.  **Clone this repository** to your local machine:
    ```bash
    git clone https://github.com/wusshit/my-hugo-vps-deploy-setup.git
    cd my-hugo-vps-deploy-setup
    ```

2.  **Configure the Inventory:** Edit the `inventory.ini` file to point to your server.
    ```ini
    [webserver]
    your_domain_or_ip ansible_user=your_ssh_user ansible_ssh_private_key_file=~/.ssh/your_private_key
    ```

3.  **Configure the Playbook:** Open `my-site.yml`. The default variables from the role are overridden here. At a minimum, set your `domain_name` and `certbot_email`.
    ```yaml
    # my-site.yml
    ...
    vars:
      domain_name: your_domain.com
      certbot_email: your-email@example.com
    ...
    ```

4.  **Run the Playbook:** Execute the main playbook from your local machine.
    ```bash
    ansible-playbook -i inventory.ini my-site.yml
    ```
    If prompted for a sudo password, add the `-K` flag: `ansible-playbook -i inventory.ini my-site.yml -K`.

## Ansible Role Structure (`roles/hugo_deploy`)

The core logic is contained within the `hugo_deploy` role:

*   `defaults/main.yml`: Default variables for the role (e.g., paths, usernames). These are meant to be overridden.
*   `tasks/`: Contains the main task files, broken down by function (`install.yml`, `nginx.yml`, etc.) for readability. `main.yml` is the entry point that includes the others.
*   `handlers/main.yml`: Contains handlers, such as reloading Nginx only when its configuration changes.
*   `templates/`: Contains Jinja2 templates for configuration files (`my_blog_nginx.conf.j2`, `post-receive.sh.j2`).

This structure makes the automation modular, reusable, and easy to maintain.