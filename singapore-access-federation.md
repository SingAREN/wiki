<!-- TITLE: Singapore Access Federation (SGAF) -->
<!-- SUBTITLE: The Singapore Access Federation (SGAF) service is a Federated Identity Management System for Singapore's research and education (R&E) community. SGAF uses SAML2.0 technology to enable scalable, trusted collaborations among Singapore's R&E community.  -->

## Local Entity Requirements to Join the SGAF
* Member of SingAREN
* Acceptance of the Singapore Access Federation Rules

### Service Provider (SP) Requirements
> * SAML2.0 compatible Service Provider such as Shibboleth SP

### Identity Provider (IdP) Requirements
* SAML2.0 compatible Identity Provider such as ADFS, Shibboleth IdP, etc or a directory service such as AD, LDAP, etc
*  Provide at minimum, the following core attributes 
	* **displayName** (oid:2.16.840.1.113730.3.1.241)
	* **email** (oid:0.9.2342.19200300.100.1.3)
	* **eduPersonPrincipalName** (oid:1.3.6.1.4.1.5923.1.1.1.6)
	* **eduPersonPrimaryAffiliation** (oid:1.3.6.1.4.1.5923.1.1.1.1)
  * **eduPersonAffiliation** (oid:1.3.6.1.4.1.5923.1.1.1.1)
	* **eduPersonTargetedID** (oid:1.3.6.1.4.1.5923.1.1.1.10)
	* **organizationName** (oid:2.5.4.10)

