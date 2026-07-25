# QS-Maßnahmen — Beleg der Durchführung (6P)

> **Projekt:** OpenStack AppStore (six7-click-n-deploy)
> **Organisation:** [`github.com/six7-click-n-deploy`](https://github.com/six7-click-n-deploy)
> **Stand:** 25.07.2026
> **Zweck:** Nachweis der durchgeführten Qualitätssicherungs­maßnahmen im Studienprojekt.

Dieses Dokument belegt die im Projekt durchgeführten QS-Maßnahmen. Alle Angaben sind reproduzierbar über die GitHub-Organisation, die CI/CD-Pipelines und die Pull-Request-Historie.

---

## 1. Überblick der QS-Maßnahmen

Die Qualitätssicherung des Projekts stützt sich auf vier ineinandergreifende Säulen:

| Säule | Maßnahme | Wo belegt |
|---|---|---|
| **1. Code-Review** | Verpflichtende Pull Requests, mind. 1 Review-Approval, Branch-Schutz | GitHub PRs + Ruleset |
| **2. Automatisierte Tests** | Unit- + Integrationstests, Coverage-Reporting | CI-Pipeline (`test`-Jobs) |
| **3. Statische Analyse** | Linting, Type-Checking, Formatierungs-Checks | CI-Pipeline (`lint`/`typecheck`) |
| **4. Security** | Dependency-Audit, Container-Scan, Secret-Scan, IaC-Scan | CI-Pipeline (`security`/`image-scan`), Gitleaks |

Der Code ist auf vier eigenständige Repositories aufgeteilt, die jeweils eine vollständige eigene Pipeline besitzen:

| Repository | Rolle | Sprache/Stack |
|---|---|---|
| [`frontend`](https://github.com/six7-click-n-deploy/frontend) | Single-Page-App (AppStore-UI) | Vue 3, TypeScript, Vite |
| [`backend`](https://github.com/six7-click-n-deploy/backend) | REST-API | FastAPI, Python 3.11 |
| [`worker`](https://github.com/six7-click-n-deploy/worker) | Async-Deployment-Worker | Celery, Python 3.11 |
| [`deployment`](https://github.com/six7-click-n-deploy/deployment) | Infrastruktur & CD | Terraform, Ansible |

---

## 2. Code-Review (Pull-Request-Prozess)

### 2.1 Prozess

Änderungen gelangen **ausschließlich über Pull Requests** nach `main` — direktes Pushen ist unterbunden. Jeder PR durchläuft:

1. **Automatische Checks** (CI muss grün sein — siehe Abschnitt 3–5)
2. **Peer-Review** durch ein anderes Teammitglied (mind. 1 Approval)
3. **Merge** erst nach Freigabe

Zur Vereinheitlichung existiert im `frontend`-Repo ein [Pull-Request-Template](https://github.com/six7-click-n-deploy/frontend/blob/main/.github/pull_request_template.md), das Reviewer durch eine strukturierte Checkliste führt (Art der Änderung, Testnachweis, Self-Review, i18n, Typisierung).

### 2.2 Branch-Schutz (technisch erzwungen)

Im `frontend`-Repository ist ein aktives **Ruleset „protect main"** hinterlegt:

- ✅ `pull_request` erforderlich, **mind. 1 genehmigendes Review** (`required_approving_review_count: 1`)
- ✅ **Löschschutz** für `main` (`deletion`)
- ✅ **Kein Force-Push** (`non_fast_forward`)

In `backend`, `worker` und `deployment` wird der PR-Workflow über die CI-Pflichtchecks und die Team-Vereinbarung durchgesetzt.

### 2.3 Belegbare Kennzahlen (Stand 25.07.2026)

| Repository | PRs gesamt | davon gemerged | PRs mit Review | Review-Vorgänge |
|---|---:|---:|---:|---:|
| `frontend` | 70 | 68 | 49 | 50 |
| `backend` | 35 | 32 | 9 | 9 |
| `worker` | 22 | 21 | 5 | 5 |
| `deployment` | 5 | 5 | 2 | 2 |
| **Summe** | **132** | **126** | **65** | **66** |

> Reproduzierbar über:
> `gh pr list -R six7-click-n-deploy/<repo> --state all --json number,state,reviews`

---

## 3. Automatisierte Tests

Jedes Code-Repository führt bei **jedem PR und jedem Push auf `main`** eine Testsuite aus. Ein PR ist erst mergefähig, wenn die Tests bestehen.

### 3.1 Backend (`backend`) — FastAPI / pytest

- **`test-unit`**: Unit-Tests (`pytest -m unit`) gegen einen PostgreSQL-16-Service-Container
- **`test-integration`**: End-to-End-Tests der FastAPI-App via `TestClient` (`pytest -m integration`)
- **`coverage`**: Kombiniert Unit- + Integrations-Coverage, rendert HTML-Report, erzeugt ein Shields.io-Coverage-Badge und veröffentlicht den Report auf GitHub Pages
- **Testumfang:** 45 Testdateien

### 3.2 Worker (`worker`) — Celery / pytest

- **`test-unit`** + **`test-integration`** (getrennte Marker), ebenfalls mit Coverage-Aggregation und Pages-Publishing
- **Testumfang:** 13 Testdateien

### 3.3 Frontend (`frontend`) — Vitest

- **`test`**: `npm run test:coverage` (Vitest) inkl. Coverage-Badge-Erzeugung
- **`typecheck`**: `vue-tsc --noEmit` als eigener Blocking-Job
- **Testumfang:** 13 Spec-Dateien

### 3.4 Coverage-Transparenz

Alle drei Code-Repos publizieren ihren Coverage-Report automatisch auf **GitHub Pages** und erzeugen ein farbcodiertes **Coverage-Badge** (grün ≥ 80 %, absteigend bis rot). Die Coverage-Daten werden zusätzlich als Workflow-Artefakt (14 Tage) abgelegt.

---

## 4. Statische Analyse (Linting, Formatierung, Typen)

Vorgeschaltete Qualitäts-Gates fangen Stil- und Typfehler ab, bevor Tests laufen:

| Repository | Werkzeug(e) | Prüfung |
|---|---|---|
| `backend` | **Ruff** | Linting (`ruff check .`) |
| `worker` | **Ruff, Black, isort, MyPy** | Linting, Formatierung, Import-Ordnung, statische Typen |
| `frontend` | **vue-tsc** | TypeScript-Typprüfung (`--noEmit`) |

Diese Jobs sind **blockierend** — schlägt das Linting fehl, kann der Build-Job nicht starten.

---

## 5. Security-Maßnahmen

Security ist als fester Pipeline-Bestandteil integriert und **blockiert Merges bei HIGH-/CRITICAL-Findings**.

### 5.1 Dependency-Scanning

| Repository | Werkzeug | Verhalten |
|---|---|---|
| `backend` / `worker` | **pip-audit** (`--strict`) | Prüft Produktions­abhängigkeiten, bricht bei Vulnerabilities ab |
| `frontend` | **npm audit** (`--audit-level=high --omit=dev`) | Prüft Produktions­abhängigkeiten |

Dokumentierte, begründete Ausnahmen werden nachvollziehbar hinterlegt — z. B. wird im Backend `PYSEC-2026-1325` (ecdsa Minerva-Timing-Seitenkanal) mit ausführlicher Begründung ignoriert, da der betroffene Code-Pfad (Signieren) nicht genutzt wird (nur Token-Verifikation).

### 5.2 Filesystem- & Misconfig-Scan (Trivy)

Der **`security`-Job** führt in jedem Repo einen `trivy fs`-Scan aus (`vuln,secret,misconfig`):

- **Blocking Gate:** Job schlägt bei HIGH/CRITICAL fehl (`exit-code: 1`)
- **SARIF-Upload:** Alle Findings (auch ignorierte) landen im **GitHub Security Tab** — dauerhaft sichtbar

### 5.3 Container-Image-Scan (Trivy)

Der **`image-scan`-Job** baut das produktive Docker-Image und scannt es mit Trivy auf CVEs (HIGH/CRITICAL, blockierend + SARIF-Upload). Erst danach darf das Image via **`push`-Job** in die GitHub Container Registry (GHCR) gelangen.

### 5.4 Secret-Scanning (Gitleaks)

Das `deployment`-Repo betreibt einen dedizierten [**Secret-Scan-Workflow**](https://github.com/six7-click-n-deploy/deployment/blob/main/.github/workflows/secret-scan.yml) mit **Gitleaks** als Pre-Merge-Gate:

- Scannt die **vollständige Git-Historie** (`fetch-depth: 0`) auf versehentlich committete Tokens, SSH-Keys und OpenStack-Credentials
- Zwei-Pass-Muster: blockierendes Gate + SARIF-Emit in den Security Tab

### 5.5 Infrastructure-as-Code-Scan

Die Staging-Deploy-Pipeline (`deployment/staging.yml`) prüft die Terraform-Konfiguration:

- `terraform fmt -check` (Formatierung)
- `terraform validate` (Syntax/Konsistenz)
- **`trivy config`** auf IaC-Misconfigurations (HIGH/CRITICAL, SARIF in Security Tab)
- Sensible Artefakte (SSH-Keys, Inventory) werden nach dem Lauf **garantiert gelöscht** (`if: always()`)

---

## 6. CI/CD-Pipeline (Gesamtablauf)

Jedes Code-Repo nutzt eine identisch strukturierte, mehrstufige Pipeline. Jobs mit Abhängigkeiten (`needs:`) garantieren, dass **kein Artefakt veröffentlicht wird, das nicht alle Qualitäts-Gates bestanden hat**:

```
lint / typecheck ─┐
test-unit ────────┤
test-integration ─┼──> build ──> image-scan ──> push (GHCR) ──> trigger-staging
security ─────────┘        │                         (nur main)      (nur main)
coverage (Pages) ──────────┘
```

- **Trigger:** jeder PR gegen `main` + jeder Push auf `main` + Version-Tags (`v*.*.*`)
- **Gate-Prinzip:** `build` benötigt `lint`, `test-integration` **und** `security` → alle müssen grün sein
- **Deploy nur von `main`:** `push` und `trigger-staging` laufen ausschließlich bei `push`-Events auf `main`
- **Continuous Deployment:** Ein erfolgreicher Push auf `main` triggert automatisch das Staging-Deployment im `deployment`-Repo

### 6.1 Pipeline-Läufe (letzte 100 Runs je Repo, Stand 25.07.2026)

| Repository | Erfolgreich | Fehlgeschlagen |
|---|---:|---:|
| `frontend` | 72 | 26 |
| `backend` | 61 | 39 |
| `worker` | 34 | 44 |
| `deployment` | 86 | 14 |

> Fehlgeschlagene Läufe belegen, dass die Gates **tatsächlich greifen** — fehlerhafte Änderungen wurden von der Pipeline abgefangen, bevor sie gemerged/deployed werden konnten.

---

## 7. Zusammenfassung — Nachweis der Durchführung

Die QS-Maßnahmen sind **nicht nur konzeptioniert, sondern nachweislich im Projektalltag gelebt worden**:

- ✅ **126 gemergte Pull Requests** über vier Repositories, davon 65 mit dokumentiertem Peer-Review
- ✅ **Technisch erzwungener Branch-Schutz** (Ruleset mit Review-Pflicht) auf `main`
- ✅ **71 Testdateien** (Unit + Integration) mit automatisiertem Coverage-Reporting auf GitHub Pages
- ✅ **Statische Analyse** (Ruff, Black, isort, MyPy, vue-tsc) als blockierende Gates
- ✅ **Vierfaches Security-Scanning**: Dependencies (pip-audit/npm audit), Filesystem + Images (Trivy), Secrets (Gitleaks), IaC (Trivy config) — mit zentraler Sichtbarkeit im GitHub Security Tab
- ✅ **Durchgängige CI/CD-Pipeline** mit strikten Job-Abhängigkeiten und automatischem Staging-Deployment

Alle Belege sind über die GitHub-Organisation, die Workflow-Dateien (`.github/workflows/`) und die `gh`-CLI reproduzierbar.
