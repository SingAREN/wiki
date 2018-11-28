<!-- TITLE: Connect a Service Provider to the SGAF -->
<!-- SUBTITLE: How to connect SAML 2.0 enabled Service Providers to the SGAF -->

# Shibboleth v3
## SGAF Registration

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

## SGAF Configuration
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

# SimpleSAMLphp
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

## `shibboleth2.xml` Configuration Snippet 
```
<ApplicationOverride id="virtualhome" entityID="https://vho.sgaf.singaren.net.sg/shibboleth" attributePrefix="AJP_">
  <Sessions checkAddress="false" consistentAddress="false" handlerSSL="true" cookieProps="https">
    <SSO discoveryProtocol="SAMLDS" ECP="false" discoveryURL="https://ds.sgaf.org.sg/discovery/DS">SAML2 SAML1</SSO>
  </Sessions>

  <MetadataProvider type="XML" url="https://ds.sgaf.org.sg/distribution/metadata/sgaf-metadata.xml" backingFilePath="virtualhome.metadata.xml" reloadInterval="1800">
    <MetadataFilter type="RequireValidUntil" maxValidityInterval="2419200"/>
    <MetadataFilter type="Signature" certificate="virtualhome/metadata.crt"/>
  </MetadataProvider>

  <AttributeExtractor type="XML" validate="true" reloadChanges="false" path="virtualhome/attribute-map.xml" />
  <AttributeResolver type="Query" subjectMatch="true"/>
  <AttributeFilter type="XML" validate="true" path="virtualhome/attribute-policy.xml" />
  <CredentialResolver type="File" key="virtualhome/sp.key" certificate="virtualhome/sp.crt" />
</ApplicationOverride>
```
# ADFS and Single-Metadata-Entity-Ingesting Service Providers
> Use SGAF Proxy Metadata: https://sgaf.singaren.net.sg/simplesaml/module.php/saml/sp/metadata.php/proxy-sp
{.is-danger}

ADFS and other Single-Metadata-Entity-Ingesting Service Providers will need to connect to the SGAF Proxy as they are unable to load or have a difficult time with multi-entity metadata.
* Use the following guide: [Connecting Service and ADFS Identity Providers to the Singapore Access Federation](https://www.singaren.net.sg/document/Connecting%20Service%20and%20ADFS%20Identity%20Providers%20to%20the%20SingAREN%20Access%20Federation.pdf) to enable your SP for IdPs connecting to the SGAF Proxy. 

> **Note:** Your Service Provider will become active within the Singapore Access Federation 24 hours after approval.
{.is-info}
