<!-- TITLE: Identity Provider Installation: Shibboleth v3 -->
<!-- SUBTITLE: A quick summary of Identity Provider Installation Shibboleth -->

# Introduction
The Shibboleth IdP Installer automates the install of version 3 for the Shibboleth IdP on a **dedicated** Redhat or CentOS 7 server.

The installer adheres to each step outlined in the official [Installation Guide](https://wiki.shibboleth.net/confluence/display/IDP30/Installation).

The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “MAY”, and “OPTIONAL” in this document are to be interpreted as described in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).

Once you’re ready to get started please continue to the requirements checklist.

# Requirements Checklist
You MUST NOT continue to installation until you’ve worked through the checklist below. Environment preparation is critical to a successful outcome.

**Required Checklist**

* A **dedicated** CentOS 7 or Redhat 7 server (virtual or physical), with the following minimum specifications:
	* 2 CPU
	* 2GB RAM
	* 10GB+ partition for OS

This server MUST NOT be used for any other purpose in the future.

**Additional requirements for CentOS 7 servers**

* Install the EPEL Repository

	```
	# yum -y install epel-release
	```
	
**Additional requirements for RedHat 7 servers**

Redhat systems also require EPEL in order to continue and the above is one option you MAY use to achieve this. In some commercial environments you may need to have the server enabled for these packages via Satellite.

In this case please speak to your system administrators and have this configured before continuing.
1. You MUST have SSH access to the server
1. You MUST be able to execute commands as root on the system without limitation
1. The server MUST be routable from the public internet with a static IP. Often this means configuring the IP on a local network interface directly but advanced environments may handle this differently.
1. The static IP MUST have a publicly resolvable DNS entry. Typically of the form idp.example.edu.sg
1. The server MUST be able to communicate with the wider internet without blockage due to firewall rules. All publicly routable servers MUST be accessible for:

	| Port  |                 Purpose                    |
	| ----- | --------------------------------- |
	|   80  | Outbound HTTP connections   |
	|  443 | Outbound HTTPS connections |

	* Each of the following commands MUST succeed when run on your server:
		
		```
		$ curl http://example.edu
		$ curl https://example.edu
		```

1. The server MUST be accessible from the wider internet without blockage due to firewall rules for:

	| Port   |                                              Purpose                                                    |
	| ------ | --------------------------------------------------------------------------- |
	|   80   |                              Outbound HTTP connections                                    |
	|  443  |                             Outbound HTTPS connections                                   |
	| 8443 | Backchannel, client verified TLS connections, used within SAML flows |

1. Environmental data for your IdP
	1. Production environment for SGAF
	1. Organisation Name
	1. Organization base domain e.g. example.edu.sg

**Optional Checklist**

1. An account which can bind to and run queries against your corporate directory service. You’ll require the following pieces of information from your directory administrator:
	1. IP Address / DNS entry for your LDAP server and connection port
	1. Base DN for user objects within your directory
	1. The Bind DN of the account you wish to connect to the directory with
	1. The password for the above account
	1. An LDAP filter attribute, often uid If you:

	*	Don’t have LDAP details available;
	*	Wish to use TLS connections; or
	*	Have an advanced deployment scenario for your directory infrastructure.

You’ll need to undertake further customisation during the installation process when prompted. Each of these scenarios are outside of the installer’s scope.

# Installation
You MUST have completed the requirements checklist before continuing installation. Environmental preparation is critical to a successful outcome.

**Actions undertaken during installation**

The installation process:

1. Perform a `# yum -y update` (system wide package upgrade). Please note that the installer uses yum for the installation of all system components (except Jetty and Shibboleth IdP).
1. Install all required dependencies via `yum` (`git`, `ansible`, `mariadb`, `apache` etc). With the previous step in mind, bootstrap will always use the latest versions of these packages.
1. Create self-signed keys for Apache. These are for initial testing of the IdP and are replaced when further customising the Shibboleth IdP.
1. Install Apache.
1. Install Jetty with Shibboleth IdP. Jetty runs on port `8080` and creates the Shibboleth IdP web app context `/idp`. Apache configuration serves this on port `443` through a reverse proxy. Jetty also listens on port `8443` to support ECP.
1. Install a MariaDB instance. A database is created (name: `idp_db`, user: `idp_admin`) with [these schemas](https://github.com/spgreen/shibboleth-idp-installer/tree/SGAF-Implementation/templates/db) populated.
1. Install NTP for time synchronisation.
	* Opens local firewall ports `80`, `443` and `8443`.

**Event logging**

The installer provides a detailed set of information indicating the steps it has under taken on your server. You MAY disregard this output if the process completes successfully.

For future review all installer output is logged to:
```
/opt/shibboleth-idp-installer/activity.log
```

**Running the installer**
The following commands MUST be executed as the **root** user. Start the process from `/root`.

1. Run the command:
	```
	curl https://raw.githubusercontent.com/spgreen/shibboleth-idp-installer/\
	SGAF-Implementation/bootstrap.sh > bootstrap.sh && chmod u+x bootstrap.sh
	```

2. Edit the bootstrap file:

	```
	vim bootstrap.sh
	```

* You MUST review, configure and uncomment each field listed in the “MANDATORY SECTION”
* If you have LDAP details you SHOULD also configure the “OPTIONAL SECTION”.

You MAY further configure LDAP connection details, including TLS if required, after bootstrap has completed.

3. Run the command:

	```
	./bootstrap.sh
	```

The bootstrap process will now configure your server to operate as a Shibboleth IdP.

**Errors during installation**

If an error occurs, the logs prior to installer termination MUST be reviewed to understand the underpinning cause.

Generally the installer SHOULD be executed once.

After the initial execution you’ll receive an error if you try to run bootstrap.sh again.

You MUST NOT re-run bootstrap.sh if the installation process completed but you made a simple mistake. e.g.

    Mistyped config in the MANDATORY SECTION
    Mistyped config in the OPTIONAL SECTION

If you force bootstrap.sh to run again once initial installation has completed the action MAY be destructive.

In this scenario you should continue with federation registration as documented below and then make any configuration changes necessary as documented within the customisation stage following completion of the installation stage as documented below.
Reasons to re-run the installer

You MUST NOT re-run bootstrap.sh, unless:

    The script indicates you did not provide all required configuration options before your initial execution
    The underpinning Ansible based installation process fails during installation

If the latter scenario occurs you MUST correct the root cause of the error before attempting to continue.
Allowing the installer to run again

If you’ve met all conditions, run the installer again:

rm /root/.lock-idp-bootstrap && ./bootstrap.sh

The bootstrap process will now start over and attempt to configure your server to operate as a Shibboleth IdP.
Registration with the federation

Once completed the bootstrap process will output information specific to your installation which you will use to register your Shibboleth IdP with the federation. Please follow the onscreen guide in order to complete the registration process.

Important: At section 3 of the registration process, SAML Configuration, you will need to manually enter the Artifact Resolution Endpoint + Index and the Attribute Resolution Endpoint if you use the Easy registration using defaults part. Please Note: The ECP Endpoint is not required.

In the following examples, please replace idp.example.edu.sg with the IdP's Fully Qualified Domain Name (FQDN) during the registration process.

Artifact Resolution Endpoint: https://idp.example.edu.sg:8443/idp/profile/SAML2/SOAP/ArtifactResolution

Artifact Resolution Endpoint Index: 2

Attribute Resolution Endpoint: https://idp.example.edu.sg:8443/idp/profile/SAML2/SOAP/AttributeQuery

After completing the registration process, you will receive an email from the federation indicating your Shibboleth IdP is pending approval by the SGAF. The approval process can take up to 24 hours.

For further assistance please contact technical-support@singaren.net.sg
Installation finalisation steps

1. Configure LDAP connectivity

If you provided basic LDAP details to the bootstrap process you MAY skip this section.

If you:

    Wish to use TLS connections; or

    Have an advanced deployment scenario for your directory infrastructure:

    1. Locate local configuration files

    You MUST make any changes you require below within:

    /opt/shibboleth-idp-installer/repository/assets/<HOST_NAME>/idp/conf

    and NOT

    /opt/shibboleth-idp/conf

    which is the default path documented in external resources. All specific config file names, e.g. ldap.properties, remain the same.

    2. Configure LDAP options

    Please see the Shibboleth IdP LDAP documentation for a description of all available LDAP options. Undertake configuration changes or certificate additions to the server as necessary.

    3. Apply Changes

    After changing LDAP configuration you MUST run the command:

    /opt/shibboleth-idp-installer/repository/update_idp.sh

    This will merge the changes as required and reload the Shibboleth IdP to apply them.

2. Receive Shibboleth IdP Approval

Following approval by the AAF you’ll receive a second email.

Please wait for at least 4 hours after receiving this email, so backend processes and data sync is definitely completed, before undertaking the instructions it contains to gain administrative rights over your Shibboleth IdP within AAF management tools.

3. Add Backchannel certificates

The final installation step involves providing your backchannel certificates to the SGAF management tool, Federation Registry.

Access your Shibboleth IdP record within Federation Registry and navigate to SAML -> Certificates

    Add the contents of /opt/shibboleth/shibboleth-idp/current/credentials/idp-backchannel.crt as a certificate for signing.
    Add the contents of /opt/shibboleth/shibboleth-idp/current/credentials/idp-encryption.crt as an encryption certificate.

Next Step

Once you’ve finalised installation please continue to the customisation stage where we’ll test your installation and show you how to tune things as necessary for your environment.