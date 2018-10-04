<!-- TITLE: Ruby Installation -->
<!-- SUBTITLE: A quick summary of Ruby Installation -->

## Update System and Install Prerequisite Software

1. Refresh yum package manager metadata, update the system and install prerequisite software.

	```
	# yum makecache
	# yum update
	# yum install "@Development Tools"
	# yum install git readline-devel zlib-devel openssl-devel ImageMagick-devel cmake
	```

## Setup Ruby Environment

1. Set the Ruby `RBENV_ROOT` path environment variable and create the path.

	```
	# sh -c 'echo RBENV_ROOT=/opt/aaf-shared/rbenv >> /etc/environment'
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

1. Install Bundler and Rugged gems from RubyGems using the `gem` executable.
	
	```
	# /opt/aaf-shared/rbenv/bin/rbenv exec gem install bundler
	# /opt/aaf-shared/rbenv/versions/2.4.2/bin/gem install rugged
	```

1. Rebuild Ruby shims and link Ruby executable into `/usr/bin`.
	
	```
	# /opt/aaf-shared/rbenv/bin/rbenv rehash
	# ln -s /opt/aaf-shared/rbenv/shims/ruby /usr/bin/ruby
	```

1. Download the AAF Ruby Scripts into the `/tmp/` directory and unzip the archive. 

	```
    $ cd /tmp/
	$ curl -o ruby-scripts.zip https://<LOCATION>
	# unzip /tmp/ruby-scripts.zip 
	```

1. Copy the unzipped contents into the appropriate directories:
	- `god-init.sh`, `unicorn-init.sh` and `install-bundle.rb` to `/opt/aaf-shared/rbenv/bin/`
	- `rbenv.sh` to `/etc/profile.d/`.

	```
	# cp god-init.sh unicorn-init.sh install-bundle.rb /opt/aaf-shared/rbenv/bin/
	# cp rbenv.sh /etc/profile.d/
	```

1. Clean scripts and package from download directory

	```
	# rm god-init.sh unicorn-init.sh install-bundle.rb rbenv.sh ruby-scripts.zip
	```

1. Set ownership, groups and permissions for the AAF Ruby scripts.

	```
	# chown root.wheel /opt/aaf-shared/rbenv/bin/god-init.sh
	# chown root.wheel /opt/aaf-shared/rbenv/bin/unicorn-init.sh
	# chown root.wheel /opt/aaf-shared/rbenv/bin/install-bundle.rb
	# chown root.wheel /etc/profile.d/rbenv.sh
	```

1.  Set permissions for the AAF Ruby scripts.

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