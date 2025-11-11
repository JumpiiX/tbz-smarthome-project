# Custom Apple SmartHome Bridge

**TBZ Höhere Fachschule für Technik Zürich**  
**Student:** David Unterguggenberger | **Klasse:** ITCNE24 - 4. Semesterarbeit  
**Zeitraum:** 27.10.2025 - 28.01.2026  
**Supervisors:** Philip Stark, Corrado Parisi

---

## Wichtige Links

- **Source Code:** [smarthome-tivoli](https://github.com/JumpiiX/smarthome-tivoli)
- **GitHub Projects:** [Sprint Board](https://github.com/users/JumpiiX/projects/3)
- **Milestones:** [Github Milestones](https://github.com/JumpiiX/tbz-smarthome-project/milestones)


---

## Einführung

Diese Semesterarbeit entwickelt eine Kubernetes-basierte Plattform für eine Apple HomeKit Bridge, die KNX-SmartHome-Systeme mit dem Apple-Ökosystem verbindet.

Die Bridge ist in Rust geschrieben und ermöglicht es Bewohnern, ihre SmartHome-Geräte über Siri, die Home App und andere Apple-Geräte zu steuern - ohne den Umweg über eine Website mit Login.

---

## Problemstellung

### Die Situation

Ich wohne in der **[Tivoli Garten Überbauung](https://www.tivoli-garten.ch/de-ch/roomfinder-wohnen.html#/objects/0) in Spreitenbach** - 400 Apartments mit KNX-SmartHome. Die offizielle Steuerung läuft über eine Website mit Login, das nach 24 Stunden abläuft.

Das bedeutet: Jeden Tag Browser öffnen, einloggen, Captcha lösen, warten. Für Apple-User wie mich ist das alles andere als smart.

### Meine Lösung

Ich habe eine Rust-Software entwickelt, die als Bridge zwischen KNX und Apple HomeKit funktioniert. Seit einer Woche läuft sie bei mir - einfach "Hey Siri, Lichter aus" statt Website-Klickerei.

### Das Problem

Es hat sich rumgesprochen und 50+ Nachbarn haben sich innerhalb von 2 Tagen gemeldet und wollen das jetzt auch. Aber ich kann nicht 50x alles manuell aufsetzen. Jedes Apartment braucht:

- Eine eigene Container-Instanz
- Verschlüsselte Login-Daten
- Strikte Isolation (niemand darf auf fremde Apartments zugreifen)

### Die Lösung

Diese Arbeit baut eine **Kubernetes-Infrastruktur**, die:
- Automatisch neue Apartments deployed
- Credentials sicher verschlüsselt (Sealed Secrets)
- Strict isoliert (Namespace pro Apartment)
- Einfach zu warten ist (GitOps mit ArgoCD)

---

## Projektziele

1. **Automatisches Deployment:** Neues Apartment via Git-Push einsatzbereit
2. **Sichere Credentials:** Alle KNX-Logins verschlüsselt, niemals im Klartext
3. **Monitoring:** Dashboard zeigt Status aller Apartments auf einen Blick
4. **Live-Demo:** Mehrere Apartments laufen und ich kann ein Licht per Siri steuern

---

## Tech-Stack

**Infrastructure:** Kubernetes (kind + Hetzner), ArgoCD, Sealed Secrets  
**Application:** Rust, Docker, Helm  
**CI/CD:** GitHub Actions  
**Hardware:** MacBook Pro M4 Max

---

## Sprint-Übersicht

| Sprint | Zeitraum | Ziel |
|--------|----------|------|
| **Sprint 1** | 27.10 - 17.11 | Software production-ready + manuell auf Hetzner deployed |
| **Sprint 2** | 18.11 - 15.12 | Kubernetes + GitOps Automation |
| **Sprint 3** | 16.12 - 23.01 | Multi-Tenant Deployment + Monitoring |
| **Finalisierung** | 24.01 - 28.01 | Doku + Präsentation |

---

# Sprint 1: Foundation & Manual Deployment (27.10 - 17.11.2025)

## Sprint 1 Planung

**Sprint-Ziel:** Die Rust-Software production-ready machen und erfolgreich auf einem Hetzner Server deployen. Am Ende soll ein komplettes End-to-End Test mit echten KNX-Geräten laufen.

### Sprint Planning Video

Nach Corrado's Empfehlung habe ich ein Sprint Planning Video erstellt, damit die Dozenten nicht jedes Mal durch GitHub-Änderungen scrollen müssen. Das Video zeigt die geplanten User Stories, Zeitschätzungen und Sprint Goals.

### User Stories & Story Points

Ich arbeite mit der **MoSCoW-Methode** für Priorisierung und **Story Points (Fibonacci)** für Schätzungen.

| User Story | Priority | Story Points | Status |
|------------|----------|--------------|--------|
| Story 1: Google Captcha Handling | Nice-to-Have | 13 | Done |
| Story 2: Generic Multi-Apartment Support | Must-Have | 8 | Done |
| Story 3: Hetzner Server Provisioning | Must-Have | 2 | Done |
| Story 4: Server Dependencies Installation | Must-Have | 8 | Done |
| Story 5: Manuelles Deployment | Must-Have | 8 | Done |
| Story 6: End-to-End Test | Must-Have | 5 | Done |
| Story 7: Sprint 1 Dokumentation | Must-Have | 3 | Done |
| Story 8: Sprint 1 Review & Retrospektive | Must-Have | 2 | Done |

**Total: 49 Story Points**

Nach Philip's Feedback wurde Story 1 von Must-Have auf Nice-to-Have verschoben.

---

## Sprint 1 Durchführung

### Story 3: Hetzner Server Setup

**Status:** Done

Hetzner Cloud Server gemietet mit folgenden Specs:
- **CX33** mit Ubuntu 24.04 LTS
- 2 vCPU AMD, 8 GB RAM
- 80 GB NVMe SSD
- Standort: Falkenstein

**Was fast schiefging:**

Beim Firewall-Setup hatte ich einen klassischen Moment. Ich wollte die SSH-Regeln "sauber" neu konfigurieren und hab kurz SSH deaktiviert. Ergebnis: Ausgesperrt vom eigenen Server.

Glücklicherweise hatte ich noch die Hetzner Web-Console offen und konnte mich darüber retten. Learning: Firewall-Regeln immer erst testen, dann aktivieren.

**Final Setup:**
```bash
# System aktualisieren
apt update && apt upgrade -y
apt install -y curl wget git build-essential vim

# Firewall richtig konfigurieren
ufw allow 22/tcp
ufw allow 80/tcp
ufw allow 443/tcp
ufw status  # Checken ob's klappt
ufw enable

# Zeitzone setzen
timedatectl set-timezone Europe/Zurich
```

### Story 4: Dependencies Installation

**Status:** Done

Installation der benötigten Software:
```bash
# Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env

# Google Chrome (für headless browsing)
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
apt install -y ./google-chrome-stable_current_amd64.deb

# X11 Forwarding Tools
apt install -y xauth xvfb

# PostgreSQL Client
apt install -y postgresql-client
```

Lief alles problemlos durch.

### Story 1: Google Captcha Handling - Die Lösung

**Status:** Done

**Das Problem:**

Die KNX-Website hat Login mit Google Captcha. Automatisches Einloggen ist schwierig wegen Bot-Detection.

**Philip's berechtigte Warnung:**

Philip hat mich darauf hingewiesen, dass das Umgehen von Google Captchas problematisch ist und zu Account-Sperrungen führen kann. Das war wichtiges Feedback.

**Meine Lösung:**

Statt Bot-Detection zu umgehen, hab ich einen pragmatischen Hybrid-Ansatz gewählt:

1. **Erstes Login:** Manuell via X11 Forwarding
   - Chrome läuft auf dem Server, Fenster auf meinem Mac
   - User loggt sich einmal manuell ein
   - Chrome speichert alle Session-Daten (Cookies, etc.)

2. **Alle weiteren Logins:** Automatisch
   - Chrome nutzt gespeicherte Sessions
   - Kein Captcha mehr nötig
   - Funktioniert zu 100%

**X11 Forwarding Setup:**

Das X11 Setup brauchte einige Versuche. Das Problem war, dass XQuartz auf meinem Mac auf Display `:2` läuft, nicht `:0` wie ich dachte.
```bash
# Auf Mac: Display checken
ps aux | grep Xquartz  # Zeigt :2
export DISPLAY=:2
ssh -Y root@server     # -Y statt -X verwenden

# Auf Server:
echo $DISPLAY          # localhost:10.0
xeyes                  # Test - Augen erscheinen auf Mac
```

Nach dem X11 Setup funktioniert der Discovery-Modus einwandfrei. Chrome öffnet sich, User loggt sich ein, `chrome_data/` wird gespeichert, und ab dann laufen alle Logins automatisch.

**Rechtliche Absicherung:**

In den Verträgen mit Kunden habe ich dokumentiert:
- As-is Software mit Lizenzzahlung
- Warnung: Wenn sich die Website ändert, funktioniert die Software nicht mehr
- Hinweis: Manuelle Erst-Login nötig, Account kann theoretisch gesperrt werden
- Jeder Kunde hat dies gelesen und akzeptiert

Die Accounts sind dedizierte Vermietungs-Accounts, keine privaten Google Accounts.

### Story 2: Generic Multi-Apartment Support

**Status:** Done

Die Software war ursprünglich nur für mein Apartment hardcoded. Jetzt ist sie generisch und funktioniert für jedes Apartment:
```rust
// Vorher: Hardcoded
let page_01_lights = vec!["06+01+00+01", "07+01+00+01"];

// Nachher: Auto-Discovery
let discovered = auto_discover_all_devices().await?;
```

Die Auto-Discovery findet automatisch alle Geräte auf allen Pages - keine manuelle Konfiguration nötig.

### Story 5 & 6: Deployment & End-to-End Test

**Status:** Done

**Deployment:**
```bash
git clone https://github.com/JumpiiX/smarthome-tivoli
cd smarthome-tivoli
cargo build --release
cargo run --release -- --discover
```

**Test-Ergebnis:**
```
Already logged in! (Session restored from chrome_data/)
Discovering devices on page 01...
  Found 11 devices on page 01
Discovering devices on page 02...
  Found 11 devices on page 02
Discovering devices on page 03...
  Found 1 devices on page 03
Discovering devices on page 04...
  Found 7 devices on page 04
  
Discovery complete! Found 38 device mappings
Saved to device_mappings_auto.toml
```

**38 Geräte automatisch gefunden**, darunter:
- Lichter (Essen, Küche, Wohnen, Loggia, Eingang, Reduit)
- Storen (Küche, Wohnen, Zentral)
- Markise (Loggia)
- Temperatursensoren
- Lüftungsstufen (1-3)
- Szenen

**Siri-Test:** "Hey Siri, Licht Küche einschalten" - funktioniert.

---

## Sprint 1 Review (17.11.2025)

### Demo

Sprint Review Video erstellt mit:
- GitHub Board Status (alle Stories Done)
- Live Discovery-Prozess
- 38 gefundene Devices

### Feedback von Corrado

**Positiv:**
- Video-Format kommt gut an
- Generische Lösung ist sinnvoll
- Rust als Tech-Stack passt
- Dependencies und Deployment gut strukturiert

**Action Items:**
- MoSCoW-Methode in Doku aufnehmen
- Story Points sichtbar machen

### Feedback von Philip

**Captcha-Warnung:**

Philip hat mich auf die Risiken von Captcha-Umgehung hingewiesen. Das war wichtig und hat mich zum Umdenken gebracht.

**Meine Reaktion:**

Story 1 von Must-Have auf Nice-to-Have verschoben und pragmatische Lösung implementiert. Philip's Rückmeldung:

> "Vielen Dank für die ausführliche Antwort. Damit ist das Thema für mich erledigt. Ich finde es toll, dass du dir die Gedanken gemacht hast."

### Sprint 1 Metriken

| Metrik | Geplant | Erreicht |
|--------|---------|----------|
| Story Points | 49 | 49 |
| Stories | 8 | 8 |
| Sprint Goal | Erreicht | Ja |

**MVP erreicht:** Software läuft production-ready auf Hetzner.

---

## Sprint 1 Retrospektive

### Keep (Beibehalten)

- Video-Updates für Stakeholder
- Proaktive Kommunikation mit Dozenten
- Pragmatische Lösungsansätze

### More (Mehr davon)

- Automatisierte Tests
- Dokumentation während der Arbeit

### Less (Weniger davon)

- Zeit für X11 Debugging (11 Versuche waren zu viel)
- Perfektionismus bei Configs

### Stop (Aufhören)

- Firewall ohne Test aktivieren

### Start (Neu beginnen)

- Estimate-Field statt Labels (ab Sprint 2)
- Regelmässige Backups

---

## Sprint 1 Fazit

**Erreicht:**
- Server läuft stabil auf Hetzner
- Software ist production-ready
- 38 Devices automatisch entdeckt
- Siri-Integration funktioniert
- Pragmatische Captcha-Lösung (100% zuverlässig)

**Gelernt:**
- X11 Forwarding ist machbar aber braucht Geduld
- Firewall-Regeln immer testen vor Aktivierung
- Pragmatische Lösungen sind besser als Hacks
- Video-Updates sparen allen Zeit
