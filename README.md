# Proxmox-Musterlösung: VPS, Traefik und Authelia

**Arbeitsstand:** Dieses Repository ist ein Ausgangspunkt für eine verständliche Anleitung, noch keine fertig getestete Musterinstallation. Screenshots und vollständige, verifizierte Konfigurationen fehlen. Verwende die folgenden Angaben als Beispiel und prüfe sie auf deinem eigenen System.

## Die Idee

Ein Proxmox-VPS betreibt getrennte LXC-Container. Traefik nimmt Webanfragen entgegen; Authelia kann Zugriffe absichern. Ein kleiner Webserver stellt Testseiten bereit. Je nach Provider braucht der Host eine interne Bridge, Routing, ausgehendes NAT und DNAT für die Webports. Welche Regeln tatsächlich nötig sind, hängt von deinem VPS und seinem bestehenden Netzwerk ab.

Beispiel aus einer bestehenden Anlage, als Orientierung – **nicht ungeprüft übernehmen**:

| Komponente | Beispiel |
| --- | --- |
| Privates Netz / Gateway | `192.168.100.0/24` / `192.168.100.1` |
| Interne Bridge | `vmbr1` |
| Traefik | LXC `1003`, `192.168.100.3` |
| Authelia | LXC `1004`, `192.168.100.4` |
| Tiny Pages | LXC `100032`, `192.168.100.32` |

Die Werte sind dokumentierte Beispiele und keine aktuelle Inventur deines Servers. Eine öffentliche WAN-IP oder Zugangsdaten sind hier bewusst nicht enthalten. Die zugehörige Benutzerverwaltung und ihr Entwicklungsstand stehen im Repository [Authelia Onboarding](https://github.com/WaldemarFech/authelia-onboarding).

## Kopiervorlage für deine KI

```text
Hilf mir, auf meinem eigenen Proxmox-VPS eine Debian-13-Umgebung mit
Traefik, Authelia und einer Testseite einzurichten. Erkläre mir jeden
Schritt verständlich und beginne mit einer lesenden Bestandsaufnahme.

Frage mich gesammelt nach Domain, DNS-Zugriff, VPS-Provider, öffentlicher
IPv4/IPv6, Proxmox-Zugriff und Provider-Konsole, ersten Benutzernamen und
bestehenden Diensten. Keine Passwörter oder Einmalcodes im Chat erfragen.

Prüfe vorhandene Bridges, Subnetze, Routen, Firewall-/NAT-Regeln, freie
Container-IDs und IP-Adressen. Nutze die Werte aus diesem Repository als
Beispiel, nur wenn sie zu meinem System passen und frei sind. Dokumentiere
einen Netzplan und einen Rückfallweg, bevor du etwas veränderst. Vermeide,
dass mein Zugang durch Netzwerkänderungen ausfällt.

Plane gegebenenfalls eine interne vmbr, IPv4-Forwarding, ausgehendes NAT,
DNAT für TCP 80/443 und passende Firewall-Regeln. Prüfe IPv6 separat und
veröffentliche sie nicht, bevor der komplette Pfad abgesichert ist. Richte
Traefik und eine HTTPS-Testseite ein und prüfe sie zuerst ohne Authelia.
Installiere dann offizielles Authelia, richte einen Benutzer und 2FA ein,
schütze die Testseite und prüfe Anmeldung, unbefugte Zugriffe, gefälschte
Header und direkte Backend-Zugriffe.

Ohne Mailserver: Prüfe den offiziellen Authelia-Filesystem-Notifier als
zeitlich begrenzte Testlösung. Ich lese etwaige Einmal-Nachrichten selbst
in einer geschützten Konsole; Codes, Links, QR-Codes und Secrets dürfen
nicht in Chat, Logs oder Git gelangen.

Lies auch https://github.com/WaldemarFech/authelia-onboarding . Die dortige
Benutzerverwaltung ist experimentell und nicht produktionsreif; befolge
die dortigen Warnungen und installiere sie nicht öffentlich, bevor ihre
Freigabebedingungen erfüllt und nachgewiesen sind. Dokumentiere getestete
Ergebnisse, Backups, offene Punkte und Rückbau. Nutze keine fremden Secrets.
```

## Offene Arbeiten

Siehe [HANDOVER.md](HANDOVER.md) für den aktuellen Stand und die nächsten Aufgaben.
