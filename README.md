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
```

## Paket herunterladenn und installieren
Hiermit wird die aktuellste Version geladen
```
VERSION=$(curl --silent "https://api.github.com/repos/juanfont/headscale/releases/latest"|grep '"tag_name"'|sed -E 's/.*"([^"]+)".*/\1/'|sed 's/v//')
wget https://github.com/juanfont/headscale/releases/download/v${VERSION}/headscale_${VERSION}_linux_amd64.deb
sudo apt install -f ./headscale_${VERSION}_linux_amd64.deb
```
Falls es nicht funktionieren sollte, dan einafach selbst die gewünschte Version herunterladen. 
```
wget --output-document=headscale.deb https://github.com/juanfont/headscale/releases/download/v0.22.3/headscale_0.22.3_linux_amd64.deb
wget --output-document=headscale.deb https://github.com/juanfont/headscale/releases/download/v0.23.0-alpha3/headscale_0.23.0-alpha3_linux_amd64.deb
```

