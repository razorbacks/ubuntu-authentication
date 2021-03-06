# Shibboleth Installation

These instructions show you how to setup Shibboleth
on an Ubuntu server running Apache 2.4

## Table of Contents

* [Prerequisites](#prerequisites)
    * [Apache SSL](#apache-ssl)
    * [Federation Firewall](#federation-firewall)
* [Installation](#installation)
    * Service Provider
    * Apache Module
* [Configuration](#configuration)
    * [Shibboleth SSL](#shibboleth-ssl)
    * [Generate XML](#generate-xml)
    * [Virtual Host](#virtual-host)
    * [Metadata](#metadata)
    * [Production IdP](#production-idp)
* [Addendum](#addendum)
    * [Additional Hosts](#additional-virtual-hosts)
    * [Active Directory Groups](#active-directory-groups)
    * [Logging Out](#logging-out)
    * [Lazy Loading](#lazy-loading)
    * [URL Rewriting](#notes-on-url-rewriting)

## Prerequisites

This document assumes you are already running Ubuntu 16.04 with Apache 2.4

### Apache SSL

Before you begin, make sure you have a valid SSL certificate installed
on the web server. You can [create a certificate signing request][7] and then
[create a Cherwell ticket][cherwell] pasting in the CSR to get one.

### Federation Firewall

`shibd` needs to download the IdP (Identity Provider) metadata in order to start up correctly,
so make sure there is no firewall blocking traffic from your server to
https://federation.uark.edu otherwise you will need to [request a policy][firewall-policy-form]
for TCP port 443.

You can check the firewall with [nmap][nmap].

    sudo apt-get install nmap
    nmap -Pn -p 443 federation.uark.edu

## Installation

Install the [service provider][1] and [Apache web server module][2].

    sudo apt install shibboleth-sp2-schemas libapache2-mod-shib2

## Configuration

There are a number of steps required to configure Shibboleth.

### Shibboleth SSL

[Download the script `keygen.sh`][8] and make it executable.

    chmod +x keygen.sh

Run the script with `sudo` to make the certificate/key pair for `_shibd` user.

```
sudo ./keygen.sh -o /etc/shibboleth -u _shibd -g _shibd
```

### Generate XML

Open https://itsforms.uark.edu/shib2xml and under `Entity id` type
`https://{YOUR HOSTNAME}/shibboleth` and click <kbd>Generate</kbd>

Replace `/etc/shibboleth/shibboleth2.xml` with generated file.

**Note** that by default, shib will use the TEST IdP (acmex.uark.edu).

[Uncomment][10] the "Examples of LDAP-based attributes" in `/etc/shibboleth/attribute-map.xml`
on roughly lines 88 to 142 by deleting the enclosing lines that contain `<!--` and `-->` around
the `Attribute` tags, such as:

```xml
<Attribute name="urn:mace:dir:attribute-def:cn" id="cn"/>
```

### Virtual Host

Create a folder in your web server directory root called `secure`
Inside you can create an `.htaccess` file with these contents:

    AuthType shibboleth
    ShibRequestSetting requireSession 1
    Require valid-user

**Note** you must have enabled [`AllowOverride AuthConfig` or `AllowOverride All`][3]
in the primary virtual host configuration file.

Alternatively, you can create a [`Directory`][4] or a [`Location`][5] tag
in the configuration file with the same contents.
This is preferred for performance reasons because Apache will not need to
parse the `.htaccess` file on every request.

Start/restart the services.

    sudo service shibd restart
    sudo service apache2 restart

In browser, verify `https://{YOUR HOSTNAME}/secure` redirects to `acmex.uark.edu`
(you will receive an error page at this point).

### Metadata

Download the Metadata from `https://{YOUR HOSTNAME}/Shibboleth.sso/Metadata`

[Create a Cherwell ticket][cherwell] requesting your site Metadata be added to the IdP
server. Attach the downloaded Metadata to the ticket.

Once entered into IdP servers, the change takes about 15 minutes to propagate.

### Production IdP

Edit `/etc/shibboleth/shibboleth2.xml` to use the production IdP server instead
of the test server by changing line 46 from:

```xml
<SSO entityID="https://acmex.uark.edu/idp/shibboleth">
```

to:

```xml
<SSO entityID="https://idp.uark.edu/idp/shibboleth">
```

Restart the services.

    sudo service shibd restart
    sudo service apache2 restart

Visit `https://{YOUR HOSTNAME}/secure` again, and you should get the login page.

## Addendum

A minimally viable installation should be running at this point.
The following sections cover additional common configurations.

### Additional Virtual Hosts

For each domain name, you’ll need to set up an `ApplicationOverride` in
`/etc/shibboleth/shibboleth2.xml` ***at the end*** of the `ApplicationDefaults` tag.
It requires an application `id` unique to your server, and an `entityId` unique
to the IdP. By convention, this is the virtual host name with `/shibboleth`
appended to the end. For example:

```xml
<ApplicationOverride id="example" entityID="https://example.uark.edu/shibboleth"/>
```

Again you must generate the Metadata for each virtual host by visiting
`https://{YOUR*OTHER*HOSTNAME}/Shibboleth.sso/Metadata` and then submitting that
in a [Cherwell ticket][cherwell] to be added to the IdP server.

Then you must declare the override `applicationID` in the document root of the
virtual host.

This must be in a [`<Directory /path/to/site/root>`][4] or a [`<Location />`][5]
tag or in an `.htaccess` in the root folder of the site.

    ShibRequestSetting applicationID example

Restart the services.

    sudo service shibd restart
    sudo service apache2 restart

### Active Directory Groups

Aside from `require valid-user` the real power lies in leveraging active directory
security groups for access control. In shibboleth, these are referred to as "entitlements"
and you can take a look at them by reading what's been populated into the php `$_SERVER`
variable after login.

```php
<?php print_r($_SERVER['entitlement']);
```

Here is a more [refined snippet][apache-print-entitlements].

For example, if you wanted to limit access to a location for just the members of `WCOB-JazzUsers`
then your `.htaccess` require directive would look like this:

    AuthType shibboleth
    ShibRequestSetting requireSession 1
    Require shib-attr entitlement "urn:mace:uark.edu:ADGroups:Walton College:Security Groups:WCOB-JazzUsers"

**Note** that currently Shibboleth only populates explicit group memberships.
Therefore you cannot enforce authorization via effective group membership rollups.
For example, if `JonUser` is a member of `FooGroup` and `FooGroup` is a member of `BarGroup`
then `JonUser` will only have the entitlement `FooGroup` and there is no way to
verify that `JonUser` is an effective member of `BarGroup` by virtue of rollups.
If you want to authorize access to a resource by the membership in `BarGroup`
then `JonUser` must be an explicit member of `BarGroup` to show that entitlement.
This is a significant limitation in lieu of common Windows access control processes.
Hopefully, support for recursive group memberships will be implemented in the future.

**Note** the distinction between Active Directory security groups and distribution groups.
Prior to the Office365 cloud migration, both memberships were available as entitlements.
However, now only on-premises security groups will be populated as entitlements.
Do not use distribution groups for authorization.

### Logging Out

To logout of the IdP server, the URL is https://idp.uark.edu/idp/exit.jsp

So to logout of your server, the url would be:

    https://{YOUR HOSTNAME}/Shibboleth.sso/Logout?return=https%3A%2F%2Fidp.uark.edu%2Fidp%2Fexit.jsp

Return parameter being url encoded. This will destroy their session on the SP,
then send them to the IdP and destroy their session there. They may, however,
still have active sessions on other SPs that will not be destroyed.

### Lazy Loading

Sometimes you want locations available to the public with an optional login.
In other words, you display the same page to everyone, but those who are logged in might see additional
content, such as their username displayed in the navigation menu.
This is called lazy shib. In the root of your site, you will need to `requireSession false` and `require shibboleth`

```xml
<Location />
 AuthType Shibboleth
 ShibRequestSetting requireSession false
 Require shibboleth
</Location>
```

You may still secure other locations normally by requiring the session and any other constraints,
such as a specific user or entitlement.

```xml
<Location /secure>
 AuthType Shibboleth
 ShibRequestSetting requireSession true
 Require valid-user
</Location>
```

### Notes on URL Rewriting

If you're using an application router to handle requests, as in Wordpress, Laravel, etc.
then you'll probably have some rewrite rules to send all non-existent files to your router page. e.g.

    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^ index.php [L]

In some cases, you may find that this rewrite rule interferes with Shibboleth's locations.
The solution is to place the Shibboleth directives in a higher precedent block than these.
For example, `Location` takes precedence over `Directory` so include a `Location` block in
the virtual host configuration file with the Shibboleth directives and place the redirects
in the `Directory` block.

Alternatively, exclude the condition to rewrite paths for Shibboleth. e.g.

    RewriteCond %{REQUEST_URI} !^/Shibboleth
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^ index.php [L]

[1]:https://shibboleth.net/products/service-provider.html
[2]:https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPApacheConfig
[3]:https://httpd.apache.org/docs/current/mod/core.html#allowoverride
[4]:https://httpd.apache.org/docs/current/mod/core.html#directory
[5]:https://httpd.apache.org/docs/current/mod/core.html#location
[cherwell]:https://help.uark.edu
[7]:https://github.com/jpuck/openssl.csr.bash
[8]:./keygen.sh
[firewall-policy-form]:https://help.uark.edu/CherwellPortal/ITHelpPortal/Command/OneStep.LaunchOneStep?Name=Firewall%20Request
[10]:http://stackoverflow.com/a/2757409/4233593
[apache-print-entitlements]:https://github.com/razorbacks/print-apache-shibboleth-entitlements
[nmap]:https://nmap.org
