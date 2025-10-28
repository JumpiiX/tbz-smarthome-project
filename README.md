# Apple SmartHome Bridge - Kubernetes Multi-Tenant Platform

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
- Automatisch neue Apartments deployed (unter 5 Minuten via Git-Push)
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

## Dokumentation
