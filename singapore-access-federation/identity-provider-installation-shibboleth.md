<!-- TITLE: Identity Provider Installation: Shibboleth v3 -->
<!-- SUBTITLE: A quick summary of Identity Provider Installation Shibboleth -->


    Guide Version: 1.1
    Installer Version: 1.3.1
    Shibboleth IdP Version: 3.3.1
    Author: Australian Access Federation
    Edited by: Simon Green, SingAREN
    Publication Date: 5 July 2017

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

**Event Logging**

The installer provides a detailed set of information indicating the steps it has under taken on your server. You MAY disregard this output if the process completes successfully.

For future review all installer output is logged to `/opt/shibboleth-idp-installer/activity.log`

## Running the Installer

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

After the initial execution you’ll receive an error if you try to run `bootstrap.sh` again.

You MUST NOT re-run `bootstrap.sh` if the installation process completed but you made a simple mistake. e.g.

* Mistyped config in the MANDATORY SECTION
* Mistyped config in the OPTIONAL SECTION

If you force `bootstrap.sh` to run again once initial installation has completed the action MAY be destructive.

In this scenario you should continue with federation registration as documented below and then make any configuration changes necessary as documented within the customisation stage following completion of the installation stage as documented below.

**Reasons to re-run the installer**

You MUST NOT re-run `bootstrap.sh`, unless:

* The script indicates you did not provide all required configuration options before your initial execution
* The underpinning Ansible based installation process fails during installation

If the latter scenario occurs you MUST correct the root cause of the error before attempting to continue.

**Allowing the installer to run again**

If you’ve met all conditions, run the installer again:

```
rm /root/.lock-idp-bootstrap && ./bootstrap.sh
```

The bootstrap process will now start over and attempt to configure your server to operate as a Shibboleth IdP.

## Registering into the SGAF

Once completed the bootstrap process will output information specific to your installation which you will use to register your Shibboleth IdP with the federation. Please follow the onscreen information in order to complete the registration process.

> **Important**: At section 3 of the registration process, SAML Configuration, you will need to manually enter the Artifact Resolution Endpoint + Index and the Attribute Resolution Endpoint if you use the Easy registration using defaults. **Please Note**: The ECP Endpoint is not required.
> In the following examples, please replace `idp.example.edu.sg` with the IdP's Fully Qualified Domain Name (FQDN) during the registration process.
> 
> **Artifact Resolution Endpoint**: https://idp.example.edu.sg:8443/idp/profile/SAML2/SOAP/ArtifactResolution
> **Artifact Resolution Endpoint Index**: 2
> **Attribute Resolution Endpoint**: https://idp.example.edu.sg:8443/idp/profile/SAML2/SOAP/AttributeQuery
{.is-warning}

After completing the registration process, you will receive an email from the federation indicating your Shibboleth IdP is pending approval by the SGAF. The approval process can take **up to 24 hours**.

For further assistance please contact technical-support@singaren.net.sg

## Finalising the Installation

1. **Configure LDAP connectivity**

If you provided basic LDAP details to the bootstrap process you MAY skip this section.
If you:
* Wish to use TLS connections; or
* Have an advanced deployment scenario for your directory infrastructure:
	1. **Locate local configuration files**
	You **MUST** make any changes you require below within:
		
		```
		/opt/shibboleth-idp-installer/repository/assets/<HOST_NAME>/idp/conf
		```
		and **NOT**
		```
		/opt/shibboleth-idp/conf
		```
		
	which is the default path documented in external resources. All specific config file names, e.g. `ldap.properties`, remain the same.

	2. **Configure LDAP options**

		Please see the Shibboleth IdP LDAP documentation for a description of all available LDAP options. Undertake configuration changes or certificate additions to the server as necessary.

	3. **Apply Changes**

		After changing LDAP configuration you MUST run the command:

		```
		/opt/shibboleth-idp-installer/repository/update_idp.sh
		```
    This will merge the changes as required and reload the Shibboleth IdP to apply them.

2. **Receive Shibboleth IdP Approval**
	You will receive a second email following approval by the SGAF.

	Please wait for at least 4 hours after receiving this email, so backend processes and data sync is completed before undertaking the instructions it contains to gain administrative rights over your Shibboleth IdP within SGAF management tools.

3. **Add Backchannel certificates**

The final installation step involves providing your backchannel certificates to the SGAF management tool, Federation Registry.

Access your Shibboleth IdP record within Federation Registry and navigate to SAML -> Certificates

1. Add the contents of `/opt/shibboleth/shibboleth-idp/current/credentials/idp-backchannel.crt` as a certificate for signing.
1. Add the contents of `/opt/shibboleth/shibboleth-idp/current/credentials/idp-encryption.crt` as an encryption certificate.

# Customisation

**Ensuring your Shibboleth IdP is functioning**

Before undertaking any customisation of your Shibboleth IdP and after each change you make to customise your Shibboleth IdP we recommend testing to ensure everything is functioning correctly.

To facilitate this the SGAF provides a useful tool, called the SGAF Attribute Validator (created by AAF). This tool will ensure that your IdP is working correctly with backend security processes and that it is capable of providing the attributes your users may be asked to present when accessing federated services.

A ‘private’ browser session as the best tool for working with SGAF Attribute Validator. Different browsers will have different names for ‘private’ mode, e.g. Incognito Mode.

Access the [SGAF Attribute Validator](https://manager.sgaf.org.sg/attributevalidator/).

Follow the flow to login, ensuring you choose your new Shibboleth IdP when prompted by the Discovery Service.

**How the Shibboleth IdP installer manages your configuration**

> **IMPORTANT:** All modifiable configuration is housed in the directory `/opt/shibboleth-idp-installer/repository/assets/<HOST_NAME>`.
{.is-warning}

The structure of your configuration directory will look like the following:
 
```
.
├── apache
│   ├── idp.conf
│   ├── intermediate.crt
│   ├── server.crt
│   └── server.key
└── idp
    ├── branding
    │   ├── css
    │   │   ├── consent.css
    │   │   └── main.css
    │   ├── error-messages.properties
    │   ├── images
    │   │   ├── logo-mobile.png
    │   │   └── logo.png
    │   └── views
    │       ├── attribute-release.vm
    │       ├── error.vm
    │       ├── expiring-password.vm
    │       ├── login-error.vm
    │       ├── login.vm
    │       ├── logout-response.vm
    │       ├── logout.vm
    │       └── resolvertest.vm
    ├── conf
    │   ├── attribute-filter.xml
    │   ├── attribute-resolver.xml
    │   ├── global.xml
    │   ├── idp.properties
    │   ├── ldap.properties
    │   ├── metadata-based-attribute-filter.xml
    │   ├── metadata-providers.xml
    │   └── services.xml
    ├── logging
    │   └── logback.xml
    └── sys
        └── jetty-profile
```

If you make configuration changes directly within `/opt/shibboleth/shibboleth-idp`, `/etc/httpd` or elsewhere your installation will become unsupported and you may have difficulties when upgrading.

## Customising your Shibboleth IdP
From the configuration directory you can make changes to customise the following areas as appropriate for your environment:
* Apache certificates and configuration
* IdP configuration (xml/properties)
* IdP branding (velocity templates, css and images).

**Customisations recommended by SingAREN for operating a production Shibboleth IdP**

Here are some of the areas you should customise when preparing a Shibboleth IdP for a production environment:

* The Shibboleth IdP MUST use valid certificates, verified by a widely trusted CA, for your Apache webserver
* The use of EV certificates is **RECOMMENDED**
* Ensure all attributes on the SGAF Attribute Validator are shown with green ticks to indicate successful release
* Branding should be consistent with your organisations corporate branding, images, logos, colour schems, etc
* Corporate links, eg Accessibility, Copyright, Disclaimers, Privacy, etc should be consistent with the corporate site
* The name known by your users for their username / password should be consistently used on the IdP login page
* Links to a corporate terms of use or similar page
* Link provided to recover lost password, manage passwords or other credentials, etc
* Display of the SingAREN logo and links to the SGAF information such as the service catalogue
* Guidance for users about effectively logging out, particularly when using publicly accessible computers
* Minimise and preferably eliminate the use of technical jargon
* Showing the name of the service the user is logging into, possibly the logo as well if it is available

## Updating the Shibboleth IdP with customisations
**Actions undertaken during an update**
The update process will perform the following:

1.	Update underlying operating system packages to ensure any security issues are addressed
2.	Apply any configuration changes made within the assets directory for: * Shibboleth IdP * Jetty * Apache HTTPD
3.	RESTART all dependent processes.

You MUST have a tested rollback plan in place before running an update to ensure any unanticipated changes can be reversed.

You MUST have a tested rollback plan in place before running an update to ensure any unanticipated changes can be reversed.
Executing the update
To update your Shibboleth IdP run the command:

```
/opt/shibboleth-idp-installer/repository/update_idp.sh
```

## Upgrading your Shibboleth IdP version
In order to upgrade your versions to the latest vetted releases you need to add the -u switch to the update_idp.sh command:

```
/opt/shibboleth-idp-installer/repository/update_idp.sh -u
```

By supplying the `-u` switch the following occurs in addition to the normal update process:
1.	Upgrade to the most recent version of the installer:
	* The update will be retrieved from: `https://github.com/spgreen/shibboleth-idp-installer.git`
	* It will be based on the most recent production release
2.	Upgrade, if necessary, to the most recently vetted versions of:
	* Shibboleth IdP
	* Jetty

# Operations
Following completion of all the previous stages the Shibboleth IdP enters an operational phase.

Administrators should be aware of the following concerns for the ongoing operation of the Shibboleth IdP.

## Common Commands

* Apply configuration changes to the IdP
	```
	# /opt/shibboleth-idp-installer/repository/update_idp
	```
* Restart the IdP (Jetty)
	```
	# systemctl restart idp
	```
* Restart apache
	```
	# systemctl restart httpd
	```
* Restart ntpd
	```
	# systemctl restart ntpd
	```
* Restart firewall
	```
	# systemctl restart firewalld
	```

## Filesystem structure

The structure of the filesystem after a successful install is as follows:

```
	/opt
	├── jetty
	│   └── jetty-distribution-9.2.10.v20150310   # Jetty installation
	├── keypairs                                  # TLS assets for Apache
	│   ├── intermediate.crt
	│   ├── server.crt
	│   └── server.key
	├── shibboleth
	│   ├── jetty                                 # Jetty base for Shib IdP
	│   ├── shibboleth-idp
	│   │   └── shibboleth-idp-3.1.1              # Shibboleth instance
	│   └── shibboleth-src                        # Shib Installation files
	│       ├── install-3.1.1.exp
	│       ├── install-3.1.1.sh
	│       └── shibboleth-identity-provider-3.1.1
	└── shibboleth-idp-installer	
	    ├── repository                            #Holds conf and source code
	    └── build                                 #Installer work directory 
	/var
	└── log
	    ├── shibboleth                            #Shibb specific logs
	    ├── httpd                                 #Apache logs
	    └── jetty                                 #Jetty base logs
```

## Backup / Resilience
The IdP installer provides no backup or monitoring of the platform.
Deployers SHOULD:

Undertake regular backups of:
* The entire VM
* The local mariadb instance
* Key directories, including, but not limited to:
	1.	`/opt/shibboleth-idp-installer`
	2.	`/opt/keypairs`
	3.	`/opt/shibboleth`
	4.	`/etc/httpd`
* Monitor service availability
* Monitor platform concerns, such as disk space and load averages

**Future Customisations**

You can return to the customisation stage in the future to make further changes.


# About the AAF IdP Installer
The AAF IdP installer, a tool developed by the AAF is aimed at easing the effort required to install and maintain a Shibboleth IdP, a key component enabling users within your organisation to access the services within the federation.

All software will requires maintenance over its life time. Continuous maintenance of your IdP will allow you to;

* reduce the risk of security incidents,
* enable new features when they become available,
* and ensure technical compliance

For new subscribers bring identities to the federation the installer will allow you to technically connect your users to services with minimal effort. Again the installer will ensure you are running the latest version of the IdP.

### Technical details
The installer uses software called [Ansible](http://www.ansible.com/) a goal-oriented configuration management tool that is secure and agentless. An AAF GitHub repository holds the configuration information. This is used by Ansible to reliably and repeatably configure the IdP avoiding the potential failures from scripting, script-based systems or manually management. All that is needed is your local information. The installer passes this to Ansible which then installs or upgrades your IdP. <em>SingAREN has modified the necessary files within the AAF GitHub repository to allow Shibboleth IdP v3 installation specifically for the SGAF environment. </em>

### Updates
As new versions of the Shibboleth IdP or any of the underpinning software become available the AAF creates a new branch in the GitHub repository and apply changes. All changes go through a though testing and review process before being committed to the repository. On completion of testing a new version of the installer is released.
When a new installer is released notifications will be announced via various channels. SingAREN will modify the updates accordingly for the SGAF environment.
On receipt of such a notification it’s time to plan for your local IdP upgrades. First in your test environment to be sure there are no issues and local configuration options continue to work. Next, with approval of your change board upgrade your production IdP.

### General
The IdP Installer is a tool to assist you maintain your IdP. If the tool is not meeting your requirements or you would like to see some improvements we welcome your [feedback](http://ausaccessfed.github.io/shibboleth-idp-installer/about/feedback.html).

If there are any issues experienced when running the tool the SGAF technical support will be happy to assist.
