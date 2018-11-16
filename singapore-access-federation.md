<!-- TITLE: Singapore Access Federation (SGAF) -->
<!-- SUBTITLE: A SAML 2.0 Federated Identity Management System to provide SSO for Singapore's R&E community-->

![Singaren Logo Transparency Small](/uploads/images/singaren-logo-transparency-small.png "Singaren Logo Transparency Small"){.pagelogo}
The Singapore Access Federation (SGAF) service is a Federated Identity Management System for Singapore's research and education (R&E) community. SGAF uses SAML2.0 technology to enable scalable, trusted collaborations among Singapore's R&E community.
# Local Entity Requirements to Join the SGAF
## General Requirements 
* Member of SingAREN
* Acceptance of the Singapore Access Federation Rules

## Service Provider (SP) Requirements
 * SAML2.0 compatible Service Provider such as Shibboleth SP

## Identity Provider (IdP) Requirements
>* SAML2.0 compatible Identity Provider such as ADFS, Shibboleth IdP, etc or a directory service such as AD, LDAP, etc
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
## Identity Providers
## Service Providers
# Support
technical-support@singaren.net.sg
