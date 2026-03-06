# MailHog — Administrator-Dokumentation

## Zweck
MailHog ist ein SMTP-Testserver zur Validierung von E-Mail-Funktionalität in Entwicklungs- und Testumgebungen. Ausgehende E-Mails werden abgefangen und über eine Weboberfläche visualisiert, ohne reale Empfänger zu kontaktieren.

---

## Systemübersicht

**Funktion:**
- SMTP-Testserver
- Weboberfläche zur Mail-Inspektion
- Basic-Auth Absicherung

**Standard-Ports:**
- SMTP: 1025/tcp
- Web UI: 8025/tcp

**Betriebsart:**
- Containerisiert (Docker Compose)

---

## Verzeichnisstruktur

```
/srv/mailhog/
 ├─ docker-compose.yml
 ├─ .env
 └─ README.md
```

---

## Voraussetzungen

- Linux-Server oder VM
- Docker Engine
- Docker Compose Plugin
- DNS-Eintrag (oder Hosts-Datei)
- htpasswd (Apache Tools) oder Docker

---

## Deployment

### 1. Arbeitsverzeichnis

```
/srv/mailhog
```

### 2. Umgebungsdatei erstellen

```
mv example.env .env
```

### 3. Passwort-Hash erzeugen

**Variante A — lokal:**

```
htpasswd -nb admin <passwort>
```

**Variante B — via Docker:**

```
docker run --rm httpd:2.4-alpine htpasswd -nb admin <passwort>
```

**Beispielausgabe:**
```
admin:$apr1$E8Xxhk8x$XzwhC5QLJnypjscOVv1E7/
```

### 4. Escape-Sequenz für Docker Compose

Alle `$`-Zeichen müssen maskiert werden:

```
$ → $$
```

**Beispiel:**
```
$$apr1$$E8Xxhk8x$$XzwhC5QLJnypjscOVv1E7/
```

### 5. .env Konfiguration

```
MAILHOG_HOST=mailhog.home.lab
MAILHOG_USER=admin
MAILHOG_PASSWORD_HASH=$apr1$E8Xxhk8x$XzwhC5QLJnypjscOVv1E7/
```

### 6. Container-Start

```
cd /srv/mailhog
docker compose up -d
```

---

## Netzwerk & DNS

### DNS-Anforderung

Hostname muss intern auf den Docker-Host auflösen.

**Beispiel:**
```
mailhog.home.lab → 192.168.178.56
```

Alternativ: Eintrag in `/etc/hosts`

---

## Zugriffspunkte

**Intern (Docker-Netzwerk):**
```
smtp://mailhog:1025
```

**Extern (Browser):**
```
http://<host-ip>:8025
https://mailhog.home.lab
```

---

## Authentifizierung

- HTTP Basic Authentication
- Benutzername: MAILHOG_USER
- Passwort: Klartextpasswort des htpasswd-Hashes

---

## Betrieb

### Container-Status
```
docker compose ps
```

### Logs
```
docker compose logs -f
```

### Neustart
```
docker compose restart
```

### Stoppen
```
docker compose down
```

---

## Sicherheitshinweise

- Nicht produktiv einsetzen
- Zugriff auf internes Netz beschränken
- TLS über Reverse Proxy empfohlen
- Zugangsdaten nicht im Klartext versionieren

---

## Typische Einsatzszenarien

- Entwicklungsumgebungen
- CI/CD Testpipelines
- Lokale Integrations-Tests
- Formular- & Registrierungs-Workflows

