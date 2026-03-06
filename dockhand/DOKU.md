# Dockhand — Administrator-Dokumentation

## Zweck

Dockhand ist eine containerisierte Verwaltungsoberfläche für Docker-Umgebungen. In diesem Setup wird Dockhand über Traefik unter einer eigenen internen Domain bereitgestellt und verwendet PostgreSQL als Datenbank.

---

## Systemübersicht

**Funktion:**

- Weboberfläche für Docker-Verwaltung
- Persistente Datenspeicherung in PostgreSQL
- Reverse-Proxy-Anbindung über Traefik
- TLS-Bereitstellung über Zertifikatsresolver

**Betriebsart:**

- Containerisiert mit Docker Compose

**Zugriff:**

- URL: `https://dockhand.home.lab/`

---

## Verzeichnisstruktur

```text
/srv/dockhand/
 ├─ compose.yml
 ├─ example.env
 ├─ DOKU.md
 └─ README.md
```

---

## Voraussetzungen

- Linux-Server oder VM
- Docker Engine
- Docker Compose Plugin
- Bereits vorhandenes externes Docker-Netzwerk `proxy`
- Laufender Traefik-Proxy im Netzwerk `proxy`
- DNS-Eintrag für `dockhand.home.lab`

---

## Konfigurationsdateien

### Compose-Datei

Die Compose-Datei definiert einen PostgreSQL-Container und einen Dockhand-Container. PostgreSQL läuft ausschließlich im internen Netzwerk, Dockhand zusätzlich im externen Proxy-Netzwerk. Traefik veröffentlicht den Dienst per HTTPS. fileciteturn1file0

### Umgebungsdatei

Die Umgebungsdatei enthält die Zugangsdaten für PostgreSQL. fileciteturn1file1

```env
POSTGRES_USER=dockhand
POSTGRES_PASSWORD=dockhandpw
POSTGRES_DB=dockhand
```

---

## Netzwerkdesign

### Internes Netzwerk

Das Netzwerk `internal` wird durch Docker Compose automatisch projektspezifisch erstellt und dient ausschließlich der Kommunikation zwischen Dockhand und PostgreSQL.

### Externes Netzwerk

Das Netzwerk `proxy` ist als `external: true` definiert und muss bereits existieren. Es wird gemeinsam mit Traefik verwendet. fileciteturn1file0

---

## Deployment

### 1. Umgebungsdatei bereitstellen

```bash
cp example.env .env
```

### 2. Werte prüfen oder anpassen

```env
POSTGRES_USER=dockhand
POSTGRES_PASSWORD=dockhandpw
POSTGRES_DB=dockhand
```

### 3. Externes Proxy-Netzwerk prüfen

```bash
docker network ls
```

Falls das Netzwerk nicht existiert:

```bash
docker network create proxy
```

### 4. Stack starten

```bash
docker compose -f compose.yml up -d
```

---

## Traefik-Integration

Dockhand wird über folgende Labels an Traefik angebunden: Host-Regel `dockhand.home.lab`, EntryPoint `websecure`, TLS-Aktivierung, Zertifikatsresolver `stepca` und interner Service-Port `3000`. fileciteturn1file0

---

## DNS

Der Hostname `dockhand.home.lab` muss auf den Traefik-Host auflösen.

**Beispiel:**

```text
dockhand.home.lab → 192.168.178.56
```

---

## Betrieb

### Container-Status

```bash
docker compose -f compose.yml ps
```

### Logs anzeigen

```bash
docker compose -f compose.yml logs -f
```

### Einzelne Logs

```bash
docker logs dockhand
docker logs dockhand-db
```

### Neustart

```bash
docker compose -f compose.yml restart
```

### Stoppen

```bash
docker compose -f compose.yml down
```

### Vollständiger Neuaufbau inklusive Volumes

```bash
docker compose -f compose.yml down -v
docker compose -f compose.yml up -d
```

---

## Fehleranalyse

### Datenbankmigration schlägt fehl

Wenn Dockhand zwar PostgreSQL erreicht, aber Migrationen nicht ausführen kann, ist häufig ein bereits vorhandenes Datenvolume mit unpassenden Berechtigungen die Ursache.

**Empfohlene Maßnahme für Test- und Homelab-Umgebungen:**

```bash
docker compose -f compose.yml down -v
docker compose -f compose.yml up -d
```

### Erreichbarkeit prüfen

```bash
curl -I https://dockhand.home.lab/
```

### Proxy-Netzwerk prüfen

```bash
docker network inspect proxy
```

---

## Sicherheitshinweise

- PostgreSQL nicht im Proxy-Netzwerk bereitstellen
- Dockhand nur über Traefik veröffentlichen
- Docker-Socket nur bewusst und in vertrauenswürdigen Umgebungen einbinden
- Zugangsdaten nicht im Klartext versionieren
- TLS über Traefik aktiv halten

---

## Typische Einsatzszenarien

- Verwaltung lokaler Docker-Stacks
- Homelab-Administration
- Test- und Entwicklungsumgebungen
- Interne Serviceverwaltung hinter Reverse Proxy
