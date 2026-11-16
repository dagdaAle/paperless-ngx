Stato: bozza generata da agente · Aggiornato: 2026-11-16 · Fonte: dagdaAle/paperless-ngx@2ce8ee998 (dev) + note vault server/Intecha-Docs-2026-10-27, server/Paperless-2026-10-13

# Architettura

L'architettura è quella di Paperless-ngx: Documentazione ufficiale upstream: https://docs.paperless-ngx.com (sezione *Architecture* / *Setup*). Qui solo il contesto Intecha.

## C4 L1 — contesto

```
  [Ale, browser] --HTTPS--> [Cloudflare Tunnel] --> [Intecha Docs]
                                                      |   |   |
                     DeepSeek API (cloud) <-- chat ---+   |   +--> NAS (backup restic, notte)
                     Ollama bge-m3 (GPU server) <-- embedding
                     Caselle IMAP (Gmail, Aruba) <-- posta  [DA FARE]
```

## C4 L2 — container sul server (servizio Coolify)

```
 +------------------- servizio Coolify sq6zcxrys7rues1umt3pnnl5 -------------------+
 |  paperless-<uuid>                                  redis-<uuid>                 |
 |  immagine intecha/paperless-ngx:latest  <--------> broker Celery                |
 |  - Django web + frontend Angular rebrandizzato                                  |
 |  - worker Celery (PAPERLESS_TASK_WORKERS=3)                                     |
 |  - consumer della cartella consume/                                             |
 |  - poller posta (process_mail_accounts ogni minuto)                             |
 |  volumi: paperless-data (SQLite, indici), paperless-media (PDF)                 |
 |  bind:   consume/, export/                                                      |
 +---------------------------------------------------------------------------------+
```

## Flussi propri del fork

1. **Ingresso documenti**: file copiati in `consume/` (dal Mac via ssh) → OCR → indicizzazione → workflow 1 (tag `da classificare`) → workflow 2 (suggerimenti AI su tag, corrispondente, tipo, storage path).
2. **AI**: le richieste di suggerimento vanno a DeepSeek (cloud, compatibile OpenAI). Gli embedding per la ricerca semantica si calcolano in casa con Ollama `bge-m3`. L'indice vettoriale sta in `data/llm_index`.
3. **Branding**: il frontend Angular carica `intecha.scss` dopo `theme.scss`; le pagine Django (login, reset password, MFA) usano `base.css` e i template modificati.
4. **Sync upstream** (PR #4): GitHub Action giornaliera → branch `sync/<tag>` → PR verso `dev` → build dell'immagine sul server a mano.
