<!-- TITLE: Service Provider Installation: Shibboleth 3 -->
<!-- SUBTITLE: A quick summary of Service Provider Installation Shibboleth -->

Original Documentation
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
* NTP
	```
	# yum install ntp
	```
	
* Apache with mod_ssl

	```
	# yum install httpd mod_ssl
	```

* Following Firewall Settings:
	* Allow inbound web server traffic on TCP port 80 and 443 
	* Allow outbound traffic on TCP port 8443 for the Shibboleth daemon (`shibd`) to connect to every remote SGAF Identity Provider (IdP) in the federation for attribute fetching.
	
# Install Shibboleth
* Setup Shibboleth YUM repository and install Shibboleth from YUM:

	```
	# wget http://download.opensuse.org/repositories/security:/shibboleth/CentOS_7/security:shibboleth.repo -P /etc/yum.repos.d
	# yum makecache
	# yum install shibboleth
	```

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
> 
>`$ cd /etc/shibboleth`
>`# ./keygen.sh -f -n sp-signing -u shibd -g shibd -y 20 -h sp.example.org -e https://sp.example.org/shibboleth`
>`# ./keygen.sh -f -n sp-encrypt -u shibd -g shibd -y 20 -h sp.example.org -e https://sp.example.org/shibboleth`
{.is-info}

* **Step 5**: Select the attribute needed by the SP and mark which of them are **Requested** and if they are **Required**. For each attribute, give a good exaplanation for why the attribute is needed as it will be displayed to users as justification for why the information is being released.
* **Step 6**: Click Submit and wait for a confirmation email.

> **Note**
> It is import to click on the link in the confirmation email that comes later - that makes you the administrator of this SP in the Federation Registry.
{.is-info}
# Configuration
Edit the `/etc/shibboleth/shibboleth2.xml` file:
* Replace all instanaces of `sp.example.org` with your hostname
* Within the `<Sessions>` element:
	* Make the session handler use SSL. Set `handlerSSL="true"`
		* **Recommended**: Go even further  in the `<Sessions>` element and change the `handlerURL` from a relative one (`"/Shibboleth.sso"`) to an absolute URL - i.e. `handlerURL="https://sp.example.org/Shibboleth.sso"`. Use the hostname used when registering the SP within the SGAF Federation Registry. This makes sure that the sure is always issuing correct endpoint URLs in outgoing requests, even when users refer to the server with an alternative name. This is particurlaly important when there are multiple hostnames resolving to your server (such as ones prefixed with "www." and one without).
* **Configure Session Initiator** by locating the `<SSO>` element
	* Remove reference to the default `idp.example.org` - delete the entityID attribute
	* Configure the Discovery Service URL in the `discoveryURL` attribute as follows:
		
		```
		discoveryURL="https://ds.sgaf.org.sg/discovery/DS"
		```
* In `AttributeExtractor`, set `reloadChanges="true"`
* Customise attributes within the `<Errors>` element to configure the error handling pages that are rendered to the user should an error occur. Definitely change the `supportContact` attribute to a more meaningful value than `root@localhost`.
* Download the SGAF metadata signing certificate into `/etc/shibboleth/`:

	```
	# wget https://ds.sgaf.org.sg/distribution/metadata/updated_metadata_cert.pem -O /etc/shibboleth/sgaf-metadata-cert.pem
	```
	
* **Load the SGAF Metadata** by adding the **following** (or equivalent) section just above the commented `MetadataProvider` sample element:
	
	```
	<MetadataProvider type="XML" uri="https://ds.sgaf.org.sg/distribution/metadata/sgaf-metadata.xml"
			backingFilePath="sgaf-metadata.xml" reloadInterval="7200" validate="true">
		<MetadataFilter type="RequireValidUntil" maxValidityInterval="2419200"/>
		<MetadataFilter type="Signature" certificate="sgaf-metadata-cert.pem" verifyBackup="false"/>
	</MetadataProvider>
	```
* The Shibboleth SP installations needs to be configured to map attributes received from teh IdP in `/etc/shibboleth/attribute-map.xml`. Change the attribute mapping definition by either editing the file and uncommencting attributes to be accepted.
	> **attribute-policy.xml**
	> In addition to mapping received attributes to local names (and thus accepting them), it is also possible to conigure filtering rules in `attribute-policy.xml`. In most cases, this can be left as-is as the default rules provide the necessary filtering for SGAF attributes.
	>Addtional information can be found at the [Shibboleth SP3 AttributeFilter](https://wiki.shibboleth.net/confluence/display/SP3/AttributeFilter) official documentation.
	{.is-warning}

# Logging
Shibboleth SP has two separate components (the `shibd` daemon and the mod_shib module running inside Apache), and they also have separate logging configuration and destinations.

* The shibd daemon logs primarily into `/var/log/shibboleth/shibd.log` (with transaction details in `/var/log/shibboleth/transaction.log`)
	* Logging configuration is in `/etc/shibboleth/shibd.logger`
	* Log files should be owned by shibd (the user account `shibd` daemon runs under)
* The `mod_shib` Apache module logs into syslog (as facility `LOCAL0`).
	* Logging configuration in `/etc/shibboleth/native.logger`

# Protect a Resource
You can protect a resource with Shibboleth SP by adding the following directives into your Apache configuration. By default, a sample configuration snippet protecting the /secure URL on the server is included in /etc/httpd/conf.d/shib.conf:
A resource can be protected by Shibboleth SP by using the following directives within your Apache configuration. The Shibboleth SP installation provides a sample configuration snippet within `/etc/httpd/conf.d/shib.conf` that protects the `/secure` URL on the server:

```
<Location /secure>
  AuthType shibboleth
  ShibRequestSetting requireSession 1
  require shib-session
</Location>
```

You can add additional access control directives either to this file or anywhere else in the Apache configuration, as it fits with your application.

Another frequently used technique is lazy sessions - access is granted also for unauthenticated users, but if a session exists, the attributes in the session are passed through to the application - and the application can then make access control decision (and initiate a login where needed).

Applying lazy sessions (making the Shibboleth sessions visible) to the whole application can be achieved e.g. with:

```
<Location />
  AuthType shibboleth
  ShibRequestSetting requireSession 0
  require shibboleth
</Location>
```

> **Lazy Sessions**
> The login needs to be triggered by ensuring that the application redirects the user to a Shibboleth SP Session Initiator such as the default relative URL `/Shibboleth.sso/Login`.
{.is-warning}

Please view the following Shibboleth SP documentation for further information:

* [Protect a Resource](https://wiki.shibboleth.net/confluence/display/SP3/ProtectContent)
* [Protect a Resource with Apache Directives](https://wiki.shibboleth.net/confluence/display/SP3/htaccess)
* [Shibboleth SP Apache Module Configuration Reference](https://wiki.shibboleth.net/confluence/display/SP3/Apache)
* [Shibboleth SP Configuration Reference](https://wiki.shibboleth.net/confluence/display/SP3/Configuration)
* [Intergrating Shibboleth SP with Your Application](https://wiki.shibboleth.net/confluence/display/SP3/ApplicationIntegration)

# Finishing Up
* Start up Apache and Shibboleth SP:

	```
	# systemctl enable httpd shibd
	# systemctl start httpd shibd
	```

# Testing