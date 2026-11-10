# Proxmox-Musterlösung – Umsetzungsplan

**Ziel:** Eine verständliche, bebilderte und anhand der tatsächlichen VPS-Anlage nachvollziehbare Vorlage samt überprüfter Authelia-Einordnung.

**Grundlage:** Separat versionierte Proxmox- und Authelia-Repositories; lesende Bestandsaufnahme und klar datierte Evidenz. Die bestehende Produktion bleibt unverändert, außer einem ausdrücklich beauftragten und eng begrenzten Testkonto, falls nötig.

1. **Inventar:** Aktuelle PVE-LXCs, interne IPs, Bridge, DNS/Proxy und die tatsächlich aktiven DNAT-Regeln für 80/443 und WireGuard feststellen. Beispiele von historischen Informationen unterscheiden; öffentliche WAN-IP und Geheimnisse nicht veröffentlichen.
2. **Verständliche Vorlage:** `README.md` und kurze thematische Dateien unter `docs/` für Proxmox, Networking, Traefik, Authelia/2FA, Tiny Pages und weitere gewünschte Dienste erstellen. Zu jedem Dienst Zweck, Beispielpfad, Voraussetzungen, Einrichtung, Funktionsprüfung und Rückbau; kein ungeprüftes Copy/Paste als aktuelle Tatsache ausgeben.
3. **Bilder:** Nach Möglichkeit echte Proxmox-/Netzwerk- und Authelia-Screenshots mit fiktiven Daten aufnehmen und vor Veröffentlichung auf WAN-IP, Namen und Geheimnisse prüfen; Architektur als verständliches Diagramm ergänzen. Fehlende Ansichten nicht als vorhanden ausgeben.
4. **Authelia-Belege:** Laufenden eigenen Build mit öffentlichem Quell-/Portstand vergleichen. Vorhandene getrennte Testidentität nutzen oder kontrolliert synthetisch anlegen; echte Anmeldung/2FA und Einladungen nur mit ausdrücklichem Umfang, ohne Token-Ausgabe oder Veränderung fremder Konten testen. Produktions-, Labor- und öffentliches Paket getrennt dokumentieren.
5. **Veröffentlichung:** Links und Bilder prüfen, finalen konkreten Inhalt samt Git-Diff und Historie auf Secrets/WAN-IP prüfen, unabhängig reviewen und über PR veröffentlichen. Erfolg und verbleibende konkrete Grenzen berichten.
