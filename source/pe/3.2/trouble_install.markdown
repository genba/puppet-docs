---
layout: default
title: "PE 3.2 » Troubleshooting » Installation"
subtitle: "Troubleshooting Installer Issues"
canonical: "/pe/latest/trouble_install.html"
---

Common Installer Problems
-----

Here are some common problems that can cause an install to go awry.

### Installing Without Internet Connectivity

By default, the master node hosts a repo that contains packages used for agent installation. In order to obtain these packages, the install script will attempt to connect to the internet in order to access a Puppet Labs-maintained repo on Amazon S3. If the script cannot access the remote repo (due to a firewall issue, IT policy, etc.), the agent tarball will not be downloaded and you will see error messages in the first and subsequent puppet runs on the master. These do not mean the installation failed, only the retrieval of the tarball.

Depending on your particular deployment there are three ways you can resolve this issue. In each case, you will need to procure the agent tarball beforehand. 

* If you already have a package management/distribution system, you can use it to install agents by adding the agent packages to your repo. In this case, you can disable the PE-hosted repo feature altogether by removing the `pe-repo` class from your master.

* If you would like to use PE-provided repo, you can copy the agent tarball into `/opt/staging/pe_repo` so the master won't attempt to download it. This will prevent the error message from recurring on subsequent puppet runs.

* Lastly, if your deployment has multiple masters and you don't wish to copy the agent tarball to each one, you can specify a path to the agent tarball. This can be done with an [answer file](./install_automated.html), by setting `q_tarball_server` to an accessible server containing the tarball, or by [using the console](./console_classes_groups.html#editing-class-parameters-on-nodes) to set the `base_path` parameter of the `pe_repo` class to an accessible server containing the tarball.

### Installing on to a Master with Separated `/var` and `/opt` Directories 

In order to manage disc space or for other reasons, some PE deployments may have the `/var` and `/opt` directories on different mount points. Due to an issue in the `puppetlabs-firewall` module, this can cause serious problems during installation. See the following to find out how to prevent the issues and/or recover from a failed upgrade.
    
* **To prevent a failed installation** follow these steps:  
  1. Unpack the PE tarball.
  2. Edit `erb/puppet.conf.erb [main]` by adding: `module_working_dir = /opt/puppet/share/puppet/module_working_dir/cache`
  3. Create a directory to use as the module working directory mkdir -p /opt/puppet/share/puppet/module_working_dir/cache
  4. Run the installer in the standard manner.
        
* **To recover from a failed installation** follow these steps:
    1. Run the [uninstaller](./install_uninstalling.html).
    2. Follow the preventative steps above.

### Upgrading a Master with Separated `/var` and `/opt` Directories
    
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

### Is DNS Wrong?

If name resolution at your site isn't quite behaving right, PE's installer can go haywire.

* Puppet agent has to be able to reach the puppet master server at one of its valid DNS names. (Specifically, the name you identified as the master's hostname during the installer interview.)
* The puppet master also has to be able to reach **itself** at the puppet master hostname you chose during installation.
* If you've split the master and console roles onto different servers, they have to be able to talk to each other as well.

### Are the Security Settings Wrong?

The installer fails in a similar way when the system's firewall or security group is restricting the ports Puppet uses.

* Puppet communicates on **ports 8140, 61613, and 443.** If you are installing the puppet master and the console on the same server, it must accept inbound traffic on all three ports. If you've split the two roles, the master must accept inbound traffic on 8140 and 61613 and the console must accept inbound traffic on 8140 and 443.
* If your puppet master has multiple network interfaces, make sure it is allowing traffic via the IP address that its valid DNS names resolve to, not just via an internal interface.

### Did You Try to Install the Console Before the Puppet Master?

If you are installing the console and the puppet master on separate servers and tried to install the console first, the installer may fail.

### How Do I Recover From a Failed Install?

First, fix any configuration problem that may have caused the install to fail. See above for a list of the most common errors.

Next, run the uninstaller script. [See the uninstallation instructions in this guide](./install_uninstalling.html) for full details.

After you have run the uninstaller, you can safely run the installer again.


* * * 

- [Next: Troubleshooting Connections & Communications ](./trouble_comms.html)
