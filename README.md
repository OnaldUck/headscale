# headscale
headscale ist eine open source, self-hosted Implementation von Tailscale control server.
Das hier basiert auf der Anleitung von computingforgeeks https://computingforgeeks.com/install-and-configure-headscale-on-ubuntu/

## Voraussetzung
Irgendein vServer, Debian oder Ubunt installieren lassen un paar kleine Tools installieren. SUDO braucht man theoretisch am Debian nicht, manches Installationsscript aber schon und deswegen kann es mit installiert werden.

`apt install htop nano mc sudo`

Auf Debian Startseite und Apache entfernen
```
mv /var/www/html/index.html index.org-html
systemctl status apache2
systemctl is-enabled apache2
systemctl disable apache2
systemctl stop apache2
systemctl mask apache2
apt remove apache2
## Reset der VMID
```



Bei Änderungen an der VMX Datei kann man auch den host-Dienst durchstarten und sich das hin- und her-registrieren der Maschine sparen.


```
/etc/init.d/hostd restart
/etc/init.d/vpxa restart
```
