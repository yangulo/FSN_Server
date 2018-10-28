# Server details
* IP address: 18.220.83.174
* Public DNS: ec2-18-220-83-174.us-east-2.compute.amazonaws.com
* SSH port: 2200

# Linux Server Configuration
This project is part of the Full Stack Web Developer Nanodegree offered by Udacity. This project covers:

* A baseline installation of a Linux server and prepares it to host a simple catalog web application. 
* It secures the server from a number of attack vectors
* It installs and configures a database server, and deploys the catalog web application previously built in the same course onto it.

## Getting Started

The steps to Configure the Linux server are listed below:

1. To run this project, you'll need a Linux server instance. This nanodegree recommends using [Amazon Lightsail](https://aws.amazon.com/) for this.
    * Log into Amazon Lightsail
    * Create an instance
    * Choose an instance: Ubuntu
    * Choose your instance plan (The lowest tier of instance is just fine. And as long as you complete the project within a month and shut your instance down, the price will be zero)
    * Give your instance a hostname
    * Wait for it to start up (It may take a few minutes for your instance to start up)
    * It's running. Once your instance is running, click the connect and follow the instrctions there, to connect you will have to copy the line under "Example". (Access to it from your terminal)
    * Now that you have a working instance, you can get right into the project!

2. Give grader access: 
    * Create a new user account named grader

    ```
     $ sudo adduser grader
    ```

    * Give Sudo Access to grader

    ```
    sudo nano /etc/sudoers.d/grader
    ```

    * Add following line to this file
    ```
    grader ALL=(ALL:ALL) ALL
    ```

3. Update all currently installed packages
    * Update the package indexes

    ```
    apt-get update
    ```

    * Upgrade the installed packages

    ```
    apt-get upgrade
    ```

4. Configure the key-based authentication for grader user
    * Generate an encryption key on your local machine (This is done on a regular new terminal window<b> not logged as ubunto nor grader</b>)

    ```
    ssh-keygen
    ```

    * You will be sked to give a file name for the key pair. Use the default directory where the key pair should exist <b>Users/xxx/.ssh </b>(within your user's home directory.)
    * Place the public key on the server <b>that we want to use</b>, as grader do:

    ```
    su grader
    mkdir .ssh
    chmod 700 .ssh
    vi authorized_keys
    ```
    
    * Here you will paste the public key generated before, then

    ```
    chmod 644 authorized_keys
    ```
    
    * Now you can log into as grader from a new terminal window

    ```
    $ ssh -i <private key file name> grader@XX.XX.XX.XX
    ```

    * <b>Note</b> that the XX.XX.XX.XX is the ip from your AWS that you use to connect.

5. Disable root login
    * Change the following line in the file ```/etc/ssh/sshd_config``` From ```PermitRootLogin without-password``` to ```PermitRootLogin no```
    * Also, uncomment the following line so it reads:

    ```
    PasswordAuthentication no
    ```

    * Do ```service ssh restart``` for the changes to take effect.

6. Change the timezone to UTC

    * Check the timezone with the ```date``` command. This will display the current timezone after the time. If this is not UTC change it with:

    ```
    sudo timedatectl set-timezone UTC
    ```

7. Change SSH port from 22 to 2200:

    * Edit the file ```/etc/ssh/sshd_config``` and change the line port 22 to:

    ```
    Port 2200
    ```

    * Then restart the SSH service:

    ```
    sudo service ssh restart
    ```

    * Use the following command to login to the server:

    ```
    $ ssh -i <private key file name> grader@XX.XX.XX.XX -p 2200
    ```

8. Configuration Uncomplicated Firewall (UFW):

    * Block all incoming connections on all ports:

    ```
    sudo ufw default deny incoming
    ```

    * Allow outgoing connection on all ports:

    ```
    sudo ufw default allow outgoing
    ```

    * Allow incoming connection for SSH on port 2200:

    ```
    sudo ufw allow 2200/tcp
    ```

    * Allow incoming connections for HTTP on port 80:

    ```
    sudo ufw allow www
    ```

    * Allow incoming connection for NTP on port 123:

    ```
    sudo ufw allow ntp
    ```

    * To check the rules that have been added before enabling the firewall use:

    ```
    sudo ufw show added
    ```

    * To enable the firewall, use:

    ```
    sudo ufw enable
    ```

    * To check the status of the firewall, use:

    ```
    sudo ufw status
    ```

9. Install Apache to serve a Python mod_wsgi application:
    * Install Apache:

    ```
    sudo apt-get install apache2
    ```

    * Install the libapache2-mod-wsgi package:

    ```
    sudo apt-get install libapache2-mod-wsgi
    ```

10. Install and configure PostgreSQL:

    * Install PostgreSQL with:

    ```
    $ sudo apt-get install libpq-dev python-dev
    $ sudo apt-get install postgresql postgresql-contrib
    ```

    * Login as postgres User 

    ```
    $ sudo su - postgres
    $ psql
    ```

    * Create a new User named catalog:

    ```
    CREATE USER catalog WITH PASSWORD 'password';
    ```

    * Create a new DB named catalog: 

    ```
    CREATE DATABASE catalog WITH OWNER catalog;
    ```

    * Connect to the database catalog:

    ```
    \c catalog
    ```

    * Revoke all rights: 

    ```
    REVOKE ALL ON SCHEMA public FROM public;
    ```

    * Lock down the permissions only to user catalog: 

    ```
    GRANT ALL ON SCHEMA public TO catalog;
    ```

    * Log out from PostgreSQL: ```\q```. Then return to the grader user: ```$ exit```

11. Install Flask, SQLAlchemy and more

    * Issue the following commands:

    ```
    sudo apt-get install python-psycopg2 python-flask
    sudo apt-get install python-sqlalchemy python-pip
    sudo pip install oauth2client
    sudo pip install requests
    sudo pip install httplib2
    sudo pip install flask-seasurf
    ```

12. Install Git version control software

    ```
    sudo apt-get install git
    ```

13. Clone the repository that contains the Catalog project:

    ```
    cd var/www/
    mkdir catalog
    clone git [project link](https://github.com/yangulo/FSN_C) catalog
    ```

    * To run this application <b>remeber to run the db_setup.py and populate.py files </b>

14. Update catalog.wsgi file for this installation

    * Make a catalog.wsgi file to serve the application over the mod_wsgi. with:
    
    ```
    cd /var/www/catalog
    vi catalog.wsgi
    ```
   
    * Add
   
    ```
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0, "/var/www/catalog")

    from catalog import app as application
    application.secret_key = 'super_secret_key'
    ```

15. Inside project.py change database connection
    
    * Note: Insert database password in <b>password@</b>
   
    ```
    engine = create_engine('postgresql://catalog:password@localhost/catalog')
    ```

    * Also update the client_secrets.json location
   
    ```
    CLIENT_ID = json.loads(
    open('/var/www/catalog/catalog/client_secrets.json', 'r').read())['web']['client_id']
    APPLICATION_NAME = "My Catalog Project"
    ```

16. Edit the default Virtual File with following content:
   
    * To serve the catalog app using the Apache web server go to ```/etc/apache2/sites-available/```, and to ```000-default.conf``` and update with the following:

    ```
    <VirtualHost *:80>
    #ServerName XX.XXX.XX.XXX
    #ServerAlias ec2-XX-XXX-XX-XXX.us-east-2.compute.amazonaws.com
    ServerAdmin admin@localhost
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

    * <b>Note:</b> that XX.XXX.XX.XXX is the Public IP you can fine in AWS

17. Restart Apache to launch the app

```
sudo apache2ctl restart
```

