
# Product Database

*Version 0.4 (development) - see [Changelog](CHANGELOG.md) for details*

**This is the Product Database development branch. I recommend to use the latest [stable version 0.3](https://github.com/hoelsner/product-database/tree/v0.3) 
for production.**

**Before updating to Version 0.4 (or this development version), please save your current configuration values. Version 0.4 
uses the database to store the configuration values, not the configuration file. After updating, you must reconfigure 
the Product Database settings.**

This version is currently in development and contains multiple major changes to the underlying operating system and the
application . **Please note that an update path from Version 0.2 is not maintained.**

This web service provides a central point of management for product information, which targets primarily
network products. It currently focuses on the following use cases:

* automatic check of lifecycle states for a list of products against the local database (bulk EoL check)
* REST API access to a central database which holds information about products from network vendors/manufacturers
* import data using Excel

This web service is based on python 3.5.1 and Django 1.9. A detailed list with all dependencies is available in the
`requirements.txt` file.

## License

See the [license](LICENSE.md) file for license rights and limitations (MIT).

## Server requirements

* recommend 4 vCPU's
* min. 2 GB RAM

## Product Database Installation

The entire setup of the Product Database is based on an Ansible playbook. This playbook contains all parameters and setup
to setup a new instance of the web-service. There are multiple ways to deploy a new instance of the Product Database:

* Dedicated Server setup (without an Ansible control machine)
* Dedicated Server setup (using an Ansible control machine)
* Vagrant VM

Two users are created during the installation process: one superuser with the username **pdb_admin** and the
password **pdb_admin** and one **api** user with the password **api** as a normal user. Any read action on the web-site
is permitted without authentication by default. To change this behavior, you can enable the "login-only mode". The
REST API is only available to registered users.

### Dedicated Server setup

To setup the **Product Database** on a dedicated server, you need a clean installation of Ubuntu 16.04. Please note, that
this web service must be the only application running on the server. Within the `deploy` directory, you find an
installation profile with some predefined default values named `stage_default`. The entire setup is based on an Ansible
playbook.

### Setup without an Ansible control machine

The following steps are required if you have only a single Ubuntu 16.04 server that should run the Product Database.
First, you need to install the following dependencies:

```
sudo apt-get install python python-dev python3 python-pip python3-pip git openssh-server build-essential libssl-dev libffi-dev sshpass
sudo -H pip install ansible
sudo -H pip3 install invoke
```

After a successful installation of these packages, clone the code from GitHub and navigate to the `stage_default` directory:

```
git clone https://github.com/hoelsner/product-database.git
cd product-database/deploy/stage_default
```

Now you can start the installation on the local host using the following command:

```
invoke deploy_local -u <username> -p <password>
```

Please note that the user must be permitted to use SSH (to get Ansible working). Furthermore, this user requires sudo
permissions and is used to run the application as a service.

#### Setup using an Ansible control node

Before you start the setup process, please make sure that the following requirements are satisfied on your Ubuntu Server:

  * python Version 2 is installed (required to run Ansible)
  * the deployment user is also used to run the required services
  * the deployment user requires sudo permissions

You need an Ansible control machine to deploy your dedicated server (Linux/Mac OS X only, see
[the Ansible installation guide for details](http://docs.ansible.com/ansible/intro_installation.html#installing-the-control-machine)).
On the Ansible control machine, you need to perform the following steps:

 1. install python2 ,ansible and invoke
 2. clone the code repository from GitHub
 3. run the invoke deploy task (just a task runner to simplify the execution)

```
sudo -H pip3 install invoke
git clone https://github.com/hoelsner/product-database.git
cd deploy/stage_default
invoke deploy <ip/hostname> -u <username> -p <password>
```

The `password` is only required if you're using SSH password-based authentication.

### Vagrant VM

This repository contains a Vagrant file that describes a Product Database VM for testing, demo and development purpose.
To create a VM using Vagrant, you simply need to clone (or copy) the repository to your laptop.
Before continue, you need to install [Vagrant](https://www.vagrantup.com/) and a an virtualization software like VirtualBox.
You need to run the following command within the local code repository:

    $ vagrant up productdb

After a successful provisioning process, the Product Database runs inside the VM and is available on
**http://localhost:16000**.

# Cisco EoX APIs within the Product Database

This version is capable to synchronize the local database with the Cisco EoX API. More information about the API is 
available at [http://apiconsole.cisco.com](http://apiconsole.cisco.com) (Cisco Partner access only). Please note the 
Terms & Conditions of this Service
([http://www.cisco.com/web/siteassets/legal/terms_condition.html](http://www.cisco.com/web/siteassets/legal/terms_condition.html)).

To use the Product Database with the Cisco API console, you need to enter your client credentials on the settings page. These 
credentials can be created in the Cisco API administration. 

**Currently, the access permission to the Cisco EoX V5 API is required**
 
If you have questions about the API access, please take a look on the following entry in the 
[Cisco Support Forums](https://supportforums.cisco.com/community/5456/partner-support-service) (Cisco CCO account required).

# Development Notes

You need to install the requirement files `requirements.txt` and the `requirements_dev.txt` for development. I recommend 
to create a new **virtualenv** for these dependencies. The test cases from this projects uses py.test and selenium. 

Before you're able to execute the test cases, you need access to the [Cisco Support APIs](http://apiconsole.cisco.com). 
After you created your account, you need to create your client credentials and save them in a file named 
`.cisco_api_credentials` in the root directory with the following format:

```
{
    "client_id": "yourclientid",
    "client_secret": "yourclientsecret"
}
```

Every test case requires a local postgres database connection using the following parameters:

* the database name `productdb_dev`
* username `productdb`
* running at `localhost` port `5432`

There are two additional options available when executing the test cases:

* the parameter `--online` will add test cases that use the online Cisco API (otherwise the test cases are mocked)
* the parameter `--selenium` will add the selenium test cases (Firefox required)
