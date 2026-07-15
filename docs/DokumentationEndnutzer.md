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

## App-Freigaben verwalten

Als Admin verfügen Sie über erweiterte Rechte, um neue App-Versionen zu prüfen, freizugeben oder zu sperren. 

### Die Freigabe-Verwaltung im Überblick

Wenn Sie die Freigabe-Verwaltung öffnen, sehen Sie standardmäßig **nur Apps, für die aktuell eine Freigabe beantragt wurde**. 

* **Neue Versionen prüfen:** Liegen ausstehende Einreichungen vor, werden diese hier aufgelistet. Sie können jede Version entweder **genehmigen** oder **ablehnen**.
* **Filter ausschalten (Alle Apps anzeigen):** Wenn Sie oben rechts den Filter deaktivieren, sehen Sie alle öffentlichen und privaten Apps. 
* **Hinweis zu privaten Apps:** Hier werden keine ausstehenden Freigaben angezeigt. Erst wenn eine App veröffentlicht wird, können Versionen zur Freigabe eingereicht werden.
* **Detailansicht:** Klicken Sie auf eine App in der Liste, um alle eingereichten Versionen und deren aktuellen Status einzusehen.

<figure>
  <img src="img/Genehmigen_Ablehnen.png" alt="Freigabe-Verwaltungsseite" width="100%">
  <figcaption>Freigabe-Verwaltung: Übersicht der ausstehenden App-Freigaben</figcaption>
</figure>




<figure>
  <img src="img/Freigabe_Verwaltung_Filter_aus.png" alt="Freigabe-Verwaltungsseite: Alle Apps" width="80%">
  <figcaption>Freigabe-Verwaltung: Deaktivierter Filter</figcaption>
</figure>


---

### App-Versionen freigeben oder ablehnen

Sie haben drei Möglichkeiten, wie Sie mit eingereichten oder bereits freigegebenen Versionen verfahren können:

| Aktion | Ablauf & Voraussetzungen | Auswirkung |
| :--- | :--- | :--- |
| **Genehmigen** | Klicken Sie bei der ausstehenden Version auf **Genehmigen**. | Die Version wird sofort für alle Nutzer freigeschaltet. |
| **Ablehnen** | Klicken Sie auf **Ablehnen**. <br>**Pflicht:** Sie müssen im Freitextfeld einen **Ablehnungsgrund** angeben. | Die Version wird nicht freigeschaltet. Der zugehörige Nutzer sieht Ihre Begründung. |
| **Widerrufen** | Sie können eine bereits genehmigte Version durch Klick auf **Widerrufen** jederzeit nachträglich sperren.<br>**Pflicht:** Sie müssen einen **Widerrufsgrund** (z. B. eine Sicherheitslücke) angeben. | Die Version wird für alle Nutzer sofort gesperrt. |


<figure>
  <img src="img/Freigabe_widerrufen.png" alt="Freigabe-Verwaltungsseite: Freigabe widerrufen" width="80%">
  <figcaption>Freigabe-Verwaltung: Freigabe widerrufen</figcaption>
</figure>




> **Gut zu wissen:** Sowohl abgelehnte als auch widerrufene Versionen sind nicht endgültig verloren. Sie können diese zu einem späteren Zeitpunkt jederzeit manuell **erneut genehmigen**, sobald die Probleme behoben wurden.

---