# Full Stack Nanodegree P5
Solution for P5 of Full Stack Nanodegree on Udacity

# IP Address of the server
52.24.79.42

# Preparing the VM
These steps are for a fresh EC2 instance on AWS. The image used is an Ubuntu 14.04 64 bits.
The text at the end of the line after '#' is not part of the commands, these are comments to explain the commands.
These commands are to be run as root.

### Update system
 apt-get update
 apt-get upgrade

### Setup the user 'grader'
 useradd grader -m -s /bin/bash
 
 openssl rand -base64 16 <b># This will generate a random string that is 16 caracters long</b>
 
 passwd grader <b># Use the string generated as a password for the user 'grader'</b>
 
 echo "grader  ALL=(ALL) ALL" >> /etc/sudoers <b># Add user 'grader' to sudoers list</b>

 <b># The following 3 commands allow you to use the SSH key of user 'root' for user 'grader'</b>
 
 mkdir /home/grader/.ssh
 
 cp /root/.ssh/authorized_keys /home/grader/.ssh/authorized_keys
 
 chown grader:grader /home/grader/.ssh/authorized_keys

### Update ssh config
 <b># Open the config file:</b>
 
 nano /etc/ssh/sshd_config
 
 <b># Update the following options:</b>
 
 Port 2200 <b># change port number to 22</b>
 
 PermitRootLogin no <b># disable root login</b>
 
 <b># Disable password login and enable public key authentication, this should be the default on a fresh install</b>
 
 ChallengeResponseAuthentication no <b># this is the default</b>
 
 PasswordAuthentication no <b># this is the default</b>
 
 PubkeyAuthentication yes <b># This is the default</b>

### Update firewall config
 ufw allow in 2200

 ufw allow in 80

 ufw allow in 123

 ufw default deny incoming
 
 ufw enable

### Update timezone
 dpkg-reconfigure tzdata <b>"None of the above" -> "UTC"</b>

### Install apache, postgres and flask
 apt-get install -qqy apache2 libapache2-mod-wsgi postgresql python-psycopg2 python-flask python-sqlalchemy python-pip
 
 pip install bleach werkzeug==0.8.3 flask==0.9 Flask-Login==0.1.3 oauth2client requests httplib2

### Create database and user 'catalog'
 su postgres

 createdb catalog
 
 createuser -PRDS catalog <b># When prompted for the password, make sure to enter 'catalog'</b>
 
    Enter password for new role: catalog
    
    Enter it again: catalog
 
 psql -c "GRANT ALL PRIVILEGES ON DATABASE catalog to catalog"
 
 exit
 
 <b># Update Postgres access, replace the content of /etc/postgresql/9.3/main/pg_hba.conf by:</b>
 local   all             all                                     peer
 
 host    all             all           127.0.0.1/32              reject
 
 host    all             all           0.0.0.0/0                 reject
 <b># First two lines to give full local access, the last line to reject everything else.</b>

### Install git and install the catalog app
 apt-get install git
 
 cd /var/www
 
 git clone https://github.com/bh-chaker/cbh-catalog.git catalog
 
 cd catalog
 
 <b>Create a Google App, you can follow this video from Authentication & Authorization: OAuth, a Udacity course. After this step you should have a valid client_secrets.json file in the catalog directory (next to project.py file)

 IMPORTANT: When adding the Authorized JavaScript Origin, make sure to use the URL 'http://52.24.79.42/' without a port number.</b>
 
 chown -R www-data:www-data /var/www <b># To avoid any permission issues, give ownership of /var/www to Apache user (www-data).</b>
 
 <b># Install and populate</b>
 
 python database_setup.py
 
 python database_populate.py
 
### Configure apache virtual host
 <b># Open config file:</b>
 
 nano /etc/apache2/sites-available/catalog.conf
 
 <b># Fill it with following content:</b>
    <VirtualHost *:80>
            ServerAdmin benhammouda.chaker@gmail.com
            WSGIScriptAlias / /var/www/catalog/catalog.wsgi
            <Directory /var/www/catalog/>
                Order allow,deny
                Allow from all
                Options -Indexes
            </Directory>
            Alias /static /var/www/catalog/static
            <Directory /var/www/catalog/static/>
                Order allow,deny
                Allow from all
            </Directory>
            ErrorLog ${APACHE_LOG_DIR}/error.log
            LogLevel warn
            CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>

 <b># Disable default virtual server and enable 'catalog' virtual server:</b>
 
 a2dissite 000-default
 
 a2ensite catalog
 
 service apache2 reload
 
### Restart the server
 reboot
