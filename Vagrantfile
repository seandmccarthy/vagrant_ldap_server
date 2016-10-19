# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANT_VERSION = 2
Vagrant.configure(VAGRANT_VERSION) do |config|
  config.vm.box = "relativkreativ/centos-7-minimal"
  config.vm.provider "virtualbox" do |v|
    v.name = "ldap_server_test"
    v.memory = 1024
    v.cpus = 1
  end

  config.vm.network "private_network", ip: "172.28.128.123"
  config.ssh.forward_agent = true
  config.ssh.insert_key = false

  # Operations requiring privileged user
  config.vm.provision "shell", inline: <<-SHELL
    yum makecache fast
    yum -y install openldap openldap-servers openldap-clients
    systemctl stop firewalld
    systemctl disable firewalld
    cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG 
    chown ldap. /var/lib/ldap/DB_CONFIG
    systemctl start slapd
    systemctl enable slapd
    ldapadd -Q -H 'ldapi:///' -f /etc/openldap/schema/core.ldif 
    ldapadd -Q -H 'ldapi:///' -f /etc/openldap/schema/cosine.ldif 
    ldapadd -Q  -H 'ldapi:///' -f /etc/openldap/schema/inetorgperson.ldif 
    ldapmodify -Q -Y EXTERNAL -H 'ldapi:///' -f /vagrant/ldif/database_suffix.ldif
    ldapmodify -Q -Y EXTERNAL -H 'ldapi:///' -f /vagrant/ldif/rootdn.ldif
    ldapmodify -Q -Y EXTERNAL -H 'ldapi:///' -f /vagrant/ldif/root_password.ldif
    ldapadd -x -w test -D 'cn=admin,dc=example,dc=com' -H ldapi:/// -f /vagrant/ldif/example_dc.ldif
    ldapadd -x -w test -D 'cn=admin,dc=example,dc=com' -H ldapi:/// -f /vagrant/ldif/user_orgunit.ldif
    ldapadd -x -w test -D 'cn=admin,dc=example,dc=com' -H ldapi:/// -f /vagrant/ldif/userlist.ldif
  SHELL
end
