<!-- TITLE: Service Provider Installation: Shibboleth 3 -->
<!-- SUBTITLE: A quick summary of Service Provider Installation Shibboleth -->

# Introduction
There is a lot of documentation on how to install a Shibboleth 3.x Service Provider (SP) as shown by the following:

* [Understanding Shibboleth: How it All Fits Together](https://wiki.shibboleth.net/confluence/display/CONCEPT/FlowsAndConfig).
* [Official Shibboleth SP 3 Installation Documentation](https://wiki.shibboleth.net/confluence/display/SP3/Installation)
* [Official Shibboleth SP 3 Configuration Documentation](https://wiki.shibboleth.net/confluence/display/SP3/Configuration)

This wiki entry provides a simple sequence of steps to setup a Shibboleth SP to work in the Singapore Access Federation (SGAF) environment.

This documentation covers Shibboleth SP 3.x. It has been tested on CentOS7 (x86_64) but should work on other RedHat based systems.

# Pre-requisites

Before starting to build and configure the Shibboleth Service Provider, assure that the following pre-requisites are met:

* CentOS 7 Minimal Installation
* Apache with mod_ssl

```
yum install httpd mod_ssl
```

* Following Firewall Settings:
	* Allow inbound web server traffic on TCP port 80 and 443 
	
		```
		# firewall-cmd --add-service=http --permanent
		# firewall-cmd --add-service=https --permanent
		# firewall-cmd --reload
		```

	* Allow outbound traffic on TCP port 8443 for the Shibboleth daemon (shibd) to connect to every remote SGAF Identity Provider (IdP) in the federation for attribute fetching.
	
# Install Shibboleth
* Setup Shibboleth YUM repository and install Shibboleth from YUM:

```
# wget http://download.opensuse.org/repositories/security:/shibboleth/CentOS_7/security:shibboleth.repo -P /etc/yum.repos.d
# yum makecache
# yum install shibboleth
```

Shibboleth is now installed and needs to be configured.
# Registering Shibboleth SP into SGAF
Installing a Shibboleth SP only becomes useful after registering the SP into the SGAF.

The registration process is self-explanatory. The key points are:

* Navigate to the [Federation Registry](https://manager.sgaf.org.sg/federationregistry) and select **Create a Service Provider**.
* **Step 1**: Select your 'Organization' and enter a Name, Description and Service URL for the SP you are registering within the SGAF.
* **Step 2**: Enter as many Additional Details as available - this will allow the SGAF to properly advertise the service through the service catalogue.
* **Step 3**: Even though this is a Shibboleth 3.x SP Installation, please select the **Shibboleth Serivce Provider (2.4.x)** under **SAML Configuration**. Enter the base URL for your system, i.e. https://sp.example.org. The Federation Registry will automatically create all of the SAML 2.0 endpoints.
* **Step 4**: Paste in the back-channel certificate generated when installing the Shibboleth package, this is generally self-signed and does not need to be validated by a remote CA. The certificate should be located in `/etc/shibboleth/sp-cert.pem`.
> **Note:**
> The CN in the certificate must match the hostname of the service being registered. If this is an alias and your system thinks of itself with a different hostname, you will need to generate a new certificate with the correct hostname.
> Run the following command and make sure to replace `sp.example.org` with the hostname of your SP.
> ```
	$ cd /etc/shibboleth
	# ./keygen.sh -f -h sp.example.org -e https://sp.example.org/shibboleth
	```


