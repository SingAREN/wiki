<!-- TITLE: Singapore Access Federation (SGAF) -->
<!-- SUBTITLE: A SAML 2.0 Federated Identity Management System for Singapore's R&E community-->

![Singaren Logo Transparency Small](/uploads/images/singaren-logo-transparency-small.png "Singaren Logo Transparency Small"){.pagelogo}

# Local Entity Requirements to Join the SGAF
## General Requirements 
* Member of SingAREN
* Acceptance of the Singapore Access Federation Rules

## Service Provider (SP) Requirements
 * SAML2.0 compatible Service Provider such as Shibboleth SP

## Identity Provider (IdP) Requirements
>* SAML 2.0 compatible Identity Provider such as ADFS, Shibboleth IdP, etc or a directory service such as AD, LDAP, etc
>*  Provide at minimum, the following core attributes 
>    * **displayName** (oid:2.16.840.1.113730.3.1.241)
>    * **email** (oid:0.9.2342.19200300.100.1.3)
>    * **eduPersonPrincipalName** (oid:1.3.6.1.4.1.5923.1.1.1.6)
>    * **eduPersonPrimaryAffiliation** (oid:1.3.6.1.4.1.5923.1.1.1.1)
>    * **eduPersonAffiliation** (oid:1.3.6.1.4.1.5923.1.1.1.1)
>    * **eduPersonTargetedID** (oid:1.3.6.1.4.1.5923.1.1.1.10)
>    * **organizationName** (oid:2.5.4.10)

> **Note:** If your institution only has a **directory service**, a SAML2.0 IdP needs to be installed and connected to the directory service before connecting to the SGAF. Please follow the Shibboleth IdPv3 Installer by AAF Guide.
{.is-warning}


# SGAF Metadata

The **SGAF Metadata** is an important part of the SAML Federation. In essence, it is a directory of registered, trusted and approved entities within the SGAF, allowing only Identity Providers (IdP) and Service Providers (SP) found within the metadata to communicate with each other.

The **SGAF Metadata Registration Practice Statement (MRPS)** describes the metadata management process conducted by the SGAF Federation Operator. 

The **SGAF SAML Web Single Sign-On Technology Profile** defines a standard that enables Identity Providers and Relying Parties to create and use Web Single Sign-On services with SAML. 

## Metadata Repository
[SGAF Metadata Sources](https://ds.sgaf.org.sg/)

## Metadata Documents
* [SGAF Metadata Registration Practice Statement](https://www.singaren.net.sg/document/SGAF-MRPS.pdf)
* [SGAF SAML Web Single Sign-On Technology Profile](https://www.singaren.net.sg/document/SGAF-SAML-Web-SSO-Technology-Profile.pdf)


# Connecting to the SGAF
## Identity Providers (IdPs)
### Shibbobleth Identity Providers

> SGAF Local Metadata: https://ds.sgaf.org.sg/distribution/metadata/sgaf-metadata.xml
> SGAF eduGAIN Metadata: https://ds.sgaf.org.sg/distribution/metadata/sgaf-edugain.xml
> SGAF Metadata Signing Certificate: https://ds.sgaf.org.sg/distribution/metadata/updated_metadata_cert.pem
{.is-info}

1. Create an Organisation for your institution. Please use the following technical document: [Create an Organisation in SGAF](https://www.singaren.net.sg/document/Creating%20an%20Organisation%20within%20SGAF.pdf),
2. Wait for the Federation Administrator to approve the new organisation,
3. [Register your Shibboleth Identity Provider](https://manager.sgaf.org.sg/federationregistry/registration/idp) using your newly created Organisation in Identity Provider Description,
4. Select the appropriate attributes that the Identity Provider will supply,
5. Submit request and wait for approval via email.
6. Load the [SGAF Local Metadata](https://ds.sgaf.org.sg/distribution/metadata/sgaf-metadata.xml), [SGAF-signed eduGAIN Metadata](https://ds.sgaf.org.sg/distribution/metadata/sgaf-edugain.xml) and the [SGAF Metadata Signing Certificates](https://ds.sgaf.org.sg/distribution/metadata/updated_metadata_cert.pem) within the Shibboleth IdP `relying-party.xml` or equivalent file.
	* Visit the [Shibboleth IdPv3 MetadataConfiguration Documentation](https://wiki.shibboleth.net/confluence/display/IDP30/MetadataConfiguration) for information on how to load metadata into the Shibboleth IdP.
	* Example [`relying-party.xml` configuration snippet](#relying-party-xml-configuration-snippet-where-the-metadata-and-signing-certificate-files-are-retrieved-externally-and-loaded-into-shibboleth)
7. Reload the Shibboleth IdP

Once you receive the confirmation email and loaded in the SGAF metadata chain within your IdP, connect to the [Federation Registry](https://manager.sgaf.org.sg/federationregistry/) and become the administrator for both the Organisation and Identity Provider.
Follow the instructions given by the confirmation emails of both the Organisation and Identity Provider to complete this process.

> **Note:** Your identity provider will become active within the Singapore Access Federation 24 hours after approval.
{.is-info}

---

#### `relying-party.xml` configuration snippet where the metadata and signing certificate files are retrieved externally and loaded into Shibboleth:

```
<metadata:MetadataProvider id="ShibbolethMetadata" xsi:type="metadata:ChainingMetadataProvider">

        <metadata:MetadataProvider id="IdPMD" xsi:type="metadata:FilesystemMetadataProvider"
                                   metadataFile="/opt/virtualhome/shibboleth/shibboleth-idp/shibboleth-idp-2.4.4/metadata/idp-metadata.xml"
                                   maxRefreshDelay="P1D" />

        <metadata:MetadataProvider id="FedMD" xsi:type="metadata:FilesystemMetadataProvider"
                                   metadataFile="/opt/virtualhome/shibboleth/shibboleth-idp/shibboleth-idp-2.4.4/metadata/federation-metadata.xml"
                                   maxRefreshDelay="PT10M">
          <metadata:MetadataFilter xsi:type="metadata:ChainingFilter">
            <metadata:MetadataFilter  xsi:type="metadata:RequiredValidUntil" maxValidityInterval="P7D" />

            <metadata:MetadataFilter xsi:type="metadata:EntityRoleWhiteList">
                <metadata:RetainedRole>samlmd:SPSSODescriptor</metadata:RetainedRole>
              </metadata:MetadataFilter>
              <metadata:MetadataFilter xsi:type="metadata:SchemaValidation"/>
          </metadata:MetadataFilter>
        </metadata:MetadataProvider>
        <metadata:MetadataProvider id="EGMD" xsi:type="metadata:FilesystemMetadataProvider"
                                   metadataFile="/opt/virtualhome/shibboleth/shibboleth-idp/shibboleth-idp-2.4.4/metadata/edugain-metadata.xml"
                                   maxRefreshDelay="PT10M">
          <metadata:MetadataFilter xsi:type="metadata:ChainingFilter">
            <metadata:MetadataFilter  xsi:type="metadata:RequiredValidUntil" maxValidityInterval="P7D" />

            <metadata:MetadataFilter xsi:type="metadata:EntityRoleWhiteList">
                <metadata:RetainedRole>samlmd:SPSSODescriptor</metadata:RetainedRole>
              </metadata:MetadataFilter>
              <metadata:MetadataFilter xsi:type="metadata:SchemaValidation"/>
          </metadata:MetadataFilter>
        </metadata:MetadataProvider></metadata:MetadataProvider>
        <security:TrustEngine id="shibboleth.MetadataTrustEngine" xsi:type="security:StaticPKIXSignature">
          <security:ValidationInfo id="AAFCredentials" xsi:type="security:PKIXFilesystem">
            <security:Certificate>/opt/virtualhome/shibboleth/shibboleth-idp/shibboleth-idp-2.4.4/credentials/metadata.crt</security:Certificate>
          </security:ValidationInfo>
          <security:ValidationInfo id="EGCredentials" xsi:type="security:PKIXFilesystem">
            <security:Certificate>/opt/virtualhome/shibboleth/shibboleth-idp/shibboleth-idp-2.4.4/credentials/metadata.crt</security:Certificate>
          </security:ValidationInfo>
        </security:TrustEngine>
				
```

### ADFS Identity Providers
> Use SGAF Proxy Metadata: https://sgaf.singaren.net.sg/simplesaml/module.php/saml/sp/metadata.php/proxy-sp
{.is-danger}

ADFS Identity Providers will need to connect to the SGAF Proxy as ADFS has issues ingesting multi-entity metadata. Thus all ADFS connections within the SGAF will flow through the proxy. 
Use the following guide: [Connecting Service and ADFS Identity Providers to the Singapore Access Federation](https://www.singaren.net.sg/document/Connecting%20Service%20and%20ADFS%20Identity%20Providers%20to%20the%20SingAREN%20Access%20Federation.pdf).

> **Note:** Once approved, your identity provider will become active in the Singapore Access Federation within 24 hours.
{.is-info}
## Service Providers (SPs)
# Support
technical-support@singaren.net.sg
