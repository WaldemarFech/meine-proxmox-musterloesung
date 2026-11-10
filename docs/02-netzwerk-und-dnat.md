# Netzwerk, Bridge und Portweiterleitung

## 1. Warum gibt es zwei Netzseiten?

Der Proxmox-Host erreicht über `vmbr0` das öffentliche Netz des Providers. Auf `vmbr1` liegt das interne Containernetz `192.168.100.0/24`. Der Host hat dort `192.168.100.1` und ist das Gateway der Container. Ein Container kann über das interne Netz angesprochen werden, ohne eine eigene öffentliche IPv4-Adresse zu benötigen.

Vor jeder Änderung: Provider-Konsole und Rückfallweg sicherstellen, vorhandene Bridge-, Routen- und Firewallkonfiguration sichern und das eigene Netzwerk prüfen. Manche Provider routen zusätzliche Adressen anders; das Beispiel ist keine universelle Providerkonfiguration.

```sh
ip -br address
ip route
cat /proc/sys/net/ipv4/ip_forward
iptables -t nat -S
```

Die Ausgabe kann öffentliche Adressen, Gerätenamen und vertrauliche Netzdetails enthalten. Vor dem Teilen öffentliche WAN-IP, Domains und personenbezogene Namen entfernen. Keine privaten Schlüssel oder komplette VPN-Konfigurationen veröffentlichen.

## 2. Was macht DNAT?

Die folgenden **aktiven Regeln** wurden am 10. November 2026 lesend auf dem Beispiel-VPS beobachtet. Sie leiten eingehende Pakete auf dem externen Interface an die internen Ziele weiter:

```text
TCP 80  über vmbr0 → 192.168.100.3:80   (Traefik / HTTP)
TCP 443 über vmbr0 → 192.168.100.3:443  (Traefik / HTTPS)
UDP 51820 über vmbr0 → 192.168.100.10:51820 (WireGuard-Hub)
```

Traefik entscheidet anschließend anhand des Hostnamens, ob eine Anfrage etwa zur Testseite oder zu Authelia gehört. DNAT selbst entscheidet nicht anhand der Domain und ersetzt keine Firewall- oder Authentifizierungsregel.

Die beobachtete Form einer Regel ist beispielsweise:

```sh
iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 443 \
  -j DNAT --to-destination 192.168.100.3:443
```

**Erklärung, kein Blindkopierbefehl:** `PREROUTING` verarbeitet eingehende Pakete, `--dport 443` wählt HTTPS und `--to-destination` das interne Ziel. Auf einem anderen VPS können Interface, Firewallverwaltung und Ziel abweichen. Eine bloß interaktiv hinzugefügte Regel ist unter Umständen nach einem Neustart weg. Prüfe Zuständigkeit, Weiterleitungsregeln, bestehendes nftables-/iptables-Regelwerk und Persistenz, bevor du etwas änderst. Nicht doppelt hinzufügen oder fremde Regeln überschreiben.

## 3. Prüfen

```sh
# Auf dem Proxmox-Host: laufende Portweiterleitungen anzeigen
iptables -t nat -S PREROUTING

# Netz und Gateway eines Containers anzeigen – ID anpassen
pct config 1003

# Im Traefik-Container: lauscht der Dienst?
pct exec 1003 -- ss -ltn
```

`pct config` kann Kommentare mit sensiblen Angaben enthalten; Ausgaben vor Weitergabe prüfen. Ein Dienst, der intern lauscht, ist dadurch nicht automatisch extern erreichbar. DNS, Provider-Firewall, Host-Firewall, DNAT, Traefik-Router, Zertifikat und Backend jeweils getrennt prüfen.

## 4. WireGuard und Standorte

Im Beispiel lauscht der Hub in CT `100010` intern auf `192.168.100.10` und nutzt `10.10.0.1/16` auf `wg0`. Der VPS-Host hatte bei der lesenden Prüfung am 10. November 2026 eine Route zu `10.10.0.5` über diesen Hub. Die ältere Standortdokumentation beschreibt ein Heimnetz `192.168.1.0/24` und eine weitere FRITZ!Box-Seite `192.168.178.0/24`; die aktuelle Erreichbarkeit dieser Gegenstellen wurde hier nicht erneut geprüft.

WireGuard verbindet Peers verschlüsselt. Welche entfernten Netze über welchen Peer laufen, bestimmen unter anderem eindeutige `AllowedIPs`, Betriebssystemrouten und Firewallregeln. Vermeide überlappende Netze. Öffne nicht automatisch alle Standorte füreinander: Zugriffe je Bedarf beschränken und Rückwege prüfen. Peer-PrivateKeys, Preshared Keys und komplette Konfigurationen gehören nie in Screenshots oder ein öffentliches Repository.

## 5. IPv6 ist ein eigener Pfad

DNAT-Regeln für IPv4 schützen IPv6 nicht. Prüfe, ob dein Provider IPv6 routet, welche Adressen die Container erhalten und welche IPv6-Firewallregeln gelten. Veröffentliche AAAA-Einträge erst, wenn der gewünschte IPv6-Pfad und seine Zugriffsbeschränkungen tatsächlich getestet sind.
