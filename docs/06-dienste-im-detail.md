# Dienste im Beispiel: Zuständigkeiten und Prüfung

Die Container teilen sich das interne `vmbr1`-Netz. Ein eigener Container pro Aufgabe macht Zuständigkeiten und Updates übersichtlicher. Die Tabelle enthält **interne** Adressen. Die Beispiele stammen aus einer lesenden Abfrage vom 10. November 2026; sie sind keine Garantie für die Erreichbarkeit oder Sicherheit einer anderen Installation.

| Dienst | LXC / interne IP | Beobachtung im Container | Zweck |
| --- | --- | --- | --- |
| Traefik | `1003` / `192.168.100.3` | HTTP(S)-DNAT des Hosts zeigt auf diesen Container | Reverse Proxy und TLS-Einstieg |
| Authelia | `1004` / `192.168.100.4` | Authelia und separater Onboarding-Dienst laufen | Anmeldung und Zugriffspolitik |
| WireGuard | `100010` / `192.168.100.10` | `wg0` trägt `10.10.0.1/16` | Verschlüsselte Standortverbindungen |
| OpenCode / ai-dev | `100012` / `192.168.100.12` | `opencode` lauscht intern auf TCP `4096` | Entwicklungsumgebung |
| Hermes | `100019` / `192.168.100.19` | Container läuft | Agentendienst |
| Forgejo | `100024` / `192.168.100.24` | `forgejo` lauscht intern auf TCP `3000` | Git-Repositories und Zusammenarbeit |
| Drop | `100029` / `192.168.100.29` | Python-Dienste lauschen intern auf TCP `8080` und `8081` | Datei-/Übergabedienst |
| Tiny Pages | `100032` / `192.168.100.32` | nginx lauscht intern auf TCP `80` und `8081` | Kleine Webseiten |
| Proxmox Backup Server | `100252` / `192.168.100.252` | Backup-Dienst lauscht intern auf TCP `8007` | Sicherungen und Wiederherstellung |

**Intern lauschen heißt nicht öffentlich erreichbar.** Ob ein Port von außen zugänglich ist, hängt von Provider-Firewall, Routing, DNAT und den Regeln auf dem Host und im Container ab. Die Tabelle dokumentiert keine pauschale Freigabe dieser Ports. Ein nur an `127.0.0.1` gebundener Dienst ist normalerweise nicht über die Container-IP für andere Container erreichbar; `ss -ltnp` zeigt die jeweilige Bind-Adresse. Welche Drop-Funktion welchen der beiden Ports nutzt, wurde bei dieser kurzen Inventur nicht zugeordnet.

## Traefik und Authelia

Traefik nimmt Anfragen entgegen und leitet sie anhand des Hostnamens an den passenden Dienst. Für geschützte Routen fragt die ForwardAuth-Middleware Authelia nach der Berechtigung. Die öffentliche Authelia-Seite selbst muss ohne Redirect-Schleife erreichbar bleiben. Backend-Dienste dürfen nicht über einen zweiten öffentlichen Weg an der Middleware vorbei erreichbar sein.

Ein praktisches Beispiel für eine erste nginx-Seite findest du unter [Traefik und Tiny Pages](05-traefik-tiny-pages.md); den gemessenen Stand der Anmeldung und die Trennung zwischen VPS-Code und öffentlichem Paket unter [Dienste und Authelia](03-dienste-und-authelia.md).

## WireGuard und Standortnetze

Der WireGuard-Container dient als Hub für verschlüsselte Verbindungen. Auf Host und Hub können die Routingentscheidungen unterschiedlich ausfallen. Die [Standort-Anleitung](04-wireguard-standorte.md) zeigt die konkret beobachteten Routen zu Heimnetz und einem weiteren Standort, einschließlich ihrer Grenzen.

## Forgejo

Forgejo stellt Git-Repositories und eine Weboberfläche bereit. Im Beispiel lauschte sein Prozess auf Port `3000` im LXC `100024`. Ein Traefik-Router kann HTTP-Anfragen an diese interne Adresse senden; die tatsächlich eingerichteten Router, SSH-Zugänge und Berechtigungen müssen für jede Anlage gesondert geprüft werden.

Für einen lesenden Basischeck auf dem Proxmox-Host:

```sh
pct status 100024
pct exec 100024 -- ss -ltnp
```

Ein geöffneter Port beweist weder die Anmeldung noch die Repository-Berechtigungen. Für eine eigene Installation dienen die [Forgejo-Dokumentation](https://forgejo.org/docs/latest/) und die Dokumentation der ausgewählten Version als Ausgangspunkt.

## Drop

Der Drop-Dienst stellt eine eigene Webfunktion für Dateiübergaben bereit. In der Beispielanlage läuft er getrennt von Traefik in CT `100029`. Für eine eigene Lösung entscheiden insbesondere Zugangskontrolle, Ablaufzeit, Speicherung und Löschung darüber, ob der Dienst zur Aufgabe passt. Teste mit einem eigenen Wegwerfbeispiel, ob Berechtigungen und Ablauf tatsächlich wie gewünscht wirken. Echte geheime Inhalte oder Zugangslinks gehören nicht in die Projektdokumentation.

## OpenCode und Hermes

OpenCode läuft im Beispiel als Dienst in CT `100012`. Die Abfrage zeigte dort einen intern lauschenden Prozess auf Port `4096`; eine erfolgreiche Webanmeldung wurde dabei nicht geprüft. Hermes läuft in CT `100019` als eigener Agentendienst. Seine Installation, Verbindungen und Arbeitsrechte sind an die jeweilige Umgebung gebunden; der laufende Containerstatus allein belegt keine funktionierende Aufgabenausführung. Zugangsdaten und private Agentenspeicher nicht als Beispielmaterial exportieren.

## Proxmox Backup Server

Der Backupserver stellt im Beispiel seinen Dienst auf internem Port `8007` im CT `100252` bereit. Ein Backupkonzept sollte mindestens klären: Welche VMs/Container werden gesichert, wo liegt die zweite Kopie, welche Schlüssel werden zum Lesen benötigt und wer darf löschen? Verschlüsselungsschlüssel müssen separat sicher aufbewahrt werden.

Ein **Restore-Test** in einer eigens dafür vorgesehenen Umgebung prüft mehr als ein grüner Backup-Status. Wähle ein eigenes Testziel, stelle keine produktive VM ungefragt zurück und protokolliere, ob Daten und Dienste tatsächlich lesbar bzw. startbar waren. Einzelheiten zur Einrichtung und Wiederherstellung stehen in der [offiziellen PBS-Dokumentation](https://pbs.proxmox.com/docs/).

## Häufige Fehlersuche

1. **Container läuft nicht:** `pct status <ID>` und den Dienststatus im Container prüfen.
2. **Interner Dienst antwortet nicht:** im Container `ss -ltnp` und anschließend vom zugelassenen internen Peer testen.
3. **Traefik zeigt 404:** Routerregel, Hostname und geladenen File-Provider prüfen.
4. **Traefik zeigt 502:** Backend-Adresse, Port, Dienststatus und interne Erreichbarkeit prüfen.
5. **403 oder Anmeldung scheitert:** vorhandene Zugriffsregeln und Logs über den vorgesehenen Administrationsweg prüfen; Sperren nicht umgehen.
6. **Tunnelroute gewählt, aber Ziel antwortet nicht:** Handshake, Peer-Zuordnung, Rückweg und Firewall an beiden Enden untersuchen.

Nie ungefilterte Diagnoseausgaben mit Zugangsdaten, privaten Schlüsseln, öffentlichen WAN-Adressen, echten Kontonamen oder Domainnamen veröffentlichen.
