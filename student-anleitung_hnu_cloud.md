# Spring Boot App auf HNU Cloud deployen - Anleitung für Studenten

## Übersicht
Diese Anleitung zeigt, wie ihr eure Spring Boot Anwendung lokal entwickelt und auf der HNU Cloud deployed.

---

## Voraussetzungen

### 1. HNU Cloud Zugang
- Zugriff auf die HNU Cloud WebConsole: **http://194.95.24.122:8080**
- Euer User-Account (swe-deployer)

### 2. Lokale Installation
- Docker Desktop auf eurem Mac/Windows installiert
- Git installiert
- IntelliJ IDEA oder VS Code

### 3. GitHub Account
- Erstellt einen kostenlosen GitHub Account (falls noch nicht vorhanden)
- GitHub Container Registry (ghcr.io) wird automatisch aktiviert
- Ihr benötigt GitHub Container Registry (ghcr.io) zum Speichern eurer Docker Images

### 4. Euer zugewiesener Port
Jeder Student bekommt einen festen Port zugewiesen:
- Student 1: Port 8081
- Student 2: Port 8082
- Student 3: Port 8083
- usw.


## Teil 1: Lokale Vorbereitung (auf eurem Rechner)

### Schritt 1: GitHub Personal Access Token erstellen

1. Geht zu GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)
2. Klickt auf "Generate new token (classic)"
3. Gebt einen Namen ein (z.B. "Docker HNU Cloud")
4. Wählt die Berechtigung: **write:packages** (damit könnt ihr Images pushen)
5. Klickt auf "Generate token"
6. ⚠️ **WICHTIG:** Kopiert den Token sofort! Er wird nur einmal angezeigt!

### Schritt 2: Docker Login zu GitHub

Öffnet euer Terminal/Command Prompt:

```bash
# Login zu GitHub Container Registry
docker login ghcr.io

# Username: euer-github-username
# Password: euer-personal-access-token (nicht euer GitHub Passwort!)
```

### Schritt 3: Projekt vorbereiten

#### application.properties anpassen

Öffnet `src/main/resources/application.properties` und setzt euren zugewiesenen Port:

```properties
# Beispiel: Euer Port ist 8083
server.port=8083
server.address=0.0.0.0

# Eure Datenbank-Konfiguration...
spring.datasource.url=jdbc:mysql://...
```

⚠️ **Wichtig:** Jeder Gruppe bekommt einen eigenen Port! (z.B. 8081, 8082, 8083, ...)

#### Dockerfile erstellen/überprüfen

Euer Dockerfile sollte so aussehen:

```dockerfile
# --------- Build-Stage ---------
FROM eclipse-temurin:17-jdk AS build
WORKDIR /workspace

# Maven Wrapper + Konfiguration kopieren
COPY mvnw pom.xml ./
COPY .mvn .mvn

# Dependencies vorab laden
RUN ./mvnw dependency:go-offline

# Source-Code kopieren und bauen
COPY src src
RUN ./mvnw package -DskipTests

# --------- Runtime-Stage ---------
FROM eclipse-temurin:17-jre
WORKDIR /app

# Jar aus der Build-Stage kopieren
COPY --from=build /workspace/target/*.jar app.jar

# Port exponieren
EXPOSE 8083

# App starten
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Schritt 4: Image bauen und pushen


#### In eurem Projekt-Verzeichnis 
cd /pfad/zu/eurem/projekt

#### oder im Terminal von IntelliJ

#### Multi-Platform Build (wichtig für Mac M1/M2 und Server-Kompatibilität)
docker buildx build --platform linux/amd64,linux/arm64 \
  -t ghcr.io/euer-github-username/meine-app:latest \
  --push .


⚠️ Ersetzt:
- `euer-github-username` mit eurem GitHub Benutzernamen
- `meine-app` mit einem sinnvollen Namen für eure App

**Beispiel:**
```bash
docker buildx build --platform linux/amd64,linux/arm64 \
  -t ghcr.io/max-mustermann/bookstore-app:latest \
  --push .
```

---

## Teil 2: Deployment auf HNU Cloud

### Schritt 1: WebConsole Login

1. Öffnet im Browser: **http://194.95.24.122:8080**
2. Loggt euch mit euren Zugangsdaten ein
3. Ihr seht nun eine Terminal-Oberfläche

### Schritt 2: Docker Image pullen

```bash
# Image von GitHub herunterladen
sudo docker pull ghcr.io/euer-github-username/meine-app:latest
```

Falls ihr nach einem Passwort gefragt werdet, gebt euer HNU Cloud Passwort ein.

### Schritt 3: Container starten

```bash
# Container starten mit eurem zugewiesenen Port
sudo docker run -d \
  -p 8083:8083 \
  --name meine-app \
  ghcr.io/euer-github-username/meine-app:latest
```

⚠️ Ersetzt `8083` mit eurem zugewiesenen Port!

**Parameter erklärt:**
- `-d` = Container läuft im Hintergrund
- `-p 8083:8083` = Port-Mapping (erster Port = außen, zweiter Port = innen im Container)
- `--name meine-app` = Name für euren Container
- `ghcr.io/...` = Euer Docker Image

### Schritt 4: Überprüfen ob die App läuft

```bash
# Logs anschauen
sudo docker logs meine-app

# Nach "Tomcat started" suchen
sudo docker logs meine-app | grep "Tomcat started"

# App testen
curl http://localhost:8083/books
```

Wenn alles funktioniert, solltet ihr eine Antwort sehen (z.B. `[]` für eine leere Liste).

---

## Häufige Befehle

### Container-Verwaltung

```bash
# Alle laufenden Container anzeigen
sudo docker ps

# Alle Container anzeigen (auch gestoppte)
sudo docker ps -a

# Container stoppen
sudo docker stop meine-app

# Container starten
sudo docker start meine-app

# Container neu starten
sudo docker restart meine-app

# Container löschen
sudo docker rm meine-app

# Container löschen (auch wenn er läuft)
sudo docker rm -f meine-app
```

### Logs und Debugging

```bash
# Logs anzeigen
sudo docker logs meine-app

# Logs live verfolgen
sudo docker logs -f meine-app

# Letzte 50 Zeilen
sudo docker logs --tail 50 meine-app

# In den Container "einloggen"
sudo docker exec -it meine-app /bin/sh
```

### Image-Verwaltung

```bash
# Alle Images anzeigen
sudo docker images

# Image löschen
sudo docker rmi ghcr.io/euer-github-username/meine-app:latest

# Neues Image pullen (für Updates)
sudo docker pull ghcr.io/euer-github-username/meine-app:latest
```

---

## Workflow für Updates

Wenn ihr eure App aktualisiert habt:

### Lokal (auf eurem Rechner):

```bash
# 1. Neues Image bauen und pushen
docker buildx build --platform linux/amd64,linux/arm64 \
  -t ghcr.io/euer-github-username/meine-app:latest \
  --push .
```

### Auf HNU Cloud (WebConsole):

```bash
# 2. Alten Container stoppen und löschen
sudo docker rm -f meine-app

# 3. Neues Image pullen
sudo docker pull ghcr.io/euer-github-username/meine-app:latest

# 4. Neuen Container starten
sudo docker run -d \
  -p 8083:8083 \
  --name meine-app \
  ghcr.io/euer-github-username/meine-app:latest

# 5. Überprüfen
sudo docker logs meine-app | grep "Tomcat started"
curl http://localhost:8083/books
```

---

## Troubleshooting

### Problem: "Container name already in use"

**Lösung:**
```bash
# Alten Container löschen
sudo docker rm -f meine-app

# Dann neu starten
sudo docker run -d -p 8083:8083 --name meine-app ...
```

### Problem: "port is already allocated"

**Fehler:** `bind: address already in use`

**Ursache:** Der Port ist schon von einem anderen Container belegt.

**Lösung:**
```bash
# Welcher Container nutzt den Port?
sudo docker ps

# Den Container stoppen der den Port nutzt
sudo docker stop <container-name>

# Oder einen anderen Port nutzen (z.B. 8084 statt 8083)
```

### Problem: "Connection refused" beim curl

**Mögliche Ursachen:**

1. **App ist noch nicht fertig gestartet:**
   ```bash
   # Warte 10 Sekunden und versuche nochmal
   sleep 10
   curl http://localhost:8083/books
   ```

2. **App ist abgestürzt:**
   ```bash
   # Logs checken
   sudo docker logs meine-app
   
   # Container läuft noch?
   sudo docker ps | grep meine-app
   ```

3. **Falscher Port in application.properties:**
   - Überprüft ob `server.port=8083` richtig gesetzt ist
   - Image neu bauen und pushen

### Problem: "no matching manifest for linux/amd64"

**Ursache:** Image wurde nur für eine Plattform gebaut.

**Lösung:** Beim Build `--platform linux/amd64,linux/arm64` verwenden:
```bash
docker buildx build --platform linux/amd64,linux/arm64 \
  -t ghcr.io/euer-github-username/meine-app:latest \
  --push .
```

### Problem: Docker Login funktioniert nicht

**Lösung:**
1. Überprüft ob ihr einen Personal Access Token verwendet (nicht euer GitHub Passwort!)
2. Token muss die Berechtigung `write:packages` haben
3. Probiert erneut:
   ```bash
   docker logout ghcr.io
   docker login ghcr.io
   ```

---

## Port-Zuweisung

Jeder Student bekommt einen festen Port:

| Student | Port |
|---------|------|
| Student 1 | 8081 |
| Student 2 | 8082 |
| Student 3 | 8083 |
| Student 4 | 8084 |
| ... | ... |

⚠️ **Wichtig:** Nutzt nur euren zugewiesenen Port!

---

## Checkliste vor dem Deployment

- [ ] GitHub Account erstellt
- [ ] Personal Access Token erstellt
- [ ] Docker Desktop installiert
- [ ] `docker login ghcr.io` erfolgreich
- [ ] Port in `application.properties` auf eigenen Port gesetzt
- [ ] Dockerfile im Projekt vorhanden
- [ ] Image lokal gebaut und zu GitHub gepusht
- [ ] Zugang zur HNU Cloud WebConsole (http://194.95.24.122:8080)
- [ ] Container auf HNU Cloud gestartet
- [ ] App mit `curl` getestet

---

## Support

Bei Problemen:
1. Logs checken: `sudo docker logs meine-app`
2. Container-Status prüfen: `sudo docker ps -a`
3. Dozenten oder Kommilitonen fragen

**Viel Erfolg!** 🚀 Zugang zur HNU Cloud
- **WebConsole:** http://194.95.24.122:8080
- **Username:** swe-deployer (oder euer zugewiesener Username)
- **Passwort:** (wird vom Dozenten bereitgestellt)

### 2. GitHub Account
- Erstellt einen kostenlosen GitHub Account auf https://github.com
- GitHub Container Registry (ghcr.io) ist automatisch dabei

### 3. Lokal installiert
- Docker Desktop (Mac/Windows)
- Git
- Java 17 JDK
- IDE (IntelliJ IDEA, VS Code, Eclipse)

### 4. Port-Zuweisung
Jeder Student bekommt einen festen Port zugeteilt:
- Student 1: Port 8081
- Student 2: Port 8082
- Student 3: Port 8083
- usw.

**Fragt euren Dozenten nach eurem zugewiesenen Port!**

---

## Schritt 1: Spring Boot Projekt vorbereiten

### 1.1 Port in application.properties setzen

Öffnet `src/main/resources/application.properties` und setzt euren zugewiesenen Port:

```properties
# Beispiel: Student 3 hat Port 8083 zugeteilt bekommen
server.port=8083
server.address=0.0.0.0
```

⚠️ **Wichtig:** Verwendet den Port, der euch zugeteilt wurde!

### 1.2 Dockerfile erstellen

Erstellt eine Datei namens `Dockerfile` im Projekt-Root:

```dockerfile
# --------- Build-Stage ---------
FROM eclipse-temurin:17-jdk AS build
WORKDIR /workspace

# Maven Wrapper + Konfiguration kopieren
COPY mvnw pom.xml ./
COPY .mvn .mvn

# Dependencies vorab laden
RUN ./mvnw dependency:go-offline

# Source-Code kopieren und bauen
COPY src src
RUN ./mvnw package -DskipTests

# --------- Runtime-Stage ---------
FROM eclipse-temurin:17-jre
WORKDIR /app

# Jar aus der Build-Stage kopieren
COPY --from=build /workspace/target/*.jar app.jar

# Port exponieren
EXPOSE 8080

# App starten
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### 1.3 .dockerignore erstellen

Erstellt eine `.dockerignore` Datei im Projekt-Root:

```
target/
.git/
.idea/
*.iml
.DS_Store
```

---

## Schritt 2: Docker Login bei GitHub

### 2.1 Personal Access Token (PAT) erstellen

1. Geht auf GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)
2. Klickt auf "Generate new token (classic)"
3. Name: `Docker Push Token`
4. Wählt die Berechtigung: **`write:packages`** ✅
5. Klickt "Generate token"
6. **⚠️ Kopiert den Token sofort - ihr seht ihn nur einmal!**

### 2.2 Docker Login durchführen

Terminal/PowerShell öffnen:

```bash
# Ersetzt EUER_GITHUB_USERNAME mit eurem GitHub-Benutzernamen
docker login ghcr.io -u EUER_GITHUB_USERNAME

# Wenn nach Passwort gefragt: Fügt euren PAT (Token) ein, NICHT euer GitHub-Passwort!
```

Erfolgreich wenn ihr seht: `Login Succeeded`

---

## Schritt 3: Docker Image bauen und pushen

### 3.1 Image bauen (Multi-Platform für Mac + HNU Server)

⚠️ **Wichtig:** Ersetzt in den folgenden Befehlen:
- `EUER_GITHUB_USERNAME` mit eurem GitHub-Benutzernamen (z.B. `max-mustermann`)
- `EUER_APP_NAME` mit einem Namen für eure App (z.B. `my-spring-app`)

```bash
# Im Projekt-Verzeichnis (dort wo das Dockerfile liegt):
cd /pfad/zu/eurem/projekt

# Multi-Platform Build und Push zu GitHub
docker buildx build --platform linux/amd64,linux/arm64 \
  -t ghcr.io/EUER_GITHUB_USERNAME/EUER_APP_NAME:latest \
  --push .
```

**Beispiel:**
```bash
docker buildx build --platform linux/amd64,linux/arm64 \
  -t ghcr.io/max-mustermann/bookstore-app:latest \
  --push .
```

### 3.2 Warten bis Build fertig ist

Der Build dauert beim ersten Mal 2-5 Minuten. Ihr seht den Fortschritt im Terminal.

✅ Erfolgreich wenn am Ende steht: `=> exporting to image` und `=> pushing layers`

---

## Schritt 4: Auf HNU Cloud deployen

### 4.1 Mit HNU Cloud verbinden

1. Öffnet Browser: http://194.95.24.122:8080
2. Loggt euch mit euren Zugangsdaten ein
3. Öffnet ein Terminal in der WebConsole

**Alternativ per SSH (falls verfügbar):**
```bash
ssh swe-deployer@194.95.24.122
```

### 4.2 Image von GitHub laden

Im HNU Cloud Terminal:

```bash
# Image pullen (ersetzt EUER_GITHUB_USERNAME und EUER_APP_NAME)
sudo docker pull ghcr.io/EUER_GITHUB_USERNAME/EUER_APP_NAME:latest
```

**Beispiel:**
```bash
sudo docker pull ghcr.io/max-mustermann/bookstore-app:latest
```

### 4.3 Container starten

⚠️ **Wichtig:** Verwendet euren zugewiesenen Port!

```bash
# Container starten (ersetzt 8083 mit EUREM Port!)
sudo docker run -d \
  -p 8083:8083 \
  --name meine-app \
  ghcr.io/EUER_GITHUB_USERNAME/EUER_APP_NAME:latest
```

**Beispiel für Student 3 (Port 8083):**
```bash
sudo docker run -d \
  -p 8083:8083 \
  --name bookstore \
  ghcr.io/max-mustermann/bookstore-app:latest
```

### 4.4 Überprüfen ob Container läuft

```bash
# Container-Status prüfen
sudo docker ps

# Logs anschauen (um zu sehen ob die App gestartet ist)
sudo docker logs meine-app

# Nach "Started" oder "Tomcat started" suchen
sudo docker logs meine-app | grep -i "started"
```

### 4.5 App testen

```bash
# Lokaler Test auf dem Server
curl http://localhost:8083/books

# Oder im Browser (außerhalb der HNU Cloud):
# http://194.95.24.122:8083/books
```

✅ Wenn ihr `[]` oder JSON-Daten seht → **Erfolg!** 🎉

---

## Häufige Befehle

### Container verwalten

```bash
# Alle laufenden Container anzeigen
sudo docker ps

# Alle Container (auch gestoppte) anzeigen
sudo docker ps -a

# Container stoppen
sudo docker stop meine-app

# Container löschen
sudo docker rm meine-app

# Container neustarten
sudo docker restart meine-app

# Logs live ansehen
sudo docker logs -f meine-app
```

### Bei Code-Änderungen: Update deployen

```bash
# 1. Lokal: Neues Image bauen und pushen
docker buildx build --platform linux/amd64,linux/arm64 \
  -t ghcr.io/EUER_GITHUB_USERNAME/EUER_APP_NAME:latest \
  --push .

# 2. Auf HNU Cloud: Alten Container stoppen und entfernen
sudo docker stop meine-app
sudo docker rm meine-app

# 3. Neues Image pullen
sudo docker pull ghcr.io/EUER_GITHUB_USERNAME/EUER_APP_NAME:latest

# 4. Neuen Container starten
sudo docker run -d \
  -p 8083:8083 \
  --name meine-app \
  ghcr.io/EUER_GITHUB_USERNAME/EUER_APP_NAME:latest
```

---

## Troubleshooting

### Problem: "Port already in use"

```bash
# Findet heraus, welcher Container den Port nutzt
sudo docker ps

# Stoppt den Container, der den Port blockiert
sudo docker stop CONTAINER_NAME
```

### Problem: "Login failed" bei docker push

- Stellt sicher, dass ihr euren **Personal Access Token** verwendet, NICHT euer GitHub-Passwort
- Token braucht die Berechtigung `write:packages`

### Problem: Container startet nicht

```bash
# Logs ansehen um Fehler zu finden
sudo docker logs meine-app

# Häufige Ursachen:
# - Falscher Port in application.properties
# - Datenbank nicht erreichbar
# - Syntax-Fehler im Code
```

### Problem: "no matching manifest for linux/amd64"

- Ihr habt vergessen `--platform linux/amd64,linux/arm64` beim Build zu verwenden
- Baut das Image nochmal mit der Multi-Platform Option

### Problem: App läuft, aber nicht erreichbar

```bash
# Prüft ob Port wirklich offen ist
sudo netstat -tulpn | grep 8083

# Test im Container selbst
sudo docker exec meine-app curl http://localhost:8083/books
```

---

## Zusammenfassung: Kompletter Workflow

### Einmalig: Setup
1. GitHub Account erstellen
2. Docker Desktop installieren
3. Personal Access Token erstellen
4. `docker login ghcr.io` ausführen
5. Port-Zuteilung vom Dozenten erhalten

### Bei jeder Änderung: Deploy
1. **Lokal:** Code ändern → Port in `application.properties` setzen
2. **Lokal:** `docker buildx build --platform linux/amd64,linux/arm64 -t ghcr.io/USER/APP:latest --push .`
3. **HNU Cloud:** Alten Container stoppen und löschen
4. **HNU Cloud:** `sudo docker pull ghcr.io/USER/APP:latest`
5. **HNU Cloud:** `sudo docker run -d -p PORT:PORT --name app ghcr.io/USER/APP:latest`
6. **Testen:** `curl http://localhost:PORT/...`

---

## Checkliste vor dem ersten Deployment

- [ ] Port vom Dozenten erhalten (z.B. 8083)
- [ ] Port in `application.properties` gesetzt
- [ ] `Dockerfile` erstellt
- [ ] GitHub Account vorhanden
- [ ] Personal Access Token erstellt (`write:packages`)
- [ ] `docker login ghcr.io` erfolgreich
- [ ] Image gebaut mit `--platform linux/amd64,linux/arm64`
- [ ] Image zu GitHub gepusht
- [ ] Zugang zur HNU Cloud WebConsole

---

## Support

Bei Problemen:
1. Logs prüfen: `sudo docker logs meine-app`
2. Container-Status prüfen: `sudo docker ps -a`
3. Dozent oder Kommilitonen fragen
4. Dokumentation nochmal durchlesen

**Viel Erfolg! 🚀**