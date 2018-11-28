<!-- TITLE: Connect a Service Provider to the SGAF -->
<!-- SUBTITLE: How to connect SAML 2.0 enabled Service Providers to the SGAF -->

# Shibboleth v3
> SGAF Local Metadata: https://ds.sgaf.org.sg/distribution/metadata/sgaf-metadata.xml
> SGAF Metadata Signing Certificate: https://ds.sgaf.org.sg/distribution/metadata/updated_metadata_cert.pem
> SGAF Discovery Service: https://ds.sgaf.org.sg/discovery/DS
{.is-danger}

## Shibboleth Registration

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
* **Step 7**: Follow [Connecting Service and ADFS Identity Providers to the Singapore Access Federation](https://www.singaren.net.sg/document/Connecting%20Service%20and%20ADFS%20Identity%20Providers%20to%20the%20SingAREN%20Access%20Federation.pdf) to enable your SP for IdPs connecting via the SGAF Proxy. 

> **Note**
> It is import to click on the link in the confirmation email that comes later - that makes you the administrator of this SP in the Federation Registry.
{.is-info}

## Shibboleth Configuration
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

The Shibboleth SP installations needs to be configured to map attributes received from the IdP in `/etc/shibboleth/attribute-map.xml`. Change the attribute mapping definitions by either editing the file and uncommenting attributes needed for the application.
> **attribute-policy.xml**
> In addition to mapping received attributes to local names (and thus accepting them), it is also possible to conigure filtering rules in `attribute-policy.xml`. In most cases, this can be left as-is as the default rules provide the necessary filtering for SGAF attributes.
>Addtional information can be found at the [Shibboleth SP3 AttributeFilter](https://wiki.shibboleth.net/confluence/display/SP3/AttributeFilter) official documentation.
{.is-warning}

# SimpleSAMLphp (SSP)

## SSP Configuration

> We will be using sp.example.org to refer to the hostname of your Service Provider - please substitute that with the actual hostname of your SP.
{.is-warning}

* Create a certificate (self-signed for 20 years)

	```
  $ cd /opt/simplesamlphp/cert
  # openssl req -newkey rsa:2048 -new -x509 -days 7304 -nodes -out saml.crt -keyout saml.pem
	```
  
  * ... and enter all information requested ("." to skip) - the crucial part is your hostname.

* Make the private key readable only to the user SimpleSAMLphp runs as (apache)
	```
	# chown apache.apache /opt/simplesamlphp/cert/saml.{crt,pem}
	# chmod 600 /opt/simplesamlphp/cert/saml.pem
	```
	
* Edit `/opt/simplesamlphp/config/authsources.php` and add references to the certificate to the default-sp definition:

	```
	        'default-sp' => array(
	                'saml:SP',
	                'privatekey' => 'saml.pem',
	                'certificate' => 'saml.crt',
	```


* Enable and configure the `metarefresh` and `cron` modules:

	```
	$ cd /opt/simplesamlphp
	# touch modules/metarefresh/enable
	# cp modules/metarefresh/config-templates/*.php config/
	# touch modules/cron/enable
	# cp modules/cron/config-templates/*.php config/
	```

* Create a directory to cache the downloaded federation metadata (writable by Apache - this also means setting the SELinux context if SELinux is enabled on your system):

	```
	# mkdir /opt/simplesamlphp/metadata/metarefresh-sgaf
	# chown apache.apache /opt/simplesamlphp/metadata/metarefresh-sgaf
	```
	
* Download the metadata signing certificate for the federation metadata into `/opt/simplesamlphp/cert/sgaf-metadata-cert.pem`:

	```
	# wget https://ds.sgaf.org.sg/distribution/metadata/updated_metadata_cert.pem -O /opt/simplesamlphp/cert/sgaf-metadata-cert.pem
	```
        
* Edit config/config-metarefresh.php:
	* Replace `'kalmar'` with the federation name (`'sgaf'`)
	* Set the download URL:

	```
   'src' => 'https://ds.sgaf.org.sg/distribution/metadata/sgaf-metadata.xml',
	```
	
* Set output directory and format (use 'serialize'format):

	```
                                'outputDir'     => 'metadata/metarefresh-sgaf/',
                                'outputFormat' => 'serialize',
	```

* Set expiry date to 7 days to match SGAF

	```
                                'expireAfter'           => 60*60*24*7, // Maximum 7 days cache time.
	```

* Change the list of accepted certificates to the metadata signing certificate downloaded above:

	```
                                                'certificates' => array(
                                                        'sgaf-metadata-cert.pem',
                                                ),
	```

* Remove/comment-out the validateFingerprint entry (see note below for explanation)
	
	>Older versions of SimpleSAMLphp did not support directly referring to a certificate and instead required embedding the certificate fingerprint in the configuration
	>
	>For historical and archival purposes, the instructions are included here - but can be ignored in favour of using the above certificates setting:
	> * Set the 'validateFingerprint' to the fingerprint value of the metadata issuing certificate
	>   * SGAF: `D9:F7:F5:5B:F4:D6:9A:BC:3F:34:18:91:B0:B7:1A:FA:B9:93:DE:F1`
	>     * To calculate the fignerprint yourself, download the metadata signing certificate and get the fingerprint value with:
	>       `$ openssl x509 -fingerprint -noout -in metadata-cert.pem`

* Edit config/config.php and add an extra entry into 'metadata.sources'
	```
	   array('type' => 'serialize', 'directory' => 'metadata/metarefresh-sgaf'),
	```
	
* Now go with your browser to your SimpleSAMLphp page: https://sp.example.org/simplesaml/
	* and go to the Configuration page (log in with the Administrator password).
	* and from there to "Cron module information page"

* Edit config/module_cron.php and change the secret 'key' from the default of 'secret'to a different password (to prevent potential abuse of the cron URL):
	```
					'key' => 'top-secret',
	```
	
* Paste the hourly cron job entry (invoking curl to localhost) into root's crontab.:

	```
	01 * * * * curl --silent "https://sp.example.org/simplesaml/module.php/cron/cron.php?key=secret&tag=hourly" > /dev/null 2>&1
	```
	
	* (run "crontab -e" and paste the line into the editor)

	> **Note - Invoking curl**
	> 1. If you have changed the cron password as instructed above, the line would be different than shown here.
	> 2. If your web server is running with a self-signed HTTPS certificate, you would need to tell curl to either trust the local host certificate, or switch off certificate checking altogether.
	>     * Otherwise, with the --silent option, curl would just silently fail
	>     * So use either
	>     `01 * * * * curl --cacert /etc/pki/tls/certs/localhost.crt --silent "https://sp.example.org/simplesaml/module.php/cron/cron.php?key=secret&tag=hourly" > /dev/null 2>&1 `
	>     * or
	>     `01 * * * * curl --insecure --silent "https://sp.example.org/simplesaml/module.php/cron/cron.php?key=secret&tag=hourly" > /dev/null 2>&1`
	>     * And wait for a confirmation email to be sent to the technical contact email address after the cronjob runs at HH:01.

	* You can force the job to run immediately by clicking on one of the hourly link at the bottom of the page (or pasting the cron-job URL into your browser - the GUI link gives more output).
  * To see the output from the metadata refresh itself, go to https://sp.example.org/simplesaml/module.php/metarefresh/fetch.php

* After confirming the cron job works (wait for up to an hour to receive a confirmation email to the SimpleSAMLphp technical contact email address), edit `config/module_cron.php` and to avoid getting a confirmation email each time the cron-job runs, set
	* either `'debug_message' => FALSE,` (to suppress the confirmation debug message)
	* or `'sendemail' => FALSE,` (to suppress all email messages from the cron module)
	* However, they will have the same effect - as any error messages from metarefresh do not propagate to the cron module and are only visible in Apache error logs (`/var/log/httpd/ssl_error_log`)

## Configure to use the SGAF Discovery Service

* Edit `config/authsources.php` and set `'discoURL'` to SGAF-Production: https://ds.sgaf.org.sg/discovery/DS':

	```
	                'discoURL' => 'https://ds.sgaf.org.sg/discovery/DS',
	```

## Configure Additional SGAF Attributes

From the list of attributes used within Tuakiri and the list of Attributes supported by SimpleSAMLphp in the default configuration, the following need to be explicitly added:

* auEduPersonSharedToken
* homeOrganizationType
* eduPersonAssurance

* Create `attributemap/sgaf-attrs.php` with the following contents (adding on to what already exists in `attributemap/oid2name.php`)

	```
	<?php
	$attributemap = array(
					'urn:oid:1.3.6.1.4.1.25178.1.2.10' => 'schacHomeOrganizationType',
					'urn:oid:1.3.6.1.4.1.5923.1.1.1.11' => 'eduPersonAssurance',
					'urn:oid:1.3.6.1.4.1.27856.1.2.5' => 'auEduPersonSharedToken',
	);      
	?> 
	```
	
* Edit `config/config-metarefresh.php` and add a reference to this file in the template for IdPs downloaded via metarefresh:

	```
	                                        'template' => array(
	                                                'tags'  => array('sgaf'),
	                                                'authproc' => array(
	                                                        51 => array('class' => 'core:AttributeMap', 'oid2name', 'sgaf-attrs'),
	                                                ),
	                                        ),
	```

* To also configure friendly attribute names, add the following after the first line of dictionaries/attributes.definition.json (note that eduPersonAssurance already has an entry there, does not need to be duplicated)

	```
	        "attribute_schachomeorganizationtype": {
	                "en": "Home organization type"
	        },
	        "attribute_auedupersonsharedtoken": {
	                "en": "Shared token"
	        },
	```

## Clean up SP configuration

Remove (comment-out) pre-configured IdPs and SPs

* Edit `metadata/saml20-idp-remote.php` - remove pre-configured openidp.feide.no
* Edit `metadata/saml20-sp-remote.php` - remove pre-configured saml2sp.example.org and google.com
* Edit `metadata/shib13-sp-remote.php` - remove pre-configured sp.shiblab.feide.no

## SSP Registration

The SGAF Federation Registry (FR) has in the initial setup only pre-configured support for Shibboleth SP implementation, not SimpleSAMLphp. Without the pre-configured support, it is necessary to enter all endpoints URLs manually. There is an ongoing project to add support to FR to support SimpleSAMLphp, until then, please use the Advanced Registration form as described in this section.

As a reference point, the metadata for your SP can be accessed at https://sp.example.org/simplesaml/module.php/saml/sp/metadata.php/default-sp?output=xhtml

For reference, please also see the attached image mapping SimpleSAMLphp metadata to Federation Registry form (credits: Bevan Rudge, University of Auckland).

Access the SGAF [Federation Registry](https://manager.sgaf.org.sg/federationregistry/) and start registering a new Service Provider.

* Enter your personal details
* Select your organization (Create an Organization first if not already listed)
* Enter the details about your SP (name, description, service URL)
* Select Advanced Registration and enter the following information (drawing from your SP metadata and using the mapping as in the image above), replacing sp.example.org with the hostname of your SP:

> **Entity Descriptor ID**: https://sp.example.org/simplesaml/module.php/saml/sp/metadata.php/default-sp
> **Assertion Consuming Service (Post)**: https://sp.example.org/simplesaml/module.php/saml/sp/saml2-acs.php/default-sp (Index: 0)
> **Assertion Consuming Service (Artifact)**: https://sp.example.org/simplesaml/module.php/saml/sp/saml2-acs.php/default-sp (Index: 2)
>
> **Single Logout Redirect Endpoint**: https://sp.example.org/simplesaml/module.php/saml/sp/saml2-logout.php/default-sp
> **Single Logout SOAP Endpoint**: https://sp.example.org/simplesaml/module.php/saml/sp/saml2-logout.php/default-sp
>
>
> **Discovery Response**: https://sp.example.org/simplesaml/module.php/saml/sp/discoresp.php
{.is-info}

> Leave other fields blank!
{.is-warning}

* **Certificates**: paste in the contents of `cert/saml.crt`

* **Attributes**: select the attributes needed by your SP (and give a reason for requesting each of the attributes)

Review the SP registration form, submit it for approval and wait for a confirmation email.

Follow [Connecting Service and ADFS Identity Providers to the Singapore Access Federation](https://www.singaren.net.sg/document/Connecting%20Service%20and%20ADFS%20Identity%20Providers%20to%20the%20SingAREN%20Access%20Federation.pdf) to enable your SP for IdPs connecting via the SGAF Proxy. 

> **Note**
> It is import to click on the link in the confirmation email that comes later - that makes you the administrator of this SP in the Federation Registry.
{.is-info}


# ADFS/Others
# Shibboleth v3 and SimpleSAMLPHP Service Providers

> SGAF Local Metadata: https://ds.sgaf.org.sg/distribution/metadata/sgaf-metadata.xml
> SGAF Metadata Signing Certificate: https://ds.sgaf.org.sg/distribution/metadata/updated_metadata_cert.pem
> SGAF Discovery Service: https://ds.sgaf.org.sg/discovery/DS
{.is-danger}

1. Create an Organisation for your institution using the following technical document: [Create an Organisation in SGAF](https://www.singaren.net.sg/document/Creating%20an%20Organisation%20within%20SGAF.pdf),
2. Wait for the Federation Administrator to approve the new organisation,
3. [Register your Service Provider](https://manager.sgaf.org.sg/federationregistry/registration/sp) using your newly created Organisation in Service Provider Description,
4. Select the attributes that the SP requires and provide reasoning to why the specfic attributes are needed,
5. Submit request and wait for approval email.
6. Load the [SGAF Local Metadata](https://ds.sgaf.org.sg/distribution/metadata/sgaf-metadata.xml), [SGAF Metadata Signing Certificate](https://ds.sgaf.org.sg/distribution/metadata/updated_metadata_cert.pem) and [Discovery URL](https://ds.sgaf.org.sg/discovery/DS) into the SP.
	*  [Shibboleth SPv3 Configuration Documentation](https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPGettingStarted)
		* [ `shibboleth2.xml` configuration snippet](#shibboleth-2-xml-configuration-snippet) for loading in metadata sources, metadata signing certificates and the discovery service.
	*  SimpleSAMLPHP - Coming Soon!
7. Once you receive the confirmation email, connect to the [Federation Registry](https://manager.sgaf.org.sg/federationregistry/) and become the administrator for both the Organisation and Service Provider.
8. Follow the instructions given by the confirmation emails of both the Organisation and Service Provider to complete this process.
9. Follow [Connecting Service and ADFS Identity Providers to the Singapore Access Federation](https://www.singaren.net.sg/document/Connecting%20Service%20and%20ADFS%20Identity%20Providers%20to%20the%20SingAREN%20Access%20Federation.pdf) to enable your SP for IdPs connecting to the SGAF Proxy. 

> **Note:** Your Service Provider will become active within the Singapore Access Federation 24 hours after approval.
{.is-info}

# ADFS and Single-Metadata-Entity-Ingesting Service Providers
> Use SGAF Proxy Metadata: https://sgaf.singaren.net.sg/simplesaml/module.php/saml/sp/metadata.php/proxy-sp
{.is-danger}

ADFS and other Single-Metadata-Entity-Ingesting Service Providers will need to connect to the SGAF Proxy as they are unable to load or have a difficult time with multi-entity metadata.
* Use the following guide: [Connecting Service and ADFS Identity Providers to the Singapore Access Federation](https://www.singaren.net.sg/document/Connecting%20Service%20and%20ADFS%20Identity%20Providers%20to%20the%20SingAREN%20Access%20Federation.pdf) to enable your SP for IdPs connecting to the SGAF Proxy. 

> **Note:** Your Service Provider will become active within the Singapore Access Federation 24 hours after approval.
{.is-info}
