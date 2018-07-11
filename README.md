# Linux-Server-Configuration
Project #5 for the Udacity Full Stack Web Developer Nanodegree

### About

For the final	 Udacity nanodegree project, I configured a linux Amazon lightsail instance and deployed my catalog app from [project #3](https://github.com/spiceybrisket/catalog-project). The application can be found [here](http://13.211.150.108.xip.io/).

### Configuration details

Here are the steps I took to secure the server and get my application running.

#####Upgrading software

After logging in to the server, the first thing I did was update all installed software packages:
```
$ apt-get update
$ apt-get upgrade
```

#####Creating new users and configuring ssh

My next order of business was to create the appropriate users, give them sudo privileges and allow them to ssh into the server. First, I added the new users:

```
$ adduser {newuser}
```

Then for each new user I created a file in the `sudoers.d` directory: `$ sudo nano /etc/sudoers.d/{newuser}` with the following text:
```
{newuser} ALL=(ALL) NOPASSWD:ALL
```
```
$ mkdir /home/{newuser}/.ssh
$ touch /home/{newuser}/.shh/authorized_keys
$ chown {newuser} /home/{newuser}/.ssh
$ chown {newuser} /home/{newuser}/.ssh/authorized_keys
$ chmod 700 /home/{newuser}/.ssh
$ chmod 600 /home/{newuser}/.ssh/authorized_keys
```

Then I added public keys into each `authorized_keys` file. Finally, I opened the `/etc/ssh/sshd_conig` file and set the following configuration settings:

```
Port 2200
...
PermitRootLogin no
...
PasswordAuthentication no
```

I then restarted the SSH service:

```
$ sudo service sshd restart
```

After this I exited the root login session and logged back into the server with my new user account and public key.

#####Setting the time zone

To set the time zone I ran the command:

```
$ sudo dpkg-reconfigure tzdata
```

This opened a dialog. From the menu I selected 'None of the above' and then 'UTC.'

#####Setting up a firewall

Next I configured a firewall to only allow incoming connections on ports 80, 123 and 2200:
```bash
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
$ sudo ufw allow 2200/tcp
$ sudo ufw allow www
$ sudo ufw allow ntp
$ sudo ufw enable
```

##### Installing and configuring fail2ban

To protect against brute force attacks I installed fail2ban, a software package that blocks ip addresses with multiple failed login attempts within a certain amount of time. I installed fail2ban from apt-get:

```
$ sudo apt-get install fail2ban
```

Configuring fail2ban was simple, as the default configuration settings were mostly adequate. All I had to do was copy the default configuration file to new file to prevent updates from overwriting my configuration settings:

```
$ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

Then, within the new `jail.local` file I changed the ssh port from 22 to 2200, saved the file and restarted fail2ban:
```
$ sudo service fail2ban restart
```


## Install and configure PostgreSQL
1. Install PostgreSQL `sudo apt-get install postgresql`
2. Check if no remote connections are allowed `sudo nano /etc/postgresql/9.5/main/pg_hba.conf`
3. Login as user "postgres" `sudo su - postgres`
4. Get into postgreSQL shell `psql`
5. Create a new database named catalog  and create a new user named catalog in postgreSQL shell

	```
	postgres=# CREATE DATABASE catalog;
	postgres=# CREATE USER catalog;
	```
5. Set a password for user catalog

	```
	postgres=# ALTER ROLE catalog WITH PASSWORD 'catalog';
	```
6. Give user "catalog" permission to "catalog" application database

	```
	postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
	```
7. Quit postgreSQL `postgres=# \q`
8. Exit from user "postgres"

	```
	exit
	```

## Install git, clone and setup your Catalog App project.
1. Install Git using `sudo apt-get install git`
2. Use `cd /var/www` to move to the /var/www directory
3. Create the application directory `sudo mkdir catalog`
4. Move inside this directory using `cd catalog`
5. Clone the Catalog App to the virtual machine `git clone https://github.com/spiceybrisket/linuxInstall.git`
6. Rename the project's name `sudo mv ./catalog-project ./catalog`
7. Move to the inner catalog directory using `cd catalog`
8. Edit `database_setup.py`, `website.py` and `functions_helper.py` and change `engine = create_engine('sqlite:///toyshop.db')` to `engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`
9. Install pip `sudo apt-get install python-pip`
10. Use pip to install dependencies `sudo pip install -r requirements.txt`
11. Install psycopg2 `sudo apt-get -qqy install postgresql python-psycopg2`
12. Create database schema `sudo python database_setup.py`

## Configure and Enable a New Virtual Host
1. Create catalog.conf to edit: `sudo nano /etc/apache2/sites-available/catalog.conf`
2. Add the following lines of code to the file to configure the virtual host.

	```
	<VirtualHost *:80>
		ServerName 52.24.125.52
		ServerAdmin qiaowei8993@gmail.com
		WSGIScriptAlias / /var/www/catalog/catalog.wsgi
		<Directory /var/www/catalog/catalog/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/catalog/catalog/static
		<Directory /var/www/catalog/catalog/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
	</VirtualHost>
	```
3. Enable the virtual host with the following command: `sudo a2ensite catalog`

## Create the .wsgi File
1. Create the .wsgi File under /var/www/catalog:

	```
	cd /var/www/catalog
	sudo nano catalog.wsgi
	```
2. Add the following lines of code to the catalog.wsgi file:

	```
	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/catalog/")

	from catalog import app as application
	application.secret_key = 'Add your secret key'
	```

## Restart Apache
1. Restart Apache `sudo service apache2 restart`

In order to view mod_status' reports I installed lynx, a terminal-based web browser:

```
$ sudo apt-get install lynx
```
After which I could view the reports with the command:
```
$ lynx http://localhost/server-status
```

#####Setting up automatic security updates

I enabled automatic security upgrades with the following commands
```
sudo apt-get install unattended-upgrades
sudo dpkg-reconfigure unattended-upgrades
```

and selected yes in the dialog that appeared.

### Sources

In addition to the Configuring Linux Web Servers Udacity course, I relied on several third-party resources to complete this project. Here they are:

 - [Setting the ssh port](http://ubuntuforums.org/showthread.php?t=1591681)
 - [Changing the timezone](https://help.ubuntu.com/community/UbuntuTime#Using_the_Command_Line_.28terminal.29)
 - [Installing and configuring fail2ban](https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04)
 - [Configuring Postgresql](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
 - [Configuring a flask application with Apache2 and mod-wsgi](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
 - [Monitoring apache2 with mod_status](http://www.tecmint.com/monitor-apache-web-server-load-and-page-statistics/)
 - [Setting file permissions for apache](http://serverfault.com/questions/125865/finding-out-what-user-apache-is-running-as)
 - [Setting up automatic security updates on Ubuntu](http://askubuntu.com/questions/9/how-do-i-enable-automatic-updates)


### Contact

For comments or questions, reach me at [adamdidthis@hotmail.com](mailto:adamdidthis@hotmail.com)