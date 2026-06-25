# Technologiestack

## 1. Frontend: Vue.js

**Beschreibung:** Das Frontend wird als Single Page Application (SPA) mit Vue.js 3 realisiert. Für das State Management (die Verwaltung des globalen Anwendungszustands) kommt Pinia zum Einsatz, während Axios die Kommunikation mit dem Backend übernimmt. Das Styling erfolgt utility-first mittels Tailwind CSS.

**Begründung:** Die Architektur setzt auf eine strikte Trennung von Benutzeroberfläche (UI), Geschäftslogik und Routing. Der Einsatz von TypeScript sorgt hierbei für Typsicherheit, wodurch viele Fehler bereits zur Entwicklungszeit und nicht erst zur Laufzeit abgefangen werden. Tailwind CSS ermöglicht eine schnelle UI-Entwicklung ohne die Wartungslast komplexer, selbstgeschriebener CSS-Dateien. Vue.js wurde gewählt, da es als leichtgewichtig und performant gilt und sich perfekt für Frontends dieser Skalierung eignet. Im Vergleich zu oft schwergewichtigeren Enterprise-Lösungen (wie Angular) bietet es eine geringere Einstiegshürde bei gleicher Leistungsfähigkeit für diesen Anwendungsfall.

---

## 2. Backend: FastAPI (Python)

**Beschreibung:** Das Backend fungiert als asynchrone REST-API, basierend auf FastAPI und Python 3.11+. Als Webserver dient Uvicorn, ein High-Performance ASGI-Server. Die Datenhaltung erfolgt in einer PostgreSQL-Datenbank, die über SQLAlchemy 2.0 (ORM) angesprochen wird.

**Begründung:** Der Fokus liegt auf Performance und modernen Standards. Pydantic validiert alle Dateneingänge rigoros und generiert automatisch die OpenAPI-Dokumentation (Swagger UI), was die Integration für Frontend-Entwickler erleichtert. PostgreSQL wurde aufgrund seiner Stabilität (ACID-Compliance) und der Unterstützung für JSONB-Datentypen gewählt, was Flexibilität bei semi-strukturierten Daten erlaubt. Die Entscheidung fiel bewusst gegen klassische Frameworks wie Django oder Flask. FastAPI ist durch die native Unterstützung von asynchroner Programmierung (Async/Await) und die Nutzung von Starlette deutlich performanter. Zudem bietet SQLAlchemy 2.0 im Gegensatz zu simpleren ORMs die Flexibilität, nahtlos zwischen objektorientiertem Zugriff und Raw SQL zu wechseln.

---

## 3. Worker: Asynchrone Verarbeitung — Celery & RabbitMQ

**Beschreibung:** Um langlaufende Prozesse zu handhaben, wird eine Task-Queue-Architektur eingesetzt. Celery übernimmt hierbei die Verwaltung der Hintergrundaufgaben, während RabbitMQ als Message Broker fungiert.

**Begründung:** Durch Celery wird die Ausführung entkoppelt: Die API nimmt den Auftrag lediglich entgegen (im Millisekunden-Bereich), während der Worker ihn im Hintergrund abarbeitet. Dies hält die UI reaktionsfähig (responsive). Als Message Broker wird RabbitMQ anstelle von Redis eingesetzt, da RabbitMQ speziell für zuverlässige Task-Verarbeitung ausgelegt ist. Durch persistente Queues, Acknowledgements und Retry-Mechanismen stellt RabbitMQ sicher, dass langlaufende Deployment-Tasks auch bei Worker-Ausfällen nicht verloren gehen. Redis ist primär ein In-Memory-Datenspeicher und Cache und bietet für komplexe, kritische Hintergrundprozesse geringere Zuverlässigkeitsgarantien. Daher wurde RabbitMQ als robustere Lösung für produktive Task-Queue-Szenarien gewählt.

> **ToDo:** Redis mit aufnehmen

---

## 4. Deployment & Containerisierung: Docker

**Beschreibung:** Die gesamte Anwendung wird mittels Docker containerisiert. Docker Compose dient zur Orchestrierung der Container. Dazu benötigt unser System folgende Docker Container:

| Container      | Aufgaben |
|----------------|----------|
| **PostgreSQL** | Zentrale Datenhaltung; stellt konsistente und dauerhafte Speicherung sicher |
| **RabbitMQ**   | Entkoppelt zeitintensive Deployment-Prozesse vom Backend; erhöht Skalierbarkeit und Ausfallsicherheit |
| **Backend**    | Verarbeitet Benutzeranfragen vom Frontend; authentifiziert und autorisiert Benutzer (Keycloak); persistiert Daten in PostgreSQL; erstellt asynchrone Deployment-Tasks für RabbitMQ |
| **Worker**     | Konsumiert Deployment-Tasks aus RabbitMQ; klont App-Repositories aus GitHub; führt Packer- und Terraform-Operationen aus; provisioniert Infrastruktur auf OpenStack |
| **Frontend**   | Ermöglicht Interaktion für Studierende, Dozenten, Powerdozenten und Administratoren; stellt App-Store- und Deployment-Funktionalitäten bereit; kommuniziert ausschließlich über die Backend API |
| **Keycloak**   | Verwaltung von Benutzern, Rollen und Rechten; Integration in die bestehende DHBW-IAM-Struktur |

Die Images werden über eine CI-Pipeline erstellt (Multi-Stage Builds) und in der GitHub Container Registry (GHCR) abgelegt.

**Begründung:** Docker garantiert Portabilität: Die Umgebung ist in Development, Staging und Production absolut identisch, was das Risiko von "Works on my machine"-Fehlern eliminiert. Multi-Stage Builds sorgen dafür, dass Build-Tools nicht im finalen Image landen, was die Images klein und sicher hält. Es wurde sich bewusst für Docker Compose und gegen Kubernetes (K8s) entschieden. Für Single-Host-Deployments ist K8s unnötig komplex ("Overhead"). Docker Compose bietet hier eine schlanke, effiziente Lösung, die leichter zu warten ist. Kubernetes bleibt eine Option für die Zukunft, falls dynamische Skalierung über mehrere Nodes hinweg notwendig wird.

> **ToDo:** Überarbeiten und Deploymentprozess überarbeiten

---

## 5. Infrastructure as Code (IaC): Terraform & Packer

**Beschreibung:** Die Infrastruktur wird nicht manuell, sondern durch Code definiert. Terraform provisioniert die Ressourcen (VMs, Netzwerke, Security Groups) auf OpenStack. Packer wird genutzt, um Abbilder für die VMs zu bauen.

**Begründung:** Dieser Ansatz ermöglicht Disaster Recovery "per Knopfdruck" und vollständige Versionierung der Infrastruktur in Git. Ein entscheidender Vorteil ist der "Golden Image"-Workflow mit Packer. Anstatt eine leere VM zu starten und dann minutenlang Software zu installieren (wie bei reinem Ansible/Shell-Scripting oft üblich), erstellt Packer ein fertiges Image (Snapshot).
