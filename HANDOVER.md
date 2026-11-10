# Handover / To-do

Stand: 10. November 2026. Die grundlegende Topologie und ausgewählte DNAT-Regeln wurden am 10. November 2026 lesend an der Beispielanlage geprüft. Eine vollständige End-to-End-Anleitung bleibt offen.

- [ ] Vorhandene, freigegebene Infrastrukturquellen gezielt auswerten und auf den tatsächlichen Stand prüfen; keine vollständige private Wissensdatenbank veröffentlichen.
- [ ] Verständliche Markdown-Anleitungen für Proxmox, Bridges, Routing, NAT/DNAT, Firewall, Debian-13-LXC, DNS, Traefik und HTTPS ergänzen und mit einer unabhängigen Testinstallation verifizieren.
- [ ] Geeignete Screenshots von Proxmox, LXC-Übersicht, Netzwerk, relevanten Traefik-DNAT-Regeln und gegebenenfalls 2FA erstellen. Öffentliche WAN-IP und sensible Angaben vor Veröffentlichung unkenntlich machen; keine echten Schlüssel, Tokens, Codes oder Personen verwenden. Der Owner erlaubt interne IPs und LXC-IDs als Beispiele.
- [ ] Authelia-Onboarding erst nach dessen eigener Sicherheits- und Installationsfreigabe als einsatzbereit beschreiben; aktueller Entwicklungsstand: https://github.com/WaldemarFech/authelia-onboarding.
- [ ] Vor jedem öffentlichen Commit Text, Bilder, Metadaten und veröffentlichte Git-Historie auf Geheimnisse und unbeabsichtigt private Informationen prüfen.

Es gibt inzwischen zwei reale, datenschutzgeprüfte Aufnahmen (Authelia-Kontoseite und Tiny Page); eine Aufnahme der Proxmox-Oberfläche fehlt, weil sie im Headless-Test HTTP 403 lieferte. Der vollständige Neuaufbau ist noch nicht durchgetestet. Bestehende Infrastruktur nicht ungefragt ändern.
