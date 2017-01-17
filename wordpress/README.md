# LDAPS authentication with Wordpress

Tested on Wordpress 4.7-beta4 with Ubuntu 16.04.1 LTS

This uses [a wordpress plugin][0] by [clifgriffin/simple-ldap-login][1]

Requires `php-ldap`

    sudo apt install php-ldap

Download the master branch or latest release ([*greater* than 1.6.0][2]) to your wordpress plugins folder

    cd wp-content/plugins/
    git clone https://github.com/clifgriffin/simple-ldap-login.git

Activate the plugin through the 'Plugins' menu in WordPress.

Update the settings by going to 'Settings' -> 'Simple LDAP Login'

![basic settings][3]

![advanced settings][4]

[0]:https://wordpress.org/plugins/simple-ldap-login/
[1]:https://github.com/clifgriffin/simple-ldap-login
[2]:https://github.com/clifgriffin/simple-ldap-login/issues/36
[3]:./images/wp-basic.png
[4]:./images/wp-advanced.png
