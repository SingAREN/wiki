<!-- TITLE: Connect a Service Provider to the SGAF -->
<!-- SUBTITLE: How to connect SAML 2.0 enabled Service Providers to the SGAF -->

# Shibboleth v3
> SGAF Local Metadata: https://ds.sgaf.org.sg/distribution/metadata/sgaf-metadata.xml
> SGAF Metadata Signing Certificate: https://ds.sgaf.org.sg/distribution/metadata/updated_metadata_cert.pem
> SGAF Discovery Service: https://ds.sgaf.org.sg/discovery/DS
{.is-danger}

## Registration - Shibboleth

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

## Configuration - Shibboleth

# SimpleSAMLphp

[this](/service-provider-installation-simplesamlphp#configuring-sp)

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
