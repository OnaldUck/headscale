# headscale
headscale ist eine open source, self-hosted Implementation von Tailscale control server.
Das hier basiert auf der Anleitung von computingforgeeks https://computingforgeeks.com/install-and-configure-headscale-on-ubuntu/

## Voraussetzung
Irgendein vServer, Debian oder Ubunt installieren lassen un paar kleine Tools installieren. SUDO braucht man theoretisch am Debian nicht, manches Installationsscript aber schon und deswegen kann es mit installiert werden.

```
apt install htop nano mc sudo
```

Auf Debian die Startseite und Apache entfernen
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
Die Konfigurationsdatei an zwei oder drei Stellen bearbeiten. 
Das `server_url: http://test.1blu.de:8080`, `listen_addr:0.0.0.0:8080` und vielleicht noch `base_domain=meine` die angepasst werden muss. Achtung wenn man **test.1blu.de** so muss es angepasst werden.

```
nano /etc/headscale/config.yaml
```

Anschliessend den Dienst neustarten

```
systemctl restart headscale.service
```

## 3. Nginx Proxy für Headscale konfigurieren
Den Nginx Webserver installieren und an zwei Stellen anpassen `server_name test.1blu.de`; `proxy_pass  http://localhost:8080;`m dann noch mit `nginx -t` die Korrektheit überprüfen.

```
apt install nginx
nano /etc/nginx/conf.d/headscale.conf
nginx -t
```

Wenn alles in Ordnung dann `server_url: http://test.1blu.de:80` anpassen

```
nano /etc/headscale/config.yaml
```

## 4. Headscale mit SSL Zertifikaten absichern
Wir machen das mit welchen von LetsEncrypt. Dazu installieren wird CERTBOT.
```
apt update && apt install snapd
snap install --classic certbot
```

```
DOMAIN=test.1blu.de
certbot --register-unsafely-without-email --agree-tos --nginx -d $DOMAIN
```

Das CERTBOT Tool fügt die erstellten Zertifikte selbst in die Nginx Konfigurtion. Der Webserver muss nur durchgestatet werden.
```
systemctl restart nginx
```
Die headscale Konfiguration muss jetzt entsprechend auf den HTTPS-Port (**443**) ummgestellt werden `server_url: https://test.1blu.de:443` und der headscale Serer neugestartet werden. (__muss schauen ob die 443 überhaupt angegeben werden muss, wenn vorne https steht__)
```
nano /etc/headscale/config.yaml
systemctl restart headscale
```

## 5. Geräte zum headscale mash aufnehmen
Zunächst müssen grundsätzlich irgenwelche Benutzer erstellt werden
```
headscale users create admin
headscale users create benutzer
```
### Preauthkeys zum registrieren erstellen
Das Ganze geschieht in zwei Schritten **am headscale Server** und **am Klientcomputer**
```
headscale --user benutzer preauthkeys create --reusable --expiration 24h
```
Den so generierten Schlüssel "mitnehmen" un an der Klientmaschien zum Einloggen benutzen
```
tailscale up --login-server https://test.1blu.de:443 --authkey 6756756757sdadsadasdasdh8978977890890
```

### Nomal registrieren
Am Klientrechner 
```
tailscale up --login-server https://test.1blu.de:443
```
Dann kommt als Antwort, so was in der Richtung.
```
To authenticate, visit:
	https://test.1blu.de:443/register/nodekey:6756756757sdadsadasdasdh8978977890890
```
Das muss man in den Webbrowser einfügen und erhält eine Anwort, wie man die Maschine am Headscale registriert.
```
headscale nodes register --user benutzer --key mkey:6756756757sdadsadasdasdh8978977890890
```



## Nützliche kommandos
Den Status kann man dann bereits wie folgt überprüfen.

`status headscale.service`
`tail -f /var/log/syslog`
`ss -tunelp | egrep '9080|9090' - Ports überprüfen.


headscale node delete -i <ID> - Delete a node in your network.
headscale node move  -i  <ID> -u <New-User> - Move node to another user
headscale node rename  -i  <ID>  <NEW_NAME> - Rename a machine in your network
headscale node expire -i <ID> - Expire (log out) a machine in your network
headscale preauthkeys --user <username> list - List Pre-auth keys:

```
```

