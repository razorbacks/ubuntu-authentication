# Shibboleth Single Sign On

Before you begin, make sure you have a valid SSL certificate installed
on the web server. You can [create a certificate signing request][7] and then
[create an AskIT ticket][6] pasting in the CSR to get one.

------------------------------------------------------------------------------

Install the [service provider][1] and [Apache web server module][2].

    sudo apt install shibboleth-sp2-schemas libapache2-mod-shib2

Open https://itsforms.uark.edu/shib2xml and under `Entity id` type
`https://{YOUR HOSTNAME}/shibboleth` and click `Generate`

Replace `/etc/shibboleth/shibboleth2.xml` with generated file.

Note that by default, shib will use the TEST IdP (acmex.uark.edu)

In `/etc/shibboleth/attribute-map.xml` Uncomment "common LDAP attributes",
roughly lines 88 to 142.

Create a folder in your web server directory root called `secure`
Inside you can create an `.htaccess` file with these contents:

    AuthType shibboleth
    ShibRequestSetting requireSession 1
    Require valid-user

**Note** you must have enabled [`AllowOverride AuthConfig` or `AllowOverride All`][3]
in the primary virtual host configuration file. Alternatively, you can create a
[`Directory`][4] or a [`Location`][5] tag in the configuration file with the
same contents.

Start/restart the services.

    sudo service shibd start
    sudo service apache2 restart

In browser, verify `https://{YOUR HOSTNAME}/secure` redirects to `acmex.uark.edu`
(you will receive an error page at this point).

Download the Metadata from `https://{YOUR HOSTNAME}/Shibboleth.sso/Metadata`

[Create an AskIT ticket][6] requesting your site Metadata be added to the IdP
server. Attach the downloaded Metadata to the ticket.

Once entered into IdP servers, the change takes about 15 minutes to propagate.

Visit `https://{YOUR HOSTNAME}/secure` again, and you should get the login page.

If all works, edit `/etc/shibboleth/shibboleth2.xml` and change line 46 from:

    <SSO entityID="https://acmex.uark.edu/idp/shibboleth">

to:

    <SSO entityID="https://idp.uark.edu/idp/shibboleth">

Restart the services.

    sudo service shibd restart
    sudo service apache2 restart

[1]:https://shibboleth.net/products/service-provider.html
[2]:https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPApacheConfig
[3]:https://httpd.apache.org/docs/current/mod/core.html#allowoverride
[4]:https://httpd.apache.org/docs/current/mod/core.html#directory
[5]:https://httpd.apache.org/docs/current/mod/core.html#location
[6]:https://askit.uark.edu/
[7]:https://github.com/jpuck/openssl.csr.bash
