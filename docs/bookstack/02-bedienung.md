# Bedienung

## Wenn ein neues Gerät auftaucht

1. Auf dem Handy kommt eine ntfy-Nachricht „Neues Gerät im Netz“ mit Name, MAC, IP und Verbindungsart.
2. Antippen (oder „Prüfen“) öffnet die Netzwerkmonitor-App direkt auf diesem Gerät. Vorher wird mit Passwort oder Fingerabdruck entsperrt.
3. **Erlauben** oder **Sperren** tippen. Sperren schaltet in der FritzBox das Internet für das Gerät ab.

Hinweis „zufällige MAC-Adresse“: Viele Handys nutzen pro WLAN eine eigene, zufällige MAC. Ein „neues“ Gerät kann also auch ein bekanntes Handy sein.

## Android-App

- **Geräteliste:** Suche (Lupe) nach Name, IP oder MAC. Filter: Alle, Online, Unbekannt, Gesperrt. Sortieren über das Menü neben der Überschrift. Zum Aktualisieren die Liste nach unten ziehen.
- **Menü (drei Punkte):** „Aktualisieren“ und „Offline-Geräte entfernen“.
- **Gerät:** alle Daten, IP und MAC mit „Kopieren“. Lange drücken markiert einen Wert. Unten die Knöpfe Erlauben und Sperren.
- **Einstellungen:**
  - **Design:** „Soft“ oder „Kacheln“
  - **Darstellung:** Hell, Dunkel oder wie System
  - **Akzentfarbe:** vier pro Design
  - **Entsperren mit:** Passwort oder Fingerabdruck
  - **Verbindung:** Adresse und Passwort des Monitors
- Nach mehr als einer Minute im Hintergrund und bei jeder Benachrichtigung muss die App neu entsperrt werden. Das Passwortfeld funktioniert immer, auch wenn der Fingerabdruck eingestellt ist.

## Weboberfläche

`http://192.168.178.207:8085`. Der Benutzername ist beliebig, das Passwort ist das der Weboberfläche.

- **Startseite:** alle Geräte, Erlauben, Sperren, Umbenennen, „Offline-Geräte entfernen“
- **Gerät:** Klick auf den Namen öffnet die Einzelansicht
- **Einstellungen** (oben rechts): FritzBox-Zugang mit „Verbindung testen“, Abfrageintervall, Quarantäne, Benachrichtigungskanäle mit „Testnachricht senden“, neues Passwort. Änderungen gelten sofort, ohne Neustart.

## Offline-Geräte entfernen

Löscht alle Geräte, die gerade offline sind, aus dem **Monitor**. Kommt ein entferntes Gerät wieder ins Netz, gilt es als neu: Es kommt eine Benachrichtigung, und man muss es neu bestätigen.

- **Gesperrte Geräte bleiben** stehen, damit ihre Sperre erhalten bleibt.
- Die Einträge in der FritzBox selbst ändern sich dadurch nicht.

## Einträge in der FritzBox aufräumen

Unter *Heimnetz → Netzwerk → Netzwerkverbindungen* gibt es ganz unten „Nicht verwendete Verbindungen entfernen“. Manche Einträge bleiben trotzdem stehen:

- **VPN-Verbindungen:** verschwinden erst, wenn man die VPN-Verbindung löscht.
- **Geräte mit „Immer die gleiche IPv4-Adresse zuweisen“:** erst den Haken entfernen.
- **Geräte mit eigenem Zugangsprofil oder Internetsperre:** erst zurücksetzen.
- **Alle übrigen:** einzeln über *Bearbeiten → Entfernen*.

**Achtung:** Wird ein gesperrtes Gerät in der FritzBox gelöscht, ist auch seine Internetsperre weg.
