# Project: Linux Server Configuration
Udacity Full Stack Web Developer Nanodegree
##  Project Overview
You will take a baseline installation of a Linux server and prepare it to host your web applications. You will secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.

## Server Details
Host name: http://ec2-18-185-240-96.eu-central-1.compute.amazonaws.com

Public IP: 18.185.240.96

SSH port: 2200

## Server set up on Amazon Lightsail
1. First, log in to [Lightsail](https://lightsail.aws.amazon.com/). If you don't already have an Amazon Web Services account, you'll be prompted to create one.
2. Once you're logged in, Lightsail will give you a friendly message with a robot on it, prompting you to create an instance.
A Lightsail instance is a Linux server running on a virtual machine inside an Amazon datacenter. click **Create instance**.
3. For the instance location, choose an AWS Region that is closest to your users.
![Instance Location](/screenshots/instance_location.PNG)
4. Lightsail supports a lot of different instance types. An instance image is a particular software setup, including an operating system and optionally built-in applications.

    For this project, you'll want a plain Ubuntu Linux image. There are two settings to make here. First, choose "OS Only" (rather than "Apps + OS"). Second, choose Ubuntu 18.04 as the operating system.
![Instance Image](screenshots/instance_image.PNG)
5. The instance plan controls how powerful of a server you get. It also controls how much money they want to charge you. For this project, the lowest tier of instance is just fine. And as long as you complete the project within a month and shut your instance down, the price will be zero.
![Instance Plan](/screenshots/instance_plan.PNG)
6. Every instance needs a unique hostname. You can use any name you like, as long as it doesn't have spaces or unusual characters in it. Your instance's name will be visible to you and to the project reviewer.
![Instance name](https://d17h27t6h515a5.cloudfront.net/topher/2017/February/589e43ce_screen-shot-2017-02-10-at-14.47.08/screen-shot-2017-02-10-at-14.47.08.png)
7. It may take a few minutes for your instance to start up.
![Instance pending](https://d17h27t6h515a5.cloudfront.net/topher/2017/February/589e43d2_screen-shot-2017-02-10-at-14.47.34/screen-shot-2017-02-10-at-14.47.34.png)
8. Once your instance has started up, you can log into it with SSH from your browser. When you SSH in, you'll be logged as the ubuntu user. When you want to execute commands as root, you'll need to use the sudo command to do it.

Source [Udacity](https://classroom.udacity.com/nanodegrees/nd004-ent/parts/416f7e1f-fce2-484f-a10f-331c5cb22f4a/modules/4266837a-8c70-45fd-b3fb-a139b32c45ec/lessons/046c35ef-5bd2-4b56-83ba-a8143876165e/concepts/c4cbd3f2-9adb-45d4-8eaf-b5fc89cc606e) and [serverpilot](https://serverpilot.io/docs/how-to-create-a-server-on-amazon-lightsail)

## SSH into the server
You can SSH into the server using the **'Connect using SSH'** button on your instance page. But after changing the SSH port to 2200 you wouldn't be able to do that. Instead, you will have to use a private key to SSH into the server.
1. Download the default Private Key from the SSH keys tab in your Account page.
2. Move the downloaded file into the ~/.ssh directory and rename it to **lightsail_key.rsa**.
3. Change the permission of the file using `chmod 600 ~/.ssh/lightsail_key.rsa`.
4. You can now SSH into server with `ssh -i ~/.ssh/lightsail_key.rsa ubuntu@<instance_public_ip_address>`.

## Server Setup
### 1. Create New User
#### 1.1 Create a new user named `grader`
1. Run `sudo adduser grader`.
2. Enter a password for the new user and fill in any necessary information about the user.
#### 1.2 Assigning Sudo privilege to `grader`
1. Run `sudo nano /etc/sudoers.d/grader`
2. Type `grader ALL=(ALL:ALL) ALL`, then exit and save the file by entering ^C+x, y, then enter.
#### 1.3 Create SSH key for `grader`
To create the SSH key we will use the ssh-keygen tool.

**On the local machine:**
1. Run `ssh-keygen` and enter a file name (path including), ex. path/to/key/aws_key, to save the key to.
This will create two files in the ~/.ssh directory: aws_key and aws_key.pub.
2. Run `cat ~/.ssh/aws_key.pub` and copy the contents of the file.

**Log into the server as `grader`:**

You can use `su - grader` and enter password, to log as `grader`. Or SSH into the server using the default Private Key.
1. In home directory run `mkdir .ssh`.
2. Then, `nano .ssh/authorized_keys` to create a file called authorized_keys and open it.
3. Paste contents of aws_key.pub that you copied.
4. Change permissions of the directory .ssh and file authorized_keys using  `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys`.
5. To force Key-based authentication, run `sudo nano /etc/ssh/sshd_config` and make sure 'passwordAuthentication' is set to no.
6. Since the SSHD service is currently running, and it reads it configuration file when it is initially started we need to restart the service so it runs with the new configuration, by running `sudo service ssh restart`.

From now on you can SSH into the server with keypair: `ssh -i ~/.ssh/aws_key grader@<instance_public_ip_address>` or `ssh -i ~/.ssh/lightsail_key.rsa ubuntu@<instance_public_ip_address>`

### 2. Update all currently installed packages
1. `sudo apt-get update`.
2. `sudo apt-get upgrade`.
3. Install **finger**, a utility software used to check users' status: `sudo apt-get install finger`.

### 3. Configure the local timezone to UTC
While logged in as grader:
1. run `sudo dpkg-reconfigure tzdata`
2. Then, choose 'None of the above', then UTC.

### 4. Firewall Configuration
#### 4.1 Install fail2ban
Fail2ban will be used to mitigate brute force attacks by users and bots alike.
1. Install Fail2ban: `sudo apt-get install fail2ban`.
2. Copy the default config file and create a file to safely customize the fail2ban functionality: `sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`.
3. Open the **jail.local** file and edit it: `sudo nano /etc/fail2ban/jail.local`.
     ```shell
        destemail = YOURNAME@DOMAIN  
        action = %(action_mwl)s  
        under [sshd] change port = ssh to port = 2220
    ```
4. Install sendmail package, to send the alerts to the admin user: `sudo apt-get install sendmail`.
5. `sudo service fail2ban restart`

#### 4.2 Configure Amazon Instance firewall
1. Visit your instance page. Then, click on the **Networking** tab.
2. In the **Firewall** section click on the "**Add another**" link.
3. Add new rules with the following values:

    | Application  | Protocol | Port range |
    | ------------- | ------------- | ------------- |
    | Custom  | UDP  | 123  |
    | Custom  | TCP  | 2200  |

#### 4.3 Modify sshd_config
This modification is needed to change the SSH port from 22 to 2200.
1. Run, `sudo nano /etc/ssh/sshd_config`.
2. Add `Port 2200` below `Port 22` (don't delete port 22 until 2200 is working).
3. To disable ssh login for the root user, find the PermitRootLogin line and edit it to no.
4. Then, write the file and exit (ctrl+o, enter, ctrl+x).
5. Restart the SSH service: `sudo service ssh restart`

#### 4.4 Configure the Uncomplicated Firewall (UFW)
Next, we will configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
1. Check the status of UFW.  
   `sudo ufw status verbose`
2. Deny any incoming traffic.  
   `sudo ufw default deny incoming`
3. Enable outgoing traffic.  
   `sudo ufw default allow outgoing`
4. Allow incoming TCP packets on port 2200 (SSH).  
   `sudo ufw allow 2200/tcp`
5. Allow incoming TCP packets on port 80 (HTTP).  
   `sudo ufw allow 80/tcp`
6. Allow incoming UDP packets on port 123 (NTP).  
   `sudo ufw allow 123/udp`
7. Allow ssh.  
   `sudo ufw allow ssh`
8. Enable UFW.  
   `sudo ufw enable`
9. Check the status and all current rules of the UFW.  
   `sudo ufw status`

Finally, reboot the server by clicking on the **Reboot** button from the aws website.
At this point you should be able to log in on port 2200 from a local terminal using:
`ssh -i .ssh/<your_private_key> <your_username>@<your_public_IP> -p 2200`
if this works, you are ready to go back to the previous steps and remove port 22 from sshd_config and from AWS firewall settings.

### 5. Prepare to deploy the project
#### 5.1 Install Apache package
1. First, we need to install and configure Apache to serve a Python mod_wsgi application: `sudo apt-get install apache2`.
2. Open a browser and type your public IP address, you should see the following page:
![Apache Homepage](/screenshots/apache.PNG)

#### 5.2 Install mod_wsgi package
1. Since the project was built with Python 3, we need to install the Python 3 mod_wsgi package on the server: `sudo apt-get install libapache2-mod-wsgi-py3`
2. Restart the Apache server for mod_wsgi to load:
`sudo service apache2 restart`

#### 5.3 Install Git
1. `sudo apt-get install git`.
2. Configure your username using: `git config --global user.name <username>`.
3. Configure your email: `git config --global user.email <email>`.

#### 5.4 Clone the Catalog app from Github
1. First navigate to /var/www/ using: `cd /var/www`.
2. Then, make a directory called **catalog** that will contain the app: `sudo mkdir catalog`.
3. Change ownership for the catalog folder: `sudo chown -R grader:grader catalog`.
4. Navigate to the catalog folder: `cd /catalog` and clone the catalog repository from Github: `git clone https://github.com/YOUR_GITHUB_NAME/YOUR_REPOSITORY.git catalog`.

#### 5.5 Create catalog.wsgi file
1. Navigate to the /var/www/catalog file: `cd /var/www/catalog`.
2. Create a new file called **catalog.wsgi** to serve the application over the mod_wsgi. That file should look like this
```python
    #!/usr/bin/env python3
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0, "/var/www/catalog/")

    from catalog import app as application
    application.secret_key = 'your_secret_key'
```
3. Restart Apache: `sudo service apache2 restart`or `sudo apache2ctl restart`

**Note:** the .git folder will be inaccessible from the web without any particular setting. The only directory that can be listed in the browser will be the static folder.

#### 5.6 Install Pip
Pip is a tool used to install Python packages.
1. Before installing pip, you will need to add a few prerequisites that will help you in setting up your virtual space. Run the following command in order to install the build-essential, libssl-dev, libffi-dev and python-dev packages to your system: `sudo apt-get install build-essential libssl-dev libffi-dev python-dev`.
3. Next, run the following commands to install the latest pip3 package: `sudo apt-get update`, `sudo apt install python3-pip`.
    * Now that pip3 is installed on your system, you can install any pip package by using the following command syntax:
`pip3 install [package-name]`

#### 5.7 Create Virtual Environment
1. In order to create the virtual environment, you need the Python3-venv package installed on your system: `sudo apt install -y python3-venv`.
2. Navigate to /var/www/catalog: `cd /var/www/catalog`. Then, run `python3 -m venv venv` to create a virtual environment called venv.
3. Change the permissions of the venv folder: `sudo chmod -R 777 venv`
3. To activate the virtual environment run `source venv/bin/activate`.

#### 5.8 Install required packages
1. Navigate to the /var/www/catalog/catalog folder: `cd catalog`
2. Move the requiremnet file to the /var/www/catalog folder: `mv requirements.txt ..` (This step is not required).
3. Navigate back to the /var/www/catalog folder: `cd ..`. Then, run `pip3 install -r requirements.txt` to download all the dependencies.
4. Deactivate the virtual environment: `deactivate`

#### 5.9 Configure the Virtual Host
1. Run `sudo nano /etc/apache2/sites-available/catalog.conf` to create a virtual host and edit it.
2. Paste in the following lines of code and change names and addresses:
    ```shell
    <VirtualHost *:80>
        ServerName <YOUR_PUBLIC_IP_ADDRESS> e.g. 18.185.240.96
        ServerAlias <YOUR_HOST_NAME> e.g. ec2-18-185-240-96.eu-central-1.compute.amazonaws.com
        ServerAdmin admin@<YOUR_PUBLIC_IP_ADDRESS> e.g. admin@18.185.240.96
        WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python3.6/site-packages
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
    </VirtualHost>
    ```

* **Note:** The WSGIDaemonProcess line specifies what Python to use. In this case, we are explicitly saying to use the virtual environment and its packages to run the application.
* **Note:** To know what is your hostname you can use this [website]( http://www.hcidata.info/host2ip.cgi).
3. Enable the virtual host: `sudo a2ensite catalog`.
4. Restart Apache: `sudo service apache2 restart` or `sudo apache2ctl restart`.

#### 5.10 Disable the default Apache site
1. Disable the default Apache site: `sudo a2dissite 000-default.conf`
2. Reload Apache: `sudo service apache2 reload`

#### 5.11 Install and Configure PostgreSQL
1. Install PostgreSQL: `sudo apt-get install postgresql`.
    * **Note:** You might need to run `sudo apt-get install python-psycopg2`, `sudo apt-get install libpq-dev`, `pip install psycopg2`
2. PostgreSQL should not allow remote connections. Open the /etc/postgresql/10/main/pg_hba.conf file, and edit it, if necessary, to make it look like this:
    ```shell
        local   all             postgres                                peer
        local   all             all                                     peer
        host    all             all             127.0.0.1/32            md5
        host    all             all             ::1/128                 md5
    ```
3. Postgres automatically creates a new user, called **'postgres'**, during its installation. To switch to the postgres user: `sudo su - postgres`
4. Next, connect to the database system with  `psql`.
5. Create a new user called **'catalog'** with a password: `CREATE USER catalog WITH PASSWORD 'YOUR_PASSWORD';`
6. Assign the catalog user the **CREATEDB capability**: `ALTER USER catalog CREATEDB;`
7. List current roles: `\du`.
8. Create the **'catalog'** database owned by catalog user: `CREATE DATABASE catalog WITH OWNER catalog;`.
9. Connect to the database: `\c catalog`.
10. Revoke all rights: `REVOKE ALL ON SCHEMA public FROM public;`.
11. Grant only permission to the catalog role: `GRANT ALL ON SCHEMA public TO catalog;`.
12. Log out from PostgreSQL: `\q`.
13. Then return to the grader user: `exit` or CTRL+d.

#### 5.12 Required Modifications to the Project's Files
1. Navigate to the /var/www/catalog/catalog folder: `cd /var/www/catalog/catalog`.
2. Rename project\.py file to \__init\__.py: `mv project.py __init__.py`.
3. Run: `sudo nano __init__.py` and change the following lines:
    * This modification changes the reference to the database.
         ```python
            #engine = create_engine('sqlite:///catalogwithuserdb.db')

            engine = create_engine('postgresql://catalog:YOUR_PASSWORD@localhost/catalog')
        ```
    * This modification is required since wgsi starts searching for files from /var/www.
        ```python
            #CLIENT_ID = json.loads(open('client_secrets.json', 'r').read())['web']['client_id']

            CLIENT_ID = json.loads(open('/var/www/catalog/catalog/client_secrets.json', 'r').read())['web']['client_id']
        ```
        ```python
            #oauth_flow = flow_from_clientsecrets('client_secrets.json', scope='')

            oauth_flow = flow_from_clientsecrets('/var/www/catalog/catalog/client_secrets.json', scope='')
        ```
         ```python
            #app_id = json.loads(open('fb_client_secret.json', 'r').read())['web']['app_id']
            #app_secret = json.loads(open('fb_client_secret.json', 'r').read())['web']['app_secret']

             app_id = json.loads(open('/var/www/catalog/catalog/fb_client_secret.json', 'r').read())['web']['app_id']
            app_secret = json.loads(open('/var/www/catalog/catalog/fb_client_secret.json', 'r').read())['web']['app_secret']
        ```
    * This changes the port to 80.
        ```python
            #if __name__ == '__main__':
            #app.secret_key = 'super_secret_key'
            #app.debug = True
            #app.run(host='0.0.0.0', port=5000, threaded=False)

            if __name__ == '__main__':
            app.secret_key = 'super_secret_key'
            app.debug = True
            app.run(host='0.0.0.0', port=80, threaded=False)
        ```
4. Next, run: `sudo nano database_setup.py` and change the following lines:
    * Change reference to the database.
        ```python
            #engine = create_engine('sqlite:///catalogwithuserdb.db')
            engine = create_engine('postgresql://catalog:YOUR_PASSWORD@localhost/catalog')
        ```
5. Create postgreSQL database schema: `python3 database_setup.py`

Finally, run `sudo apache2ctl restart`, `sudo service apache2 restart`, `sudo systemctl reload apache2`.  

#### 5.13 Update OAuth Information for Google and Facebook Logins
**Google OAuth**
1. First, log in to your Google account. Then visit [Google Developers Console](https://console.developers.google.com/).
2. Choose credentials from the menu on the left.
3. From the **OAuth 2.0 client IDs** section, click **Item Catalog**.
4. In the **Authorized JavaScript origins** section add http://ec2-18-185-240-96.eu-central-1.compute.amazonaws.com.
5. Next, add http://ec2-18-185-240-96.eu-central-1.compute.amazonaws.com/gconnect to the **Authorized redirect URIs**.
    **Note:** You may be required to add this domain to the **Authorized domains**.
6. Download the OAuth 2.0 client ID and save it as **"client_secrets.json"**.
7. Copy the content of the **"client_secrets.json"** file.
8. Next, navigate to the /var/www/catalog/catalog folder: `cd /var/www/catalog/catalog`.
9. Run, `touch client_secrets.json` to create the **"client_secrets.json"** file. Then, `nano client_secrets.json` and paste the contents copied in step (7).

**Facebook OAuth**
1. First, log in to your Facebook account. Then navigate to your [Apps page]( https://developers.facebook.com/apps/).
2. Click on your App, go to Settings > Basics and change **Site URL** to http://ec2-18-185-240-96.eu-central-1.compute.amazonaws.com/
3. Before making your app public, so others can log in as well, fill in a contact email address in the respective field and add a link to your privacy policy in the **Privacy Policy URL** field. Then, click **"Save Changes"**.
4. Next, navigate to the /var/www/catalog/catalog folder: `cd /var/www/catalog/catalog`.
5. Run, `touch fb_client_secret.json` to create the **"fb_client_secret.json"** file. Then, run `nano fb_client_secret.json` and paste the following content.
      ```json
         {
            "web" : {
                "app_id": "YOUR_APP_ID",
                "app_secret" : "YOUR_APP_SECRET"
            }
        }
     ```
Finally, run `sudo apache2ctl restart`, `sudo service apache2 restart`, `sudo systemctl reload apache2`.
### 6. Restart Apache to launch the app
`sudo service apache2 restart`
## Issues
1. **Command 'pip' not found**  
    First, run `sudo apt install python3-pip`. Then, ` sudo apt-get install build-essential libssl-dev libffi-dev python-dev`.
    **Note:** You might also need to run these commands `sudo apt-get install python-psycopg2` `sudo apt-get install libpq-dev`
2. **No module named 'database_setup', No module named 'forms'**  
    To solve this error change the following lines:
    ```python
        #from database_setup import Base, User, Cuisine, Recipe
        #from forms import CuisineForm, DeleteForm, RecipeForm

        from catalog.database_setup import Base, User, Cuisine, Recipe
        from catalog.forms import CuisineForm, DeleteForm, RecipeForm
    ```
3. **No module named 'sqlalchemy', No module named 'PACKAGE_NAME'**:
    * If this type of error occured, then you may need to re-install the package outside the venv. Sometimes there are some minimal compatibility issues between venv and some module installation.
    `sudo pip install sqlalchemy` OR `pip3 install sqlalchemy` OR `sudo apt-get install python-<missing package>`
    * Another solution is to reinstalling libapache2-mod-wsgi-py3
    `sudo apt-get install libapache2-mod-wsgi-py3`. Then, reinstalling SQLalchemy `pip3 uninstall sqlalchemy` OR `sudo pip install sqlalchemy`. Finally, reload and reset apache `sudo a2ensite catalog`, `service apache2 reload`, `service apache2 restart`.

    **Note:** Generally, after any change in the files of the application, wgsi, or virtual host you need to restart apache.

4. **Error : (InvalidClientSecretsError: ('Error opening file', 'client_secrets.json', 'No such file or directory', 2)**
   * Check that the Python version in the server (`python -V` OR `python3 -V`) and the version used in the sheebang line in the application's files are the same.
   * Make sure that you did step 5.12-3.
 5. **Connection refused**  
     If this error occurred that means that you did not configure the ports correctly. It is better to create a new instance and follow the instructions.

For any of the wsgi errors make sure that you have one version of wsgi and it is the one compatible with the version of python used in the application. To remove wsgi used with python2 run `sudo apt-get remove libapache2-mod-python libapache2-mod-wsgi`

## Helpful commands
1. ` sudo tail -20 /var/log/apache2/error.log`  
    Shows the last 20 errors occurred while running apache.
2. `sudo apt list --installed`  
    Lists all the installed packages.

## References
* [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
* [Connecting to Your Linux Instance Using SSH
](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html)
* [Udacity Knowledge](https://knowledge.udacity.com/questions/19521)
* [Deploying python Flask web app on Amazon Lightsail
](https://umar-yusuf.blogspot.com/2018/02/deploying-python-flask-web-app-on.html)
* [How to Create a Server on Amazon Lightsail
](https://serverpilot.io/docs/how-to-create-a-server-on-amazon-lightsail)
* [How to view Apache log files
](https://www.a2hosting.com/kb/developer-corner/apache-web-server/viewing-apache-log-files)
* Github Repositories
    * [iliketomatoes/linux_server_configuration](https://github.com/iliketomatoes/linux_server_configuration)
    * [stueken/FSND-P5_Linux-Server-Configuration](https://github.com/stueken/FSND-P5_Linux-Server-Configuration)
    * [garaujo23/Linux-Server-Configuration](https://github.com/garaujo23/Linux-Server-Configuration)
    * [Wynand91/linux-server-configuration](https://github.com/Wynand91/linux-server-configuration)
* [The Best Way to Restart an Apache Web Server](https://www.lifewire.com/restart-apache-web-server-3464025)
* [List installed packages on Ubuntu 18.04 Bionic Beaver Linux](https://linuxconfig.org/list-installed-packages-on-ubuntu-18-04-bionic-beaver-linux)
* [Minimal Apache configuration for deploying a flask app (Ubuntu 18.04)](https://www.codementor.io/abhishake/minimal-apache-configuration-for-deploying-a-flask-app-ubuntu-18-04-phu50a7ft)
* [Deploy Flask App to Digital Ocean with Python3](http://leonwang.me/post/deploy-flask)
* [How to upgrade to python 3.7 on ubuntu 18.10](https://jcutrer.com/linux/upgrade-python37-ubuntu1810)
* [Install Python 3.6 on Ubuntu](https://www.pytorials.com/install-python36-on-ubuntu/)
