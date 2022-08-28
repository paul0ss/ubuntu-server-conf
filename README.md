# Konfiguration und Härtung von Ubuntu 20.04 Server

## Installationsprozess  

* Installer: Ubuntu 22.04.2 live server amd64  
* Reserved RAM:4112Mb  
* Reserved memory: 55G  
* Language: English  
* Layout: English (US)Variant: English (US)  
* Type of install: Ubuntu Server (minimized)  
* Proxy: none  
* Mirror: default  
* Storage layout:  

    | Partition | Speicherkapüazität | Dateisystem | Mount Optionen | 
    | :------- | :------: | -------: | -------: |
    | / | 10G | ext4 | Rechts |
    | /boot | 1G | ext4 | rw |
    | /home | 8G | xfs | rw, nosuid, nodev |
    | /var | 15G | ext4 | Rechts |
    | /var/log | 8G | ext4 | nosuid, nodev, noexec |
    | /var/log/audit | 4G | ext4 | nosuid, nodev, noexec |
    | /var/tmp | 4G | ext4 | nosuid, nodev, noexec |
    | SWAP | 4G | swap | sw |

* Name: Andriy Lyubar  
* Server’s name: uhtw  
* Username: securitas94  
* Password: sV56$nP!g9  
* OpenSSH: noinstall  
* Additional packages: none  

**System aktualisieren**
```
> sudo apt-get update && upgrade
```

## Post-Installation Konfiguration
**Notwendige Pakete und das Härtungsskript installieren:**
```
> sudo apt-get -y install git net-tools procps --no-install-recommends
> git clone https://github.com/konstruktoid/hardening.git
```

**In ~/hardening/ubuntu.cfg Datei folgende Parameter einstellen:** 
* FW_ADMIN sollte die IP-Adresse vom Admin sein, von der die SSH-Verbindung hergestellt wird 
* SSH_PORT sollte sich vom Standardport unterscheiden
* ADMINEEMAIL ist die Emailadresse von Admin
* CHANGEME indiziert, dass die Konfigurationsdatei vom Benutzer gelesen und entsprechend verändert wurde

**Skript ausführen:**  
```
> sudo ~/hardening/ubuntu.sh
```

**Letztendlich wird die alte fstab-Datei aus dem Backup eingelesen, sodass der Inhalt aus der alten fstab-Datei, sich vor den Partitionen befindet, die von dem Hardening-Skript definiert wurden. Damit die Änderungen in Kraft treten, muss das System neugestartet werden.**

```
> sudo vim /etc/fstab
:r fstab bck
> sudo reboot
```

**Falls dabei Komplikationen auftreten, kann die Muster-fstab-Datei aus dem Repository benutzt werden.**

## Grub2 Passwort
**Hashwert für das Passwort generieren. Mit dem s-Flag kann die Länge von Salz eingestellt werden.** 
```
> sudo grub-mkpasswd-pbkdf2 -s 10
```
**Den „superuser“ und das gehashte Passwort am Ende der Datei, unter /etc/grub.d/> > > 40_custom, einfügen.**  
> set superusers=’root’  
> password_pbkdf2 root <password-hash>
**Grub Konfiguration aktualisieren.** 
```
> sudo update-gruB
```

## UFW Einrichten
Mit der Freischaltung von den Ports 80 und 443 werden die Verbindungen zu dem später installierten Webserver erlaubt.
```
> sudo ufw allow 80  
> sudo ufw allow 443
```
## SSH
**2-F Authentifizierung**  
Bei der SSH 2-Faktor-Authentifizierung geht es um die Benutzung einer Public-Key Authentifizierungsmethode zusammen mit einem Passwort. Dafür soll ein Schlüsselpaar auf der Seite des Clients erstellt werden und im Nachhinein der öffentliche Schlüssel auf den Server kopiert werden. Bei der Erstellung eines Schlüsselpaars ist zu beachten, dass der private Schlüssel auch mit einem Passwort abgesichert werden muss. Damit im Falle eines Diebstahls, dieser nicht von Dritten benutzt werden kann. Letztendlich muss auch der ssh-Dienst auf dem Server entsprechend konfiguriert werden.  

Client:  
```
> ssh-keygen
> scp -P 1617 C:\Users\Andriy\id_rsa.pub securitas94@192.168.178.159:~/.ssh/
```

## AuditD Zugriffsrechte
1.	Alle oben erwähnte Dateien sollten nur dem root-Benutzer und der root-Gruppe gehören
```
> sudo chown root:root /var/log/audit/*  /etc/audit/* /etc/audit/rules/*
```
2.	Die Änderung der Gruppe muss auch in der Konfigurationsdatei /etc/audit/audit.cfg festgehalten werden
>log_group = root
3.	Nur der root-Benutzer sollte über die Zugriffsrechte verfügen
```
> sudo chmod 600 /var/log/audit/*  /etc/audit/* /etc/audit/rules/*
```
## Apache
**Instllation**  
```
> sudo apt install apache2
```
**Module aktivieren**  
```
> sudo a2enmod headers ssl proxy proxy_html rewrite
```
**HTTP-Methoden einschränken**  
HTTP 1.1 unterstützt viele Methoden die im Normalbetrieb gar nicht gebraucht werden. Manche stellen ein Risiko dar und sollten deswegen abgeschaltet werden. Die erlaubten Methoden wurden in der Hauptkonfigurationsdatei ***/etc/apache2/apache2.conf*** auf GET POST HEAD limitiert.  

>\<LimitExcept GET POST HEAD>  
>deny from all  
>\</LimitExcept>

Zusätzlich wurde die TRACE-Methode explizit in der Sicherheitskonfigurationsdatei ***/etc/apache2/conf-available/security.conf*** deaktiviert.  Diese wird im Regelfall von Webentwicklern für Debugging-Zwecke verwendet. Die Angreifer können sie aber für eine Cross-Site Tracing (XST) Attacke ausnutzen, da sie den kompletten Request, inklusive Cookies, und Autorisierung-Header, in dem Response-Body widerspiegelt.
>TraceEnable off

**Serverinformationen verbergen**  
Um die sensiblen Serverdetails vor den Angreifern zu verbergen, sollten folgende Headereinstellungen in der Sicherheitstkonfigurationsdatei ***/etc/apache2/conf-available/security.conf*** vorgenommen werden:
>ServerTokens prod  
>ServerSignature Off  
>FileETag None  

Directorylisting in ***/etc/apache2/apache2.conf*** ausmachen:
>\<Directory /var/www  
>  Options None  
>\</Directory>  

**Response Header Härtung**  

Betroffene Header:  
* X-Content-Type-Options 
* X-Permitted-Cross-Domain-Policies
* X-XSS-Protection 
* X-Frame-Options 

/etc/apache2/conf-available/security.conf:  
> Header set X-Content-Type-Options: "nosniff"  
> Header set X-Permitted-Cross-Domain-Policies: "master-only"  
> Header set X-XSS-Protection: "1; mode=block"  
> Header set X-Frame-Options: "sameorigin"  

**Verschlüsselung**  
Das Überschreiben vom Basisverzeichnis im ***/etc/apache2/apache2.conf*** erlauben:
>\<Directory /var/www/html>  
>AllowOverride All  
>\</Directory>  

Verzeichnis für die Ablagerung von Zertifikaten und Schlüsseln erstellen:
```
> mkdir /etc/apache2/cert
> cd /etc/apache2/cert
```
Zertifikat und Schlüssel mit OpenSSL generieren:
```
> openssl req -new -newkey rsa:4096 -x509 -sha256 -days 365 -nodes -out apache-certificate.crt -keyout apache.key
```
Konfiguration von dem virtuellen Host anpassen und Umleitung auf Port 443 in ***/etc/apache2/sites-available/000-default.conf*** einrichten:
><VirtualHost *:80>  
        RewriteEngine On  
        RewriteCond %{HTTPS} !=on  
        RewriteRule \^\/\?\(\.\*\) https://%{SERVER_NAME}\/\$1 [R=301,L]  
>\</virtualhost>  
>\<VirtualHost *:443>  
        ServerAdmin webmaster@localhost  
        DocumentRoot /var/www/html  
        ErrorLog \$\{APACHE_LOG_DIR\}/error.log  
        CustomLog \$\{APACHE_LOG_DIR\}/access.log combined  
        SSLEngine on  
        SSLCertificateFile /etc/apache2/cert/apache-certificate.crt  
        SSLCertificateKeyFile /etc/apache2/cert/apache.key  
>\</VirtualHost>

Unsichere SSL Versionen in ***/etc/apache2/mods-available/ssl.conf*** blockieren: 
> SSLProtocol all -SSLv3 -TLSv1 -TLSv1.1  

**Web Application Firewall**  
**Pakete installieren:**
```
> sudo apt install libapache2-mod-security2
```
**Modul aktivieren:**
```
> sudo cp ./modsecurity.conf-recommended/ ./modsecurity.conf
```

/etc/modsecurity/modsecurity.conf:  
>SecRuleEngine On

**OWASP Regeln runterladen und installieren:**
```
> wget https://github.com/coreruleset/coreruleset/archive/refs/tags/v3.3.2.tar.gz
> sha1sum vFileName.zip && echo 63aa8ee3f3c9cb23f5639dd235bac1fa1bc64264
> tar -xvf v3.3.2.tar.gz
> sudo mv coreruleset-3.3.2/crs-setup.conf.example /etc/modsecurity/crs-setup.conf
> sudo mv coreruleset-3.3.2/rules/ /etc/modsecurity/
```
Pfade in den Apacheeinstellungen setzten.  
**/etc/apache2/mods-enabled/security2.conf:**  
>IncludeOptional /etc/modsecurity/*.conf  
>Include /etc/modsecurity/rules/*.conf 

## MySQL
**libpam-tmpdir löschen**
```
> sudo apt remove libpam-tmpdir
> sudo reboot
```
**Installation**
```
> sudo apt install mysql-server
```
**Kontoverwaltung und Zugriffskontrolle**  
| Benutzername | Berechtigungen | Host | Passwort |
| :------- | :------: | -------: | -------: |
| rootkowski1960 | ALL ON *.* | localhost | p64T.?21ab |
| smartuser407 | SELECT, INSERT, UPDATE, DELETE ON ukrainedb| localhost | cVb35-420z |

Zugriff auf die Datenbankverwaltungssystem auf *lokalhost* einschrenken.  
***/etc/mysql/mysql.conf.d/mysqld.cnf:***
>bind-address            = 127.0.0.1

Benutzer laut der forgegebene Tabelle mit eigenen Passwörtern einrichten:
```
mysql> CREATE USER ‘smartuser407’@’localhost’ IDENTIFIED BY ‘cVb35-420z’;
mysql> CREATE DATABASE ukrainedb;
mysql> GRANT SELECT, INSERT, DELETE, UPDATE ON ukrainedb.* TO ‘smartuser407’@’localhost’;
mysql> ALTER USER ‘root’@’localhost’ IDENTIFIED WITH mysql_native_password BY ‘p64T.?21ab’;
mysql> USE mysql;
mysql> UPDATE user SET user = ‘rootkowski1960’ where user = ‘root’;
mysql> FLUSH PRIVILEGES; 
mysql> SHOW GRANTS FOR ‘smartuser407’@’localhost’;
```

**Entfernen von Testdatenbanken und anonymen Benutzerkonten**  
```
> sudo mysql_secure_installation -u rootkowski1960
```
**MySQL mit eingeschränkten Rechten ausführen**  
***/lib/systemd/system/mysql.service:***  
>[Service]
>User=mysql
>Group=mysql

***/etc/mysql/mysql.conf.d/mysqld.cnf:***  
>[mysqld]  
>\#  
>\# \* Basic Settings  
>\#  
>user   =mysql  

**Aufzeichnung von MySQL Befehle deaktivieren**  
```
> rm ~/.mysql_history
> echo ‘export MYSQL_HISTFILE=/dev/null’ >> ~/.profile
> source ~/.profile
```
**Sicherheitskritische Befehle deaktivieren**  
***/etc/mysql/mysql.conf.d/mysqld.cnf***
>[mysqld]  
>\#  
>\# \*Basic Settings  
>\#                                                         
>user            = mysql  
>skip-show-database  
>local-infile=0  

