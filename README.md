
# Mail Server for Local and Test purposes on Ubuntu 24.04

Realizing Mail Server on Ubuntu 24.04 for local and test purposes from the begining with setting up the KVM to the Antivirus protection through the Postfix, Dovecot, Roundcube.


## Set up Kernel-based Virtual Machine (KVM)
Virtualization is a critical part of modern computing, enabling the creation of isolated environments for testing, development, or production workloads. **Kernel-based Virtual Machine (KVM)**, built into the Linux kernel, is one of the most efficient and widely-used virtualization solutions.

Before we can deploy and manage virtual machines effectively, we need to set up KVM on our host machine and configure it to meet our network requirements. A critical step in this process is ensuring that our virtual machines can access the internet via a **network bridge**, which allows them to share the host machine's network connection seamlessly. This setup replicates the behavior of a physical machine on the same network, making it ideal for scenarios like hosting servers or testing network configurations.
### Prerequisites
- Pre-Install Ubuntu 24.04 Instance (Server or Desktop)
- A local user with sudo rights.
- Stable Internet Connection

## Table of Contents
1. [Update the System](#update-the-system)
2. [Install KVM and Related Packages](#install-kvm-and-related-packages)
3. [Start and Enable the Libvirtd Service](#start-and-enable-the-libvirtd-service)
4. [Add Your Local User to the KVM and Libvirt Group](#add-your-local-user-to-the-kvm-and-libvirt-group)
5. [Create a Network Bridge Using Netplan](#create-a-network-bridge-using-netplan)
6. [Launch Virt-Manager and Create a Virtual Machine](#launch-virt-manager-and-create-a-virtual-machine)


### Update the System
It is highly recommended, start by updating your package list to ensure you have the latest versions. Run the following command from the terminal.

`$ sudo apt update && sudo apt upgrade -y`

### Install KVM and Related Packages
KVM and its related packages are available in the default apt repository of Ubuntu 24.04, so for its installation, run

`$ sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager -y`

**Note**: This command installs KVM, QEMU, virt-manager and other tools required to manage virtual machines.

### Start and Enable the Libvirtd Service
Start and enable the libvirtd service using following systemctl command.

`$ sudo systemctl enable --now libvirtd`

Check libvirtd service status

`$ sudo systemctl status libvirtd`

If the service is active and running, your installation was successful. Otherwise you can investigate the issue with:

`$ sudo journalctl -xeu libvirtd`


### Add Your Local User to the KVM and Libvirt Group

For non-root users to manage virtual machines, add your user to the `libvirt` and `kvm` groups:

`$ sudo usermod -aG libvirt,kvm $USER`

**Note**: Log out and log back in for these changes to take effect.

### Create Network Bridge using netplan

To access KVM virtual machines from outside your Ubuntu 24.04 system, you need to map the VM’s interface to a network bridge. While KVM creates a default virtual bridge called virbr0 for testing, it’s not suitable for external connections. To set up a proper network bridge, you should create a configuration file with extension ***.yaml** in the **/etc/netplan** directory. This configuration ensures that your VMs can communicate with other devices on the network efficiently.

```
$ cd /etc/netplan/
$ ls
01-network-manager-all.yaml
$
```
**Note**: This file name may vary according to your setup but extension will be yaml.
Modify this file and add the following content to it.

```
$ sudo nano 01-network-init.yaml
```

https://www.linuxtechi.com/how-to-install-kvm-on-ubuntu/#Prerequisites


## Set up DNS zone and configure /etc/hosts


## Install the software packages
Your server should now be booted up and ready. You can now login at the console. But I personally prefer to login from my workstation via SSH. Use the “mpvm” user (or whatever you named it) to login.

Let us install the necessary packages to make it an actual mail server. You will install:


- **mysql-server** - The MySQL server that will store information about your email accounts and domains.
- **postfix** - The MTA (mail transport agent) that speaks SMTP and receives and sends emails.
- **postfix-mysql** - An extension that allows Postfix to get its information from a MySQL database.
- **dovecot-mysql** - The IMAP/POP3 mail server including an extension to get its information from a MySQL database.
- **dovecot-pop3d** - An extension to Dovecot that allows users to fetch emails using the POP3 protocol.
- **dovecot-imapd** - An extension to Dovecot that allows users to access emails using the IMAP protocol.
- **dovecot-lmtpd** - Enabled Dovecot to receive LMTP connections. We will need it later to allow email communication between Postfix and Dovecot.
- **dovecot-managesieved** - An extension to Dovecot that allows users to define filter rules that are automatically run on the server when a new email arrives.
- **phpmyadmin** - A PHP-based web interface to manage your MySQL database.
- **spamassassin** - A software to score emails. Helps you determine how likely an email is spam.
- **spamass-milter** - An extension that allows communication between SpamAssassin and Postfix using the milter protocol.
- **pwgen** - A tool to create passwords. (Optional.)
- **roundcube** - A PHP-based webmail software. (Optional.)
- **roundcube-plugins** - Extra plugins that add functionality to Roundcube. (Optional.)
- **swaks** - The Swiss Army Knife of Smtp. A utility to send emails through SMTP for testing purposes.
- **mutt** - A console-based program that can speak IMAP and also read Maildirs directly. Very helpful for testing the functionality of your mail server.

```
$ su -
$ apt-get update
$ apt-get upgrade
```
We will use the following utility which creates a random string with the mentioned length (e.g. 20).:

```
$ apt-get install pwgen
$ pwgen -s 20 1

```
Copy the output, because we will use it to create and set the MySQL **root** user. 

**Optional** You can safe the password for example in */root/.my.cnf* like: 

```
[client]
password=...
```

To make the file only readable for the system’s “root” user run:
```
chmod u=rw,go= /root/.my.cnf
```

Next install the MySQL server package:

`$ apt-get install mysql-server`

If all went well you can now run “mysql” and get a connection to your MySQL database.

Exit the MySQL shell with “exit” or just press CTRL-D.




