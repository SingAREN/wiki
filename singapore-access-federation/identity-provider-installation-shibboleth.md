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

Redhat systems also require EPEL in order to continue and the above is one option you MAY use to achieve this.

In some commercial environments you may need to have the server enabled for these packages via Satellite.

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

1. Environment data for your IdP
	a. Production environment for SGAF
	b. Organisation Name
	c. Organization base domain e.g. example.edu.sg

Optional Checklist

    An account which can bind to and run queries against your corporate directory service. You’ll require the following pieces of information from your directory administrator:
        IP Address / DNS entry for your LDAP server and connection port
        Base DN for user objects within your directory
        The Bind DN of the account you wish to connect to the directory with
        The password for the above account
        An LDAP filter attribute, often uid If you:

    Don’t have LDAP details available;
    Wish to use TLS connections; or
    Have an advanced deployment scenario for your directory infrastructure.

You’ll need to undertake further customisation during the installation process when prompted. Each of these scenarios are outside of the installer’s scope.

Next Step

Once you’ve finalised these checklists please continue to the installation st