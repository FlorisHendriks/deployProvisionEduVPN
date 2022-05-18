**Note: We have put the RPM repository from where you retrieve the RPM packages offline. This document is now for archiving purposes only**
## Introduction

With this deploy script you will be able to deploy a Fedora eduVPN server with custom support for provisioning.

[This repository is part of making eduVPN a system VPN rather than a client VPN.](https://github.com/FlorisHendriks98/eduVPN-provisioning)

Additional scripts are available after deployment:

* Use [Let's Encrypt](https://letsencrypt.org/) for automatic web server 
  certificate management;

## Requirements

* Clean Fedora installation with all updates installed;
* SELinux MUST be enabled;
* You SHOULD use static IPv4 and IPv6 address configured on your external 
  interface;
* Network equipment/VM platform allows access to the very least `tcp/80`, 
  `tcp/443`, `udp/1194` and `tcp/1194` for basic functionality, the deploy 
  script will take care of the host firewall;
* Working DNS entry for your VPN server, e.g. `vpn.example.org`.
* A root CA certificate stored on the server.

We test only with the official Fedora 
[Cloud Base Images](https://alt.fedoraproject.org/cloud/).
 
If you have a more complicated setup, we recommend to manually walk through 
the deploy script and follow the steps.

## Step 1: Base Deploy

**NOTE**: make sure you have `git` installed:

    $ sudo dnf -y install git

Perform these steps on the host where you want to deploy:

    $ git clone https://github.com/FlorisHendriks98/deployProvisionEduVPN.git
    
Traverse to the git repository:

    $ cd deployProvisionEduVPN/

Run the script (as root):

    $ sudo -s
    # ./deploy_fedora.sh

Specify the hostname you want to use for your VPN server. The recommended 
hostname SHOULD already be the one you want to use... If not, set the hostname
correctly first.

**NOTE**: you can NOT use `localhost` as a hostname, nor an IP address!

**NOTE**: by default there is **NO** firewall for the traffic between VPN 
client and VPN server. So if you have SSH running on your server, the clients
will be able to connect to it when you don't take additional steps! Look 
[here](FIREWALL.md).

## Step 2: Set up mutual TLS
Client authentication will be done via mutual TLS. We therefore need to check if the certificate the client provides us is signed by the ADCS root CA. Moreover, we need to check if this certificate is revoked or not. To set this up, we configure the /etc/httpd/conf.d/vpn.example.org.conf:

  $ vim /etc/httpd/conf.d/vpn.example.org.conf

Then add the following between <VirtualHost \*:443> \</VirtualHost> :

`SSLVerifyClient optional` \
`SSLVerifyDepth 1` \
`SSLOptions +StdEnvVars` \
`SSLCACertificateFile /etc/httpd/ADCA.crt` \
`SSLUserName SSL_CLIENT_S_DN_CN`

Where /etc/httpd/ADCA.crt is the path to the the stored CA certificate.

## Configuration

### VPN

See [PROFILE_CONFIG](PROFILE_CONFIG.md) on how to update the VPN server 
settings.

### Authentication 

#### Username & Password

By default there is a user `demo` and `admin` with a generated password for 
portal access. Those are printed at the end of the deploy script.

If you want to update/add users you can use `vpn-user-portal-account`. 
Provide an existing account to _update_ the password:

    $ sudo vpn-user-portal-account --add foo
    Setting password for user "foo"
    Password: 
    Password (repeat): 

You can configure which user(s) is/are an administrator by setting the 
`adminUserIdList` option in `/etc/vpn-user-portal/config.php`, e.g.:

    'adminUserIdList' => ['admin'],

#### LDAP

It is easy to enable LDAP authentication. This is documented separately. See
[LDAP](LDAP.md).

#### RADIUS

It is easy to enable RADIUS authentication. This is documented separately. See
[RADIUS](RADIUS.md).

#### SAML

It is easy to enable SAML authentication for identity federations, this is 
documented separately. See [SAML](SAML.md).

### ACLs

If you want to restrict the use of the VPN a bit more than on whether someone
has an account or not, e.g. to limit certain profiles to certain (groups of)
users, see [ACL](ACL.md).

## Optional

### Web Server Certificates

By default a self-signed certificate is used for the web server. You can 
install your own certificates, and tweak 
`/etc/httpd/conf.d/vpn.example.org.conf` to point to them, or use Let's Encrypt 
using the script mentioned below.

#### Let's Encrypt

Run the lets_encrypt_fedora.sh script (as root) from the cloned repository:

    $ sudo -s
    # ./lets_encrypt_fedora.sh

Make sure you use the exact same DNS name you used when running 
`deploy_fedora.sh`! 

After completing the script, the certificate will be installed and the system 
will automatically replace the certificate before it expires.

### Let's Connect! Branding

See [BRANDING](BRANDING.md).

### Port Sharing

If you also want to allow clients to connect with the VPN over `tcp/443`, see 
[Port Sharing](PORT_SHARING.md).
