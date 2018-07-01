# Mailserver
Folgende Applikationen werden mit diesem Vagranfile installiert:
* Debian 9 “Stretch”
* Postfix
* Dovecot
* Rspamd
* Redis
* MariaDB

# Guide
Mit dem folgenden Befehl den Server aufsetzen:
  $ Vagrant up
  
# Eine Verbindung zum Mailserver herstellen
Eine Verbindung könnt ihr über jeden IMAP-fähigen E-Mail-Client und den folgenden Verbindungsparametern herstellen:
* IMAP-Server: mail.mysystems.tld | Port: 143 | STARTTLS
* SMTP-Server: mail.mysystems.tld | Port: 587 | STARTTLS
* (optional Managesieve: mail.mysystems.tld | Port: 4190 | STARTTLS)
* Benutzername & Passwort eingeben

# Spamfilter testen
Man kann den Spamfilter testen in dem man einfach ein Text mit zufälligen Zahlen und Zeichen schickt Beispiel:
   XJS*C4JDBQADN1.NSBN3*2IDNEN*GTUBE-STANDARD-ANTI-UBE-TEST-EMAIL*C.34X
