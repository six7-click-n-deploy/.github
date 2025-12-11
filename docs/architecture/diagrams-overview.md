# Architektur-Diagramme Übersicht

Diese Seite bietet eine Schnellübersicht aller UML-Diagramme mit Beschreibungen.

---

## 1️⃣ Komponentendiagramm

**Datei**: [`component-diagram.puml`](./component-diagram.puml)

### Zweck
Zeigt die **logische Struktur** des Systems mit allen Hauptkomponenten und deren Interaktionen.

### Inhalt
- **Frontend-Komponenten**: Vue 3 SPA, Router, Stores, Views
- **Backend-Komponenten**: FastAPI, Routers, Services, ORM
- **Worker-Komponenten**: Celery Worker, Git Ops, Terraform Ops
- **Infrastruktur**: PostgreSQL, RabbitMQ, Redis
- **Externe Systeme**: GitHub, OpenStack

### Wichtige Erkenntnisse
- Klare Trennung zwischen Frontend, Backend und Worker
- Message-Queue-basierte asynchrone Kommunikation
- Integration mit GitHub für Template-Repositories
- OpenStack als Deployment-Ziel

---

## 2️⃣ Deployment-Diagramm

**Datei**: [`deployment-diagram.puml`](./deployment-diagram.puml)

### Zweck
Beschreibt die **physische Infrastruktur** und wie Komponenten auf Server/Container verteilt sind.

### Inhalt
- **Docker-Container**: Frontend, Backend, Worker, PostgreSQL, RabbitMQ, Redis
- **Docker-Netzwerke**: frontend-network, backend-network, worker-network
- **Volumes**: Persistente Datenspeicherung und Hot-Reload
- **Port-Mapping**: Externe Zugriffe auf Services
- **OpenStack-Integration**: Compute, Network, Storage Nodes

### Wichtige Erkenntnisse
- Alle Services laufen in Docker-Containern
- Docker Compose orchestriert die Umgebung
- Worker-Container können dynamisch skaliert werden
- Entwicklung und Produktion nutzen unterschiedliche Compose-Dateien

---

## 3️⃣ Sequenzdiagramm

**Datei**: [`sequence-diagram.puml`](./sequence-diagram.puml)

### Zweck
Illustriert den **zeitlichen Ablauf** eines typischen App-Deployments von Anfang bis Ende.

### Inhalt
- **Phase 1**: App-Auswahl & Konfiguration durch Nutzer
- **Phase 2**: Deployment-Erstellung im Backend
- **Phase 3**: Asynchrone Worker-Verarbeitung (Git Clone, Terraform)
- **Phase 4**: Status-Updates und Deployment-Abschluss
- **Phase 5**: Nutzer-Benachrichtigung mit Zugriffsdaten

### Wichtige Erkenntnisse
- Nutzer-Interaktion erfolgt über Frontend (Vue)
- Backend erstellt Deployment-Record und triggert Celery-Task
- Worker klont GitHub-Repo und führt Terraform aus
- Status-Updates erfolgen in Echtzeit über Polling/WebSocket
- Fehlerbehandlung mit Rollback-Mechanismus

### Ablauf-Schritte
1. Nutzer wählt App aus Template-Store
2. Nutzer konfiguriert Deployment-Parameter (Terraform-Variablen)
3. Backend erstellt Deployment (Status: PENDING)
4. Celery-Task wird in RabbitMQ-Queue geschrieben
5. Worker konsumiert Task und startet Verarbeitung
6. Git Clone von Template-Repository
7. Terraform Init/Plan/Apply auf OpenStack
8. Status-Update (RUNNING → SUCCESS/FAILED)
9. Nutzer erhält Zugriffsdaten (IP, URL, SSH)

---

## 4️⃣ Klassendiagramm

**Datei**: [`class-diagram.puml`](./class-diagram.puml)

### Zweck
Dokumentiert das **Datenbank-Schema** und die Beziehungen zwischen Entitäten.

### Kern-Entitäten

#### User
- Rollen: STUDENT, TEACHER, ADMIN
- Authentifizierung: JWT + bcrypt
- Beziehungen: Course, Apps, Deployments, Teams

#### Course
- Lehrveranstaltungen
- Beziehungen: Users, UserGroups

#### App
- Templates (referenziert GitHub-Repos)
- Terraform-basierte Deployment-Definitionen
- Beziehungen: User (Creator), Deployments

#### Deployment
- Konkrete Instanzen von App-Deployments
- Status: PENDING, RUNNING, SUCCESS, FAILED
- Enthält: Terraform-Variablen, Commit-Info, Outputs
- Beziehungen: User, App, UserGroup

#### UserGroup
- Gruppiert Nutzer für gemeinsame Deployments
- Nützlich für Team-Projekte in Kursen
- Beziehungen: Deployment, Users, Courses, Teams

#### Team
- Teams innerhalb von UserGroups
- Beziehungen: UserGroup, Users

### Junction Tables (N:M Beziehungen)
- **UserToUserGroup**: User ↔ UserGroup
- **CourseToUserGroup**: Course ↔ UserGroup
- **UserToTeam**: User ↔ Team

### Service-Layer
- **CeleryTask**: Task-Queue-Management
- **GitService**: Repository-Operationen
- **TerraformService**: Infrastructure-as-Code
- **OpenStackClient**: Cloud-Provider-Integration

### Wichtige Erkenntnisse
- Rollen-basiertes Access Control (RBAC)
- Flexible Gruppen- und Team-Strukturen
- Apps sind wiederverwendbare Templates
- Deployments speichern vollständige Historie
- Service-Layer entkoppelt Business-Logic von Persistenz

---

## 🔍 Zusammenfassung

### System-Architektur auf einen Blick

```
┌─────────────────┐
│  Vue Frontend   │ ← Nutzer-Interaktion
└────────┬────────┘
         │ REST API
┌────────▼────────┐
│  FastAPI        │ ← Authentication, Authorization
│  Backend        │ ← CRUD Operations
└────────┬────────┘
         │ Celery Task
┌────────▼────────┐
│  RabbitMQ       │ ← Message Queue
└────────┬────────┘
         │ Consume
┌────────▼────────┐
│  Celery Worker  │ ← Git Clone, Terraform
└────────┬────────┘
         │ Deploy
┌────────▼────────┐
│  OpenStack      │ ← VMs, Networks, Storage
└─────────────────┘
```

### Datenfluss

1. **Nutzer → Frontend**: Auswahl & Konfiguration
2. **Frontend → Backend**: HTTP POST /deployments
3. **Backend → Database**: Deployment-Record erstellen
4. **Backend → RabbitMQ**: Celery-Task publishen
5. **RabbitMQ → Worker**: Task konsumieren
6. **Worker → GitHub**: Template-Repository klonen
7. **Worker → OpenStack**: Terraform apply ausführen
8. **Worker → Database**: Status aktualisieren
9. **Backend → Frontend**: Deployment-Informationen
10. **Frontend → Nutzer**: Zugriffsdaten anzeigen

### Technologie-Mapping

| Ebene | Technologie | Zweck |
|-------|-------------|-------|
| **Frontend** | Vue 3 + TypeScript | SPA, Nutzer-Interface |
| **API** | FastAPI | REST API, Business Logic |
| **ORM** | SQLAlchemy | Datenbank-Abstraktion |
| **Database** | PostgreSQL | Persistente Datenspeicherung |
| **Queue** | RabbitMQ | Message Broker |
| **Cache** | Redis | Task Results, Session Cache |
| **Worker** | Celery | Asynchrone Task-Verarbeitung |
| **IaC** | Terraform | Infrastructure-as-Code |
| **Cloud** | OpenStack | Deployment-Ziel |
| **VCS** | GitHub | Template-Repositories |
| **Container** | Docker | Service-Isolation |
| **Orchestration** | Docker Compose | Multi-Container-Management |

---

## 📚 Weiterführende Dokumentation

- **[Architektur-Übersicht](./architecture-overview.md)**: Detaillierte Beschreibung des Gesamtsystems
- **[README](./README.md)**: Anleitung zur Visualisierung der Diagramme
- **[Backend README](../../backend/README.md)**: Backend-spezifische Dokumentation
- **[Frontend README](../../frontend/README.md)**: Frontend-spezifische Dokumentation
- **[Deployment README](../../deployment/README.md)**: Deployment-Anleitung

---

## 🎓 Verwendung für Prüfungsleistungen

### Empfohlene Struktur für Präsentation

1. **Einführung** (5 Min)
   - Projektziel: App-Store für Lehrveranstaltungen
   - Use Cases: Entwicklungsumgebungen, Jupyter, K8s-Cluster
   - Zielgruppe: Studierende, Dozierende, Admins

2. **System-Übersicht** (5 Min)
   - **Komponentendiagramm** zeigen
   - Erklären: Frontend, Backend, Worker, External Systems
   - Hervorheben: Message-Queue-Architektur

3. **Infrastructure** (5 Min)
   - **Deployment-Diagramm** zeigen
   - Erklären: Docker-Container, Netzwerke, Volumes
   - Hervorheben: Skalierbarkeit der Worker

4. **Deployment-Flow** (10 Min)
   - **Sequenzdiagramm** durchgehen
   - Schritt-für-Schritt: Von Nutzer-Auswahl bis OpenStack-Deployment
   - Erklären: Asynchrone Verarbeitung, Status-Updates

5. **Datenmodell** (5 Min)
   - **Klassendiagramm** zeigen
   - Erklären: User, Course, App, Deployment
   - Hervorheben: Rollen-Konzept, Gruppen-Strukturen

6. **Q&A** (5 Min)

### Tipps für die Präsentation

✅ **DO:**
- Diagramme in hoher Auflösung (SVG) verwenden
- Schritt-für-Schritt durch Sequenzdiagramm gehen
- Konkrete Beispiele nennen (z.B. "K8s-Cluster für 30 Studierende")
- Technologie-Entscheidungen begründen (Warum RabbitMQ? Warum Terraform?)

❌ **DON'T:**
- Zu viel Text auf Folien
- Implementierungs-Details ohne Kontext
- Diagramme ohne Erklärung zeigen
- Annahmen treffen ohne Begründung

---

**Erstellt**: Dezember 2025  
**Projekt**: SIX7 Click'n Deploy  
**Organisation**: six7-click-n-deploy
