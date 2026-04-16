# Einrichtung Spark (Handout — anonymisiert)

Diese Fassung ist für **Veröffentlichung oder Weitergabe** gedacht: organisationsspezifische Namen, Pfade, Rechnernamen und personenbezogene Angaben sind durch **Platzhalter** ersetzt. Konkrete Werte erhältst du von **IT oder Administrator**.

**Hinweis:** Wenn du den Spark nicht selbst aufsetzt, sondern nur **deinen eigenen Computer** vorbereiten sollst, genügt in der Regel **Kapitel 4**.

**Stand:** April 2026

---

## 1. Einleitung

Diese Anleitung richtet sich an **Teams und Einzelpersonen**, die einen **NVIDIA DGX Spark** einsetzen möchten: KI-Modelle und sensible Daten **lokal** auf eigener Hardware betreiben und dabei mit **Cursor** auf dem Spark arbeiten. Damit Cursor von **überall** zuverlässig auf den Spark zugreifen kann, braucht es ein **sicheres Netz zwischen deinem Laptop und dem Spark** — dafür ist **Tailscale** vorgesehen. **NVIDIA Sync** verbindet den Spark mit der **Cursor-Installation auf deinem Rechner** und erleichtert die Einrichtung.

### Begriffe in Kürze

| Begriff | Kurz erklärt |
|--------|----------------|
| **DGX Spark** | Ein leistungsstarker KI-Arbeitsplatz (Server) bei euch im Unternehmen oder beim Anbieter. |
| **Tailscale** | Erstellt ein **verschlüsseltes Netz** zwischen deinem Computer und dem Spark. |
| **SSH** | **Verschlüsselte Verbindung** zum Spark; Cursor nutzt das im Hintergrund. |
| **NVIDIA Sync** | Programm von NVIDIA: verbindet den Spark mit **Cursor** auf deinem Rechner. |
| **Cursor** | Editor zum Arbeiten auf dem Spark (ähnlich Visual Studio Code). |

Ohne funktionierendes **Tailscale** funktioniert der Fernzugriff in der Regel **nicht** — deshalb zuerst Tailscale sauber einrichten.

---

## 2. Spark aufsetzen

Dieses Kapitel richtet sich an die Person, die den **Spark physisch** betreut und **Linux-Dienste** einrichtet.

### 2.1 Kurzüberblick

- Spark anschliessen (Strom, Netzwerk laut Herstellerangaben).
- Nach dem ersten Start: Linux-Oberfläche oder Terminal laut NVIDIA-Dokumentation nutzen.
- **Tailscale** und **SSH** auf dem Spark installieren und prüfen.

### 2.2 Recovery (nur falls nötig)

- <https://docs.nvidia.com/dgx/dgx-spark/system-recovery.html>
- Recovery-Image (Version kann wechseln): <https://developer.nvidia.com/downloads/dgx-spark/>

### 2.3 Tailscale auf dem Spark installieren

Als Benutzer mit `sudo`-Rechten:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

### 2.4 SSH-Server auf dem Spark

```bash
sudo apt update
sudo apt install -y openssh-server
sudo systemctl enable --now ssh
sudo systemctl status ssh --no-pager
```

**Soll-Zustand:** `active (running)`.

### 2.5 Gleiches Tailnet für alle Geräte

**Spark**, **dein Laptop** und alle, die zugreifen sollen, müssen im **gleichen Tailscale-Tailnet** sein — **konkreten Tailnet-Namen von der IT erfragen**. Sonst gibt es keinen `ping`, `tailscale ping` meldet u. U. „no matching peer“, und SSH wirkt „tot“.

---

## 3. Benutzer erstellen

Neue Linux-Benutzer auf dem Spark (Beispiel — **Benutzername** anpassen):

```bash
sudo adduser <benutzername>
sudo usermod -aG sudo,adm,audio,dip,plugdev,lpadmin <benutzername>
```

### 3.1 Gemeinsamer Projektordner (Beispiel)

| Thema | Inhalt |
|--------|--------|
| **Pfad** | Von der IT vorgegeben, z. B. `/home/<erstbenutzer>/Dokumente/Repository` — **nicht** blind aus dieser Anleitung übernehmen. |
| **Freigabe** | Typisch: Gruppe `users`, Lesen und Schreiben für die Gruppe; neue Dateien erben die Gruppe (Setgid). |
| **In Cursor** | **File → Open Folder** und den **von der IT genannten** Pfad wählen. |

### 3.2 Repository im Home-Verzeichnis sichtbar machen

Damit in Cursor unter jedem Benutzer (z. B. `BENUTZERNAME [SSH: …]`) der Eintrag **Repository** direkt sichtbar ist, beim neuen Benutzer einen Symlink anlegen — **Quellpfad von der IT**:

```bash
ln -s <PFAD_ZUM_GEMEINSAMEN_REPOSITORY> /home/<benutzername>/Repository
```

**Prüfen:**

```bash
ls -ld /home/<benutzername>/Repository
```

**Erwartung:** Ausgabe enthält `->` und den korrekten Zielpfad.

**Wichtig:** Falls `/home/<benutzername>/Repository` bereits als echter Ordner existiert, zuerst umbenennen oder Inhalt migrieren — danach erst den Symlink setzen.

**Ansicht in Cursor:**

- Wenn du **`/home`** öffnest, siehst du die Benutzerordner deiner Installation.
- Wenn du nur **deinen eigenen Ordner** öffnest, siehst du nur diesen Bereich; `Repository` erscheint dort, **falls der Symlink gesetzt ist**.
- Nur der gemeinsame Projektordner ist typischerweise geteilt; private Dateien im restlichen Home bleiben getrennt.

### 3.3 Konten und Passwörter

Linux-Benutzer, Passwörter und Berechtigungen werden **nicht** in dieser anonymisierten Anleitung aufgeführt — **ausschliesslich bei IT/Administration erfragen**.

---

## 4. Einrichtung auf deinem Computer (Cursor, Tailscale, NVIDIA Sync)

### 4.1 Voraussetzungen

| Was | Details |
|-----|---------|
| **Computer mit Internet** | Mac, Windows oder Linux. |
| **Benutzername und Passwort** | Für deinen Linux-Account auf dem Spark — von **IT / Administrator**. |
| **Konto für Tailscale** | Z. B. Google, GitHub oder Microsoft — **so, wie es eure Organisation vorgibt**. |
| **Richtiges Tailnet** | Alle müssen im **gleichen** Tailnet sein — **Name von der IT**. |

### 4.2 Tailscale herunterladen und installieren

| Schritt | Mac | Windows | Linux |
|--------|-----|---------|-------|
| **1. Download** | **https://tailscale.com/download/** — Mac-Version. | Dieselbe Seite — **Windows**. | Dieselbe Seite — **Linux**. |
| **2. Installation** | Paket öffnen, Anweisungen folgen. | Installer ausführen. | Je nach Distribution: Paketmanager oder Anleitung auf der Tailscale-Seite. |
| **3. Start & Login** | Tailscale starten, **Log in**. | Taskleiste, **Anmelden**. | Anwendungsmenü, anmelden. |

### 4.3 Tailnet prüfen

- Im Browser: **https://login.tailscale.com/admin/machines**
- **Dein Computer** und der Spark müssen in derselben Liste erscheinen (Spark-Name von der IT).

### 4.4 Terminal öffnen

| System | So geht’s |
|--------|-----------|
| **Mac** | **Cmd + Leertaste**, „Terminal“, **Enter**. |
| **Windows** | **Windows-Taste**, „PowerShell“ oder „Terminal“. |
| **Linux** | Terminal-App aus dem Menü. |

### 4.5 Verbindung testen (Tailscale-Ping)

```bash
tailscale ping DEINE-SPARK-IP
```

**Erwartung:** Meldungen mit **„pong from …“**. **Spark-IP** von der IT oder aus der Geräteliste.

### 4.6 SSH testen (optional)

```bash
ssh DEIN-BENUTZERNAME@DEINE-SPARK-IP
```

Beim ersten Mal: `yes` bei Host-Key-Frage. Passwort von der IT. Beenden mit `exit`.

### 4.7 NVIDIA Sync installieren

| Schritt | Aktion |
|--------|--------|
| 1 | **https://build.nvidia.com/spark/connect-to-your-spark/sync** |
| 2 | Installationsdatei für **Mac / Windows / Linux** herunterladen. |
| 3 | Installieren und starten, EULA prüfen. |
| 4 | Optional **Cursor** als IDE auswählen. |

### 4.8 Spark in NVIDIA Sync eintragen

| Feld | Inhalt |
|------|--------|
| Hostname / IP | **Tailscale-IP** des Spark (vom Admin). |
| Benutzername | Dein Spark-Login |
| Passwort | Dein Spark-Passwort |

### 4.9 Mit dem Spark verbinden und Cursor starten

1. In NVIDIA Sync **Connect** bis **Connected**.
2. **Cursor** starten.
3. **File → Open Folder** → Pfad **von der IT**.
4. Optional: **File → Open Workspace from File…** und die **gemeinsame Workspace-Datei** wählen (Name von der IT).

### 4.10 Ohne NVIDIA Sync (Alternative)

| Schritt | Aktion |
|--------|--------|
| Cursor installieren | Von der Cursor-Webseite |
| Erweiterung **Remote - SSH** | In Cursor unter Extensions |
| Verbinden | **Remote-SSH: Connect to Host** |
| Host | `benutzer@tailscale-ip` |

### 4.11 Fehlerbehebung

| Problem | Was tun |
|--------|---------|
| `tailscale: command not found` | Tailscale installieren, ggf. Rechner neu starten. |
| `tailscale ping` / Timeout | Tailscale neu verbinden; **gleiches Tailnet** prüfen. |
| **no matching peer** | Administrator. |
| SSH „hängt“ | **Strg + C**, dann `tailscale ping`. |
| NVIDIA Sync | Gerät löschen, mit **Tailscale-IP** neu anlegen. |
| Cursor verbindet nicht | Zuerst SSH im Terminal testen. |
| **Permission denied** | Administrator. |
| `Repository` fehlt | Symlink prüfen/anlegen (Pfad von der IT). |

### 4.12 Referenzwerte (nur Platzhalter)

| Angabe | Hinweis |
|--------|---------|
| **Spark-Hostname** | Von der IT, z. B. `spark-<kennung>` |
| **Tailnet** | Von der IT |
| **Tailscale-IP** | Aus **deiner** Geräteliste — ändert sich je nach Netz; nicht aus fremden Dokumenten übernehmen. |

### 4.13 Hilfe

**IT oder Administrator** kontaktieren — mit **kurzer Fehlermeldung** oder Screenshot (**keine Passwörter**).
