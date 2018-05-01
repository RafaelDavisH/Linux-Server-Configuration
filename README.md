# Linux Server Configuration
Baseline installation of a Ubuntu Linux distribution on a virtual machine and prepared to host web applications, including updates installations, securing it from attack vectors and installing/configuring web and database servers.

The following instructions are written specifically for hosting a web application called [Pot Of Quotes](https://quotes.rafaeldavis.me) on an Amazon Lightsail instance.

### Server Specifics

- IP Address: _54.218.182.68_
- SSH port: _2200_
- URL: _https://quotes.rafaeldavis.me_

### INSTRUCTIONS

  **Get your server Ready**

I. Ubuntu Linux server instance on [Amazon Lightsail](https://lightsail.aws.amazon.com/).

  * Create an instance.

  <img src="assets/create_instance.png">

  * Login to the server's terminal.

  <img src="assets/servers-terminal.png">

> ---

II. Update all currently installed packages.

  `$ sudo apt-get update`

  `$ sudo apt-get upgrade`

  * Install finger package.

  `$ sudo apt-get install finger`


  * To connect to your server via your terminal you must download your default private key from the `Account page`.

  * **Recommendation**: _Create new SSH Key and name it whatever you want and can remember later._

  <img src="assets/ssh-keys.png">

  <img src="assets/ssh-keys-page.png">

  * Make sure your downloaded SSH Key is placed inside: `~/.ssh/`. The SSH Key will have the ` .pem` suffix.

> ---

III. Change the SSH port from **22** to **2200**.

  `$ sudo nano /etc/ssh/sshd_config.`

  * Change port 22 to 2200

  * Look for the line where it says `PasswordAuthentication yes` and change yes to **no**. Now all users will be forced to to log in using a key pair.

  `$ sudo service ssh restart`

  * Now you can connect via your local terminal with the following:

  `ssh -i ~/.ssh/name_you_created.pem ubuntu@XX.XXX.XXX.XX -p2200`

> ---

IV. Configure the _Uncomplicated Firewall_ (UFW) to only allow incoming connections for SSH(port 2200), HTTP(port 80), and NTP(port 123).

  `$ sudo ufw default deny incoming`

  `$ sudo ufw default allow outgoing`

  `$ sudo ufw allow 2200/tcp`

  `$ sudo ufw allow 80/tcp`

  `$ sudo ufw allow 123/udp`

  `$ sudo ufw enable.`

  * Verify the ports that are allowed and denied.

  `$ sudo ufw status`


```
      To                         Action      From
    --                         ------      ----
    2200/tcp                   ALLOW       Anywhere                  
    80/tcp                     ALLOW       Anywhere                  
    123/tcp                    ALLOW       Anywhere                  
    22                         DENY        Anywhere                  
    2200/tcp (v6)              ALLOW       Anywhere (v6)             
    80/tcp (v6)                ALLOW       Anywhere (v6)             
    123/tcp (v6)               ALLOW       Anywhere (v6)             
    22 (v6)                    DENY        Anywhere (v6)
```

V. Creating a new user account named **grader**.

  `$ sudo adduser grader`

> ---

VI. Give **grader** the permission to **sudo**.

  `$ sudo nano /etc/sudoers.d/grader`

  * Type the following inside the file:

  `grader ALL=(ALL) NOPASSWD:ALL`

  * Save file (control X, Type `y`, and press enter/return).

> ---

VII. Create an SSH key pair for **grader** using the **ssh-keygen** tool.

  * On your local terminal (not connected to your server) create a SSH key pair.

  `$ ssh-keygen -f ~/.ssh/name_you_created.rsa`

  * You will be asked to enter a passpharse.

  * `ssh-keygen` will generated two files.

  `name_you_created` and `name_you_created.pub`

  * Copy the `name_you_created.pub` content.

  `$ cat ~/.ssh/name_you_created.pub`

  * Go back to your AWS Lightsail server terminal to create `.ssh` directory and file named `authorized_keys`.

  `$ cd /home/grader`

  `$ mkdir .ssh`

  `$ nano .ssh/authorized_keys`

  * Paste the content of your `name_you_created.pub` into the `authorized_keys` file. Save the file.

  * Change the permissions for the directory and file.

  `$ chmod 700 .ssh`

  `$ chmod 644 .ssh/authorized_keys`

  * Now you can connect via your local terminal as `grader`.

  `$ ssh -i ~/.ssh/name_you_created grader@XX.XXX.XXX.XX -p2200`

  * Enforce the key-based authentication.

  `$ sudo nano /etc/ssh/sshd_config`

  * Look for the line that says `PasswordAuthentication`, make sure it has **no** not yes.

  * Restart ssh service.

  `$ sudo service ssh restart`

> ---

VIII. Configuring the local timezone to UTC.

  `$ sudo dpkg-reconfigure tzdata`

  * Select `none of the above` and then `UTC`.

> ---

XI. Installing and configuring Apache to serve a Python mod_wsgi application.

  * **If you are building your project with Python 3, you will need to install the Python 3 mod_wsgi package on your server:**

    `$ sudo apt-get install libapache2-mod-wsgi-py3`.


  `$ sudo apt-get install apache2`

  * Installing the `mod_wsgi` package (a tool that allows Apache to serve Flask applications) along with `python-dev` (a package with header files required when building Python extensions) using the following command:

  `$ sudo apt-get install libapache2-mod-wsgi python-dev`

  * Enable `mod_wsgi`.

  `$ sudo a2enmod wsgi`

> ---

X. Installing and configuring PostgreSQL:

  `sudo apt-get install postgresql`

  * Creating a new database user named `catalog` that has limited permissions to your catalog application database.

  * Postgres will create a user named `postgres`.

  `$ sudo su - postgres` and connect to database system `$ psql`.

  `~$ CREATE USER catalog WITH PASSWORD password;`

  `~$ CREATE DATABASE catalog WITH OWNER catalog;`

  `~$ \q` to exit `psql`

> ---

XI. Install `git`.

  `sudo apt-get install git`

  * Create a directory named 'catalog' inside `/var/www/`.

  `$ sudo mkdir /var/www/catalog`

  `$ cd catalog`

> ---

XII. Cloning and setting up Flask Application from the GitHub repository.

  * Copy the Flask Web App project's clone link from GitHub.

  `$ git clone [GitHub Repository link] catalog`

  * The full PATH for the app should look like this:

  `/var/www/catalog/catalog(app directory)/[GitHub Repository files]`.

> ---

XIII. Setting up the server to function correctly when visiting on a browser.

  * **Get out of the app directory one step back**

  `$ cd .. `

  * Create a `.wsgi file`.

  `$ sudo nano catalog.wsgi`

  * Include the following content inside `catalog.wsgi` file.

  ```

  activate_this = '/var/www/catalog/catalog/venv/bin/activate_this.py'
  execfile(activate_this, dict(__file__=activate_this))

  #!/usr/bin/python
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0,"/var/www/catalog/")

  from catalog import app as application
  application.secret_key = 'supersecretkey'

  ```
  --> ** If your app is based on Python2.7 you must have `#!/usr/bin/python2.7` above `import sys`.** <--

  * Move to the app directory.

  `$ cd catalog`

  * Change the `application.py` to `__init__.py`.

  `$ mv application.py __init__.py`

  * Edit `__init__.py`.

  `$ sudo nano __init__.py`

  * Change `app.run(hots='0.0.0.0', port=8000)` to `app.run()`

  * In order for Google signin to work **Authorized JavaScript Origins** and **Authorized redirect URIs** must be updated.

  * Log in into Google APIs console to edit your credentials.

  **Google signing will not work properly sometimes by using the raw IP address. To make sure it works well it's recommended to have a domain name where you can have a subdomain for your app that points to your server's IP address.**

  * Add your app domain address for **Authorized JavaScript origins.**

  _https://yourappdomain.com_

  * Add the **Authorized redirect URIs.**

  _https://yourappdomain.com/login_

  _https://yourappdomain.com/gconnect_

  _https://yourappdomain.com/oauth2callback_

  * Save file and download the `JSON` file.

  * Open the `JSON` file and copy its content.

  * Add or edit `client_secrets.json`. The file must be inside the app directory. `var/www/catalog/catalog/`.

  **Example**

  ```json

    {"web":{
  	"client_id":"XXXXXXXXXXXXXXXXXXXX",
  	"client_secret": "XXXXXXXXXXXXXXX",
  	"project_id":"XXXXXXXXXXXXXXXX",
  	"token_uri":"https://accounts.google.com/o/oauth2/token",
  	"auth_provider_x509_cert_url":"https://www.googleapis.com/oauth2/v1/certs",
  	"redirect_uris":["https://yourappdomain.com/login","https://yourappdomain.com/gconnect","https://yourappdomain.com/oauth2callback"],
  	"javascript_origins":["https://yourappdomain.com"]}}

  ```

  * You need to add `"client_secret":"XXXXXX",` to the `client_secrets.json` as shown in the above example code.

  * Edit `__init__.py` to properly find `client_secrets.json`.

  `$ sudo nano __init__.py`

  * Change the path for `client_secrets.json` in the following lines:

  ```python

  CLIENT_ID = json.loads(
  open('/var/www/catalog/catalog/client_secrets.json', 'r').read())['web']['client_id']

  ```

  ```python

  try:
      # Upgrade the authorization code into a credentials object
      oauth_flow = flow_from_clientsecrets('/var/www/catalog/catalog/client_secrets.json', scope='')
      oauth_flow.redirect_uri = 'postmessage'
      credentials = oauth_flow.step2_exchange(code)
  ```

  * The following line needs to be changed in `__init__.py` and in `database_setup_app.py`  to connect to the new database with `postgresql`.

  ```python

    engine = create_engine('postgresql://catalog:password@localhost/catalog')

  ```

  **Notice that when `catalog` user was created the word `password` was given as password. If you gave it a different password, make sure to use that instead.**


  * Install `pip` if isn't installed already.

  `$ sudo apt-get install python-pip`

  * Install and start the virtual machine.

  `$ sudo pip install virtualenv`

  `$ sudo virtualenv venv`

  `$ sudo chmod -R 777 venv`  

  `$ sudo venv/bin/activate`

  * Install the dependencies.

  `$ pip install httplib2`

  `$ pip install requests`

  `$ pip install --upgrade oauth2client`

  `$ pip install sqlalchemy`

  `$ pip install flask`

  `$ pip install psycopg2`

  * Deactivate the virtual environment.

  `$ deactivate`

  * Set up and enable a virtual host by creating a file named `catalog.conf`.

  `$ sudo nano /etc/apache2/sites-available/catalog.conf`

* Paste the following code and save the file.

```
<VirtualHost *:80>
    ServerName [www.yourappdomain.com]
    ServerAdmin [your_admin_email@yours.com]
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/catalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/catalog/catalog/static
    <Directory /var/www/catalog/catalog/static/>
        Order allow,deny
        Allow from all
        Options -Indexes
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

* Enable the virtual host.

`$ sudo a2ensite catalog`

* The following prompt will be returned.

```

Enabling site catalog.
To activate the new configuration, you need to run:
service apache2 reload

```

`$ sudo service apache2 reload`

* Disable the default Apache site.

`$ a2dissite 000-default.conf`

* The following prompt will be returned.

```

Site 000-default disabled.
To activate the new configuration, you need to run:
service apache2 reload

```

`$ sudo service apache2 reload`

### Hooray! Your App should be working by now. Enter your app domain in the browser.
