# udacity-linux-conf-catalog
Udacity Full Stack Web Developer Nanodegree - Linux Server Configuration 

This project consist of deploying a Flask web application to a linux server using AWS Lightsail and Apache.

Host name/URL: http://ec2-34-220-61-4.us-west-2.compute.amazonaws.com/

IP Address: http://34.220.61.4/

Accessible SSH port: 2200.

Amazon Lightsail Process:

1) Create an account
     a. https://aws.amazon.com/lightsail/?sc_channel=PS&sc_campaign=pac_ps_q4&sc_publisher=google&sc_medium=AW_PAC_CORE_Enterprise_Lightsail_b_Adopt&sc_content=lightsail_e&sc_detail=amazon%20lightsail&sc_category=lightsail&sc_segment=243316795586&sc_matchtype=e&sc_country=US&sc_geo=namer&sc_outcome=pac&s_kwcid=AL!4422!3!243316795586!e!!g!!amazon%20lightsail&trk=ps_a131L000005iyYnQAI&ef_id=VM81PgAAAW72rpx1:20180619072250:s

2) Create an instance
    a. click Linux as platform 
    b. select OS only for blue print followed by Ubuntu
    c. name your instance
    d. wait for the instance to say "running"
    e. click on status card
    f. click on Account page link located at the bottom
    g. navigate to ssh key tab and click the Download link
    h. Navigate to connect using ssh page and click on the networking tab
    i. click on 'add another' and add a port for 123 and 2200

Server Configuration

1) Go to terminal and type $ killall Finder
2) Next type $ defaults write com.apple.finder AppleShowAllFiles TRUE
3) Now that your hidden file are accessible drag the .pem file into the .ssh directory
4) Type $ chmod 600 ~/.ssh/YourAWSKey.pem to make the public accessible 
5) Log into Amazon lightsail server $ ssh -i ~/.ssh/YourAWSKey.pem ubuntu@yourpublicipaddress
6) Switch to root user by typing $ sudo su - 
7) Type $ sudo adduser grader
8) Create sudoers directory $ sudo nano /etc/sudoers.d/grader 
9) Add this 'grader ALL=(ALL:ALL) ALL' withot single quotes (control x, type yes and press enter to save and quit)
10) Open new terminal and input $ ssh-keygen -f ~/.ssh/udacity_key.rsa
11) On new terminal paste $ cat ~/.ssh/udacity_key.rsa.pub and copy the public key that appears
12) Go back to first terminal and $ cd /home/grader
13) Create .ssh directory into Amazon Lightsail as the root user: $ mkdir .ssh
14) Create a file to store the copied public key $ touch .ssh/authorized_keys
15) Paste in $ nano .ssh/authorized_keys
16) Change the permission $ sudo chmod 700 /home/grader/.ssh and $ sudo chmod 644 /home/grader/.ssh/authorized_keys
17) Change the owner from root to grader: $ sudo chown -R grader:grader /home/grader/.ssh
18) $ sudo service ssh restart 
19) Type $ ~.
20) Log into server as grader $ ssh -i ~/.ssh/udacity_key.rsa grader@yourpublicipaddress
21) Enforce the key-based authentication: $ sudo nano /etc/ssh/sshd_config
    a. Find the PasswordAuthentication line and change text after to 'no'
    b. $ sudo service ssh restart
22) Change port from 22 to 2200 $ sudo nano /etc/ssh/ssdh_config
    a. Find the Port line and change 22 to 2200
    b. $ sudo service ssh restart
23) Disconnect the server by $ ~.
24) Log back through port 2200: $ ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@yourpublicipaddress
25) Disable ssh login for root user $ sudo nano /etc/ssh/sshd_config
    a) Find the PermitRootLogin line and edit to no 
    b) Restart ssh $ sudo service ssh restart
26) Configure UFW to fulfill the requirement
    a. $ sudo ufw allow 2200/tcp
    b. $ sudo ufw allow 80/tcp
    c. $ sudo ufw allow 123/udp
    d. $ sudo ufw enable

Deploy Catalog using apache

1) Install required packages
    a. $ sudo apt-get install apache2
    b. $ sudo apt-get install libapache2-mod-wsgi python-dev
    c. $ sudo apt-get install git

2) $ sudo a2enmod wsgi
3) $ sudo service apache2 start
4) You should input the public IP address and you should see a default page 
5) Set up folder structure
    a. $ cd /var/www
    b. $ sudo mkdir catalog
    c. $ sudo chown -R grader:grader catalog
    d. $ cd catalog
6) Now we clone the project from Github: $ git clone [your link] catalog
7) Create a .wsgi file: $sudo nano catalog.wsgi
    a. Copy and paste this:
      import sys
      import logging
      logging.basicConfig(stream=sys.stderr)
      sys.path.insert(0, "/var/www/catalog")
      sys.path.insert(0, "/var/www/catalog/catalog/venv/lib/python2.7/site-packages")

      from catalog import app as application
      application.secret_key = 'supersecretkey'

8) Rename the application.py to __init__.py
9) Install and start the virtual machine cd catalog/catalog
    a. $ sudo pip install virtualenv
    b. $ sudo virtualenv venv
    c. $ source venv/bin/activate
    d. $ sudo chmod -R 777 venv
    e. you should see (venv) before your teminal username
10) Install the Flask and other packages needed for this application
    a. $ sudo apt-get install python-pip
    b. $ sudo pip install Flask
    c. $ sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlaclemy_utils requests render_template, redirect, psslib [anything else you have built within this application
11) Use the nano __init__.py command to change the client_secrets.json line to /var/www/catalog/catalog/client_secrets.json for your CLIENT_ID
    a. CLIENT_ID = json.loads(open('the path above', 'r').read())['web']['client_id']
12) Change your host to your Amazon Lightsail public ip address and port to 80
13) Configure and enable the virtual host
    a. $ sudo nano /etc/apache2/sites-available/catalog.conf
    b. paste this
      <VirtualHost *:80>
        ServerName 34.220.61.4
        ServerAlias ec2-34-220-61-4.us-west-2.compute.amazonaws.com
        ServerAdmin admin@34.220.61.4
        WSGIScriptAlias / /var/www/catalog/catalog/catalog.wsgi
        <Directory /var/www/catalog/catalog/>
            Order allow,deny
            Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
      </VirtualHost>
14) Set up the database
    1. $ sudo apt-get install libpq-dev python-dev
    2. $ sudo apt-get install postgresql postgresql-contrib
    3. $ sudo su - postgres -i
15) $ psql
16) Create a user to create and set up the database
    a. $ CREATE USER catalog WITH PASSWORD [your password];
    b. $ ALTER USER catalog CREATEDB;
    c. $ CREATE DATABASE catalog WITH OWNER catalog;
    d. Connect to database $ \c catalog
    e. $ REVOKE ALL ON SCHEMA public FROM public;
    f. $ GRANT ALL ON SCHEMA public TO catalog;
    g. Quit the postgrel command line: $ \c and then $ exit
17) Use sudo nano command to change all engine to engine = create_engine('postgresql://catalog:[your password]@localhost/catalog 
18) Initiate the database python database_setup.py
19) sudo service apache2 restart
20) Enter your public IP address or host name into the browser. Completed!

Reference
1) I would like to thank these Alumini for having there respository available.
    a. https://github.com/callforsky/udacity-linux-configuration
    b. https://github.com/iliketomatoes/linux_server_configuration
    c. https://github.com/stueken/FSND-P5_Linux-Server-Configuration

