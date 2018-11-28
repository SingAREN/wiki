<!-- TITLE: Connect a Service Provider to the SGAF -->
<!-- SUBTITLE: How to connect SAML 2.0 enabled Service Providers to the SGAF -->

# Shibboleth v3
> SGAF Local Metadata: https://ds.sgaf.org.sg/distribution/metadata/sgaf-metadata.xml
> SGAF Metadata Signing Certificate: https://ds.sgaf.org.sg/distribution/metadata/updated_metadata_cert.pem
> SGAF Discovery Service: https://ds.sgaf.org.sg/discovery/DS
{.is-danger}

* Shibboleth Service Provider SGAF - [Registration](/singapore-access-federation/service-provider-installation-shibboleth#sgaf-registeration)
* Shibboleth Service Provider SGAF - [Configuration](/singapore-access-federation/service-provider-installation-shibboleth#sgaf-configuration)

# SimpleSAMLphp
> SGAF Local Metadata: https://ds.sgaf.org.sg/distribution/metadata/sgaf-metadata.xml
> SGAF Metadata Signing Certificate: https://ds.sgaf.org.sg/distribution/metadata/updated_metadata_cert.pem
> SGAF Discovery Service: https://ds.sgaf.org.sg/discovery/DS
{.is-danger}

* Service Provider Installation: SimpleSAMLphp - [Configuration](/singapore-access-federation/service-provider-installation-simplesamlphp#sgaf-configuration).
* Service Provider Installation: SimpleSAMLphp - [Registration](/singapore-access-federation/service-provider-installation-simplesamlphp#sgaf-registration).
# ADFS/Others
# ADFS and Single-Metadata-Entity-Ingesting Service Providers
> Use SGAF Proxy Metadata: https://sgaf.singaren.net.sg/simplesaml/module.php/saml/sp/metadata.php/proxy-sp
{.is-danger}

ADFS and other Single-Metadata-Entity-Ingesting Service Providers will need to connect to the SGAF Proxy as they are unable to load or have a difficult time with multi-entity metadata.
* Use the following guide: [Connecting Service and ADFS Identity Providers to the Singapore Access Federation](https://www.singaren.net.sg/document/Connecting%20Service%20and%20ADFS%20Identity%20Providers%20to%20the%20SingAREN%20Access%20Federation.pdf) to enable your SP for IdPs connecting to the SGAF Proxy. 

> **Note:** Your Service Provider will become active within the Singapore Access Federation 24 hours after approval.
{.is-info}
