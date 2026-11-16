Stato: bozza generata da agente · Aggiornato: 2026-11-16 · Fonte: dagdaAle/paperless-ngx@2ce8ee998 (dev) + note vault server/Intecha-Docs-2026-10-27, server/Paperless-2026-10-13

# Configurazione

Solo i valori che differiscono dal default o che contano per il fork. Elenco completo: Documentazione ufficiale upstream: https://docs.paperless-ngx.com/configuration/
Nessun segreto qui: le chiavi stanno nel DB o in Vaultwarden.

## Variabili d'ambiente

| Variabile | Valore / uso |
|---|---|
| `PAPERLESS_TIME_ZONE` | `Europe/Rome` |
| `PAPERLESS_OCR_LANGUAGE` | `ita+eng` |
| `PAPERLESS_TASK_WORKERS` | `3` (default 1, troppo lento con centinaia di documenti in coda) |
| `PAPERLESS_FILENAME_FORMAT` | `{{ created_year }}-{{ created_month }}/{{ title }}` (doppie graffe in 3.x) |
| `PAPERLESS_AI_LLM_EXTRA_PARAMS` | `{"reasoning_effort": "none"}` — spegne il thinking di `deepseek-flash`. **Si legge solo dall'env** |
| `PAPERLESS_AI_LLM_ALLOW_INTERNAL_ENDPOINTS` | default `true`: serve per Ollama su IP privato, non va toccata |
| `PAPERLESS_EMAIL_*` | non impostate |

## ApplicationConfiguration (DB, modificabile a caldo)

| Campo | Valore |
|---|---|
| `ai_enabled` | `true` |
| Backend LLM | `openai-like` → DeepSeek, endpoint `https://api.deepseek.com/v1` |
| Modello | `deepseek-flash` |
| API key | Vaultwarden → voce DeepSeek usata da JobSpy (`jobspy-api`) |
| Lingua output | `it` |
| Embedding | `ollama`, modello `bge-m3`, Ollama sulla GPU del server |
| Chunk size | 1024 |

Si cambia da UI *Settings → Application configuration* o da shell.

## Servizi esterni

| Servizio | A cosa serve |
|---|---|
| DeepSeek | suggerimenti AI (titolo, tag, tipo, corrispondente) |
| Ollama (server di casa) | embedding `bge-m3` per la ricerca semantica |
| NAS | destinazione backup restic |
| Gmail / Aruba IMAP | posta in entrata — DA FARE |

## Dipendenze aggiunte dal fork

- `@fontsource/cairo` (frontend, `src-ui/package.json`).
- Le pagine Django di login caricano il font Cairo da Google Fonts.
