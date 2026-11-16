Stato: bozza generata da agente · Aggiornato: 2026-11-16 · Fonte: dagdaAle/paperless-ngx@2ce8ee998 (dev) + note vault server/Intecha-Docs-2026-10-27, server/Paperless-2026-10-13

# Intecha Docs — panoramica

Archivio documenti con OCR e ricerca full-text. È un **fork rebrandizzato di Paperless-ngx 3.3.1**: stesso motore, branding Intecha sopra.
Questa documentazione copre **solo ciò che il fork aggiunge o cambia**. Per tutto il resto: Documentazione ufficiale upstream: https://docs.paperless-ngx.com

| | |
|---|---|
| Stato | produzione |
| URL | https://paperless.intecha.dev |
| Per chi | uso personale di Ale (utente `ale`, superuser) |
| Repo | https://github.com/dagdaAle/paperless-ngx — branch di default **`dev`** (non `main`) |
| Upstream | https://github.com/paperless-ngx/paperless-ngx |
| Punto di fork | upstream `6e717817c` (12/10/2026) |
| Versione | 3.3.1 |
| Immagine | `intecha/paperless-ngx:latest`, **costruita sul server**, non su un registry |
| Stack | Django + Celery + Redis (backend), Angular (frontend), SQLite, OCR ocrmypdf/tesseract |
| Credenziali | Vaultwarden → raccolta *Server intecha* → voce "Paperless-ngx — paperless.intecha.dev" |

## Cosa cambia rispetto all'upstream

| Area | Dove | Tipo |
|---|---|---|
| Branding (palette Deep Ocean & Cairo, logo, titoli) | codice del fork, PR #1 e #2 | codice |
| Navbar leggibile + cache-busting dei bundle | PR #3, **aperta** | codice |
| Sync automatico con le release upstream | PR #4, **aperta** | CI |
| Dashboard con le viste salvate | `ui_settings` nel DB | configurazione |
| AI: suggerimenti DeepSeek + indice semantico bge-m3 locale | `ApplicationConfiguration` + env | configurazione |
| Workflow AI prudente sui documenti nuovi | Workflow nel DB | configurazione |
| Posta in entrata (IMAP) | non ancora configurata | DA FARE |
| Backup restic sul NAS | cron sul server | operations |

## Pagine

- 01 · Architettura
- 02 · Modello dati
- 03 · API e tool MCP
- 04 · Configurazione
- 05 · Deploy e operations
- 06 · Runbook
- 07 · Decisioni (ADR)
- 08 · Sviluppo locale
