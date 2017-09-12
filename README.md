# linux-server-configuration

- IP Address: 35.156.85.227
- SSH port: 2200
- URL: http://35.156.85.227/catalog


## Summary of the software installed and configuration changes

### Server preparation
1. Created an Amazon lightsail instance
2. Updated the available packages lists via `sudo apt-get update`
3. Upgraded installed packages via `sudo apt-get upgrade`
4. Removed packages that are not longer used via `sudo apt-get autoremover`

### User management
5. Created new users via `sudo adduser username`
    - A personal user
    - User 'grader' for grading purposes
6. Gave both users sudo rights via `sudo nano /etc/sudoers.d/grader` and added the following line
    - grader ALL=(ALL) NOPASSWD:ALL
7. Genereted a key pair for both users via 'ssh-keygen' on my local machine
    - default directory: ~/.ssh/
8. Installing the public key on the amazon server
    - logged in as user grader
    - created a folder .ssh/ via `mkdir .ssh/`
    - created a file in the .ssh/ folder via `touch .ssh/authorized_keys`
    - updated this file via `nano .ssh/authorized_keys` with the content of the public key
    - set file permissions
        - `chmod 700 .ssh/`
        - `chmod 644 .ssh/authorized_keys`
9. Disable password login —> edited the sshd config file via `sudo nano /etc/ssh/sshd_config`
    - set PasswordAuthentication to no (was already set to no)
    - Set PermitRootLogin to no
    - reset ssh service via `sudo service ssh restart`

### Firewall settings
10. Verified firewall (ufw) was inactive via `sudo ufw status`
11. Configured ports in UFW:
    - `sudo ufw default deny incoming`
    - `sudo ufw default allow outgoing`
    - `sudo ufw status`
    - `sudo ufw allow ssh`
    - `sudo ufw allow 2200/tcp`
    - `sudo ufw allow www`
    - `sudo ufw allow nap`
    - `sudo ufw enable`

### Install applications
12. Installed an Apache HTTP server via `sudo apt-get install apache2`
13. Installed the WSGI module to handle incoming requests via python
    - Created a wsgi file in folder /var/www/catalog/catalog.wsgi with the following content
        -   `import sys
            sys.path.insert(0, '/home/kkriek/item-catalog')

            from application import app as application`
    - Edited the /etc/apache2/sites-enabled/000-default.conf config file. I added the following content in the VirtualHost element
        -   `WSGIDaemonProcess catalog
            WSGIScriptAlias / /var/www/catalog/catalog.wsgi

            <Directory /var/www/catalog>
               WSGIProcessGroup catalog
               WSGIApplicationGroup %{GLOBAL}
               Order deny,allow
               Allow from all
            </Directory>`
    - restarted the apache server via `sudo apache2ctl restart`
14. Installed Postgres via `sudo apt-get install postgresql`
15. Installen git via `sudo apt-get install git` (git was already installed)

### Create a database in Postgres
16. Swithed to the postgres user via `sudo su - postgres`
17. Created a database via `createdb catalog`
18. Opened postgres via `psql`
19. Created a database user via `create user catalog`
    - Revoked all access rights for user catalog via `REVOKE ALL ON SCHEMA public FROM catalog;`
    - Granted access rights for user catalog for database catalog via `GRANT ALL ON DATABASE catalog TO catalog;`

### install python modules
20. Installed pip via `sudo apt-get install python-pip`
21. Installed flask via `sudo pip install Flask`
22. Installed SQLAlchemy via `sudo pip install SQLAlchemy`
23. Installed oauth2client via `sudo apt-get install python-oauth2client`
24. Installed requests via `sudo apt-get install python-requests`
25. Installed psycopg via `sudo pip install psycopg2`

### Configuration of the application
25. Used git to clone the github repository into my home folder
    -  /home/kkriek/item-catalog
26. Changed the connection strings in my application in order to use the postgres database
    instead of the sqlite database
27. Created tables in Postgres via `python database_setup.py`
28. Populated tables in Postgres with some data via `python lotsofcatalogitems.py`
29. Added the IP address of the server to the google API key
30. Changed the client_secrets.json file of the application


### Resources
- Udacity course Linux Security
- Udacity course Web Application Servers
- http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/
- http://docs.python-requests.org/en/v1.0.0/community/out-there/
- https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps
- https://www.howtoinstall.co/en/ubuntu/utopic/universe/python-oauth2client/
- http://initd.org/psycopg/docs/
- https://askubuntu.com/questions/27559/how-do-i-disable-remote-ssh-login-as-root-from-a-server

