# LDAPS authentication with Wordpress

Tested on Wordpress 4.7.3 with Ubuntu 16.04.1 LTS

This uses [a wordpress plugin][0] by [clifgriffin/simple-ldap-login][1]

Requires `php-ldap`

    sudo apt install php-ldap

This should create a file called `/etc/ldap/ldap.conf` in which an environment variable is set
pointing to the system trusted SSL certificate bundle. If you already have this file from a different
installation configuration, then you'll need to ensure the variable is set. You can also set that
to a different path if you'd like to use a special certificate, like a self-signed one.

    # TLS certificates (needed for GnuTLS)
    TLS_CACERT	/etc/ssl/certs/ca-certificates.crt

Download the master branch or latest release ([*greater* than 1.6.0][2]) to your wordpress plugins folder

    cd wp-content/plugins/
    git clone https://github.com/clifgriffin/simple-ldap-login.git

Activate the plugin through the 'Plugins' menu in WordPress.

Update the settings by going to 'Settings' -> 'Simple LDAP Login'

| Attribute            | Value                    |
| ---------------------|--------------------------|
| Account Suffix       | @uark.edu                |
| Base DN              | ou=people,dc=uark,dc=edu |
| Domain Controller(s) | ldaps://ds.uark.edu      |

![basic settings][3]

![advanced settings][4]

[0]:https://wordpress.org/plugins/simple-ldap-login/
[1]:https://github.com/clifgriffin/simple-ldap-login
[2]:https://github.com/clifgriffin/simple-ldap-login/issues/36
[3]:./images/wp-basic.png
[4]:./images/wp-advanced.png
