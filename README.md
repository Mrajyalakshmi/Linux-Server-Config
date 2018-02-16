# LinuxServerConfig
Udacity-FullStack-Project6

## 1 Server details:
VM IP: ec2-52-36-184-198.us-west-2.compute.amazonaws.com ( 52.36.184.198 )
Access Movies Catalog web application: http://ec2-52-36-184-198.us-west-2.compute.amazonaws.com
SSH Port : 2200

## 2 Access Server for user : `grader`
Access the server using Grader private key (provided in "Notes to Reviewer" field).
Use the following command to create a ssh session with the server:  `$ ssh -i <path-to-grader-private-key> grader@52.36.184.198 -p 2200 `

## 3 Software installation Steps

Update all currently installed packages : `$ sudo apt-get update`
Upgrade all installed packages: `$ sudo apt-get upgrade` and `$ sudo apt-get dist-upgrade`

Install software:

```
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get dist-upgrade
$ sudo apt-get install unattended-upgrades
$ sudo apt-get install git
$ sudo apt-get install apache2
$ sudo apt-get install libapache2-mod-wsgi
$ sudo apt-get install postgresql

$ sudo apt-get install python python-pip
$ sudo apt-get install libapache2-mod-wsgi python-dev
$ pip install flask
 Notice: Don't use sudo with pip.
$ sudo pip2 install packaging oauth2client redis passlib flask-httpauth
$ sudo pip2 install sqlalchemy flask-sqlalchemy psycopg2 bleach requests

```

## 4 configuration changes

### 4.1 Configure Firewall :

Run following command lines to configure UFW:

```
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
$ sudo ufw allow 2200/tcp
$ sudo ufw deny 22
$ sudo ufw allow www
$ sudo ufw allow ntp
$ sudo ufw enable
```
### 4.2  Create new user `grader` and give `sudo` permission

```
sudo adduser grader
sudo ls /etc/sudoers.d
sudo cp /etc/sudoers.d/vagrant /etc/soders.d/grader
vi /etc/sudoers.d/grader
--modify to have  : grader ALL=(ALL) NOPASSWD:ALL
```

### 4.3 Configure key based authentication for `grader`

Generate private /public key pairs on client machine:
`$ ssh-keygen`

Copy grader.pub to the server and change owner

```
$ mkdir .ssh
$ touch .ssh/authorized_keys
- copy content of grader.pub to authorized_keys
$ chmod 700 .ssh
$ chmod 644 .ssh/authorized_keys
$ sudo chown -R grader:grader /home/grader/.ssh
```

### 4.4 Configure ssh to restrict root login and login using password.

In `/etc/ssh/sshd_config` modify to have and restart service:
```
PermitRootLogin no
PasswordAuthentication no
sudo service ssh restart
```

### 4.5 Configure the local timezone to UTC
`sudo timedatectl set-timezone Etc/UTC`

### 4.6 Configure PostgreSQL

Configure postgres using below commands:
```
sudo su - postgres
psql
\password postgres
CREATE USER catalog WITH PASSWORD 'password';
CREATE DATABASE moviedb OWNER catalog;
GRANT ALL PRIVILEGES ON DATABASE moviedb to catalog;
```

### 4.7 Clone MovieCatalog project:
Run the below commands to clone the github repo:
```
cd /var/www/
sudo git clone https://github.com/Mrajyalakshmi/MovieCatalog
```
Changes to MovieCatalog:

```
@@ -24,7 +24,7 @@ DBSession = sessionmaker(bind=engine)
 session = DBSession()

 CLIENT_ID = json.loads(
-     open('client_secrets.json', 'r').read())['web']['client_id']
+     open('/var/www/moviecatalog/client_secrets.json', 'r').read())['web']['client_id']
 APPLICATION_NAME = 'Movie Catalog'


@@ -257,7 +257,7 @@ def gconnect():

     try:
         # Upgrades auth code into credentials object
-        oauth_flow = flow_from_clientsecrets('client_secrets.json', scope='')
+        oauth_flow = flow_from_clientsecrets('/var/www/moviecatalog/client_secrets.json', scope='')
         oauth_flow.redirect_uri = 'postmessage'
         credentials = oauth_flow.step2_exchange(code)
     except FlowExchangeError:
@@ -416,6 +416,5 @@ def movie_json(genre_id, movie_id):


 if __name__ == '__main__':
-    app.secret_key = 'super_secret_key'
     app.debug = True
-    app.run(host='0.0.0.0', port=8000)
+    app.run()
diff --git a/client_secrets.json b/client_secrets.json
old mode 100644
new mode 100755
index ba45b74..92b6039
--- a/client_secrets.json
+++ b/client_secrets.json
@@ -1 +1 @@
-{"web":{"client_id":"632856727534-pq164h8in1ivftr6nvnurnbrn4g8u6vf.apps.googleusercontent.com","project_id":"movie-catalog-193522","auth_uri":"https://accounts.google.com/o/oauth2/auth","token_uri":"https://accounts.google.com/o/oauth2/token","auth_provider_x509_cert_url":"https://www.googleapis.com/oauth2/v1/certs","client_secret":"SuBW1057XTkEjlmlHdG8vBvM","redirect_uris":["http://localhost:8000/genre"],"javascript_origins":["http://localhost:8000"]}}
\ No newline at end of file
+{"web":{"client_id":"632856727534-pq164h8in1ivftr6nvnurnbrn4g8u6vf.apps.googleusercontent.com","project_id":"movie-catalog-193522","auth_uri":"https://accounts.google.com/o/oauth2/auth","token_uri":"https://accounts.google.com/o/oauth2/token","auth_provider_x509_cert_url":"https://www.googleapis.com/oauth2/v1/certs","client_secret":"SuBW1057XTkEjlmlHdG8vBvM","redirect_uris":["http://ec2-52-36-184-198.us-west-2.compute.amazonaws.com"],"javascript_origins":["http://ec2-52-36-184-198.us-west-2.compute.amazonaws.com"]}}
diff --git a/database_init.py b/database_init.py
old mode 100644
new mode 100755
diff --git a/database_setup.py b/database_setup.py
old mode 100644
new mode 100755
index 39dcf75..666f807
--- a/database_setup.py
+++ b/database_setup.py
@@ -8,7 +8,7 @@ from sqlalchemy import asc, desc

 Base = declarative_base()

-engine = create_engine('sqlite:///catalog.db')
+engine = create_engine('postgresql://catalog:password@localhost/catalogdb')

 Base.metadata.bind = engine
 DBSession = sessionmaker(bind=engine)
```

Modify `000-default.conf` in `/etc/apache2/sites-enabled/`:

```
<VirtualHost *:80>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName www.example.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/moviecatalog
        WSGIScriptAlias / /var/www/moviecatalog/catalog.wsgi

        <directory /var/www/moviecatalog>
                WSGIApplicationGroup %{GLOBAL}
                Order deny,allow
                Allow from all
        </directory>


        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        # For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        #Include conf-available/serve-cgi-bin.conf
</VirtualHost>

``` 

Restart Apache Server:
`$ sudo apache2ctl restart`


## 5 Third-party resources
 https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
 
 https://www.cyberciti.biz/tips/setup-ssh-to-run-on-a-non-standard-port.html
 
 https://askubuntu.com/questions/16650/create-a-new-ssh-user-on-ubuntu-server
 
 http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/
 
 https://www.thegeekstuff.com/2010/09/change-timezone-in-linux/
 
 https://github.com/TianyangChen/
