---
title: Setting up OpenLDAP
date: 2019-07-25
layout: post
author: andrew
image: assets/images/ldap.png
categories: [ linux, "system administration" ]
---

# Setting up OpenLDAP
## Ubuntu 18.04 LTS

    $ sudo apt-get update
    $ sudo apt-get upgrade
    
    $ sudo apt-get instal slapd ldap-utils

Enter an "Administrator" password when prompted, `foobar123`

    $ sudo dpkg-reconfigure slapd

- Omit OpenLDAP Server Configuration ⇒ No
- DNS Domain Name: `ldap.mydomain.com`
- Organization Name: `mydomain`
- Administrator Password: `foobar123`
- Database: MDB
- Remove DB when SlapD is purged? No
- Move old DB? Yes

> You can also edit the `/etc/ldap/ldap.conf` file to make these changes.

## Create your OUs

    # ldap_setup.ldif
    
    #Create "People" OU
    dn: ou=People,dc=ldap,dc=mydomain,dc=com
    objectClass: organizationalUnit
    ou: People
    
    # Create "Groups" OU
    dn: ou=Groups,dc=ldap,dc=mydomain,dc=com
    objectClass: organizationalUnit
    ou: Groups
    
    # Create "IT" Group OU
    dn: cn=it,ou=Groups,dc=ldap,dc=mydomain,dc=com
    objectClass: posixGroup
    cn: it
    gidNumber: 5000
    
    # Create "Accounting" Group OU
    dn: cn=accounting,ou=Groups,dc=ldap,dc=mydomain,dc=com
    objectClass: posixGroup
    cn: accounting
    gidNumber: 5001

Then import them:

    $ ldapadd \
        -x \                             #=> Simple Auth (not SASL)
        -D cn=admin,dc=ldap,dc=mydomain,dc=com \  #=> Use this account to make changes
        -W \                             #=> Use "Simple Authentication" (ask for password on CLI)
        -f ldap_setup.ldif               #=> File to use, see above
    
    Enter LDAP Password: [foobar123]
    adding new entry "ou=People,dc=ldap,dc=mydomain,dc=com"
    
    adding new entry "ou=Groups,dc=ldap,dc=mydomain,dc=com"
    
    adding new entry "cn=it,ou=Groups,dc=ldap,dc=mydomain,dc=com"

## Adding a user

    # ldap_aduss.ldif
    dn: uid=aduss,ou=People,dc=ldap,dc=mydomain,dc=com
    objectClass: inetOrgPerson
    objectClass: posixAccount
    objectClass: shadowAccount
    uid: aduss
    sn: Duss
    givenName: Andy
    cn: Andy Duss
    displayName: aduss
    uidNumber: 1000
    gidNumber: 1000
    userPassword: apple123
    gecos: Andy Duss
    loginShell: /bin/bash
    homeDirectory: /home/mbp/aduss

and import again:

    $ ldapadd -x -D cn=admin,dc=ldap,dc=mydomain,dc=com -W -f ldap_aduss.ldif 
    
    Enter LDAP Password: [foobar123]
    adding new entry "uid=aduss,ou=People,dc=ldap,dc=mydomain,dc=com"

## Query for User

    $ ldapsearch \
        -x \                                   #=> Simple Authentication
        -LLL \                                 #=> Removes unnecessary output, see below
        -b dc=ldap,dc=mydomain,dc=com \   #=> Searchbase, start the ldap query here
        'uid=aduss' cn gidNumber               #=> Search for 'uid=aduss', display 'cn' and 'gidNumber'

Digging into what `-LLL` does:

    $ man ldapsearch
    
    -L     Search results are display in LDAP Data Interchange Format  detailed  in
           ldif(5).  A single -L restricts the output to LDIFv1.
           A  second  -L  disables  comments.  A third -L disables printing of the
           LDIF version.  The default is to use an extended version of LDIF.

## Changing a User's Password

    $ ldappasswd \
        -x \
        -D "cn=admin,dc=ldap,dc=mydomain,dc=com" \         #=> Use this account to make changes
        -W \                                                    #=> Supply password for '-D' account via input
        -S \                                                    #=> Supply new user password via input
        "uid=aduss,ou=People,dc=ldap,dc=mydomain,dc=com"   #=> Object to change
    
    New password: [aster123]
    Re-enter new password: [aster123]
    Enter LDAP Password: [foobar123]
    
    (Nothing returned if successful)
