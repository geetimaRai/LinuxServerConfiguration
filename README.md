iProject 5 for Udacity Full-Stack Nanodegree 

Public IP Address: 52.42.97.13 
Project Browse Link: http://ec2-52-42-97-13.us-west-2.compute.amazonaws.com/
SSH Port: 2200

List of tasks and steps to accomplish the task:

1.	Launch the Virtual Machine- Followed the steps as mentioned in the Udacity account.

•	Downloaded private key
•	Moved the private key file into the folder ~/.ssh –
mv ~/Downloads/udacity_key.rsa ~/.ssh/
•	Changed permissions on the key – 
chmod 600 ~/.ssh/udacity_key.rsa
•	SSH’d into the VM – 
ssh -i ~/.ssh/udacity_key.rsa root@<Public-IP-Address>

2.	Create a new user named grader

•	Create a new user grader – 
sudo adduser grader
•	Entered password and name as grader
•	To confirm that the user was created successfully, I installed finger and confirmed -
apt-get install finger
finger grader
•	Output:
Login: grader                     Name: grader
Directory: /home/grader                 Shell: /bin/bash
Never logged in.
No mail.
No Plan.

3.	Give the grader the permission to sudo
•	I first entered the command below that opened a file – 
sudo visudo
•	Added the following line below the root user in "#User privilege specification" –
grader ALL=(ALL:ALL) ALL
•	Created file grader in /etc/sudoers.d – 
sudo vim /etc/sudoers.d/grader
•	Added the line below and saved –
grader ALL=(ALL:ALL) ALL
•	Created file root in /etc/sudoers.d – 
sudo vim /etc/sudoers.d/root
•	Added the line below and saved –
root ALL=(ALL:ALL) ALL

4. Update all currently installed packages

•	Ran the following commands from the prompt –
sudo apt-get update
sudo apt-get upgrade

5. Change the SSH port from 22 to 2200
•	Opened sshd_config file –
vim /etc/ssh/sshd_config 
•	Performed the following changes to the file –
Port 22 to Port 2200
PermitRootLogin without-password to PermitRootLogin no (disallow root login)
PasswordAuthentication no to PasswordAuthentication yes. (Will be changed back once the SHH login setup is complete as instructed in the video lectures.)
•	Added the following line to allow user grader to login through ssh 
AllowUsers grader
•	Restarted ssh –
sudo service ssh reload

6. Create SSH keys locally and add to server:
•	On local machine generate SSH key pair –
ssh-keygen
•	Save keygen file in the ssh directory at location Users/<Username>/.ssh/project5
•	Login with grader account using password set during user creation –
ssh -v grader@< Public-IP-Address > -p 2200 
•	You will need to enter a password that you used when you created grader.
•	Create .ssh directory in home (~) –
mkdir .ssh
•	Create a file to store the public key –
vim .ssh/authorized_keys
•	Copy the contents of the file .ssh/project5.pub on your local machine
•	On your VM, paste the above in .ssh/authorized_keys paste contents
•	Set permissions for files –
chmod 700 .ssh 
chmod 644 .ssh/authorized_keys

•	Open sshd_config file –
sudo vim /etc/ssh/sshd_config
•	Change PasswordAuthentication from yes back to no. (enter grader password when asked for grader password)

•	You should be able to login with key pair successfully –
ssh grader@52.42.97.13 -p 2200 -i ~/.ssh/project5
 
7. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
•	Enter the following command for UFW status to make sure its inactive –
sudo ufw status
•	Deny all incoming by default –
sudo ufw default deny incoming
•	Allow outgoing by default –
sudo ufw default allow outgoing
•	Allow SSH on port 2200 –
sudo ufw allow 2200/tcp
•	Allow HTTP on port 80 –
sudo ufw allow 80/tcp
•	Allow NTP on port 123 –
sudo ufw allow 123/udp
•	Save the settings and activate the firewall –
sudo ufw enable

8. Configure the local timezone to UTC
•	Run the following command –
sudo dpkg-reconfigure tzdata from prompt
•	This displays a screen with multiple options
•	Select None of the above on the first screen and select UTC on the second screen.

9. Install and configure Apache to serve a Python mod_wsgi application
•	Open the hostname file and copy the hostname–
cat /etc/hostname.
•	Add the hostname to the hosts file –
sudo vim /etc/hosts
•	Install apache2 –
sudo apt-get install apache2 
•	Visit the ip address and make sure it works.
http://<Public-IP-Address>
•	Install mod_wsgi –
sudo apt-get install libapache2-mod-wsgi
•	Configure Apache to handle requests using the WSGI module –
sudo vim /etc/apache2/sites-enabled/000-default.conf
•	Add the following to the file right before the </VirtualHost> closing line –
WSGIScriptAlias / /var/www/html/catalogapp.wsgi 
•	Restart Apache –
sudo apache2ctl restart

10. Install and configure PostgreSQL:
•	Install postgres –
sudo apt-get install postgresql
•	Install additional models –
sudo apt-get install postgresql-contrib
•	Add catalog user –
sudo adduser catalog
•	Login as postgres super user –
sudo su - postgres
•	Enter postgres –
psql
•	Create user catalog with password 'db-password' –
CREATE USER catalog WITH PASSWORD 'db-password';
•	Change role of user catalog to create database –
ALTER USER catalog CREATEDB;
•	List all users and roles to verify –
\du
•	Create new DB "itemcatalog" with owner as catalog –
CREATE DATABASE itemcatalog WITH OWNER catalog;
•	Connect to the database –
\c itemcatalog
•	Revoke all rights –
REVOKE ALL ON SCHEMA public FROM public;
•	Give access to only catalog role –
GRANT ALL ON SCHEMA public TO catalog;
•	Quit postgres –
\q
•	Exit from postgres –
exit

11. Install git, clone and setup your Catalog App project (from your GitHub repository from earlier in the Nanodegree program) so that it functions correctly when visiting your server’s IP address in a browser. Remember to set this up appropriately so that your .git directory is not publicly accessible via a browser!

•	Install git –
sudo apt-get install git
•	Create keypair to fetch the remote repository and save in /home/grader/.ssh/github –
ssh-keygen -t rsa -b 4096 -C "github catalog"  
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/github
•	Copy the contents of /home/grader/.ssh/github.pub
•	Go to your github account,find settings, and add the ssh key
•	Clone the project –
git clone git@github.com:geetimaRai/catalog.git
•	Change directory and create new directory catalog –
cd /var/www/
mkdir catalog
cd catalog
•	Move the catalog project to the current directory
mv ~/catalog .
•	Install python-dev package –
sudo apt-get install python-dev
•	Verify wsgi is enabled –
sudo a2enmod wsgi
•	Change to catalog directory and create a new file __init__.py –
cd /var/www/catalog/catalog
sudo vim __init__.py
•	Add the following to the file __init__.py
from flask import Flask
app = Flask(__name__)
@app.route("/")
def hello():
	return "Hello, World!
if __name__ == "__main__":
	app.run()
•	Open database_setup.py
sudo vim database_setup.py
•	Remove the  line with engine initialization and add the following line:
engine = create_engine('postgresql://catalog:db-password@localhost/itemcatalog')
•	Open application.py and make the same change as above
•	Copy the application.py file into the __init__.py file –
mv appliation.py __init__.py
•	Install flask and other dependencies with permission changes –
sudo apt-get install python-pip
sudo pip install virtualenv
sudo virtualenv env
sudo chmod -R 777 env
source env/bin/activate
pip install Flask
python __init__.py
deactivate
•	Configure And Enable New Virtual Host
•	Create host config file –
sudo vim /etc/apache2/sites-available/catalog.conf
•	Add the following:
<VirtualHost *:80>
  ServerName 52.42.97.13
  ServerAdmin admin@52.42.97.13
 ScriptAlias / /var/www/catalog/catalogapp.wsgi
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

•	Enable catalog –
sudo a2ensite catalog
•	Create the wsgi file –
sudo vim /var/www/catalog/catalogapp.wsgi
•	Add the following –
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from catalog import app as application
application.secret_key = 'Add your secret key'



•	Restart apache server –
sudo service apache2 restart 
•	Make .git inaccessible
•	Create file .htaccess –
sudo vim /var/www/catalog/.htaccess
•	Add the following line to the file –
RedirectMatch 404 /\.git
•	Install dependencies:
cd /var/www/catalog/catalog
source env/bin/activate
sudo pip install httplib2
sudo pip install requests
sudo pip install --upgrade oauth2client
sudo pip install sqlalchemy
sudo pip install Flask-SQLAlchemy
sudo pip install flask
sudo pip install python-psycopg2
•	Setup the database schema –
python database_setup.py 

 I encountered some issues in installing psycopg2 stack overflow post helped.
 
•	In the google developer console settings for the project, add your host name and IP address to Authorized Javascript origins. Download the new json file client_secrets.json and replace with the existing one.
•	Similarly visit the facebook developers console and add your hostname in the allowed sites section and save.
•	The application was not working initially because the client_secrets.json file could not be located. I added absolute file paths everywhere and got it working
•	Retstart apache –
sudo service apache2 restart

Resources:
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
StackOverflow/Google

