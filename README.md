# Linux Server Configuration

## Overview
In this project, I deployed a Flask app to AWS lightsail ubuntu instance by following security practices that I have gaind from Udacity FSND program.

## Server Specs
* 512 MB RAM
* 1 vCPU
* 20 GB SSD
* Ubuntu 16.04 LTS

## Connection Details
* Public IP: 3.133.131.148

## Deployment Steps

### Step 1: Create Ubuntu instance
1. GO to [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp/home) and login using your AWS account
2. Once you are logged in click Create instance.
3. Choose Linux/Unix platform Then OS Only then  Ubuntu 16.04 LTS.
4. Choose the cheapest instance plan (Free for the first month).
5. Choose your instance name.
6. Click on Create instance button.
7. Wait for the instance to start-up.

### Step 2: SSH into the server 
1. Download your private key from your account page.
2. Download SSH PuTTY (Optional) or yous your terminal.
3. Follow the link to [setup PuTTY](https://lightsail.aws.amazon.com/ls/docs/en_us/articles/lightsail-how-to-set-up-putty-to-connect-using-ssh)

### Step 3: Change default SSH port
This an important step to change the default SSH port from 22 to 2200.
1. Edit /etc/ssh/sshd_config file
    `sudo nano /etc/ssh/sshd_config`
2. In line 5 change the port number from 22 to 2200.
3. Restart SSH service 
    `sudo service ssh restart`

### Step 4: Update installed packages
Run the following command to get updates
    `sudo apt-get update`
Then `sudo apt-get upgrade` to commit updates

### Step 5: Configure Firewall
1.Check firwall status #Should be inactive.
`sudo ufw status`

2.We deny all incoming connections
`sudo ufw default deny incoming`

3.Allow all outgoing
`sudo ufw default allow outgoing`

4.Allow alternative SSH TCP port 2200 
`sudo ufw allow 2200/tcp`

5.Allow HTTP packets 
`sudo ufw allow www`

6.Allow UDP port 123
`sudo ufw allow 123/udp`

7.Deny the default SSH port 22
`sudo ufw deny 22`

8.Enable Firwall
`sudo ufw enable`

9.Close SSH connection.

10.GO to instance page on AWS.

11.Click on Manage then Networking.

12.Allow the following ports:   
* 80/TCP
* 123/UDP
* 2200/TCP
* Delete 22/TCP

### Step 6: Create grader account
1. `sudo adduser grader` and choose password
2. Edit sudoers file
   `sudo visudo`
   and add `grader  ALL=(ALL:ALL) ALL` below `root ALL=(ALL:ALL) ALL`
3. Verify that grader has sudo permissions
   `su - grader` then enter password then `sudo -l` you should see a message " Matching Defaults entries for grader on IPxx "

### Step 7: Create grader SSH keys
On your local terminal:
1. Run `ssh-keygen`
2. Follow message instruction.
3. Copy the contents of ***.pub file.
4. Login to the grader account.
   
On grader account:
1. Create a new directory called ~/.ssh (mkdir .ssh)
2. Run `sudo nano ~/.ssh/authorized_keys` and paste the content into this file
3. Run `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys`
4. Check /etc/ssh/sshd_config file if <code>PasswordAuthentication</code> is set to no
5. Restart SSH service 
   `sudo service ssh restart`

Now you can login `ssh -i ~/.ssh/grader_key -p 2200 grader@3.124.1.112`

### Step 8: Configure timezone to UTC
Run `sudo dpkg-reconfigure tzdata` the choose None of the above then UTC

### Step 9: Install Apache
1. Run `sudo apt-get install apache2`
2. Configure Apache to support python3 by running the following command `sudo apt-get install libapache2-mod-wsgi-py3`

### Step 10: Install PostgreSQL
1. Run `sudo apt-get install postgresql`
2. Make sure that PostgreSQL dosn`t allow remote connection by checking /etc/postgresql/9.5/main/pg_hba.conf file 
3. Switch to postgres user `sudo su - postgres`
4. open PostgreSQL `psql`
5. Create database catalog
   `CREATE USER catalog WITH PASSWORD 'password'`
6. Create Role
   `ALTER ROLE catalog CREATEDB;`
7. exit `\q`
8. Switch back to grader account `exit`
9. Create a new user `sudo adduser catalog`
10. Give catalog user sudo permissions 
    `sudo visudo`
    insert `catalog  ALL=(ALL:ALL) ALL`
11. Login to catalog account `su - catalog`
12. Create database `createdb catalog`
8. Switch back to grader account `exit`

### Step 11: Clone Movies Catalog Repository
1. Install git `sudo apt-get install git`
2. `cd /var/www`
3. `mkdir MCApp`
4. `cd MCApp`
5. `sudo git clone https://github.com/cshummod/Movies-Catalog.git`
6. `cd ..`
7. Change owner to grader `sudo chown -R grader:grader MCApp/`
8. `cd MCApp`
9. Rename application.py to __init__.py `mv application.py __init__.py.`
10. In database.py and application.py change the engine to `engine = create_engine('postgresql://catalog:password@localhost/catalog')`

### Step 12: Setup Google OAuth
1. Go to Google devloper console.
2. Click APIs & services.
3. Click Credentials.
4. Create an OAuth Client ID.
5. Add http://3.133.131.148 and http://ec2-3-133-131-148.us-east-2.compute.amazonaws.com/ as authorized JavaScript origins.
Add http://ec2-3-133-131-148.us-east-2.compute.amazonaws.com/oauth2callback as authorized redirect URI.
1. Download JSON secrets file then open and copy all its content
2. Open /var/www/MCApp/MCApp/client_secret.json and paste contents into it.
3. Enter client ID in templates/login.html.

Remember to use IPXX.compute... so google login works.

### Step 13: Install Dependencies 
1.While logged in as grader install pip
`sudo apt-get install python3-pip`

2.Install virtual environment `sudo apt-get install python-virtualenv`

3.`cd /var/www/MCApp/MCApp/`
   
4.`sudo virtualenv -p python3 venv3`
   
5.Change ownership to grader `sudo chown -R grader:grader venv3/`
   
6.Activate the new environment `. venv3/bin/activate`
    
7.Install the following dependencies:
`pip install httplib2`

`pip install requests`

`pip install --upgrade oauth2client`

`pip install sqlalchemy`

`pip install flask`

`sudo apt-get install libpq-dev`

`pip install psycopg2`

8.Run python3 __init__.py you should see 

`* Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)`

9.Deactivate the virtual environment `deactivate`

### Step 14: Setup virtual host
1.Add the following line in /etc/apache2/mods-enabled/wsgi.conf file to use Python 3.
   `WSGIPythonPath /var/www/MCApp/MCApp/venv3/lib/python3.5/site-packages`

2.Create /etc/apache2/sites-available/MCApp.conf and add the following lines to configure the virtual host:

```XML
<VirtualHost *:80>
    ServerName 3.124.1.112
  ServerAlias ec2-3-124-1-112.eu-central-1.compute.amazonaws.com
    WSGIScriptAlias / /var/www/MCApp/MCApp.wsgi
    <Directory /var/www/MCApp/MCApp/>
    	Order allow,deny
  	  Allow from all
    </Directory>
    Alias /static /var/www/MCApp/MCApp/static
    <Directory /var/www/MCApp/MCApp/static/>
  	  Order allow,deny
  	  Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

1. Enable virtual host `sudo a2ensite MCApp`
2. Reload Apache `sudo service apache2 reload`

### Step 15: Setup Flask MCApp
   Create /var/www/MCApp/MCApp.wsgi file add the following lines:

```python
activate_this = '/var/www/MCApp/MCApp/venv3/bin/activate_this.py'
with open(activate_this) as file_:
    exec(file_.read(), dict(__file__=activate_this))

#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/MCApp/MCApp/")
sys.path.insert(1, "/var/www/MCApp/")

from MCApp import app as application
application.secret_key = 'super_secret_key'
```

2.Restart Apache `sudo service apache2 restart`
3.Disable default Apache site `sudo a2dissite 000-default.conf`
4.Run `python databasesetup.py` to setup the database
5.Run `seed.py` to seed the data
6.Restart Apache `sudo service apache2 restart`

## Contributors
Mohammed Mahdi Ibrahim

## Support
For any related questions about the tool you can contact me at wmm@hotmail.it
