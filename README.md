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

## 1. Paket herunterladenn und installieren
Hiermit wird die aktuellste Version geladen und ins "Autostart" gelegt
```
VERSION=$(curl --silent "https://api.github.com/repos/juanfont/headscale/releases/latest"|grep '"tag_name"'|sed -E 's/.*"([^"]+)".*/\1/'|sed 's/v//')
wget https://github.com/juanfont/headscale/releases/download/v${VERSION}/headscale_${VERSION}_linux_amd64.deb
sudo apt install -f ./headscale_${VERSION}_linux_amd64.deb
systemctl enable headscale
```
## 2. Konfiguration
Die Konfigurationsdatei an 2-3 Stellen bearbeiten.
Das `server_url: http://test.1blu.de:8080`, `listen_addr:0.0.0.0:8080` und vielleicht noch `base_domain=meine` die angepasst werden muss.

```
nano /etc/headscale/config.yaml
```

Anschliessend den Dienst neustarten

```
systemctl restart headscale.service
```

## 3. Nginx Proxy für Headscale konfigurieren
Den Nginx Webserver installieren und anpassen. `server_name test.1blu.de`; `proxy_pass  http://localhost:8080;`.

```
apt install nginx
nano /etc/nginx/conf.d/headscale.conf
sudo nginx  -t
```

Wenn alles in Ordnung dann `server_url: http://test.1blu.de:80` anpassen

```
nano /etc/headscale/config.yaml
```

## 4. Headscale mit SSL Zertifikaten absichern





## Nützliche kommandos
Den Status kann man dann bereits wie folgt überprüfen.

`status headscale.service`

Ports überprüfen.

ss -tunelp | egrep '9080|9090'
```
```

