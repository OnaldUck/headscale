# headscale
headscale ist eine open source, self-hosted Implementation von Tailscale control server.
Das hier basiert auf der Anleitung von computingforgeeks https://computingforgeeks.com/install-and-configure-headscale-on-ubuntu/

## Voraussetzung
Irgendein vServer, Debian oder Ubunt installieren lassen fertg
`apt install htop nano mc`


## Reset der VMID
Beim registrieren, deregistrieren der Maschinen wird die Nummer immer erhöt. Vor dem registrieren des Maschinen **hostd++ neustarten.

Bei Änderungen an der VMX Datei kann man auch den host-Dienst durchstarten und sich das hin- und her-registrieren der Maschine sparen.


```
/etc/init.d/hostd restart
/etc/init.d/vpxa restart
```
