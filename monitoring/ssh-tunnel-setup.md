# SSH-Tunnel-Setup für Prometheus und Alertmanager

Diese Anleitung ist bewusst eigenständig gehalten, damit sie separat verlinkt werden kann.

Ziel ist ein lokaler Zugriff auf Prometheus und Alertmanager, auch wenn die Dienste nicht öffentlich erreichbar sind.

---

## Voraussetzungen

- SSH-Zugang zum Server funktioniert bereits
- Login erfolgt per **SSH-Key**, nicht per Passwort
- der private Key kann lokal eine **Passphrase** haben
- auf dem Server sind die Dienste lokal oder intern erreichbar

Die Passphrase schützt nur den **lokalen privaten Schlüssel**. Auf dem Server muss dafür kein Passwort-Login aktiviert sein.

---

## Einfacher Tunnel mit SSH-Key

### Windows PowerShell

```powershell
ssh -i "$HOME/.ssh/example/vServer" -L 9093:127.0.0.1:9093 john@23.456.789.12
```

### Linux / macOS

```bash
ssh -i ~/.ssh/example/vServer -L 9093:127.0.0.1:9093 john@23.456.789.12
```

Danach ist der Alertmanager lokal erreichbar unter:

```text
http://127.0.0.1:9093
```

Wenn der Key mit einer Passphrase geschützt ist, fragt SSH diese beim Verbindungsaufbau ab.

---

## Zwei Tunnel gleichzeitig

Für Prometheus und Alertmanager zusammen:

### Windows PowerShell

```powershell
ssh -i "$HOME/.ssh/example/vServer" -L 9090:127.0.0.1:9090 -L 9093:127.0.0.1:9093 john@23.456.789.12
```

### Linux / macOS

```bash
ssh -i ~/.ssh/example/vServer -L 9090:127.0.0.1:9090 -L 9093:127.0.0.1:9093 john@23.456.789.12
```

Danach lokal erreichbar:

- Prometheus: `http://127.0.0.1:9090`
- Alertmanager: `http://127.0.0.1:9093`

---

## SSH-Config verwenden

Wenn ein Alias in der lokalen SSH-Config hinterlegt ist, wird der Befehl deutlich kürzer.

Beispiel für `~/.ssh/config` bzw. unter Windows `C:\Users\DEINNAME\.ssh\config`:

```sshconfig
Host vServer
    HostName 23.456.789.12
    User john
    IdentityFile ~/.ssh/example/v_server
```

Dann reicht später:

```bash
ssh -L 9090:127.0.0.1:9090 -L 9093:127.0.0.1:9093 vServer
```

---

## Optional: ssh-agent für Keys mit Passphrase

Wenn die Passphrase nicht bei jeder Verbindung neu eingegeben werden soll, kann der Key in den SSH-Agent geladen werden.

### Windows PowerShell

```powershell
Get-Service ssh-agent | Set-Service -StartupType Automatic
Start-Service ssh-agent
ssh-add "$HOME/.ssh/example/vServer"
```

### Linux

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/example/vServer
```

Danach fragt SSH die Passphrase in der Regel nur einmal pro Sitzung.

---

## Typische Prüf-URLs

Prometheus-Regeln prüfen:

```text
http://127.0.0.1:9090/rules
```

Prometheus-Alerts prüfen:

```text
http://127.0.0.1:9090/alerts
```

Alertmanager-Alerts prüfen:

```text
http://127.0.0.1:9093/#/alerts
```

---

## Nagstamon mit SSH-Tunnel

Wenn der Tunnel aktiv ist, kann in Nagstamon ein neuer Server so angelegt werden:

- **Monitor type**: `Alertmanager`
- **Monitor URL**: `http://127.0.0.1:9093`
- **Username**: leer
- **Password**: leer

Der Tunnel muss aktiv bleiben, solange der Zugriff über `127.0.0.1` genutzt wird.

---

## Wann SSH-Tunnel, wann WireGuard?

### SSH-Tunnel ist sinnvoll, wenn:

- nur kurz getestet werden soll
- kein VPN verbunden ist
- nur 1–2 Dienste lokal erreichbar sein sollen

### WireGuard ist sinnvoll, wenn:

- dauerhaft auf mehrere Dienste zugegriffen wird
- Nagstamon, Grafana und Prometheus direkt erreichbar sein sollen
- keine einzelnen Tunnel pro Port gepflegt werden sollen

---

## Fehlerbilder

### `connection refused`

Mögliche Ursachen:

- Dienst läuft auf dem Server nicht
- Port ist im Container oder auf dem Host nicht veröffentlicht
- falscher lokaler oder entfernter Port im Tunnel
- Zieladresse `127.0.0.1:9093` existiert auf dem Server nicht

### `Permission denied (publickey)`

Mögliche Ursachen:

- falscher SSH-User
- falscher Key-Pfad
- Key nicht auf dem Server hinterlegt
- SSH-Config verweist auf den falschen Schlüssel

### Tunnel steht, aber Nagstamon zeigt nichts

Dann ist meist nicht der Tunnel das Problem, sondern:

- noch kein `FIRING`-Alert vorhanden
- in Nagstamon falscher Monitor-Typ gewählt
- versehentlich `/#/alerts` statt der Basis-URL eingetragen

Richtig für Nagstamon:

```text
http://127.0.0.1:9093
```
