# Überblick

Der **Netzwerkmonitor** überwacht das Heimnetz über die FritzBox. Er erkennt neue Geräte, meldet sie aufs Handy und lässt sie per Knopfdruck **erlauben** oder **sperren**.

## Was macht er?

- Fragt alle 30 Sekunden die Geräteliste der FritzBox ab (TR-064-Schnittstelle).
- Merkt sich jedes Gerät in einer eigenen Datenbank. Geräte aus dem allerersten Scan gelten als **erlaubt**, alle später auftauchenden als **unbekannt**.
- Schickt bei jedem neuen Gerät eine Benachrichtigung (ntfy, optional Telegram oder Home Assistant).
- **Sperren** trennt das Gerät vom **Internet** (Internetsperre der FritzBox). Im Heimnetz bleibt es erreichbar. Ein kompletter Rauswurf aus dem WLAN geht über die Schnittstelle der FritzBox nicht.
- Optional: Unbekannte Geräte werden sofort vom Internet getrennt, bis man sie erlaubt („Quarantäne“).

## Bausteine

| Baustein | Wo | Zweck |
|---|---|---|
| Dienst `networkmonitor` | Docker-Container auf Unraid, Host-Netzwerk | Abfrage der FritzBox, Datenbank, Weboberfläche, API |
| Weboberfläche | `http://192.168.178.207:8085` | Geräteliste, Einzelgerät, Einstellungen |
| ntfy-Server | Docker-Container `ntfy` auf Unraid, Port **8091** | Zustellung der Benachrichtigungen aufs Handy |
| Android-App „Netzwerkmonitor“ | Handy (APK aus GitHub Releases) | Liste, Einzelgerät mit Erlauben/Sperren, Entsperren per Passwort oder Fingerabdruck |
| ntfy-App | Handy (Play Store/F-Droid) | Empfängt die Benachrichtigungen, Antippen öffnet die Netzwerkmonitor-App |
| FritzBox-Benutzer `networkmonitor` | FritzBox | Zugang für den Dienst |

## Wo liegt was?

| Was | Pfad / Adresse |
|---|---|
| Quellcode | GitHub: `Archonw/Networkmonitor` (privat) |
| Code auf Unraid (Build-Quelle) | `/mnt/user/appdata/networkmonitor/src` |
| Datenbank und Einstellungen | `/mnt/user/appdata/networkmonitor/data/networkmonitor.db` |
| Optionale Startwerte | `/mnt/user/appdata/networkmonitor/.env` |
| ntfy-Daten | `/mnt/user/appdata/ntfy` |
| App-Download | GitHub → Releases → `netzwerkmonitor.apk` |

**Wichtig:** Alle Einstellungen aus der Weboberfläche (FritzBox-Zugang, Benachrichtigungen, Passwort) stehen in der Datenbank im Ordner `data`. Diesen Ordner sichern, dann ist eine Neuinstallation schnell erledigt.

## Erreichbarkeit

Weboberfläche, App und ntfy funktionieren nur im Heimnetz. Von unterwegs braucht man eine VPN-Verbindung zur FritzBox.
