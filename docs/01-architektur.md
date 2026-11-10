# Aufbau: Ein VPS verbindet Dienste und Standorte

Diese Seite zeigt unsere Anlage als nachvollziehbares Beispiel. Interne IPs und Container-IDs sind bewusst erhalten; öffentliche WAN-IP, Domains und persönliche Angaben sind nicht Teil der Darstellung. Du kannst auf deinem System andere IDs und Netze wählen.

```mermaid
flowchart TB
    internet["Internet / DNS"] -->|"TCP 80 + 443"| pve["Proxmox auf VPS<br/>vmbr0: extern"]
    pve -->|"DNAT auf 192.168.100.3"| traefik["CT 1003 · Traefik<br/>192.168.100.3"]
    traefik -->|"ForwardAuth"| auth["CT 1004 · Authelia<br/>192.168.100.4"]
    traefik -->|"Webseiten"| tiny["CT 100032 · Tiny Pages<br/>192.168.100.32"]
    traefik --> forgejo["CT 100024 · Forgejo<br/>192.168.100.24"]
    traefik --> drop["CT 100029 · Drop<br/>192.168.100.29"]
    pve -->|"UDP 51820 DNAT"| wg["CT 100010 · WireGuard-Hub<br/>192.168.100.10 / 10.10.0.1"]
    wg <-->|"verschlüsselter Tunnel"| home["Heimnetz / Proxmox-Hosts<br/>Beispiel: 192.168.1.0/24"]
    wg <-->|"verschlüsselter Tunnel"| site["Weiterer Standort / FRITZ!Box<br/>Beispiel: 192.168.178.0/24"]
    ai["CT 100012 · OpenCode/ai-dev<br/>192.168.100.12"]
    hermes["CT 100019 · Hermes<br/>192.168.100.19"]
    pbs["CT 100252 · Backup Server<br/>192.168.100.252"]
    pve --- ai
    pve --- hermes
    pve --- pbs
```

Die Linien zu Forgejo und Drop zeigen deren Einordnung als über Traefik veröffentlichbare Webdienste; konkrete Router und Zugriffspolitiken müssen für die eigene Installation geprüft werden. Die Heimnetz- und FRITZ!Box-Strecken veranschaulichen die dokumentierte Hub-Topologie; eine aktuelle Ende-zu-Ende-Erreichbarkeit der Gegenstellen wurde bei dieser Bestandsaufnahme nicht behauptet.

## Die Bausteine

| Bestandteil | Beispiel | Aufgabe |
| --- | --- | --- |
| Proxmox VE | VPS, `vmbr1`: `192.168.100.1/24` | Host für LXC-Container und internes Gateway |
| Traefik | CT `1003`, `192.168.100.3` | Nimmt HTTP/HTTPS an und leitet nach Hostname weiter |
| Authelia | CT `1004`, `192.168.100.4` | Anmeldung, Richtlinien und zweiter Faktor |
| WireGuard | CT `100010`, `192.168.100.10`, Tunnel `10.10.0.1/16` | Verschlüsselte Standortverbindungen |
| OpenCode / ai-dev | CT `100012`, `192.168.100.12` | Entwicklungsumgebung |
| Hermes | CT `100019`, `192.168.100.19` | Agentendienst |
| Forgejo | CT `100024`, `192.168.100.24` | Git-Hosting |
| Drop | CT `100029`, `192.168.100.29` | Datei-/Übergabedienst |
| Tiny Pages | CT `100032`, `192.168.100.32` | Kleiner Webserver für statische Seiten |
| Proxmox Backup Server | CT `100252`, `192.168.100.252` | Backup-Ziel und Wiederherstellung |

**Stand der Messung:** Am 10. November 2026 wurden Containerstatus und die genannten LXC-Adressen lesend auf dem VPS abgefragt. `vmbr1`, die WireGuard-Interfaceadresse und die Route des Hosts zu `10.10.0.5` über `192.168.100.10` wurden ebenfalls ausgelesen. Diese Messungen zeigen keine aktuelle Anmeldung oder Ende-zu-Ende-Verbindung jedes Dienstes.

Weiter: [Netzwerk und DNAT](02-netzwerk-und-dnat.md) · [Anmeldung, Tiny Pages und Dienste](03-dienste-und-authelia.md) · [KI-Kopiervorlage](../README.md#kopiervorlage-für-deine-ki)
