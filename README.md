# LDAP Server Builder

This repository contains a Vagrant file (and support files) that creates a VM that provides an LDAP server

# Requirements

On your machine (the host) you need to install

* [Vagrant](https://www.vagrantup.com/downloads.html)
* [VirtualBox](https://www.virtualbox.org/wiki/Downloads)

# Setup

Clone this repository to your machine. Change into the directory you cloned into (ldap_server by default), then run:

    $ ssh-add
    $ vagrant up

This will take some time to complete.

# Usage

With the Virtual Machine now built and ready/running, you can create/query/manipulate the LDAP server. It is available at the local private IP of 172.28.128.123 on port 389.

    $ ldapsearch -D 'cn=admin,dc=example,dc=com' -W -x -b 'dc=example,dc=com' -H ldap://172.28.128.123

The auth credentials are:

    dn: cn=admin,dc=example,dc=com
    password: test

One user, testuser, is created by default.

You can connect to the server with Ruby

    irb(main):140:0> require 'net-ldap'
    irb(main):141:0> options = { host: '172.28.128.123', port: 389, auth: { method: :simple, username: 'cn=admin,dc=example,dc=com', password: 'test'}}
    irb(main):142:0> ldap = Net::LDAP.new options

You can query the LDAP server:

    irb(main):150:0> treebase = 'dc=example,dc=com'
    irb(main):151:0> filter = Net::LDAP::Filter.eq( "cn", "*" )
    irb(main):152:0> ldap.search( :base => treebase, :filter => filter ) do |entry|
    irb(main):153:1*   puts "DN: #{entry.dn}"
    irb(main):154:1> end
    DN: cn=testuser,ou=users,dc=example,dc=com

Try binding as a given user:

    irb(main):143:0> ldap.bind_as(base: 'ou=Users,dc=example,dc=com', method: :simple, filter: "(cn=usermanager)", password: 'test')=> [#<Net::LDAP::Entry:0x007f8d7dc48eb8 @myhash={:dn=>["cn=testuser,ou=Users,dc=example,dc=com"], :cn=>["UserManager", "testuser"], :sn=>["Manager"], :objectclass=>["inetOrgPerson"], :userpassword=>["test"], :uid=>["testuser"]}>]

Try adding a user:

    irb(main):155:0> ldap.add(dn: 'cn=foobar,ou=Users,dc=example,dc=com', attributes: {:objectclass=>[ "inetorgperson"], :cn=>"foobar", :sn=>"Bar", :mail=>"foobar@example.com", :userPassword=>"test"})
