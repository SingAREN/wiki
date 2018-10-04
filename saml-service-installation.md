SAML Services is SAML2.0 Metadata Aggregator created by the Australian Access Federation (AAF). Used in conjunction with the AAF Discovery Service.


## Update System and Install Prerequisite Software

1. Refresh yum package metadata and update the system.

	```
	# yum makecache
	# yum update
	```

1. Install prerequisite software

	```
	# yum install "@Development Tools"
	# yum install git readline-devel zlib-devel openssl-devel ImageMagick-devel cmake
	```

## Setup Ruby Environment


1. Set the default Ruby environment root path (`RBENV_ROOT`)

	```
	# sh -c 'echo RBENV_ROOT=/opt/aaf-shared/rbenv >> /etc/environment'
	```

1. Create the Ruby environment root path

	```
	# mkdir /opt/aaf-shared
	```

1. Clone rbenv and ruby builds into root path

	```
	# git clone https://github.com/sstephenson/rbenv.git  /opt/aaf-shared/rbenv
	# git clone https://github.com/sstephenson/ruby-build.git /opt/aaf-shared/rbenv/plugins/ruby-build
	```


1. Install Ruby version 2.4.2 and set it as the global version

	```
	# /opt/aaf-shared/rbenv/bin/rbenv install 2.4.2
	# /opt/aaf-shared/rbenv/bin/rbenv global 2.4.2
	# sh -c 'echo 2.4.2 > /opt/aaf-shared/rbenv/version'
	```

1. Install Bundler Gem from RubyGems which will be used to install other Ruby dependencies later on.
	
	```
	#/opt/aaf-shared/rbenv/bin/rbenv exec gem install bundler
	```

1. Install Rugged Gem which is required by SAML Services.
	
	```
	# /opt/aaf-shared/rbenv/versions/2.4.2/bin/gem install rugged
	```

1. Rebuild Ruby shims and link Ruby executable into `/usr/bin`.
	
	```
	# /opt/aaf-shared/rbenv/bin/rbenv rehash
	# ln -s /opt/aaf-shared/rbenv/shims/ruby /usr/bin/ruby
	```

1. Download the AAF Ruby Scripts into `/tmp/` directory

	```
	$ curl ruby-scripts.zip
	```

1.	Unzip AAF Ruby Scripts

	```
	# unzip ruby-scripts.zip
	```

1. Copy `god-init.sh`, `unicorn-init.sh` and `install-bundle.rb` to `/opt/aaf-shared/rbenv/bin/` and `rbenv.sh` to `/etc/profile.d/`.

	```
	# cp /tmp/god-init.sh /tmp/unicorn-init.sh /tmp/install-bundle.rb /opt/aaf-shared/rbenv/bin/
	# cp /tmp/rbenv.sh /etc/profile.d/
	```

1. Clean scripts and package from download directory

	```
	# rm /tmp/god-init.sh /tmp/unicorn-init.sh /tmp/install-bundle.rb /tmp/rbenv.sh /tmp/ruby-scripts.zip
	```

1. Set ownership and group for the AAF Ruby scripts.

	```
	# chown root.wheel /opt/aaf-shared/rbenv/bin/god-init.sh
	# chown root.wheel /opt/aaf-shared/rbenv/bin/unicorn-init.sh
	# chown root.wheel /opt/aaf-shared/rbenv/bin/install-bundle.rb
	# chown root.wheel /etc/profile.d/rbenv.sh
	```

1.  Set permissions for the AAF Ruby cripts.

	```
	# chmod 0750 /opt/aaf-shared/rbenv/bin/god-init.sh
	# chmod 0750 /opt/aaf-shared/rbenv/bin/unicorn-init.sh
	# chmod 0750 /opt/aaf-shared/rbenv/bin/install-bundle.rb
	# chmod 0755 /etc/profile.d/rbenv.sh
	```

Ruby and RBENV is now setup.

## Install Redis
1. Setup EPEL Repository in yum.

	```
	# yum install epel-release
	```

1. Install Redis, an in-memory key-value database used by the AAF SAML Services and Discovery Service

	```
	# yum install redis
	```

1. Start Redis and enable it at start-up

	```
	# systemctl start redis
	# systemctl enable redis
	```

1. Ensure Redis is running by entering the Redis console and typing in ping. 

	```
	$ redis-cli
	127.0.0.1:6379> ping
	```
	Expected Output:
	```
	PONG
	```

The Redis service is now up and running.


## Install SAML Services
1. Create a SAML user with home directory as `/opt/saml`.

	```
	# useradd saml –m –d /opt/saml
	```

1. Install prerequisite software and dependencies.
	
	```
	# yum install httpd mod_ssl git mysql-devel
	```

1. Clone SAML Services from the AAF Repository.

	```
	# git clone https://github.com/ausaccessfed/saml-service.git /opt/saml/repository
	```

1. Create directories
		
	```
	# mkdir /opt/saml/keypairs
	# mkdir /opt/saml/scripts
	```

	1. Set directory protection
	
		```
		# chmod 0750 /opt/saml/keypairs
		# chmod 0750 /opt/saml/scripts
		# chmod 0750 /opt/saml/repository
		```
	
	1. Set directory ownership
		```
		# chown root.saml /opt/saml/keypairs
		# chown root.saml /opt/saml/scripts
		# chown root.saml /opt/saml/repository
		```

1. Create write-able directories in SAML Repository
	```
	# mkdir /opt/saml/repository/tmp
	# mkdir /opt/saml/repository/tmp/pids
	# mkdir /opt/saml/repository/vendor
	# mkdir /opt/saml/repository/vendor/torba
	```

	1. Set directory protection

		```
		# chmod 0700 /opt/saml/repository/schema
		# chmod 0700 /opt/saml/repository/tmp
		# chmod 0700 /opt/saml/repository/tmp/logs
		# chmod 0700 /opt/saml/repository/tmp/pids
		# chmod 0700 /opt/saml/repository/vendor
		# chmod 700 /opt/saml/repository/vendor/torba
		```

	1. Set directory ownership

		```
		# chown saml.saml /opt/saml/repository/schema
		# chown saml.saml /opt/saml/repository/tmp
        # chown saml.saml /opt/saml/repository/tmp/logs
		# chown saml.saml /opt/saml/repository/tmp/pids
		# chown saml.saml /opt/saml/repository/vendor
		# chown saml.saml /opt/saml/repository/vendor/torba
		```

1. Create log directories

	```
	# mkdir /var/log/aaf
	# mkdir /var/log/aaf/saml
	# mkdir /var/log/aaf/saml/application
	```

	1. Set directory protection
	
	```
	# chmod 0750 /var/log/aaf
	# chmod 0700 /var/log/aaf/saml
	# chmod 0700 /var/log/aaf/saml/application
	```
	
	1. Set ownership and group

	```
	# chown saml.saml /var/log/aaf/saml
	# chown saml.saml /var/log/aaf/saml/application
	```

1. Create symbolic link from SAML Service log to log directory

	```
	# ln -s /var/log/aaf/saml/application /opt/saml/repository/log
	```

1. Install Ruby Gem dependencies

	```
	# cd /opt/saml/repository
	# /opt/aaf-shared/rbenv/bin/rbenv exec ruby \
	    /opt/aaf-shared/rbenv/bin/install-bundle.rb
	```

1. Create Passwords to be used by SAML Services
		
	```
	# mkdir /opt/saml/keypairs/passwords
	# openssl rand -base64 32 > /opt/saml/keypairs/passwords/saml_app_password
	# openssl rand -base64 128 | tr -d '\n' > /opt/saml/keypairs/passwords/saml_secret_key_base
	# openssl rand -base64 64 > /opt/saml/keypairs/passwords/fr_export_api_secret
	```

1. Install configuration files

	```
	# echo "production:
	  metadata:
	    negative_cache_ttl: 600" > /opt/saml/repository/config/saml_service.yml

	# echo "---
	puma:
	  bind: tcp://0.0.0.0:8080
	  stdout: tmp/logs/stdout.log
	  stderr: tmp/logs/stderr.log" > /opt/saml/repository/config/deploy.yml

    # echo "---
	production:
	  secret_key_base: $(cat /opt/saml/keypairs/passwords/saml_secret_key_base)" \
	> /opt/saml/repository/config/secrets.yml
	```

1. Set file permissions of configuration files

	```
	# chmod 0640 /opt/saml/repository/config/saml_service.yml
	# chmod 0640 /opt/saml/repository/config/deploy.yml
	# chmod 0640 /opt/saml/repository/config/secrets.yml
	```

1. Set ownership of configuration files

	```
	# chown root.saml /opt/saml/repository/config/saml_service.yml
	# chown root.saml /opt/saml/repository/config/deploy.yml
	# chown root.saml /opt/saml/repository/config/secrets.yml
	```

1. Install Certificates and keys
	
	The following keys and certificates need to be created and stored in the directory **/opt/saml/keypairs/**

	- **signing.crt** and **signing.key**. These are the certificate and key currently used to sign your existing Federation Metadata.
	-	**apache.crt**, **apache.key** and **intermediate.crt**. These are the certificates and keys for the web server. You need to obtain these from your certificate vendor.
	

14. Install Certificates for external metadata sources

	Add certificates for any external metadata sources that will feed into the SAML service, e.g. edugain. These will be held in the directory **/opt/saml/keypairs/external-metadata**. File names are up to you.

1. Setup the sync.sh and console.sh scripts in ```/opt/saml/scripts```.
   
	Replace [SAML_APP_PASSWORD] with contents of ```/opt/saml/keypairs/passwords/saml_app_password``` and [KEY] with contents of ```/opt/saml/keypairs/passwords/saml_secret_key_base```/
   
	1. ```sync.sh```
      
		````
		#!/bin/sh
		export RBENV_ROOT=/opt/aaf-shared/rbenv
		export RAILS_ENV=production
		export SAML_DB_HOST=127.0.0.1
		export SAML_DB_PORT=3306
		export SAML_DB_NAME="saml_production"
		export SAML_DB_USERNAME=saml_app
		export SAML_DB_PASSWORD="[SAML_APP_PASSWORD]"
		export SECRET_KEY_BASE="[KEY]"
		export TORBA_HOME_PATH=vendor/torba
		export TORBA_CACHE_PATH=tmp/cache/assets/torba
		
		cd /opt/saml/repository
		$RBENV_ROOT/bin/rbenv exec ruby bin/sync.rb $1
		 ````

	1. ```console.sh```
       
       Use same environment variables as sync.sh

		```
		#!/bin/sh
		export RBENV_ROOT=/opt/aaf-shared/rbenv
		export RAILS_ENV=production
		export SAML_DB_HOST=127.0.0.1
		export SAML_DB_PORT=3306
		export SAML_DB_NAME="saml_production"
		export SAML_DB_USERNAME=saml_app
		export SAML_DB_PASSWORD="[SAML_APP_PASSWORD]"
		export SECRET_KEY_BASE="[KEY]"
		export TORBA_HOME_PATH=vendor/torba
 		export TORBA_CACHE_PATH=tmp/cache/assets/torba

		/opt/saml/repository/bin/rails c
		````

1. Create `saml.service` service file

	Create the file ```/lib/systemd/system/saml.service```. Update the [SAML_APP_PASSWORD] and [KEY] before installing the file.

	````
	[Unit]
	Description=AAF SAML Service
	After=syslog.target network.target haproxy.target redis.target
	
	[Service]
	WorkingDirectory=/opt/saml/repository
	SyslogIdentifier=saml
	User=saml
	Environment='RBENV_ROOT=/opt/aaf-shared/rbenv' \
	  'RAILS_ENV=production' \
	  'SAML_DB_HOST=127.0.0.1' \
	  'SAML_DB_PORT=3306' \
	  'SAML_DB_NAME=saml_production' \
	  'SAML_DB_USERNAME=saml_app' \
	  'SAML_DB_PASSWORD=[SAML_APP_PASSWORD]' \
	  'SECRET_KEY_BASE=[KEY]' \
	  'TORBA_HOME_PATH=vendor/torba' \
	  'TORBA_CACHE_PATH=tmp/cache/assets/torba' \
	  'run_rake=/opt/aaf-shared/rbenv/bin/rbenv exec bundle exec rake' \
	
	ExecStartPre=/usr/bin/env $run_rake db:migrate xsd:all
	ExecStart=/usr/bin/env /opt/aaf-shared/rbenv/bin/rbenv exec bundle exec  puma -C config/puma.rb
	ExecReload=/usr/bin/env $run_rake db:migrate xsd:all
	ExecReload=/bin/kill -s USR2 $MAINPID
	TimeoutSec=300
	Restart=always
	
	[Install]
	WantedBy=multi-user.target
	````

1. Reload systemctl daemon as a new script has been added.
	
	```
	# systemctl daemon-reload
	``` 

---
## Setup SAML Service database

1. Install MariaDB

	The database must be running version MariaDB 10.0.x and not version 5.5.x. To install a local version of MariaDB you need to create the ```/etc/yum.repos.d/mariadb.repo``` file with the following content.

	```
	[mariadb]
	name = MariaDB
	baseurl = http://yum.mariadb.org/10.0/centos7-amd64
	gpgkey = https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
	gpgcheck = 1
	```

1. Ensure the system will use the new repo file
	
	```		
	# yum clean all
	```

1. Install the latest version of MariaDB

	```
	# yum install MariaDB-server
	```
	This installs both the server and client components.

1. Start and enable the database.

	```
	# systemctl enable mysql
	# systemctl start mysql
	```

1. Secure the database

	```
	# /usr/bin/mysql_secure_installation
	```

1. Access the MariaDB database as `root` user using the password set at the previous step.

	```
	$ mysql -u root -p
	```

1. Create the ```saml_app``` database user and provide user with access to the ```saml_production``` database. Change ```[SAML_APP_PASSWORD]``` to the contents of ```/opt/saml/keypairs/passwords/saml_app_password```

		> CREATE USER 'saml_app' IDENTIFIED BY '[SAML_APP_PASSWORD]';
		> GRANT USAGE ON *.* TO 'saml_app'@'localhost' IDENTIFIED BY '[SAML_APP_PASSWORD]';
		> GRANT ALL ON saml_production.* to 'saml_app'@'localhost';
		> FLUSH PRIVILEGES;
		> EXIT

1. Create the database and tables by using the following rake command. This relies on a number environment variables to be set to allow it to access the database server.

	1. Set environment variables. Change ```[SAML_APP_PASSWORD]``` to the password set at previous step
			
		```
		export RBENV_ROOT=/opt/aaf-shared/rbenv
		export RAILS_ENV=production
		export SAML_DB_HOST=127.0.0.1
		export SAML_DB_PORT=3306
		export SAML_DB_NAME="saml_production"
		export SAML_DB_USERNAME=saml_app
		export SAML_DB_PASSWORD="[SAML_APP_PASSWORD]"
		```

	1. The following commands must be run from the directory */opt/saml/repository*
	
		```	
		$ cd /opt/saml/repository
		```
	
	1. Create the database

		```
		$ bin/rake db:create
		```

	1. Create the tables in the database

		```
		$ bin/rake db:schema:load
		```
		An empty database should now be ready to load data into.

1. Start the SAML Service

		$ systemctl enable saml
		$ systemctl start saml

## Setup Metadata Sources

1. Set up the Federation Registry as your first metadata source.

	1. 	Obtain the FR API key from the file **/opt/federationregistry/application/config/fr-config.groovy**. 
	
	1.  Search for the value aaf.fr.export.key. You will find a long string that you need to copy into the following command. Replace [SECRET] with the key value. 
	
		Replace [TAG] a short name for the metadata source, for instance ‘sgaf’.
	
			$ /opt/aaf-shared/rbenv/bin/rbenv exec ruby bin/configure.rb fr_source \
			    --hostname manager.[domain-name] \
			    --secret [SECRET] \
			    --registration-authority https://[domain-name] \
	            --registration-policy  https://[domain.name]/policy/ \
				--lang en \
				--source_tag [TAG]
	
	3. Load Federation Registry metadata into SAML Service
	
		You are now ready to load the federation metadata from your Federation Registry. This is done by running the **sync.sh** script passing it the TAG you chose in the previous step:
	
			$ /opt/saml/scripts/sync.sh [TAG]
	
	1. Add metadata signing key
	
		Next load the metadata signing key into the database. You will require access to both the public and private halves of the signing key.
		
			$ cd /opt/saml/repository
			$ /opt/aaf-shared/rbenv/bin/rbenv exec ruby bin/configure.rb keypair \
			  --cert /opt/saml/keypairs/signing.crt \
	          --key /opt/saml/keypairs/signing.key

1. Setup eduGAIN Feed
	
	1. Obtain eduGAIN metadata signing certificate

			$ cd /opt/saml/keypairs
			$ wget https://technical.edugain.org/mds-2014.cer
			$ mv mds-2014.cer edugain-mds-2014.cer

	1. Add eduGAIN configuration to SAML Service
	
			$ /opt/aaf-shared/rbenv/bin/rbenv exec ruby bin/configure.rb raw_entity_source  \
			  --rank 102 \
			  --url http://mds.edugain.org/ \
			  --cert /opt/saml/keypairs/edugain-mds-2014.cer \
			  --source_tag edugain

	1. Load eduGAIN metadata into SAML Service

			$ /opt/saml/scripts/sync.sh edugain

		This may take some time due to the size of the metadata file.

1. Create Metadata Instances

	Now we add a Metadata instance to the database. This is used in the creation of metadata files. For now we only create a tag  for federation metadata and ignore the edugain metadata.

		/opt/aaf-shared/rbenv/bin/rbenv exec ruby bin/configure.rb md_instance \
		--cert /opt/saml/keypairs/signing.crt --name  https://FQDN/of/the/metadata.xml \
		--identifier [TAG] --tag [TAG] --hash sha256 --publisher https://[domain-name] \
		--lang en

1. Verify that you can extract metadata from the SAML service. Replace ```[TAG]``` with metadata instance tag. 

	```
	curl --insecure --connect-timeout 500 -H 'Accept: application/samlmetadata+xml' http://localhost:8081/mdq/[TAG]/entities
	```


## Cron Jobs

A CRON job is required to regularly refresh the metadata from the various sources. With this install guide there are two source – Federation Registry and edugain. The cron job runs the sync.sh script for each of the sources. It uses their [TAG] to identify the source.

1. Create the file ```/etc/cron.d/saml```

	```
	MAILTO=[ADMIN Email Address]
	
	*/30 * * * * saml /opt/saml/scripts/sync.sh [TAG]
	19 */6 * * * saml /opt/saml/scripts/sync.sh edugain
	```

	Replace [TAG] with the tag you used when registering the FR feed.  The ```[ADMIN Email Address]``` will be emailed if any errors occur.

- This cron config will update the FR metadata every 30 minutes. This may be too frequently, you may want to make run less frequently.
- The edugain metadata will be refreshed every 6 hours at 19 minutes past the hour.


## Setup Apache Configuration

1. Start Apache and enable it at start-up.

	```
	# systemctl start httpd
	# systemctl enable httpd
	```

2. Create the SAML Services Apache configuration in your preferred text editor (nano, vim, etc). Replace `[DOMAIN-NAME]` and the VirtualHost IP with the IP address of the server (currently set to 111.111.111.111).

	```
	# vim /etc/httpd/conf.d/saml.[DOMAIN-NAME].conf
	```
	```
	<VirtualHost 111.111.111.111:80>
	  ServerName saml.[DOAMIN-NAME]:80
	  DocumentRoot /var/www/vhosts/saml.[DOMAIN-NAME]
	
	  Redirect permanent / https://saml.[DOMAIN-NAME]/
	</VirtualHost>
	
	<VirtualHost 111.111.111.111:443>
	  Include conf.d/ssl.conf.include
	
	  ServerName saml.[DOMAIN-NAME]:443
	  DocumentRoot /var/www/vhosts/saml.[DOMAIN-NAME]
	
	  CustomLog logs/saml_access_log common
	  CustomLog logs/saml_ssl_request_log ssl
	  ErrorLog logs/saml_ssl_error_log
	  LogLevel warn
	
	  SSLCertificateFile /opt/saml/keypairs/apache.crt
	  SSLCertificateKeyFile /opt/saml/keypairs/apache.key
	
	  SSLCertificateChainFile /opt/saml/keypairs/intermediate.crt
	
	#  SSLCACertificateFile /opt/saml/keypairs/client-ca.crt
	  SSLVerifyDepth 2
	
	  ProxyPass /assets !
	
	  ProxyPass / http://localhost:8080/ retry=2 timeout=600
	  ProxyPassReverse / http://localhost:8080/
	
	  Alias /assets /opt/saml/repository/public/assets
	
	  <Directory /opt/saml/repository/public/assets>
	    Require all granted
	  </Directory>
	
	  <Location /api>
	    SSLVerifyClient optional
	  </Location>
	
	  RequestHeader set X509_DN "%{SSL_CLIENT_S_DN}s"
	  RequestHeader add X-Forwarded-Proto https
	</VirtualHost>
	```

1. Create SAML Services web directory. Replace `[DOMAIN-NAME]` as done previo
2. usly.

	```
	# mkdir /var/www/vhosts/saml.[DOMAIN-NAME]
	```

1. Reload Apache server

	```
	# systemctl reload httpd
	```

1. Try accessing SAML Services via Apache. Replace `[DOMAIN-NAME]` with the domain name set in Apache and `[TAG]` with the SAML Metadata Instance tag.

	```
	# curl --insecure --connect-timeout 500 -H 'Accept: application/samlmetadata+xml' https://saml.[DOMAIN-NAME]/mdq/[TAG]/entities
	```

You should receive the SAML metadata that was requested.

SAML Services is now running as expected.