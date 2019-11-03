# Linux Server Configration Project

- This project is an explination on how to deploy the previous project [Item Catalog](https://github.com/ali-karkazan/final-item-catalog) and setup the network and firewall settings,

I will take you throuh the project step by step 

- Create Ubuntu VM on Amazon lightsail:

	* Login to Amazon [Lightsail](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fconsole.aws.amazon.com%2Fconsole%2Fhome%3Fnc2%3Dh_ct%26src%3Dheader-signin%26state%3DhashArgs%2523%26isauthcode%3Dtrue&client_id=arn%3Aaws%3Aiam%3A%3A015428540659%3Auser%2Fhomepage&forceMobileApp=0)
	* From the main page click on create instance 
	* Select Linux/Unix, OS only and Ubuntu 16.04 LTS
	* Choose The Plan depends on your requirement 
	* Finally Click on `Create`

	Public IP adress: http://13.229.203.195/
	Link : http://ec2-13-229-203-195.ap-southeast-1.compute.amazonaws.com

- Login to your instance from your terminal using SSH:

    * From lightsail account => SSHKeys download the key file (*.PEM)
    * Rename it and save it in ~/.ssh folder in home directory 
			`sudo mv LightsailDefaultKey-ap-south-1.pem project_key.rsa` 
			`sudo mv project_key.rsa project_key.rsa ~/.ssh`
	* configure the file permition:
		`sudo chmod 600 ~/.ssh /project_key.rsa`
	* Connect to your instance using your public IP address ,my instance public IP address is 13.229.203.195
		`ssh -i ~/.ssh/project_key.rsa ubuntu@13.229.203.195`
- Update the server softwear:
	* `sudo apt-get update`
	* `sudo apt-get upgrade`

	"please note that this step might use a resnoble amount of time"

- As requirment of the project you have to change the ssh port to 2200 
	* use the following command to access the VM configration file
		`sudo nano /etc/ssh/sshd_config`
	* on the 5th line change the port to 2200, then exit and save the file.
	* reboot your ssh using the command `sudo /etc/init.d/ssh restart`


- Create new user under the name (grader) and grant him sudo access

	* use the command `sudo adduser grader` in your terminal.
	* To give `sudo` access enter 
			`sudo visudo` 
		just below `root ALL=(ALL:ALL) ALL` 
		add the following line `grader  ALL=(ALL:ALL) ALL`
	* Install the unattended-upgrades package:
		`sudo apt-get install unattended-upgrades`
	* Enable the unattended-upgrades package:
		`sudo dpkg-reconfigure -plow unattended-upgrades`
	* Change the SSH port from 22 to 2200, block root user and password authentication log in :
		Open the config file:
			`sudo nano /etc/ssh/sshd_config`
			- Change port to 2200
			- Change "PermitRootLogin from without-passwort" to "no".
			- Change "PasswordAuthentication" from "yes" to "no".
	
	* - Create SSH key pair for the user grader:

	* Open a new termial on your local machine:
		- Run the following command `ssh-keygen`
		- Save the file to `~/,ssh/key.rsa` 
		- Two files will be genrated, copy the content of "key.rsa.pub" file
			use the command: `cat ~/.ssh/key.rsa.pub` 
		- Log in to the user (grader) `su - grader`
		- Create new dirctory and call it .ssh
			`sudo mkdir .ssh`
		- Create a file and call it authorized_keys
			`sudo nano ~/.ssh/authorized_keys`
		- Paste the content  into (authorized_keys)
		- Set up the file permission:
				`sudo chmod 700 .ssh`
				`sudo chmod 644 .ssh/authorized_keys`
		- Move the folder ownership to grader:
			`sudo chown -R grader:grader /home/grader/.ssh`
		- Restart ssh service 
			`sudo service ssh restart`
		- Now you can login into the instance as grader:
			`ssh -i ~/.ssh/key.rsa grader@13.229.203.195 -p 2200` 


- Configuration of the firewall(UFW): 

	*  First `sudo ufw status` 	this command to make sure ufw is inactive 
	* if ufw status show enabled or active please deactivate by entering the following command
			into your terminal 
				`sudo ufw disable`
	* cConfigre UFW
		- `sudo ufw default deny incoming`
		- `sudo ufw default allow outgoing`
		- `sudo ufw allow ssh`
		- `sudo ufw allow 2200/tcp`
		- `sudo ufw allow www`
		- `sudo ufw allow 123/udp`
		- `sudo ufw deny 22` 

	* Activae ufw by:
		`sudo ufw enable`
	* Log out of your instance by typing `exit` or `ctrl^d`

	* Now go back to Amazon Lightsail home page 
		- from your instance menu select manage
		- from there choose networking 
		- Amend the firewall settings to be same as entered above

- Deployment of the project the Project deployment:

	* Change time zonr to UTC :
			`sudo dpkg-reconfigure tzdata`
			Choose none of the above then UTC 

	* Required softwear to be installed: 

		`sudo apt-get install apache2`
		`sudo apt-get install libapache2-mod-wsgi python-dev`
		`pip install httplib2`
		`pip install requests`
		`pip install oauth2client`
		`pip install sqlalchemy`
		`pip install flask`
		`pip install psycopg2`
		`sudo apt-get install git`

	* Enable mod_wsgi:
		`sudo a2enmod wsgi`
		Restart the web server: 
		`sudo service apache2 restart`	

	* Clone the inteded project:
		navigate to `cd /var/www/`
		create new directory
		`mkdir catalog`
		set the ownership
		`sudo chown -R grader:grader catalog`
		clone the project 
		`git clone https://github.com/ali-karkazan/final-item-catalog.git catalog`
	* Create WSGI file 
		`sudo nano /var/www/catalog/catalog.wsgi`
		Paste the following in it 

		`#!/usr/bin/python
		import sys
		import logging
		logging.basicConfig(stream=sys.stderr)
		sys.path.insert(0, "/var/www/catalog/")

		from catalog import app as application
		application.secret_key = 'super_secret_key'`

	* CD to the project directory:
		`sudo cd /var/www/catalog/catalog/`
		change application.py file name into __init__.py
			`sudo mv apllication.py __init__.py`

	* install and activate virtual inviromant:
		`sudo apt-get install python-pip`
		`sudo pip install virtualenv`
		`sudo virtualenv venv`
		`source venv/bin/activate`
		set permition
		`sudo chmod -R 777 venv`

	Note for the reviwer:

	I had to reinstall all above mentioned sofwear and libraries again using -H tag to make it work
		`sudo -H pip install requests`
		and so on for sqlalchemy etc ...

	* Changes for __init__.py file:
		app.run(0.0.0.0, port=5000) changed to app.run()

	* Change the path for client_secret.JSON file:
		to show as /var/www/catalog/catalog/client_secret.json

	* Configre apache2 sites-availabe file 
		`sudo nano /etc/apache2/sites-available/catalog.conf `
		paste the following:

		`<VirtualHost *:80>
	    ServerName 13.229.203.195
	    ServerAlias ec2-13-229-203-195.ap-southeast-1.compute.amazonaws.com
	    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
	    WSGIProcessGroup catalog
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
		</VirtualHost>`

	* Database Create and configre:
		log in to Data base:
		`http://ec2-13-229-203-195.ap-southeast-1.compute.amazonaws.com`
		`psql`
		Create a new database user named catalog that has limited permissions to your catalog 
		application database.
		`CREATE USER catalog WITH PASSWORD 'catalog';
		ALTER USER catalog CREATEDB;
		CREATE DATABASE catalog WITH OWNER catalog;`
		Connect to database $ \c catalog
		`REVOKE ALL ON SCHEMA public FROM public;
		GRANT ALL ON SCHEMA public TO catalog;`

		log ou `\q`

	* Do not allow remote connections to database:
		actually the default configration is not to allow remote connection,
		it could be verfied through:
		`sudo nano /etc/postgresql/9.5/main/pg_hba.conf`

	* Change the data base details in database_setup.py and __init__.py files 
		using `sudo nano`
		change engine into 
		`engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`

	* Last step run the database:
		`python database_setup.py`

	* Restart apache service:
		`sudo service apache2 restart`

The Application is now accessable via http://13.229.203.195/