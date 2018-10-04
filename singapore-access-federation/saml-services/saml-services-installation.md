<!-- TITLE: SAML Services Installation -->
<!-- SUBTITLE: A quick summary of SAML Services Installation -->

# SAML Services Installation 

1. Create a SAML user with home directory as `/opt/saml`.

	```
	# useradd saml –m –d /opt/saml
	```

1. Install prerequisite software and dependencies.
	
	```
	# yum install httpd mod_ssl git mysql-devel
	```

## Setup Directories
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

## Configure SAML Services
1. Install Ruby Gem dependencies

	```
	# cd /opt/saml/repository
	# /opt/aaf-shared/rbenv/bin/rbenv exec ruby \
	    /opt/aaf-shared/rbenv/bin/install-bundle.rb
	```

1. Create the passwords to be used by SAML Services
		
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

### Setup x509 Keypairs 

1. Install certificates and keys
	
	The following keys and certificates need to be created and stored in the directory `/opt/saml/keypairs/`

	- `signing.crt` and `signing.key`. These are the certificate and key currently used to sign your existing Federation Metadata.
	-	`apache.crt`, `apache.key` and `intermediate.crt`. These are the certificates and keys for the web server. You need to obtain these from your certificate vendor.
	

14. Install Certificates for external metadata sources

	Add certificates for any external metadata sources that will feed into the SAML service, e.g. edugain. These will be held in the directory `/opt/saml/keypairs/external-metadata`. File names are up to you.

## Setup Scripts and systemd Service File
1. Setup the sync.sh and console.sh scripts in `/opt/saml/scripts`.
   
	Replace `[SAML_APP_PASSWORD]` with contents of `/opt/saml/keypairs/passwords/saml_app_password` and [KEY] with contents of `/opt/saml/keypairs/passwords/saml_secret_key_base`/
   
	1. `sync.sh`
      
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
		
		cd /opt/saml/repository
		$RBENV_ROOT/bin/rbenv exec ruby bin/sync.rb $1
		 ```

	1. `console.sh`
       
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
		```

1. Create `saml.service` service file

	Create the file `/lib/systemd/system/saml.service`. Update the `[SAML_APP_PASSWORD]` and `[KEY]` before installing the file.

	```
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
	```

1. Reload systemctl daemon as a new script has been added.
	
	```
	# systemctl daemon-reload
	``` 

---

# Database Setup

1. Install MariaDB

	The database must be running version MariaDB 10.0.x and not version 5.5.x. To install a local version of MariaDB you need to create the ```/etc/yum.repos.d/mariadb.repo``` file with the following content.

	```
	[mariadb]
	name = MariaDB
	baseurl = http://yum.mariadb.org/10.0/centos7-amd64
	gpgkey = https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
	gpgcheck = 1
	```

1. Ensure the system will use the new repo file by cleaning yum packet manager.
	
	```
	# yum clean all
	```

1. Install the latest version of MariaDB

	```
	# yum install MariaDB-server
	```
	`MariaDB-server` installs both the server and client components.

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

1. Create the `saml_app` database user and provide user with access to the `saml_production` database. Change `[SAML_APP_PASSWORD]` to the contents of `/opt/saml/keypairs/passwords/saml_app_password`

		> CREATE USER 'saml_app' IDENTIFIED BY '[SAML_APP_PASSWORD]';
		> GRANT USAGE ON *.* TO 'saml_app'@'localhost' IDENTIFIED BY '[SAML_APP_PASSWORD]';
		> GRANT ALL ON saml_production.* to 'saml_app'@'localhost';
		> FLUSH PRIVILEGES;
		> EXIT

1. Create the database and tables by using the following rake command. This relies on a number environment variables to be set to allow it to access the database server.

	1. Set environment variables. Change `[SAML_APP_PASSWORD]` to the password set at previous step
			
		```
		export RBENV_ROOT=/opt/aaf-shared/rbenv
		export RAILS_ENV=production
		export SAML_DB_HOST=127.0.0.1
		export SAML_DB_PORT=3306
		export SAML_DB_NAME="saml_production"
		export SAML_DB_USERNAME=saml_app
		export SAML_DB_PASSWORD="[SAML_APP_PASSWORD]"
		```

	1. Create the SAML Services database and populate the tables. Ensure that the commands are run within the `/opt/saml/repository` directory.
	
		```
		$ cd /opt/saml/repository
		$ bin/rake db:create
		$ bin/rake db:schema:load
		```
		An empty database should now be ready to load data into.

1. Enable and start SAML Services.

		$ systemctl enable saml
		$ systemctl start saml
		
		SAML Services is now ready to accept metadata!
