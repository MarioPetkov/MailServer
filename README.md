
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

**Let's understand these terms**:

* **MUA**: A mail user agent is an interface that enables a user interaction to view and send emails.
* **MTA**: A mail transfer agent transfers email from the sender to the receiver.
* **SMTP**: A Simple Mail Transfer Protocol is a protocol that the MUA uses to send emails to the MTA.
* **MDA**: All emails sent from the MTA get received and stored at the mail delivery agent.
* **IMAP**: Internet Message Access Protocol is a protocol that MDAs use to deliver mail to the MUA.

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

### MySQL server

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

![Screenshot showing successfully login into MySQL and exit.](https://github.com/MarioPetkov/MailServer/blob/main/images/mysql-connected.png)

### Postfix

Now on to the Postfix packages:


`$ apt-get install postfix postfix-mysql spamassassin spamass-milter swaks`

**Note** - When you get asked for the mail server configuration type please choose “Internet site”. Enter your own mail server name (the fully qualified domain name) or just press enter. The host name and domain does not need to match any of your email domains.

In my case here I set up for *FQDN* = **mail.mariopetkov.com** 

### Dovecot
In addition to Postfix (that handles SMTP communication) you will need Dovecot to store received emails and allow IMAP (and optionally POP3) access for your users:

`$ apt-get install dovecot-mysql dovecot-pop3d dovecot-imapd dovecot-managesieved dovecot-lmtpd`

### Roundcube
`$ apt-get install roundcube roundcube-plugins roundcube-plugins-extra php-net-sieve`

Roundcube also creates a small MySQL database to store its management information. So you will be asked whether that database should be configured automatically for you:


## SSL encryption key and certificate
I consider any password immediately wasted that went through the internet unencryptedly. Let's create an encryption key and a certificate for Postfix (SMTP), Dovecot (IMAP/POP3) and Roundcube (Webmail/HTTPS).

**Note**: Do not use different certificates for Postfix and Dovecot!!!

### Cryptographical security
The concept of one-way functions in mathematics allows data to be easily decrypted with a key while making it nearly impossible to crack the encryption without it. For years, the SHA1 algorithm was widely used, but Edward Snowden's revelations have led to the assumption that it is too easily compromised. As a result, using at least SHA256 (part of the SHA2 family) is now strongly recommended. Similarly, RSA signatures should be upgraded from 1024 bits to 4096 bits for enhanced security. Fortunately, generating secure keys and certificates is straightforward, thanks to the availability of specialized tools.

There is self-signed CA (Certificate Authority), No-cost certificate from LetsEncrypt or Paid certificate.
I will proceed with self-signed certificate, which is free and for test and local purposes is perfect. However if you plan to deploy the mail server I recommend you to get paid certificate or at least get LetsEncrypt certificate which is valid for 90 days.

```
openssl req -x509 -nodes -days 365 -newkey rsa:4096 -keyout /etc/ssl/private/mailserver.key -out /etc/ssl/certs/mailserver.pem -subj "/C=BG/ST=Plovdiv/L=Plovdiv/O=MailServer/OU=IT/CN=mail.mariopetkov.com" -addext "basicConstraints=critical,CA:FALSE" -addext "subjectAltName=DNS:mail.mariopetkov.com,DNS:mariopetkov.com,IP:10.55.10.10"
```

**Note**: Whichever method you use – always create the key on your own server. Never trust a key that has been created by anyone else. It appears simpler because you can omit one step. But the other party now knows your secret key and could in theory intercept your encrypted traffic.


## Set up Apache web server
Now that you have a valid key and certificate you are ready to set up your web server. You will need it for the Roundcube webmail interface.

The Apache web server installation on Debian stores all virtual host configurations in /etc/apache2/sites-available/.

```
$ cd /etc/apache2/sites-available/
$ cat roundcube.conf
<VirtualHost *:443>
    ServerName mail.mariopetkov.com

	ServerAdmin admin@mariopetkov.com
	DocumentRoot /var/www/roundcube

    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/mailserver.pem
    SSLCertificateKeyFile /etc/ssl/private/mailserver.key

    Include /etc/roundcube/apache.conf
	Alias / /var/lib/roundcube/
    
    ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined

    <Directory /var/www/roundcube>
	Options -Indexes +FollowSymLinks
	AllowOverride All
	Require all granted
	</Directory>

	<FilesMatch "\.php$">
	SetHandler "proxy:unix:/run/php/php8.3-fpm.sock|fcgi://localhost"
	</FilesMatch>
</VirtualHost>
```
Redirect all HTTP traffic to HTTPS in roundcube-http.conf

```
$ cat roundcube-http.conf
<VirtualHost *:80>
    ServerName mail.mariopetkov.com
    ServerAdmin admin@mariopetkov.com

    RewriteEngine On
    RewriteCond %{HTTPS} off
    RewriteRule ^(.*)$ https://%{HTTP_HOST}$1 [L,R=301]

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
Execute the following commands to enable the ssl encryption, enable the virtual host for HTTPS and reload apache2.
```
a2enmod ssl
a2ensite roundcube
service apache2 reload

```

Now you should have working web apache server, if encounter any issues you can invsetigate with `systemctl apache2 status` or `journalctl -xeu apache2`
## Set up the MySQL database
A table in database terms is pretty much the same as a spreadsheet. You have rows and columns. And columns are also called fields. SQL statements can be entered in the “mysql” shell that you get when you run the “mysql” command as root on your server.

Let's create the database and user. Give the user permissions to access the db. You can use for password the **pwgen** utility again.

`CREATE DATABASE roundcubemail;`

`GRANT SELECT,INSERT,UPDATE,DELETE ON roundcubemail.* TO 'mailuser'@'127.0.0.1' IDENTIFIED BY 'GeneratedPassword123';`

`USE roundcubemail;`

Instead of "GeneratedPassword123" use your own password of course.

Let's project the table scheme.

**virtual_domains**
This table just holds the list of domains that you will use as virtual_mailbox_domains in Postfix.
```
CREATE TABLE `virtual_domains` (
 `id` int(11) NOT NULL auto_increment,
 `name` varchar(50) NOT NULL,
 PRIMARY KEY (`id`)
 ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
 ```

**virtual_users**
The next table contains information about your users. Every mail account takes up one row.
```
CREATE TABLE IF NOT EXISTS `virtual_users` (
 `id` int(11) NOT NULL auto_increment,
 `domain_id` int(11) NOT NULL,
 `email` varchar(100) NOT NULL,
 `password` varchar(150) NOT NULL,
 PRIMARY KEY (`id`),
 UNIQUE KEY `email` (`email`),
 FOREIGN KEY (domain_id) REFERENCES virtual_domains(id) ON DELETE CASCADE
 )  DEFAULT CHARSET=utf8;
 ```

**virtual_aliases**
The last table contains forwardings from an email address to other email addresses.
```
CREATE TABLE IF NOT EXISTS `virtual_aliases` (
 `id` int(11) NOT NULL auto_increment,
 `domain_id` int(11) NOT NULL,
 `source` varchar(100) NOT NULL,
 `destination` varchar(100) NOT NULL,
 PRIMARY KEY (`id`),
 FOREIGN KEY (domain_id) REFERENCES virtual_domains(id) ON DELETE CASCADE
 ) DEFAULT CHARSET=utf8;
 ```

### Populate the database

```
INSERT INTO `roundcubemail`.`virtual_domains` ( `id` , `name` ) VALUES ( '1', 'mariopetkov.com' );
INSERT INTO `roundcubemail`.`virtual_users` ( `id` , `domain_id` , `password` , `email` ) VALUES ('1', '1', '{SHA256-CRYPT}$5$RnBlJ1Gk3vCW2ey/$9DPQ97bbMXoGTqPkIf3d.GlPopEFOoyrRrL8V..s3F3' , 'testuser1@mariopetkov.com');
INSERT INTO `roundcubemail`.`virtual_aliases` (`id`,`domain_id`,`source`,`destination`) VALUES ('1', '1', 'testuser2@mariopetkov.com', 'testuser1@mariopetkov.com');
```
I ran “dovecot pw -s SHA256-CRYPT” to create a secure hash of the simple password “summer”.


## Configure Postfix
By default, Postfix configuration files are in /etc/postfix. The two most important files are main.cf and master.cf; these files must be owned by root. Giving someone else write permission to main.cf or master.cf (or to their parent directories) means giving root privileges to that person.

In /etc/postfix/main.cf you will have to set up a minimal number of configuration parameters. Postfix configuration parameters resemble shell variables, with two important differences: the first one is that Postfix does not know about quotes like the UNIX shell does.

**Note**: Good practice is to back up the original files as I did main.cf > main.cf.bk
```
$ cd /etc/postfix/
$ ls
dynamicmaps.cf    main.cf     main.cf.proto  master.cf        mysql-virtual-alias-maps.cf       mysql-virtual-mailbox-maps.cf  postfix-files.d  post-install
dynamicmaps.cf.d  main.cf.bk  makedefs.out   master.cf.proto  mysql-virtual-mailbox-domains.cf  postfix-files                  postfix-script   sasl

```
You need to set up the following mysql-virtual-*.cf, like the following and will make Postfix and MySQL database would communicate each other.

**Important**: Use the credentials for the database login

```
root@mail:/etc/postfix# cat mysql-virtual-mailbox-maps.cf 
user = roundcube
password = GeneratedPassword123
hosts = 127.0.0.1
dbname = roundcubemail
query = SELECT 1 FROM virtual_users WHERE email='%s'
```

```
root@mail:/etc/postfix# cat mysql-virtual-mailbox-domains.cf 
user = roundcube
password = GeneratedPassword123
hosts = 127.0.0.1
dbname = roundcubemail
query = SELECT 1 FROM virtual_domains WHERE name='%s'
```

```
root@mail:/etc/postfix# cat mysql-virtual-alias-maps.cf 
user = roundcube
password = GeneratedPassword123
hosts = 127.0.0.1
dbname = roundcubemail
query = SELECT destination FROM virtual_aliases WHERE source='%s'
```
Use the postmap command to change /etc/postfix/mysql-virtual-* to a format recognizable by Postfix. Run this command every time you edit the file, for instance, after adding more domains to the file.

Edit the master.cf configuration file to enable the SMTP service.

`$ sudo nano master.cf`

Find the entry below and remove the pound symbol at the beginning of the line.
```
...
#submission inet n       -       y       -       -       smtpd
...
```
Enter the information from below in the **main.cf**. Ensure you have properly entered **smtpd_tls_cert_file** and **smtpd_tls_key_file** with you certificate.
I have also configured also myhostname, mydomain, mynetworks, relayhost.

```
$ cat main.cf
# TLS parameters
smtp_use_tls = yes
smtp_tls_security_level = may
smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache

smtpd_use_tls = yes
smtpd_tls_security_level = may
smtpd_tls_session_cache_database = btree:${data_directory}/smtpd_scache
smtpd_tls_cert_file = /etc/ssl/certs/mailserver.pem
smtpd_tls_key_file = /etc/ssl/private/mailserver.key
smtpd_relay_restrictions = permit_mynetworks, permit_sasl_authenticated,  reject_unauth_destination

smtpd_sasl_auth_enable = yes
smtpd_sasl_type = dovecot
smtpd_sasl_path = private/auth

virtual_transport = lmtp:unix:private/dovecot-lmtp
virtual_mailbox_domains = mysql:/etc/postfix/mysql-virtual-mailbox-domains.cf
virtual_mailbox_maps = mysql:/etc/postfix/mysql-virtual-mailbox-maps.cf
virtual_alias_maps = mysql:/etc/postfix/mysql-virtual-alias-maps.cf

myhostname = mail.mariopetkov.com
mydomain = mariopetkov.com
myorigin = /etc/mailname
mydestination =  localhost.$mydomain, localhost, $myhostname
relayhost = mail.viscomp.bg:587
mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128 192.168.99.0/24 10.55.10.0/24
relay_domains = $mydestination
mailbox_size_limit = 0
recipient_delimiter = +
inet_interfaces = all
inet_protocols = all
alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases
```
## Making Postfix send emails to Dovecot

In a previous chapter we made sure that Postfix knows which emails it is allowed to receive. Now what to do with the email? It has to be stored to disk for your users. You could let Postfix handle that using its built-in mail delivery agent (MDA) called “virtual”. However compared to the capabilities that Dovecot provides like sieve rules or quotas the Postfix delivery agent is pretty basic. We are using Dovecot anyway to provide the IMAP (and optionally POP3) service. So let’s use it.

We will use LMTP (local mail transport protocol). It can handle multiple recipients at the same time and has a permanently running process which provides a better performance than using the LDA. In short LMTP is a variant of SMTP with fewer features that is meant for email communication in a trusted environment.

```
$ cd /etc/dovecot/conf.d/
$ ls
10-auth.conf      10-logging.conf  10-master.conf  10-tcpwrapper.conf  15-mailboxes.conf  20-lmtp.conf         90-acl.conf     90-quota.conf  90-sieve-extprograms.conf    auth-deny.conf.ext  auth-master.conf.ext      auth-sql.conf.ext     auth-system.conf.ext
10-director.conf  10-mail.conf     10-ssl.conf     15-lda.conf         20-imap.conf       20-managesieve.conf  90-plugin.conf  90-sieve.conf  auth-checkpassword.conf.ext  auth-dict.conf.ext  auth-passwdfile.conf.ext  auth-static.conf.ext
```

Deal with the LMTP daemon and create UNIX socket:
```
service lmtp {
  unix_listener /var/spool/postfix/private/dovecot-lmtp {
    group = postfix
    mode = 0600
    user = postfix
  }
}
```
`service dovecot restart`
Execute the following command to add it into the main.cf.

`postconf virtual_transport=lmtp:unix:private/dovecot-lmtp`

Edit the file /etc/dovecot/conf.d/20-lmtp.conf to tell lmtp service we will use Dovecot's plug, should be:
`$ mail_plugins = $mail_plugins sieve`
`$ service dovecot restart`

## Configure Dovecot
Dovecot service is used for:
- gets emails from Postfix and saves them to disk
- executes user-based “sieve” filter rules (can be used to e.g. move emails to different folders based on certain criteria or send automated vacation responses)
- allows the user to fetch emails using POP3 or IMAP

We will create new user for security reason that will own all virtual mailboxes with proper permissions.
```
groupadd -g 5000 vmail
useradd -g vmail -u 5000 vmail -d /var/vmail -m
```

Mostly we configure Dovecot in /etc/dovecot/conf.d/, so to include these configurations, we have to ensure that dovecot.conf includes

`!include conf.d/*.conf`

#### 10-auth.conf
```
auth_mechanisms = plain login
disable_plaintext_auth = yes
!include conf.d/*.conf

```

#### auth-sql.conf.executes
```
userdb {
  driver = static
  args = uid=vmail gid=vmail home=/var/vmail/%d/%n
}
```

#### 10-mail.conf
`mail_location = maildir:/var/vmail/%d/%n/Maildir`
Look for the “namespace inbox” section. `separator = .`

#### 10-master.conf
```
# Postfix smtp-auth
unix_listener /var/spool/postfix/private/auth {
  mode = 0660
  user = postfix
  group = postfix
}

inet_listener imaps {
   port = 993
   ssl = yes
}

 inet_listener pop3s {
   port = 995
   ssl = yes
}
````
#### 10-ssl.conf
```
ssl_cert = </etc/ssl/certs/mailserver.pem
ssl_key = </etc/ssl/private/mailserver.key
ssl = yes
```
#### 15-mailboxes.conf
```
mailbox INBOX.Junk {
  auto = subscribe
  special_use = \Junk
}
mailbox INBOX.Trash {
  auto = subscribe
  special_use = \Trash
}
```

#### /etc/dovecot/dovecot-sql.conf.ext
```
driver = mysql
connect = 'host=127.0.0.1 dbname=roundcubemail user=roundcube password=AdminPassword123'
default_pass_scheme = SHA256-CRYPT
password_query = SELECT email as user, password FROM virtual_users WHERE email='%u';
```
#### Last step
You should also make sure that only root can access the SQL configuration file so nobody else is reading your database access passwords:
```
chown root:root /etc/dovecot/dovecot-sql.conf.ext
chmod go= /etc/dovecot/dovecot-sql.conf.ext
```
`service dovecot restart`

Look at your /var/log/mail.log logfile. You should see:

`... dovecot: master: Dovecot v2.2.13 starting up for imap, sieve, pop3 (core dumps disabled)`
## Configure Roundcube

To access Postfix and Dovecot servers, install Roundcube email client.

Ensure you have the following rows in the file /etc/apache2/sites-available/roundcube.conf between <VirtualHost> tags.
```
    Include /etc/roundcube/apache.conf
	Alias / /var/lib/roundcube/
```
and after that reload the apache2 and check for any errors.

```
service apache2 reload
apache2ctl configtest
cat /var/log/apache2/error.log
```
Now it should be able to see the webmail interface on https://mail.mariopetkov.com/roundcube

**Note**: The “Server” is always “localhost”. So edit the /etc/roundcube/config.inc.php file and set:

`$config['default_host'] = 'localhost';`

Also to add plugins list them in the same configuration file as:
```
$config['plugins'] = array(	 	 
 'archive',	 	 
 'zipdownload',	 	 
 'managesieve',	 	 
 'password',	 	 
);
```
#### Configure the managesieve plugin
A default configuration can be found at /usr/share/roundcube/plugins/managesieve/config.inc.php.dist on your system. Copy it to the location where Roundcube will look for it:

`cp /usr/share/roundcube/plugins/managesieve/config.inc.php.dist /etc/roundcube/plugins/managesieve/config.inc.php`

No further changes are required.

#### Configure the password plugin
To let the users change their passwords we use this plugin and let's copy the default configuration file to the right place:

`cp /usr/share/roundcube/plugins/password/config.inc.php.dist /etc/roundcube/plugins/password/config.inc.php`

I have updated only the following settings:

```
$config['password_db_dsn'] = 'mysql://roundcube:GeneratedPassword123@127.0.0.1/roundcubemail';
$config['password_query'] = ' UPDATE virtual_users SET password = ENCRYPT(%p, CONCAT(\'$6$\', SUBSTRING(SHA256(RAND()), -16))) WHERE email = %u'
```
Let's try to login into our account. 

If you are not able to login check your /var/log/mail.log and /var/log/roundcube/errors files for errors.



## To be continued...
Until now you should be able to send and receive mails locally via the Roundcube webmail.
