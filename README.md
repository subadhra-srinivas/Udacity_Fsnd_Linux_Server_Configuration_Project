# Udacity_Fsnd_Linux_Server_Configuration_Project

**Introduction**
This project is about making baseline installation of Linux Server and prepare it to host
Item Catalog web application. The server is secured from number of attack vectors. The
Apache web server and postgres servers are installed and configured.</br>

We can visit the http://ec2-18-218-86-21.us-east-2.compute.amazonaws.com for the Item
Catalog application.</br>

IP Address 18.218.86.21</br>

**1. Creating a New User:**
   sudo adduser grader</br>

   sudo ls /etc/sudoers.d</br>
   sudo cp /etc/sudoers.d/vagrant /etc/soders.d/grader</br>
   vim /etc/sudoers.d/grader</br>

   grader ALL=(ALL) NOPASSWD:ALL</br>

**2. Public Key Encryption:**
   We will generate our key pair on our local machine using ssh-keygen</br>

   the file name for key is linuxserver</br>

   Save the private key in ~/.ssh on local machine</br>
   Once done ssh-keygen has generated two files one is linuxserver. and linuxserver.pub</br>
   linuxserver.pub is what we will place on the server to enable key based authentication.</br>

   Place the public key into the remote server</br>
   We login to server as grader</br>
   mkdir .ssh</br>

   Create a new file authorized keys</br>
   vim .ssh/authorized_keys</br>

   Copy the contents of .ssh/linuxserver.pub then back on my server as grader. Edit the
   authorized_keys file and just paste the contents and save it.</br>

   Set permissions .ssh directory and authorized_keys file.</br>
    chmod 700 .ssh</br>
    chmod 644 .ssh/authorized_keys</br>

   We can now login as grader</br>
   ssh grader@18.218.86.21 -p 2200 -i ~/.ssh/linuxserver</br>

**3. Disable password based Authentication:**
   Edit the sshd_config file</br>
   sudo vim /etc/ssh/sshd_config</br>

**4. Change the Port to 2200:**
   Set the PasswordAuthentication no</br>
   Edit the Port to 2200 insted of 22</br>

**5. Diable Root login:**
   Edit /etc/ssh/ssh_config</br>
   Set PermitRootLogin no</br>

   Restart the service.</br>
   sudo service ssh restart</br>

**6. Upgrading installed packages**
   All of the available package source are listed in the file</br>
   cat /etc/apt/sources.list</br>
   Updating available package list</br>
   sudo apt-get update</br>
   Upgrading the installed packages</br>
   sudo apt-get upgrade</br>

**7. Configure uncomplicated Firewall (UFW):**
   configure ufw to allow incoming connections from ssh, http and ntp</br>
   To check the status of firewall</br>
   sudo ufw status</br>
   sudo ufw default deny incoming</br>
   sudo ufw default allow outgoing</br>
   sudo ufw allow 2200/tcp</br>
   sudo ufw allow 80/tcp</br>
   sudo ufw allow 123/udp</br>
   sudo ufw enable</br>

**8. Configure local timezone**
   sudo dpkg-reconfigure tzdata</br>
   select the continent and select the timezone</br>

**9. Install and Configure Apache**
   sudo apt-get install apache2</br>
   sudo apt-get install libapache2-mod-wsgi</br>
   sudo service apache2 restart</br>

**10. Install and Configure Postgres sql**
    Install PostgreSQL sudo apt-get install postgresql</br>
    Check if no remote connections are allowed sudo vim /etc/postgresql/9.6/main/pg_hba.conf</br>
    sudo su - postgres</br>
    psql</br>
    create database catalog;</br>
    create user catalog;</br>
    create role catalog with password ‘password’;</br>
    alter role catalog with password ‘password’;</br>
    revoke all on schema public from public;</br>
    grant all on schema public to catalog;</br>
    alter user catalog with login;</br>
    grant all on database catalog to catalog;</br>

**11. Install Git**
    Install Git using sudo apt-get install git</br>

**12. Install Flask, SQLAlchemy, etc**
    Give the following commands:</br>

    sudo apt-get install python-psycopg2 python-flask</br>
    sudo apt-get install python-sqlalchemy python-pip</br>
    sudo pip install oauth2client</br>
    sudo pip install requests</br>
    sudo pip install httplib2</br>

**13. Clone and setup Catalog App:**
    cd /var/www/</br>

    Clone the catalog App to the Virtual machine</br>
    git clone https://github.com/subadhra-srinivas/Udacity_Fsnd_Item_Catalog_Project.git</br>

    sudo mkdir FlaskApp cd FlaskApp</br>
    Make another FlaskApp directory</br>
    sudo mkdir FlaskApp cd FlaskApp</br>

    Move all the files from catalog directory to second FlaskApp directory</br>

    Rename item_catalog_project.py to __init__.py</br>

    Change the engine = create_engine('postgres//catalog:password@localhost/catalog') in __init__.py,
    databse_setup.py and lotsofmenu.py

    Change the directory stucture for client_secrets.json and fb_client_secrets.json to
    /var/www/FlaskApp/FlaskApp/client_secrets.json</br>

    /var/www/FlaskApp/FlaskApp/fb_client_secrets.json</br>
    Make this change in fbconnect and gconnect functions also</br>

    Change the host = 'ec2-18-218-86-21.us-east-2.compute.amazonaws.com' and port 80</br>

    Update the Google OAuth client secrets file</br>
    Update the redirect_uri's and javascript_origins</br>
    "redirect_uris":["http://ec2-18-218-86-21.us-east-2.compute.amazonaws.com/login",</br>
		     "http://ec2-18-218-86-21.us-east-2.compute.amazonaws.com/gconnect"],</br>
    "javascript_origins":["http://ec2-18-218-86-21.us-east-2.compute.amazonaws.com"]</br>

    Update the Facebook OAuth client secrets file</br>
    Update the Facebook information from the https://developers.facebook.com/</br>
    on the Settings page, the website URL needs to read http://ec2-18-218-86-21.us-east-2.compute.amazonaws.com.</br>
    Then in the "Advanced" tab, in the "Client OAuth Settings" section, add</br> 
    http://ec2-18-218-86-21.us-east-2.compute.amazonaws.com and 18.218.86.21 to the "Valid OAuth redirect URIs"</br>
    field. Then save these changes.</br>

    Create the database schema by running the sudo python database_setup.py</br>

**14. Create and enable a new virtual host**
    1. Create FlaskApp.conf and edit</br>
       sudo vim /etc/apache2.sites-available/FlaskApp.conf</br>

    2. Add the following lines of code save and close</br>

       <VirtualHost *:80></br>
                ServerName 18.218.86.21</br>
                ServerAdmin subadhra.srinivas@gmail.com</br>
                WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi</br>
                <Directory /var/www/FlaskApp/FlaskApp/></br>
                        Order allow,deny</br>
                        Allow from all</br>
                </Directory></br>
                Alias /static /var/www/FlaskApp/FlaskApp/static</br>
                <Directory /var/www/FlaskApp/FlaskApp/static/></br>
                        Order allow,deny</br>
                        Allow from all</br>
                </Directory></br>
                ErrorLog ${APACHE_LOG_DIR}/error.log</br>
                LogLevel warn</br>
                CustomLog ${APACHE_LOG_DIR}/access.log combined</br>
       </VirtualHost></br>

    3. Enable the virtual host with the following command</br>
       sudo a2ensite FlaskApp</br>

**15. Create the .wsgi file**

    1. Move to /var/www/FlaskApp</br>
       sudo vim flaskapp.wsgi</br>
    2. Add the following lines of code</br>

       #!/usr/bin/python</br>
       import sys</br>
       import logging</br>
       logging.basicConfig(stream=sys.stderr)</br>
       sys.path.insert(0,"/var/www/FlaskApp/")</br>

       from FlaskApp import app as application</br>
       application.secret_key = 'Add your secret key'</br>

**16. Restart Apache**
    Restart Apache sudo service apache2 restart</br>

    **References:**

    https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps</br>
    https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps</br>

