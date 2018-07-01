#Virtualbox für Mailserver
Vagrant.configure("2") do |config|
  #config.ssh.username = "admin"
  #config.ssh.password = "admin"
  config.vm.define "mailserver" do |mailserver|
  mailserver.vm.box = "debian/stretch64"
      mailserver.vm.hostname = "mailserver"
      mailserver.vm.network "private_network", ip: "192.168.7.3"
    mailserver.vm.provider "virtualbox" do |vb|
	    vb.memory = "1024"
    end
    mailserver.vm.provision "shell", inline: <<-SHELL
        #Hostname wird angepasst
        sudo apt-get update
        sudo apt-get upgrade
        sudo hostnamectl set-hostname --static mail
        #Hostname wird angepasst
        sudo cat /vagrant/test > /etc/hosts
        sudo echo $(hostname -f) > /etc/mailname

        #Unbound wird installiert
        sudo apt install -y unbound
        #DNSSEC Root Key wird aktuallisiert und Unbound wird neu geladen
        sudo su -c "unbound-anchor -a /var/lib/unbound/root.key" - unbound
        sudo systemctl reload unbound

        
        #Certbot installieren
        sudo apt install -y certbot
        #Neues Zertifikat erstellen mit folgenden Befehl (nach der Installation)
        certbot certonly --standalone --rsa-key-size 4096 -d mail.mysystems.tld -d imap.mysystems.tld -d smtp.mysystems.tld
        #MySQL Datenbank einrichten
        sudo apt install -y mariadb-server
        #DB und Tables werden erstellt
        sudo mysql < /vagrant/DB.sql

        #vmail-Benutzer und Verzeichnis einrichten
        sudo mkdir /var/vmail
        sudo adduser --disabled-login --gecos --disabled-password --home /var/vmail vmail
        sudo mkdir /var/vmail/mailboxes
        sudo mkdir -p /var/vmail/sieve/global
        sudo chown -R vmail /var/vmail
        sudo chgrp -R vmail /var/vmail
        sudo chmod -R 770 /var/vmail

        #Apache mit SSL einrichten
        sudo apt-get install -y apache2 openssl
        sudo openssl req -new -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt

        #Dovecot installieren und konfigurieren
        sudo apt install -y dovecot-core dovecot-imapd dovecot-lmtpd dovecot-mysql dovecot-sieve dovecot-managesieved
        sudo systemctl stop dovecot
        sudo rm -r /etc/dovecot/*
        sudo cp /vagrant/dovecot.conf /etc/dovecot
        sudo cp /vagrant/dovecot-sql.conf /etc/dovecot
        sudo cd /etc/dovecot/
        sudo chmod 440 dovecot-sql.conf

        #Spam Script
        sudo cp /vagrant/spam-global.sieve /var/vmail/sieve/global/

        #Spam Learning Scripts
        sudo cp /vagrant/learn-spam.sieve /var/vmail/sieve/global/
        sudo cp /vagrant/learn-ham.sieve /var/vmail/sieve/global/

        #Postfix installieren und Konfigurieren
        sudo debconf-set-selections <<< "postfix postfix/mailname string mail.mysystems.tld"
        sudo debconf-set-selections <<< "postfix postfix/main_mailer_type string 'Internet Site'"
        sudo apt-get install -y postfix
        sudo systemctl stop postfix
        sudo cd /etc/postfix
        sudo rm -r sasl
        sudo rm master.cf main.cf.proto master.cf.proto
        sudo cp /vagrant/main.cf /etc/postfix

        #Entfernt Datenschutz-relevante Header aus E-Mails von MTUAs
        sudo cp /vagrant/submission_header_cleanup /etc/postfix/submission_header_cleanup

        #SQL-Konfigurieren
        sudo mkdir /etc/postfix/sql && cd $_
        sudo cp -a /vagrant/SQL-Konfig/. /etc/postfix/sql
        sudo chmod -R 640 /etc/postfix/sql

        #Postfix Konfig
        touch /etc/postfix/without_ptr
        touch /etc/postfix/postscreen_access
        sudo postmap /etc/postfix/without_ptr
        sudo systemctl reload postfix
        sudo postmap /etc/postfix/without_ptr
        sudo newaliases

        sudo apt install -y lsb-release wget
        sudo wget -O- https://rspamd.com/apt-stable/gpg.key | apt-key add -
        sudo echo "deb http://rspamd.com/apt-stable/ $(lsb_release -c -s) main" > /etc/apt/sources.list.d/rspamd.list
        sudo echo "deb-src http://rspamd.com/apt-stable/ $(lsb_release -c -s) main" >> /etc/apt/sources.list.d/rspamd.list
        sudo apt update
        sudo apt install rspamd
        sudo systemctl stop rspamd
        sudo cp /vagrant/dkim_signing.conf /etc/rspamd/local.d/dkim_signing.conf
        sudo cp -R /etc/rspamd/local.d/dkim_signing.conf /etc/rspamd/local.d/arc.conf
        sudo systemctl start rspamd

  SHELL
  end
end
