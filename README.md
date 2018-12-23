# Ansible-Install-Wordpress
>How to Use an Ansible Playbook To Install WordPress on Ubuntu 18.04 and 16.04

One such mundane task is installing WordPress. You might need to install WordPress for testing, development and main production website, and instead of repeating the installation three times we will write one Ansible playbook which can be run from a different location to install WordPress on your target system(s)

### Step 1 The Server(s) Step up:
To allow Ansible to connect to our WordPress server, you would need to have a non-root user with sudo privileges. To allow this, create ssh keys on your build server.

```
$ssh-keygen 4096
```
### Step 2: Writing the Ansible Playbook
The playbook, in Ansible terminology, consists of a set of hosts on which the automation is to be performed, roles that are to be played out, for example, a server acting as database, another as front-end and so on

Okay, first we need to create a directory where all of our configurations will be stored:
```
$ mkdir wordpress-playbook
$ cd wordpress-playbook
$ mkdir roles
$ touch hosts 
$ touch playbook.yml
```
1. Config Hosts You can let the first line be the same and just modify the second line with an appropriate ip address. You can put multiple IPs under [wordpress] to install on multiple servers
```
[wordpress]
wordpress_ip
```
2. Once that is done, let’s define the different roles in the roles sub-directory:
```
$ cd roles
$ ansible-galaxy init server 
$ ansible-galaxy init php 
$ ansible-galaxy init mysql
$ ansible-galaxy init wordpress
```
This brings in template configurations for individual components from ansible-galaxy which is a repository for many standard ansible configurations. You will notice there are now 4 folders created inside wordpress-playbook/roles directory named MySQL, PHP, server and WordPress. Now we need to configure these.

### Step 3: Creating Roles
Ansible instructions are written in .yml files which are quite human readable. We will be writing a few ourselves, for each individual roles of the LAMP stack and WordPress build on top of it

1. Playbook.yml 

First, we would write the following contents into the file wordpress-playbook/playbook.yml
```
- hosts: all
  gather_facts: False
  
  tasks:
  - name: install python 2
    raw: test -e /usr/bin/python || (apt -y update && apt install -y python-minimal)

- hosts: wordpress

  roles:
    - server
    - php
    - mysql
    - wordpress
```
*`Note: This installs Python 2.7 onto your all your target servers (which is an Ansible dependency) and then it goes on to assigning 4 roles to the hosts labelled WordPress`*.

2. Server 

Role Using your favourite text editor (for example nano or vim), open the file wordpress-playbook/roles/server/tasks/main.yml and write the following contents into it
```
---
# tasks file for server
- name: Update apt cache
  apt: update_cache=yes cache_valid_time=3600
  become: yes

- name: Install required software
  apt: name={{ item }} state=present
  become: yes
  with_items:
    - apache2
    - mysql-server
    - php7.2-mysql
    - php7.2
    - libapache2-mod-php7.2
    - python-mysqldb
```
*`Note: The first two lines would be there by default`*.

2. PHP

Next, we need to install additional PHP modules for that write the following contents to the file wordpress-playbook/roles/php/tasks/main.yml
```
---
# tasks file for php
- name: Install php extensions
  apt: name={{ item }} state=present
  become: yes
  with_items:
    - php-common
    - php7.0
    - php7.0-cli
    - php7.0-common
    - php7.0-fpm
    - php7.0-json
    - php7.0-opcache
    - php7.0-readline
    - libapache2-mod-php7.0
    - php7.0-mysql
    - php7.0-curl
    - php7.0-mbstring
    - php7.0-gd
    - php7.0-xml
    - php7.0-xmlrpc
    - php7.0-intl
    - php7.0-soap
    - php7.0-zip
```



4. MySQL

MySQL would need some extra information before it starts creating our database. In the file wordpress-playbook/roles/mysql/defaults/main.yml
```
---
# defaults file for mysql
wp_mysql_db: wordpress
wp_mysql_user: wordpress
wp_mysql_password: randompassword
```

1. Create user database

user access to the newly created database. In the file, wordpress-playbook/roles/mysql/tasks/main.yml write the following:

```
---
- name: Create mysql database
  mysql_db: name={{ wp_mysql_db }} state=present
  become: yes

- name: Create mysql user
  mysql_user: 
    name={{ wp_mysql_user }} 
    password={{ wp_mysql_password }} 
    priv=*.*:ALL

  become: yes  
```
5. WordPress

write the following contents into your wordpress-playbook/roles/wordpress/tasks/main.yml
```
---
- name: Download WordPress
  get_url: 
    url=https://wordpress.org/latest.tar.gz 
    dest=/tmp/wordpress.tar.gz
    validate_certs=no

- name: Extract WordPress
  unarchive: src=/tmp/wordpress.tar.gz dest=/var/www/ copy=no
  become: yes

- name: Update default Apache site
  become: yes
  lineinfile: 
    dest=/etc/apache2/sites-enabled/000-default.conf 
    regexp="(.)+DocumentRoot /var/www/html"
    line="DocumentRoot /var/www/wordpress"
  notify:
    - restart apache

- name: Copy sample config file
  command: mv /var/www/wordpress/wp-config-sample.php /var/www/wordpress/wp-config.php creates=/var/www/wordpress/wp-config.php
  become: yes

- name: Update WordPress config file
  lineinfile:
    dest=/var/www/wordpress/wp-config.php
    regexp="{{ item.regexp }}"
    line="{{ item.line }}"
  with_items:
    - {'regexp': "define\\('DB_NAME', '(.)+'\\);", 'line': "define('DB_NAME', '{{wp_mysql_db}}');"}        
    - {'regexp': "define\\('DB_USER', '(.)+'\\);", 'line': "define('DB_USER', '{{wp_mysql_user}}');"}        
    - {'regexp': "define\\('DB_PASSWORD', '(.)+'\\);", 'line': "define('DB_PASSWORD', '{{wp_mysql_password}}');"}
  become: yes
```
6. And finally, because we would need to restart Apache to see WordPress work, add the following snippet to your wordpress-playbook/roles/wordpress/handlers/main.yml
```
---
# handlers file for wordpress
- name: restart apache
  service: name=apache2 state=restarted
  become: yes
```
### Step 4: Running Ansible

Running Ansible
Hopefully, you have executed the above steps correctly, if so then running the following command, from the directory wordpress-playbook, would install WordPress into your target host
```
$ ansible-playbook playbook.yml -i hosts -u $username -kK -b -vvv
```


>### Reference & Tutorials dotlaye : 
*Search Engine Optimization and Applications
How to Encrypt & Decrypt Files with OpenSSL on Ubuntu and Mac OS X*

*SEO 101: Search Engine Optimization and Applications*


# HAPPY NEW YEAR

