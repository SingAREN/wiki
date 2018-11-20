<!-- TITLE: Connect a Service Provider to the SGAF -->
<!-- SUBTITLE: A quick summary of Connect A Service Provider To The SGAF -->

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

