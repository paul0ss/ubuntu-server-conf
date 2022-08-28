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





## Post-Installation Konfiguration
Notwendige Pakete und das Härtungsskript installieren:

    sudo apt-get -y install git net-tools procps --no-install-recommends  
    git clone https://github.com/konstruktoid/hardening.git  

In ~/hardening/ubuntu.cfg Datei folgende Parameter einstellen:  
* FW_ADMIN sollte die IP-Adresse vom Admin sein, von der die SSH-Verbindung hergestellt wird 
* SSH_PORT sollte sich vom Standardport unterscheiden
* ADMINEEMAIL ist die Emailadresse von Admin
* CHANGEME indiziert, dass die Konfigurationsdatei vom Benutzer gelesen und entsprechend verändert wurde

Skript ausführen:

    sudo ~/hardening/ubuntu.sh

/etc/fstab Datei aus dem Backup wiederherstellen:

## Grub2 Passwort
1) Hashwert für das Passwort generieren. Mit dem s-Flag kann die Länge von Salz eingestellt werden.
2) Den „superuser“ und das gehashte Passwort am Ende der Datei, unter /etc/grub.d/40_custom, einfügen.
3) Grub Konfiguration aktualisieren.  
    sudo grub-mkpasswd-pbkdf2 -s 10
    sudo update-grub


## Dateisystem und Partitionierung
## UFW Einrichten
## SSH
## Apache
## MySQL