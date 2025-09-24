# Headscale
Headscale ist eine open source, self-hosted Implementation von Tailscale Control Server.
Das hier basiert auf der Anleitung von [ComputingForGeeks](https://computingforgeeks.com/install-and-configure-headscale-on-ubuntu/)

# Inhaltsverzeichnis

- [Paket herunterladen und installieren](https://github.com/OnaldUck/headscale#Paket-herunterladen-und-installieren)
- [Update](https://github.com/OnaldUck/headscale#Update)
- [Konfiguration](https://github.com/OnaldUck/headscale#Konfiguration)
- [Nodes registrieren](https://github.com/OnaldUck/headscale#Nodes-zum-headscale-mash-aufnehmen)
- [Subnet routes](https://github.com/OnaldUck/headscale#Subnet-routes)
- [Fehlersuche](https://github.com/OnaldUck/headscale#Fehlersuche)
- [Headscale Befehle](https://github.com/OnaldUck/headscale#Headscale-Befehle)


# Voraussetzung
Irgendein vServer mit Debian installieren lassen un paar kleine Tools installieren. SUDO braucht man theoretisch am Debian nicht, manches Installationsscript aber schon und deswegen kann es mit installiert werden.

```
apt install curl htop nano mc sudo wget -y
```

Auf Debian die Startseite und Apache entfernen
```
mv /var/www/html/index.html index.org-html

systemctl disable apache2
systemctl stop apache2
systemctl mask apache2
apt remove apache2
```

# Paket herunterladen und installieren
Hiermit wird die aktuellste Version geladen und ins "Autostart" gelegt
```
VERSION=$(curl --silent "https://api.github.com/repos/juanfont/headscale/releases/latest"|grep '"tag_name"'|sed -E 's/.*"([^"]+)".*/\1/'|sed 's/v//')
wget https://github.com/juanfont/headscale/releases/download/v${VERSION}/headscale_${VERSION}_linux_amd64.deb

apt install -f ./headscale_${VERSION}_linux_amd64.deb

systemctl enable headscale
```

## Update

Identisch, wie die Installation, ggf. muss der Dienst noch gestartet werden
```
systemctl enable headscale
systemctl start headscale
```
# Konfiguration
Die Konfigurationsdatei an zwei oder drei Stellen bearbeiten. 
Das `server_url: http://0.0.0.0:8080`, `listen_addr:0.0.0.0:8080` und vielleicht noch `base_domain=meine` die angepasst werden muss. Achtung  **test.1blu.de** jeweils seinem System entsprechend anpassen.

```
nano /etc/headscale/config.yaml
```

Anschliessend den Dienst neustarten

```
systemctl start headscale
```

## Nginx Proxy für Headscale konfigurieren
Den Nginx Webserver installieren und an zwei Stellen anpassen `server_name test.1blu.de`; `proxy_pass  http://localhost:8080;`m dann noch mit `nginx -t` die Korrektheit überprüfen.

```
apt install nginx
```
Dann **headscale.conf** erstellen
```
nano /etc/nginx/conf.d/headscale.conf
```
Und mit folgenden Inhalt befüllen
```
map $http_upgrade $connection_upgrade {
    default      keep-alive;
    'websocket'  upgrade;
    ''           close;
}

server {
    listen 80;
	listen [::]:80;

    server_name test.1blu.de;

    location / {
        proxy_pass  http://localhost:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        proxy_set_header Host $server_name;
        proxy_redirect http:// https://;
        proxy_buffering off;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $http_x_forwarded_proto;
        add_header Strict-Transport-Security "max-age=15552000; includeSubDomains" always;
    }
}
```

```
nginx -t
```

Wenn Nginx in Ordnung dann kann in der **config.yaml** folgendes angepaßt werden `server_url: http://test.1blu.de:80` anpassen

```
nano /etc/headscale/config.yaml
```

## Headscale mit SSL Zertifikaten absichern
Wir machen das mit welchen von LetsEncrypt. Dazu installieren wird CERTBOT.
```
apt install certbot python3-certbot-nginx

DOMAIN=test.1blu.de
certbot --register-unsafely-without-email --agree-tos --nginx -d $DOMAIN
```

Das CERTBOT Tool fügt die erstellten Zertifikte selbst in die Nginx Konfigurtion. Der Webserver muss nur durchgestatet werden.
```
systemctl restart nginx
```
Die headscale Konfiguration muss jetzt entsprechend auf den HTTPS-Port (**443**) ummgestellt werden `server_url: https://test.1blu.de:443` und der headscale Serer neugestartet werden. _(muss schauen ob die 443 überhaupt angegeben werden muss, wenn vorne https steht)_
```
nano /etc/headscale/config.yaml
systemctl restart headscale
```

# Nodes zum headscale mash aufnehmen
Zunächst müssen grundsätzlich irgenwelche Benutzer erstellt werden
```
headscale users create admin
headscale users create benutzer
```
## Preauthkeys zum registrieren erstellen
Das Ganze geschieht in zwei Schritten **am headscale Server** und **am Klientcomputer**
```
headscale --user benutzer preauthkeys create --reusable --expiration 24h
Seit 2025
headscale preauthkeys create -u 4
```
Den so generierten Schlüssel "mitnehmen" un an der Klientmaschien zum Einloggen benutzen
```
tailscale up --login-server https://test.1blu.de:443 --authkey 6756756757sdadsadasdasdh8978977890890
```

## Nomal registrieren
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

# Subnet routes
Damit man nicht an jede einzelne Maschine Tailscale installierne muss, kann man einen **Subnet router** definieren. Vorzugsweise an einer Mschien die dauerhaft läuft
```
tailscale up --advertise-routes=192.168.0.0/24 --unattended --login-server=https://test.1blu.de:443
```
Es muss nicht nur **Advertised** sondern auch **Enabled** sein, das man wie folgt erledigt. 
```
headscale routes list

headscale routes enable -r 1
```

# Fehlersuche
`tailscale ping 100.64.0.2` - bin ich dirkt oder durch einen DERP Server verbunden

```
pong from hyper2019 (100.64.0.2) via DERP(fra) in 149ms
pong from hyper2019 (100.64.0.2) via DERP(fra) in 22ms
pong from hyper2019 (100.64.0.2) via DERP(fra) in 42ms
pong from hyper2019 (100.64.0.2) via DERP(fra) in 15ms
pong from hyper2019 (100.64.0.2) via DERP(fra) in 27ms
pong from hyper2019 (100.64.0.2) via DERP(fra) in 44ms
pong from hyper2019 (100.64.0.2) via DERP(fra) in 39ms
pong from hyper2019 (100.64.0.2) via DERP(ams) in 248ms
pong from hyper2019 (100.64.0.2) via DERP(ams) in 28ms
pong from hyper2019 (100.64.0.2) via DERP(ams) in 48ms
direct connection not established
```
tailscale ping 100.64.0.2



## Headscale Befehle
Den Status kann man dann bereits wie folgt überprüfen.

`systemctl status headscale.service` - Status abfragen

`tail -f /var/log/syslog` - Log mitschauen

`headscale routes delete -r 1` - Route löschen

`ss -tunelp | egrep '9080|9090'` - Ports überprüfen.

`headscale node delete -i <ID>` - Delete a node in your network.

`headscale node move  -i  <ID> -u <New-User>` - Move node to another user

`headscale node rename  -i  <ID>  <NEW_NAME>` - Rename a machine in your network

`headscale node expire -i <ID>` - Expire (log out) a machine in your network

`headscale preauthkeys --user <username> list` - List Pre-auth keys:

`alias tailscale='/Applications/Tailscale.app/Contents/MacOS/Tailscale'` - tailscale Alias am MacOS erstellen, damit man im Terminal einfach `tailscale status` abfragen kann

`alias tailscale='/Applications/Tailscale.app/Contents/MacOS/Tailscale'` - am MacBook einen Alias erstellen, damit man im Terminal einfach Tailscale CLI benutzen kann

