# Endnutzer Dokumentation

### Autoren
1. **Tom Weber** *(Dokumentationsteil: Student)*
2. **Paulina Clauss** *(Dokumentationsteil: Lehrer)*
3. **Monika Piano** *(Dokumentationsteil: Admin)*

### Reviewer (Prüfer)
1. **[Name Reviewer 1]**
2. **[Name Reviewer 2]** 

---

## 1. Studenten

**Beschreibung:** Das Frontend wird als Single Page Application (SPA) mit Vue.js 3 realisiert. Für das State Management (die Verwaltung des globalen Anwendungszustands) kommt Pinia zum Einsatz, während Axios die Kommunikation mit dem Backend übernimmt. Das Styling erfolgt utility-first mittels Tailwind CSS.

**Begründung:** Die Architektur setzt auf eine strikte Trennung von Benutzeroberfläche (UI), Geschäftslogik und Routing. Der Einsatz von TypeScript sorgt hierbei für Typsicherheit, wodurch viele Fehler bereits zur Entwicklungszeit und nicht erst zur Laufzeit abgefangen werden. Tailwind CSS ermöglicht eine schnelle UI-Entwicklung ohne die Wartungslast komplexer, selbstgeschriebener CSS-Dateien. Vue.js wurde gewählt, da es als leichtgewichtig und performant gilt und sich perfekt für Frontends dieser Skalierung eignet. Im Vergleich zu oft schwergewichtigeren Enterprise-Lösungen (wie Angular) bietet es eine geringere Einstiegshürde bei gleicher Leistungsfähigkeit für diesen Anwendungsfall.

---

## 2. Lehrer

**Beschreibung:** Das Backend fungiert als asynchrone REST-API, basierend auf FastAPI und Python 3.11+. Als Webserver dient Uvicorn, ein High-Performance ASGI-Server. Die Datenhaltung erfolgt in einer PostgreSQL-Datenbank, die über SQLAlchemy 2.0 (ORM) angesprochen wird.

**Begründung:** Der Fokus liegt auf Performance und modernen Standards. Pydantic validiert alle Dateneingänge rigoros und generiert automatisch die OpenAPI-Dokumentation (Swagger UI), was die Integration für Frontend-Entwickler erleichtert. PostgreSQL wurde aufgrund seiner Stabilität (ACID-Compliance) und der Unterstützung für JSONB-Datentypen gewählt, was Flexibilität bei semi-strukturierten Daten erlaubt. Die Entscheidung fiel bewusst gegen klassische Frameworks wie Django oder Flask. FastAPI ist durch die native Unterstützung von asynchroner Programmierung (Async/Await) und die Nutzung von Starlette deutlich performanter. Zudem bietet SQLAlchemy 2.0 im Gegensatz zu simpleren ORMs die Flexibilität, nahtlos zwischen objektorientiertem Zugriff und Raw SQL zu wechseln.

---

## 3. Admin

**Beschreibung:** Um langlaufende Prozesse zu handhaben, wird eine Task-Queue-Architektur eingesetzt. Celery übernimmt die Verwaltung der Hintergrundaufgaben und setzt dabei auf zwei spezialisierte Infrastruktur-Komponenten: **RabbitMQ** als Message Broker (Aufgaben werden vom Backend an den Worker zugestellt) und **Redis** als Result Backend (Task-Ergebnisse + Live-Status werden für Abfragen aus dem Frontend zwischengespeichert). Zusätzlich dient Redis als Pub/Sub-Layer für die Server-Sent-Events, über die das Frontend Deployment-Logs in Echtzeit empfängt.

**Begründung:** Durch Celery wird die Ausführung entkoppelt: Die API nimmt den Auftrag im Millisekunden-Bereich entgegen, während der Worker ihn im Hintergrund abarbeitet. Dies hält die UI reaktionsfähig. RabbitMQ und Redis übernehmen dabei bewusst unterschiedliche Rollen, weil ihre Stärken komplementär sind: RabbitMQ ist auf zuverlässige Task-Zustellung ausgelegt (persistente Queues, Acknowledgements, Retry-Mechanismen) und stellt sicher, dass Deployment-Tasks auch bei Worker-Ausfällen nicht verloren gehen — diese Garantien wären bei einem reinen Redis-Broker schwächer. Redis hingegen ist als schneller In-Memory-Speicher genau richtig für flüchtige Daten: Task-Ergebnisse, die das Frontend kurz nach Abschluss abfragt, sowie das Pub/Sub-Forwarding der Live-Log-Events an die SSE-Streams. Die Kombination ist gegenüber einer reinen RabbitMQ- oder reinen Redis-Lösung robuster und gleichzeitig performanter.

---