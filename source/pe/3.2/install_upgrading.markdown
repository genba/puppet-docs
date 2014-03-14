---
layout: default
title: "PE 3.2 » Installing » Upgrading"
subtitle: "Upgrading Puppet Enterprise"
canonical: "/pe/latest/install_upgrading.html"
---


Summary
-----

The Puppet Installer script is used to perform both installations and upgrades. The script will check for a prior version and run as upgrader or installer as needed. You start by [downloading][downloading] and unpacking a tarball with the appropriate version of the PE packages for your system. Then, when you run the `puppet-enterprise-installer` script, the script will check for a prior installation of PE and, if it detects one, will ask if you want to proceed with the upgrade. The installer will then upgrade all the PE components (master, agent, etc.) it finds on the node to version 3.2.

The process involves the following steps, which *must be performed in the following order:*

1. Provision and prepare a node for use by PuppetDB
2. Upgrade Master
3. Install PuppetDB/Database support role
4. Upgrade Console
5. Upgrade Agents

If more than one of these roles is present on a given node (for example your master and console run on the same node), the installer will upgrade each role in the correct order.

> ![windows logo](./images/windows-logo-small.jpg) To upgrade Windows agents, simply download and run the new MSI package as described in [Installing Windows Agents](./install_windows.html). However, be sure to upgrade your master, console, and database nodes first.


Important Notes and Warnings
---

- **Upgrading Split Console and Custom PostgreSQL Databases**  When upgrading from 3.1 to 3.2, the console database tables are upgraded from 32-bit integers to 64-bit. This helps to avoid ID overflows in large databases. In order to migrate the database, the upgrader will temporarily require disc space equivalent to 20% more than the largest table in the console's database (by default, located here: `/opt/puppet/var/lib/pgsqul/9.2/console`). If the database is in this default location, on the same node as the console, the upgrader can successfully determine the amount of disc space needed and provide warnings if needed. However, there are certain circumstances in which the upgrader cannot make this determination automatically. Specifically, the installer cannot determine the disc space requirement if:

    1. The console database is installed on a different node than the console.
    2. The console database is a custom instance, not the database installed byPE.

In case 1, the installer can determine how much space is needed, but it will be up to the user to determine if sufficient free-space exists.
In case 2, the installer is unable to obtain any information about the size or state of the database. The user will not to locate the large console database table 

- **Upgrading from PE 2.x or 3.0.0?** Check the [upgrade instructions for PE 3.1](../3.1/install_upgrading.html) for specific instructions, issues, and warnings.

-**Upgrading a Master with Separated `/var` and `/opt` Directories** In order to manage disc space or for other reasons, some PE deployments may have the `/var` and `/opt` directories on different mount points. Due to an issue in the `puppetlabs-firewall` module, this can cause serious problems during upgrades. See the following to find out how to prevent the issues and/or recover from a failed upgrade.
    
* **To prevent a failed upgrade** follow these steps:  
   1. Create a directory to use as the module working directory mkdir -p /opt/puppet/share/puppet/module_working_dir/cache
    2. Define the directory created above as the `module_working_dir` by adding the following to `/etc/puppetlabs/puppet/puppet.conf [main]`: `module_working_dir = /opt/puppet/share/puppet/module_working_dir/cache`
        
* **To recover from a failed upgrade** follow these steps:
    1. Create a directory to use as the module working directory mkdir -p /opt/puppet/share/puppet/module_working_dir/cache
    2. Define the directory created above as the `module_working_dir` by adding the following to `/etc/puppetlabs/puppet/puppet.conf [main]`: `module_working_dir = /opt/puppet/share/puppet/module_working_dir/cache`
    3. To prevent the installer from detecting that PE 3.2.0 is already installed, first echo the version of the existing PE version (e.g. '3.1.3') into the `pe_version` file
`echo '3.1.3' > /opt/puppet/pe_version` Then, delete the `pe_build` file: `rm /opt/puppet/pe_build`.
    4. Rerun the installer: 
        `cd /<path to puppet-enterprise-3.2.0-installer>`
        `./puppet-enterprise-installer `       


Downloading PE
-----

If you haven't done so already, you will need a Puppet Enterprise tarball appropriate for your system(s). See the [Installing PE][downloading] section of this guide for more information on accessing Puppet Enterprise tarballs, or go directly to the [download page](http://info.puppetlabs.com/download-pe.html).

[downloading]: ./install_basic.html#downloading-pe

Once downloaded, copy the appropriate tarball to each node you'll be upgrading.


Running the Upgrade
-----

Before starting the upgrade, all of the components (agents, master, console, etc.) in your current deployment should be correctly configured and communicating with each other, and live management should be up and running with all nodes connected.

> **Important:** All installer commands should be run as `root`.

> **Note:** PE3 has moved from the MySQL implementation used in PE 2.x to PostgreSQL for all database support. PE3 also now includes PuppetDB, which requires PostgreSQL. When upgrading from 2.x to 3.x, the installer will automatically pipe your existing data from MySQL to PostgreSQL.

### Prepare a Node for PostgreSQL

You will need to have a node available and ready to receive an installation of PuppetDB and PostgreSQL. This can be the same node as the one running the master and console (if you have a monolithic, all-on-one implementation), or it can be a separate node (if you are running a split role implementation). In a split role implementation, **the database node must be up and running and reachable at a known hostname before starting the upgrade process on the console node.**

The upgrader can install a pre-configured version of PostgreSQL (must be version 9.1 or higher) along with PuppetDB on the node you select. If you prefer to use a node with an existing instance of PostgreSQL, that instance needs to be manually configured with the [correct users and access](./install_basic.html#database-support-questions). This also needs to be done BEFORE starting the upgrade.

### Upgrade Master

Start the upgrade by running the `puppet-enterprise-installer` script on the master node. You can use any of the flags described in the [install instructions](http://docs.puppetlabs.com/pe/latest/install_basic.html). The script will detect any previous versions of PE roles and stop any PE services that are currently running. The script will then step through the install script, providing default answers based on the roles it has detected on the node (e.g., if the script detects only an agent on a given node, it will provide "No" as the default answer to installing the master role). The installer should be able to answer all of the questions based on your current installation except for the hostname and port of the PuppetDB node you prepped before starting the install.

As with installation, the script will also check for any missing dependent vendor packages and offer to install them automatically.

Lastly, the script will summarize the upgrade plan and ask you to go ahead and perform the upgrade. Your answers to the script will be saved as usual in `/etc/puppetlabs/installer/answers.install`.

The upgrade script will run and provide detailed information as to what it installs, what it updates and what it replaces. It will preserve existing certificates and `puppet.conf` files.

### Install PuppetDB/PostgreSQL

On the node you provisioned for PuppetDB before starting the upgrade, unpack the PE 3.2 tarball and run the `puppet-enterprise-installer` script. If you are upgrading from a 2.8 deployment, you will need to provide some answers to the upgrader, as follows:

*  `?? Install puppet master? [y/N]` Answer N. This will not be your master. The master was upgraded in the previous step.
*  `?? Puppet master hostname to connect to? [Default: puppet]` Enter the FQDN of the master node you upgraded in the previous step.
*  `?? Install PuppetDB? [y/N]` Answer Y. This is the reason we are performing this installation on this node.
*  `?? Install the cloud provisioner? [y/N]` Choose whether or not you would like to install the cloud provisioner role on this node.
*  `?? Install a PostgreSQL server locally? [Y/n]` If you want the installer to create a PostgreSQL server instance for PuppetDB data, answer 'Y'. If you are using an existing PostgresSQL instance located elsewhere, answer 'N' and be prepared to answer questions about its hostname, port, database name, database user, and password.
*  `?? Certname for this node? [Default: my_puppetdb_node.example.com ]` Enter the FQDN for this node.
*  `?? Certname for the master? [Default: hostname.entered.earlier ]` You only need to change the default if the hostname and certname of your master are different.

The installer will save auto-generated users and passwords in `/etc/puppetlabs/installer/database_info.install`. Do not delete this file, you will need its information in the next step.

#### Potential Database Transfer Issues

* The node running PostgreSQL must have access to the en_US.UTF8 locale. Otherwise, certain non-ASCII characters will not translate correctly and may cause issues and unpredictability.

* If you have manually re-ordered the columns in your old MySQL database, the transfer may fail or may import values into inappropriate columns, leading to incorrect data and unpredictable behavior.

* If some string values (e.g. for "group name") are literals written *exactly* as `NULL`, they will be transferred as undefined values or, if the target PostgreSQL column has a not-null constraint, the import may fail altogether.

### Upgrade the Console

On the node serving the console role, unpack the PE 3.2 tarball and run the `puppet-enterprise-installer` script. The installer will detect the version from which you are upgrading and answer as many installer questions as possible based on your existing deployment.

> **Note:** When upgrading a node running the console role, the upgrader will pipe the current MySQL databases into the new PostgreSQL databases. If your databases contain a lot of data, this transfer may take some time to complete.
>
> * [Pruning the MySQL data](./maintain_console-db.html#cleaning-old-reports) before starting the upgrade will make things go faster. While not absolutely necessary, to make the transfer go faster we recommend deleting all but two-four weeks worth of reports.
> * If you are running the console on a VM, you may also wish to temporarily increase the amount of RAM available.
>
> Note that your old database will NOT be deleted after the upgrade completes. After you are sure the upgrade was successful, you will need to delete the database files yourself to reclaim disk space.

The installer will also ask for the following information:

* The hostname and port number for the PuppetDB node you created in the previous step.
* Database credentials; specifically, the database names, user names, and passwords for the `console`, `console_auth`, and `pe-puppetdb` databases. These can be found in `/etc/puppetlabs/installer/database_info.install` on the PuppetDB node.

**Note:** If you will be using your own instance of PostgreSQL (as opposed to the instance PE can install) for the console and PuppetDB, it must be version 9.1 or higher.


#### Disabling/Enabling Live Management During an Upgrade

The status of live management is not managed during an upgrade of PE unless you specifically indicate a change is needed in an answer file. In other words, if your previous version of PE had live management enabled (the PE default), it will remain enabled after you upgrade unless you add or change `q_disable_live_manangement={y|n}` in your answer file. 

Depending on your answer, the `disable_live_management` setting in `/etc/puppetlabs/puppet-dashboard/settings.yml` on the puppet master will be set to either `true` or `false` after the upgrade is complete.

(Note that you can enable/disable Live Management at any time during normal operations by editing the aforementioned `settings.yml` and then running `service pe-httpd restart`.)

### Upgrade Agents and Complete the Upgrade

The simplest way to upgrade agents is to upgrade the `pe-agent` package in the repo your package manager (e.g., Satellite) is using. Similarly, if you are using the PE package repo hosted on the master, it will get upgraded when you upgrade the master. You can then [use the agent install script](./install_basic.html#installing-agents-using-pe-package-management)as usual to upgrade your agent.

For nodes running an OS that doesn't support remote package repos (e.g., RHEL 4, AIX) you'll need to use the installer script on the PE tarball as you did for the master, etc. On each node with a puppet agent, unpack the PE 3.2 tarball and run the `puppet-enterprise-installer` script. The installer will detect the version from which you are upgrading and answer as many installer questions as possible based on your existing deployment. Note that the agents on your master and console nodes will have been updated already when you upgraded those nodes. Nodes running 2.x agents will not be available for live management until they have been upgraded.

PE services should restart automatically after the upgrade. But if you want to check that everything is working correctly, you can run `puppet agent -t` on your agents to ensure that everything is behaving as it was before upgrading. Generally speaking, it's a good idea to run puppet right away after an upgrade to make sure everything is hooked and has gotten the latest configuration.

Checking For Updates
-----

[Check here][updateslink] to find out what the latest maintenance release of Puppet Enterprise is. To see the version of PE you are currently using, you can run `puppet --version` on the command line .

{% comment %} This link is the same one as the console's help -> version information link. We only have to change the one to update both. {% endcomment %}

[updateslink]: http://info.puppetlabs.com/download-pe.html

**Note: By default, the puppet master will check for updates whenever the `pe-httpd` service restarts.** As part of the check, it passes some basic, anonymous information to Puppet Labs' servers. This behavior can be disabled if need be. The details on what is collected and how to disable checking can be found in the [answer file reference](./install_answer_file_reference.html#puppet-master-answers).

* * *


- [Next: Uninstalling](./install_uninstalling.html)
