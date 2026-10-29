# Updates, Sicherung und Fehlersuche

## Update auf eine neue Version

Neue Versionen entstehen auf GitHub (Pull Request mergen). Danach:

**Server** (Weboberfläche, Benachrichtigungen):

```
cd ~/Networkmonitor && git pull
rsync -a --delete --exclude .git --exclude .env --exclude data ~/Networkmonitor/ root@192.168.178.207:/mnt/user/appdata/networkmonitor/src/
```

Dann auf Unraid:

```
docker build -t networkmonitor:local /mnt/user/appdata/networkmonitor/src
docker rm -f networkmonitor
docker run -d --name networkmonitor --network host --cap-add NET_ADMIN --cap-add NET_RAW \
  --env-file /mnt/user/appdata/networkmonitor/.env \
  -v /mnt/user/appdata/networkmonitor/data:/data --restart unless-stopped \
  --log-driver json-file --log-opt max-size=50m --log-opt max-file=1 networkmonitor:local
```

`--env-file` nur angeben, wenn es die `.env` gibt. `.env` und `data` bleiben beim Update unberührt.

**App:** Unter GitHub *Releases* die neue `netzwerkmonitor.apk` laden und über die alte installieren. Die Einstellungen bleiben erhalten.

## Sicherung

Es reicht der Ordner `/mnt/user/appdata/networkmonitor/data`. Er enthält die Datenbank mit:

- allen Geräten, Namen und Sperren
- dem Verlauf
- allen Einstellungen aus der Weboberfläche

Dazu optional die `.env` und `/mnt/user/appdata/ntfy`. Der Ordner `src` kommt jederzeit neu von GitHub.

## Passwort der Weboberfläche vergessen

Gespeichertes Passwort löschen und Container neu starten:

```
docker exec networkmonitor python -c "import sqlite3; c=sqlite3.connect('/data/networkmonitor.db'); c.execute(\"DELETE FROM meta WHERE key='cfg.ui_password'\"); c.commit()"
docker restart networkmonitor
```

Danach gilt:

- das `UI_PASSWORD` aus der `.env`, falls dort eins steht, oder
- wieder das Standardpasswort `netzwerkmonitor` mit der Aufforderung, ein neues festzulegen.

In der App muss das neue Passwort dann unter *Einstellungen → Verbindung* eingetragen werden.

## Fehlersuche

| Problem | Prüfen |
|---|---|
| Weboberfläche nicht erreichbar | `docker ps` (läuft `networkmonitor`?), `docker logs --tail 50 networkmonitor` |
| „FritzBox noch nicht eingerichtet“ | In den Einstellungen FritzBox-Passwort eintragen und speichern |
| „Keine Verbindung“ beim FritzBox-Test | Benutzer und Passwort, Recht „FRITZ!Box Einstellungen“, „Zugriff für Anwendungen zulassen“ in der FritzBox |
| Keine Benachrichtigung | In den Einstellungen „Testnachricht senden“. Prüfen, ob der ntfy-Container läuft (`docker ps`), ob die App den Server `:8091` nutzt und das Thema `netzwerkmonitor` abonniert hat |
| App: „Monitor nicht erreichbar“ | Handy im Heimnetz bzw. im VPN? Adresse in der App richtig? |
| App: „Ersteinrichtung: bitte zuerst im Browser …“ | Am Server ist noch kein eigenes Passwort gesetzt, siehe Neuinstallation, Schritt 5 |
| Sperren wirkt nicht | Sperren trennt nur das Internet, nicht das WLAN. Das Gerät braucht eine IP. In der FritzBox unter *Internet → Filter → Kindersicherung* prüfen |

Schnelltest der Funktionen in der Kommandozeile:

```
docker exec -it networkmonitor networkmonitor-probe
```

Zeigt Modell und FritzOS der FritzBox und prüft, ob Geräteliste und Internetsperre funktionieren.
