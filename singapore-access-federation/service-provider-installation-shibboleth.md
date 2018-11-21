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
* NTP
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
>`# ./keygen.sh -f -h sp.example.org -e https://sp.example.org/shibboleth`
{.is-info}

* **Step 5**: Select the attribute needed by the SP and mark which of them are 'Requested' and if they are 'Required' or not. For each attribute, give a good exaplanation for why they attribute is needed. This information will later be displayed to users as justification for why the information is being released.
* **Step 6**: Click Submit and wait for a confirmation email.

> **Note**
> It is import to click on the link in the confirmation email that comes later - that makes you the administrator of this SP in the Federation Registry.
{.is-info}
# Configuration
* Download the SGAF metadata signing certificate

	```
	# wget https://ds.sgaf.org.sg/distribution/metadata/updated_metadata_cert.pem -O /etc/shibboleth/sgaf-metadata-cert.pem
	```

* Edit `/etc/shibboleth/shibboleth2.xml`:
	* Replace all instanaces of `sp.example.org` with your hostname
	* Within the `<Sessions>` element:
		* Make the session handler use SSL. Set `handlerSSL="true"`
			* **Recommended**: Go even further  in the `<Sessions>` element and change the `handlerURL` from a relative one (`"/Shibboleth.sso"`) to an absolute URL - i.e. `handlerURL="https://sp.example.org/Shibboleth.sso"`. Use the hostname used when registering the SP within the SGAF Federation Registry. This makes sure that the sure is always issuing correct endpoint URLs in outgoing requests, even when users refer to the server with an alternative name. This is particurlaly important when there are multiple hostnames resolving to your server (such as ones prefixed with "www." nad one without).

	*  Change the `SupportContact` attribute to be a more meaningful value than `root@localhost`. Users will see the appropriate support contact if any errors occur during SP access.
	*  Load the federation metadata
		*  Add the **following** (or equivalent) section just above the commented sample `MetadataProvider` element.
		```
		<MetadataProvider type="XML" uri="https://ds.sgaf.org.sg/distribution/metadata/sgaf-metadata.xml"
				backingFilePath="sgaf-metadata.xml" reloadInterval="7200">
			<MetadataFilter type="RequireValidUntil" maxValidityInterval="2419200"/>
			<MetadataFilter type="Signature" certificate="sgaf-metadata-cert.pem"/>
		</MetadataProvider>
		```
		