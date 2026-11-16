Stato: bozza generata da agente · Aggiornato: 2026-11-16 · Fonte: dagdaAle/paperless-ngx@2ce8ee998 (dev) + note vault server/Intecha-Docs-2026-10-27, server/Paperless-2026-10-13

# Modello dati

Il fork **non aggiunge modelli né migrazioni**. Lo schema è quello di Paperless-ngx 3.3.1: Documentazione ufficiale upstream: https://docs.paperless-ngx.com

Le personalizzazioni Intecha stanno come **dati** nel DB, non nel codice:

| Cosa | Dove | Note |
|---|---|---|
| Viste in dashboard e sidebar | `UiSettings.settings["saved_views"]` dell'utente `ale` | in 3.x non sono più campi del modello SavedView |
| Configurazione AI | `ApplicationConfiguration` (singleton) | modificabile a caldo, senza redeploy |
| Workflow 1 e 2 | `Workflow` + trigger + azioni | 2 = AI prudente |
| Tassonomia | Tag, DocumentType, StoragePath, CustomField | tag per area (`01_Universita`…), 11 tag materia, coda `da classificare` |
| Posta | `MailAccount`, `MailRule` | 0 record al 27/10 |

## Dove stanno i dati

| Dato | Posizione |
|---|---|
| DB SQLite, indice full-text, indice vettoriale `llm_index` | volume `paperless-data` |
| PDF originali e archiviati | volume `paperless-media` |
| Import | `/data/coolify/services/<servizio>/consume/` |

`Document.checksum` è **SHA-256** (non md5): usarlo per deduplicare prima di importare.

## Struttura file

- `PAPERLESS_FILENAME_FORMAT={{ created_year }}-{{ created_month }}/{{ title }}` per i documenti senza storage path.
- Lo storage path è un **template di percorso**, non il nome di una cartella: `01_Universita/<Materia>/{{ title }}`.
- Dopo ogni modifica a percorsi o formato: `manage.py document_renamer` (idempotente).
