<!-- TITLE: Service Provider Installation: Shibboleth 3 -->
<!-- SUBTITLE: A quick summary of Service Provider Installation Shibboleth -->

# Introduction
There is a lot of documentation on how to install a Shibboleth 3.x Service Provider (SP) as shown by the following:

* [Understanding Shibboleth: How it All Fits Together](https://wiki.shibboleth.net/confluence/display/CONCEPT/FlowsAndConfig).
* [Official Shibboleth SP 3 Installation Documentation](https://wiki.shibboleth.net/confluence/display/SP3/Installation)
* [Official Shibboleth SP 3 Configuration Documentation](https://wiki.shibboleth.net/confluence/display/SP3/Configuration)

This wiki entry provides a simple sequence of steps to setup a Shibboleth SP to work in the Singapore Access Federation (SGAF) environment.

This documentation covers Shibboleth SP 3.x. It has been tested on CentOS7 (x86_64) but should work on other RedHat based systems.

# Download and Installation
## Pre-requisites

Before starting to build and configure the Shibboleth Service Provider, assure that the following pre-requisites are met:

* Install Apache with mod_ssl

```
yum install httpd mod_ssl
```

*Firewall Settings:
	* Allow inbound web server traffic on TCP port 80 and 443 
	
		```
		# firewall-cmd --add-service=http --permanent
		# firewall-cmd --add-service=https --permanent
		# firewall-cmd --reload
		```

	* Allow outbound traffic on TCP port 8443 for the Shibboleth daemon (shibd) to connect to every remote SGAF Identity Provider (IdP) in the federation for attribute fetching.
	
	> **Note:**
	> All outgoing ports are open by default on a base CentOS 7 installation.
	{.is-info}
