# SIX7 Click'n Deploy - Architektur-Dokumentation

> **Projektziel**: App-Store für Lehrveranstaltungen zur Bereitstellung von OpenStack-basierten Übungsumgebungen

## 📋 Übersicht

SIX7 Click'n Deploy ist eine Plattform, die es Dozierenden ermöglicht, vorkonfigurierte Anwendungen und komplette Übungsumgebungen für Studierende bereitzustellen. Die Deployments erfolgen auf OpenStack-Infrastruktur und können per Klick gestartet, gestoppt und aktualisiert werden.

## 🎯 Use Cases

### Typische Lehrveranstaltungs-Szenarien:
- **Entwicklungsumgebungen** für Programmierübungen
- **GitLab/Jenkins** für Kursprojekte
- **Jupyter-Server** für Data Science Übungen
- **Pentesting-Labs** in isolierten Netzwerken
- **Kubernetes-Cluster** für Microservices oder KI-Anwendungen

## 🏗️ Architektur-Prinzipien

### Deployment-Workflow:
1. Dozierende/Admins erstellen Apps (Templates) via GitHub-Repository-Link
2. Studierende/Dozierende wählen App aus und konfigurieren Deployment-Parameter
3. Backend erstellt Celery-Task in RabbitMQ-Queue
4. Worker-Container (dynamisch skaliert) führt OpenStack-Deployment aus
5. Deployment-Status wird in Echtzeit aktualisiert

### Technologie-Stack:
- **Frontend**: Vue 3 + TypeScript + Tailwind CSS
- **Backend**: FastAPI + SQLAlchemy + PostgreSQL
- **Worker**: Celery + Terraform + Git
- **Message Queue**: RabbitMQ (Broker) + Redis (Result Backend)
- **Deployment-Ziel**: OpenStack
- **IaC**: Terraform

## 👥 Rollen-Konzept

### Student (STUDENT)
- Kann Apps deployen
- Kann eigene Deployments verwalten
- Teil von Kursen und Teams

### Dozent (TEACHER)
- Kann Apps erstellen und verwalten
- Kann Kurse anlegen
- Kann Deployments für Kurse verwalten

### Administrator (ADMIN)
- Vollständige Plattform-Verwaltung
- Nutzer- und Kursverwaltung
- System-Monitoring

## 📊 UML-Diagramme

Die folgenden UML-Diagramme beschreiben die Architektur im Detail:

1. **[Komponentendiagramm](./component-diagram.puml)** - System-Übersicht und Komponenten-Interaktion
2. **[Deployment-Diagramm](./deployment-diagram.puml)** - Infrastructure und Container-Architektur
3. **[Sequenzdiagramm](./sequence-diagram.puml)** - Deployment-Flow (typischer Use Case)
4. **[Klassendiagramm](./class-diagram.puml)** - Datenbank-Schema und Domänenmodell

## 🔐 Sicherheits-Konzept

### Authentifizierung:
- JWT-basierte Token-Authentifizierung
- Password-Hashing mit bcrypt

### Autorisierung:
- Rollen-basiertes Access Control (RBAC)
- Nutzer können nur eigene Deployments verwalten
- Dozierende können Kurse und zugehörige Deployments verwalten

### GitHub-Integration:
- Deploy-Keys für private Repository-Zugriff
- Worker klont Templates mit hinterlegtem SSH-Key

## 📦 Datenmodell-Übersicht

### Kern-Entitäten:
- **User**: Nutzer mit Rollen (Student/Teacher/Admin)
- **Course**: Lehrveranstaltung
- **App**: Template/Anwendung (referenziert GitHub-Repo)
- **Deployment**: Konkrete Instanz eines App-Deployments
- **Team**: Gruppen innerhalb von Kursen
- **UserGroup**: Deployment-bezogene Nutzergruppen

### Beziehungen:
- Nutzer gehören zu Kursen
- Apps werden von Nutzern erstellt
- Deployments gehören zu Apps und Nutzern
- UserGroups verbinden Deployments mit mehreren Nutzern
- Teams organisieren Nutzer innerhalb von UserGroups

## 🔄 Deployment-Lifecycle

```
[Nutzer wählt App] 
    ↓
[Konfiguration (Terraform-Variablen)]
    ↓
[Backend erstellt Deployment-Record (PENDING)]
    ↓
[Celery-Task in RabbitMQ-Queue]
    ↓
[Worker startet (dynamisch)]
    ↓
[Git Clone von Template-Repository]
    ↓
[Terraform Init/Plan/Apply auf OpenStack]
    ↓
[Status-Updates (RUNNING → SUCCESS/FAILED)]
    ↓
[Deployment-Informationen zurück an Nutzer]
```

## 🚀 Skalierbarkeit

### Worker-Skalierung:
- Worker können horizontal skaliert werden
- Pro Task in Queue wird ein Worker-Container gestartet
- Automatisches Scale-Down nach Task-Completion

### Queue-Management:
- RabbitMQ verwaltet Task-Verteilung
- Priority-Queues für kritische Deployments möglich
- Dead-Letter-Queues für fehlgeschlagene Tasks

## 📝 Hinweise für Entwickler

### PlantUML-Diagramme rendern:
```bash
# Online: http://www.plantuml.com/plantuml/uml/
# Lokal mit PlantUML JAR:
java -jar plantuml.jar *.puml

# VS Code Extension:
# PlantUML (jebbs.plantuml)
```

### Diagramme aktualisieren:
Bei Änderungen am Datenmodell oder der Architektur müssen die entsprechenden UML-Diagramme aktualisiert werden.

---

**Letzte Aktualisierung**: Dezember 2025  
**Version**: 1.0.0
