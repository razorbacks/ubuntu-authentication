Instructions for setting up integrated authentication via LDAP for Ubuntu 16.04.

Install dependencies:

    sudo apt install -y ldap-utils libpam-ldap libnss-ldap nscd

Edit 3 files:

1. `/etc/ldap/ldap.conf`

  Add these lines:

      URI ldaps://ds.uark.edu/
      BASE ou=people,dc=uark,dc=edu
      tls_cacertfile /etc/ssl/certs/ca-certificates.crt

2. `/etc/ldap.conf`

  Exactly the same as step 1:

      URI ldaps://ds.uark.edu/
      BASE ou=people,dc=uark,dc=edu
      tls_cacertfile /etc/ssl/certs/ca-certificates.crt

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
