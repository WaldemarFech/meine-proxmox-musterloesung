# Erste Testseite: Debian-LXC, nginx und Traefik

Dieses Beispiel führt von einem vorhandenen Proxmox-Host und einer bereits funktionierenden Traefik-Installation zu einer kleinen öffentlichen HTTPS-Seite. Es verwendet die Adressen der Beispielanlage. Die Befehle sind für einen **neuen eigenen Labor-Container** gedacht; sie wurden hier nicht auf der laufenden Anlage ausgeführt.

## Voraussetzungen prüfen

Du brauchst eine eigene Domain, Zugriff auf deren DNS, eine erreichbare Provider-Konsole und einen Host, auf dem Netzwerk und Firewall bereits sicher funktionieren. Lies zuerst [Netzwerk und DNAT](02-netzwerk-und-dnat.md). Prüfe insbesondere:

- `vmbr1` existiert und das Gateway `192.168.100.1` funktioniert für neue Container.
- Die ID `100032` und IP `192.168.100.32` sind auf **deinem** Host frei. Auf unserer Beispielanlage sind sie bereits belegt; dort keinen zweiten Container damit erzeugen.
- Der für deinen Host gewählte Storage und ein Debian-13-Template stehen bereit.
- DNS, Provider-Firewall und gegebenenfalls DNS-Proxy sind für deine Domain eingerichtet; eingehendes TCP 80/443 kann den eigenen Traefik erreichen.
- Traefik hat einen HTTPS-Entrypoint und einen passenden, getesteten Zertifikatsresolver. Resolver- und Entrypoint-Namen unterscheiden sich je Installation.

Auf dem Proxmox-Host helfen diese Abfragen:

```sh
pct list
pvesm status
pveam list local
ip -br address show vmbr1
ip route
```

`local` und `local-lvm` in den folgenden Beispielen sind Storage-Namen und können bei dir anders heißen. `pct config` und Netzwerkausgaben können sensible Kommentare oder öffentliche IP-Adressen enthalten: vor Weitergabe prüfen.

## 1. Container erstellen

Auf der Beispielanlage war bei der Abfrage ein Template mit dem folgenden Namen vorhanden. Wähle auf deinem Host ein verfügbares, aktuelles Debian-13-Template und passe den Wert an. Ist es noch nicht vorhanden, lade es über die Proxmox-Oberfläche oder `pveam` auf den passenden Template-Storage.

```sh
# Beispiel für einen neuen Host: ID, Template, Storage, Bridge und IP prüfen!
pct create 100032 local:vztmpl/debian-13-standard_13.6-1_amd64.tar.zst \
  --hostname tiny-pages --ostype debian --unprivileged 1 \
  --cores 1 --memory 256 --swap 256 --rootfs local-lvm:4 --onboot 1 \
  --net0 name=eth0,bridge=vmbr1,ip=192.168.100.32/24,gw=192.168.100.1 && \
  pct start 100032 && pct status 100032
```

Verwalte den Container zunächst über `pct enter 100032` vom autorisierten Proxmox-Host. Ein zusätzlicher öffentlicher SSH-Port ist dafür nicht nötig. Stimmen Storage, Template, ID oder Netz nicht, vor dem Erstellen anpassen. Ein bereits vorhandener Container darf nicht überschrieben werden.

## 2. nginx installieren und intern testen

Prüfe zuerst ausgehendes Routing/NAT und DNS-Auflösung **im neuen Container**. Falls die Auflösung fehlschlägt, dort stoppen und Netzwerk/DNS untersuchen, bevor Pakete installiert werden:

```sh
pct exec 100032 -- ip route
pct exec 100032 -- getent hosts deb.debian.org
```

Wenn die DNS-Auflösung erfolgreich ist und der Container ausgehend erreichbar ist:

```sh
pct exec 100032 -- apt-get update
pct exec 100032 -- apt-get install -y nginx-light
pct exec 100032 -- systemctl is-active nginx
pct exec 100032 -- ss -ltn

# Vom Host: Antwort des internen Webservers prüfen
curl -I http://192.168.100.32/
```

Die Standardseite ist für einen ersten Verbindungstest ausreichend. Eigene statische Inhalte können anschließend in den konfigurierten Webroot gelegt werden; keine Zugangsdaten dort ablegen. Die interne Adresse ist eine **Backend-Adresse** und kein DNS-Ziel für öffentliche Besucher.

## 3. DNS und Traefik-Router zuordnen

Lege bei deinem DNS-Anbieter eine eigene Test-Subdomain an. Sie zeigt direkt auf die öffentliche Adresse deines VPS oder – falls du einen DNS-/CDN-Proxy verwendest – auf dessen vorgesehenen Weg zum VPS. HTTP-01 benötigt einen passenden öffentlich erreichbaren Port 80; bei DNS-01 muss dein DNS-Anbieter die benötigte API unterstützen. Beide Wege und ihre Zugangsdaten müssen zum eigenen Provider passen.

Die folgende **dynamische Traefik-Datei** zeigt die Zuordnung. Ersetze die Beispiel-Domain, die Backend-IP sowie `websecure` und `letsencrypt` durch die Namen deiner tatsächlich vorhandenen Entrypoints und Resolver:

```yaml
http:
  routers:
    tiny-pages:
      rule: "Host(`test.example.com`)"
      entryPoints:
        - websecure
      service: tiny-pages
      tls:
        certResolver: letsencrypt
  services:
    tiny-pages:
      loadBalancer:
        servers:
          - url: "http://192.168.100.32:80"
```

Diese Datei allein installiert Traefik nicht und legt keinen Resolver an. Der File-Provider muss im statischen Traefik-Setup auf das Verzeichnis zeigen, in dem du die Datei ablegst. Sichere deine bestehende Konfiguration, prüfe die vollständige Konfiguration und übernimm die Datei erst, wenn die Namen eindeutig sind. Die offiziellen Referenzen erklären den [File-Provider](https://doc.traefik.io/traefik/providers/file/) und [HTTPS/ACME](https://doc.traefik.io/traefik/https/acme/).

## 4. Von außen und innen prüfen

```sh
# Auf einem Rechner mit DNS-Auflösung, Domain ersetzen:
nslookup test.example.com
curl -I https://test.example.com/

# Auf dem Proxmox-Host: läuft das Backend?
pct exec 100032 -- systemctl is-active nginx
```

Im Browser prüfen: korrekter Hostname, gültiges Zertifikat und nginx-Seite. Ein `404` von Traefik deutet häufig auf einen nicht passenden Router, ein `502` auf ein nicht erreichbares Backend; Logs und tatsächliche Konfiguration entscheiden. Bei vorgeschaltetem CDN/Proxy belegt das Browserzertifikat zunächst nur die Verbindung bis zum Proxy. Prüfe Proxy-TLS-Modus und das Origin-Zertifikat über den jeweils vorgesehenen, autorisierten Administrationsweg gesondert; gesperrte Origin-Routen nicht umgehen.

**Erst danach** die Seite für einen separaten Authentifizierungstest mit Authelia schützen: Die öffentliche Anmeldung, ein nicht angemeldeter Zugriff, eine erfolgreiche 2FA-Anmeldung und ein Versuch mit gefälschten Identitätsheadern müssen jeweils überprüft werden. Die [Authelia-Anleitung](03-dienste-und-authelia.md) unterscheidet die laufende Beispielanlage vom öffentlichen Entwicklungsstand. Der hier gezeigte öffentliche nginx-Test ist selbst noch nicht durch Authelia geschützt.

## Rückbau

Für eine nur zum Test angelegte Seite zuerst deren DNS-Eintrag und Traefik-Router gezielt entfernen oder deaktivieren und prüfen, dass andere Dienste weiter funktionieren. Container und Daten nur löschen, wenn eindeutig feststeht, dass sie zu diesem Test gehören und nicht mehr benötigt werden. Vorher benötigte Daten sichern.
