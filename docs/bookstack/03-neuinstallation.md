# Neuinstallation von Grund auf

So wird der Netzwerkmonitor komplett neu aufgesetzt, z. B. nach einem Unraid-Neuaufbau oder auf einem anderen Server. Eine `.env` ist nicht nötig, alles wird in der Weboberfläche eingerichtet.

**Gibt es noch eine Sicherung des Ordners `data`?** Dann bei Schritt 4 einfach zurückkopieren. Geräte, Sperren und alle Einstellungen sind dann wieder da, und die Ersteinrichtung (Schritt 6) entfällt.

## 1. FritzBox vorbereiten

1. *Heimnetz → Netzwerk → Netzwerkeinstellungen*: „Zugriff für Anwendungen zulassen“ und „Statusinformationen über UPnP übertragen“ einschalten.
2. *System → FRITZ!Box-Benutzer → Benutzer hinzufügen*:
   - Name `networkmonitor` und ein eigenes Passwort
   - Recht **„FRITZ!Box Einstellungen“**
   - **ohne** Zugang aus dem Internet

   Gibt es den Benutzer schon, reicht das vorhandene Passwort.

## 2. ntfy-Server starten (für Benachrichtigungen aufs Handy)

Auf Unraid im Terminal:

```
docker run -d --name ntfy --restart unless-stopped -p 8091:80 \
  -v /mnt/user/appdata/ntfy:/var/cache/ntfy \
  binwiederhier/ntfy serve --cache-file /var/cache/ntfy/cache.db
```

Port 8091, weil 8090 von SABnzbd belegt ist. Test im Browser: `http://192.168.178.207:8091` zeigt die ntfy-Seite.

## 3. Code auf den Server holen

Das Repo ist privat, deshalb braucht man ein GitHub-Login. Am einfachsten geht es auf dem Linux-Rechner (Jan-Linux):

```
git clone https://github.com/Archonw/Networkmonitor.git ~/Networkmonitor
rsync -a --delete --exclude .git ~/Networkmonitor/ root@192.168.178.207:/mnt/user/appdata/networkmonitor/src/
```

Alternativ auf GitHub *Code → Download ZIP*, entpacken und den Inhalt nach `/mnt/user/appdata/networkmonitor/src` kopieren (z. B. über die Unraid-Freigabe `appdata`).

## 4. Image bauen und Container starten

Auf Unraid im Terminal:

```
mkdir -p /mnt/user/appdata/networkmonitor/data
docker build -t networkmonitor:local /mnt/user/appdata/networkmonitor/src
docker run -d --name networkmonitor --network host \
  --cap-add NET_ADMIN --cap-add NET_RAW \
  -v /mnt/user/appdata/networkmonitor/data:/data \
  --restart unless-stopped \
  --log-driver json-file --log-opt max-size=50m --log-opt max-file=1 \
  networkmonitor:local
```

- Eine Sicherung von `data` gehört **vor** `docker run` nach `/mnt/user/appdata/networkmonitor/data`.
- Gibt es eine `.env` mit Startwerten (z. B. `UI_PASSWORD`), zusätzlich `--env-file /mnt/user/appdata/networkmonitor/.env` angeben. Nötig ist sie nicht.

Prüfen:

```
docker logs --tail 30 networkmonitor
curl http://localhost:8085/health
```

`/health` meldet vor der Einrichtung „FritzBox noch nicht eingerichtet“. Das ist normal.

## 5. Erste Anmeldung

1. `http://192.168.178.207:8085` öffnen.
2. Benutzername beliebig, Passwort **`netzwerkmonitor`** (Standardpasswort).
3. Es erscheint „Willkommen beim Netzwerkmonitor“. Ein eigenes Passwort festlegen (mindestens 8 Zeichen).
4. Der Browser fragt erneut: jetzt mit dem neuen Passwort anmelden.

Bis das eigene Passwort gesetzt ist, ist nichts anderes erreichbar, auch nicht für die App.

## 6. Einstellungen in der Weboberfläche

Nach dem Passwort öffnet sich die Einstellungsseite.

1. **FritzBox:**
   - Adresse `192.168.178.1`, Benutzer `networkmonitor`, Passwort aus Schritt 1
   - „Verbindung testen“ muss „Verbunden: … Geräte“ zeigen
2. **Überwachung:**
   - Abfrageintervall (Standard 30 s)
   - „Geräte aus dem allerersten Scan als erlaubt übernehmen“ eingeschaltet lassen
3. **Benachrichtigungen:**
   - „ntfy“ anhaken
   - ntfy-Server `http://192.168.178.207:8091`
   - Thema `netzwerkmonitor`
   - Adresse dieser Weboberfläche `http://192.168.178.207:8085/`
4. **Speichern.** Danach „Testnachricht senden“.

Mit dem Speichern beginnt die Überwachung. Der erste Scan übernimmt alle vorhandenen Geräte als erlaubt.

## 7. Handy einrichten

1. **ntfy-App** installieren (Play Store oder F-Droid):
   - in den Einstellungen den Standardserver auf `http://192.168.178.207:8091` setzen
   - das Thema `netzwerkmonitor` abonnieren
2. **Netzwerkmonitor-App:**
   - auf GitHub unter *Releases* `netzwerkmonitor.apk` herunterladen und installieren („Installation aus unbekannten Quellen“ für den Browser erlauben)
   - beim ersten Start die Adresse `http://192.168.178.207:8085` und das Passwort der Weboberfläche eintragen und „Speichern und testen“ tippen
3. Optional in der App unter *Einstellungen → Entsperren mit* auf Fingerabdruck umstellen.
4. Test: In der Weboberfläche „Testnachricht senden“. Die Nachricht antippen, dann öffnet sich die App.

## Alternative: Unraid-Vorlage

Statt `docker run` geht auch die Vorlage `unraid/networkmonitor.xml` aus dem Repo:

1. Die Datei nach `/boot/config/plugins/dockerMan/templates-user/my-networkmonitor.xml` kopieren.
2. Unter *Docker → Add Container* „networkmonitor“ wählen.

Die Vorlage nutzt das Image `ghcr.io/archonw/networkmonitor:latest`. Weil das Repo privat ist, braucht Unraid dafür vorher `docker login ghcr.io` mit einem GitHub-Token. Der Weg über den eigenen Build (Schritt 3 und 4) kommt ohne aus.
