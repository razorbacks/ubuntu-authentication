Instructions for setting up integrated authentication via LDAP for Ubuntu.

> **NOTE:** This uses the insecure ldap protocol instead of secure ldaps,
so passwords are transmitted across the network in plain text.

Install dependencies:

    sudo apt install -y ldap-utils libpam-ldap libnss-ldap nscd

Edit 3 files:

1. `/etc/ldap/ldap.conf`

  Add these lines:

      URI ldap://ds.uark.edu/
      BASE ou=people,dc=uark,dc=edu
      TLS_CACERTDIR /etc/openldap/cacerts

2. `/etc/ldap.conf`

  Exactly the same as step 1:

      URI ldap://ds.uark.edu/
      BASE ou=people,dc=uark,dc=edu
      TLS_CACERTDIR /etc/openldap/cacerts

3. `/etc/nsswitch.conf`

  Edit these 3 lines:

      passwd:         compat
      group:          compat
      shadow:         compat

  Add `ldap` to the end like this:

      passwd:         compat ldap
      group:          compat ldap
      shadow:         compat ldap

Now restart the service:

    /etc/init.d/nscd restart
