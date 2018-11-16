<!-- TITLE: Connecting to the SGAF -->
<!-- SUBTITLE: How to connect SAML 2.0 entities (IdPs and SPs) to the SGAF -->

# Identity Providers (IdPs)
## Shibbobleth Identity Providers

> SGAF Local Metadata: https://ds.sgaf.org.sg/distribution/metadata/sgaf-metadata.xml
> SGAF eduGAIN Metadata: https://ds.sgaf.org.sg/distribution/metadata/sgaf-edugain.xml
> SGAF Metadata Signing Certificate: https://ds.sgaf.org.sg/distribution/metadata/updated_metadata_cert.pem
{.is-danger}

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


## ADFS and Single-Metadata-Entity-Ingesting Identity Providers
> Use SGAF Proxy Metadata: https://sgaf.singaren.net.sg/simplesaml/module.php/saml/sp/metadata.php/proxy-sp
{.is-danger}

ADFS and other Single-Metadata-Entity-Ingesting Identity Providers will need to connect to the SGAF Proxy as they are unable to load or have a difficult time with multi-entity metadata.
Use the following guide: [Connecting Service and ADFS Identity Providers to the Singapore Access Federation](https://www.singaren.net.sg/document/Connecting%20Service%20and%20ADFS%20Identity%20Providers%20to%20the%20SingAREN%20Access%20Federation.pdf).

> **Note:** Once approved, your identity provider will become active in the Singapore Access Federation within 24 hours.
{.is-info}
# Service Providers (SPs)

### `relying-party.xml` configuration snippet where the metadata and signing certificate files are retrieved externally and loaded into Shibboleth:

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
