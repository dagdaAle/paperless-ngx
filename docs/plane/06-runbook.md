Stato: bozza generata da agente · Aggiornato: 2026-11-16 · Fonte: dagdaAle/paperless-ngx@2ce8ee998 (dev) + note vault server/Intecha-Docs-2026-10-27, server/Paperless-2026-10-13

# Runbook

Problemi noti del fork e di questa installazione. Problemi generici di Paperless: https://docs.paperless-ngx.com/troubleshooting/

## Deploy e CI

| Sintomo | Causa | Fix |
|---|---|---|
| PR mergiata e verde, ma in produzione non cambia niente | **PR verde ≠ produzione**: l'immagine si costruisce sul server | build dell'immagine + `docker compose up -d --no-deps paperless` (pag. 05) |
| Stato CI confuso, `gh run watch` guarda il run sbagliato | i run vanno legati al commit | verificare **per SHA**: `gh run list --json name,status,conclusion,headSha` e filtrare sullo SHA del commit |
| CSS e JS non caricati, 404 su `styles.css%3Fv%3D…` | `?v=` messo **dentro** il path passato a `{% static %}`: `filepath_to_uri` codifica `?` in `%3F` | la versione va in una variabile di contesto separata (`frontend_version`) e `?v=` si aggiunge nel template **dopo** `{% static %}` (PR #3, commit `4b1f1af29`). Il bug non è mai andato online |
| Dopo un rebuild il browser mostra la grafica vecchia | i bundle hanno nomi fissi e cache lunga | Cmd+Shift+R; fix strutturale = cache-busting della PR #3 |
| Lint rosso su `views.py` | file non passato da `ruff format` | `ruff format` prima del commit |
| `No such image` al `compose up`, Coolify non ricrea nulla, in silenzio | `intecha/paperless-ngx:latest` senza tag sul server | `docker tag <image-id> intecha/paperless-ngx:latest` |
| `Temporary failure in name resolution` verso `redis:6379` | dopo un `up` fallito con `--no-deps`, redis resta in `Created` | `docker compose up -d --no-deps redis`, poi paperless |
| Env tornata al valore vecchio dopo un deploy | modificato solo il `.env` a mano | la verità è Coolify: scrivere il valore anche via API |
| `Permesso negato` in `/data/coolify` | cartella padre uid 9999, modo 700 | `sudo -n bash -c "cd … && …"` |

## Dashboard e interfaccia

| Sintomo | Causa | Fix |
|---|---|---|
| Dashboard vuota pur con 6 viste salvate | in 3.x la visibilità è per utente in `ui_settings.settings.saved_views`, non nel modello | scrivere `dashboard_views_visible_ids`, `sidebar_views_visible_ids`, `dashboard_views_sort_order`, `sidebar_views_sort_order`; poi Cmd+Shift+R |
| La UI non riflette un'operazione in blocco | cache dell'interfaccia | Cmd+Shift+R; la verità è il DB (`Document.objects.count()`) |

## AI

| Sintomo | Causa | Fix |
|---|---|---|
| `400 Thinking mode does not support this tool_choice` | `deepseek-flash` è un modello thinking, Paperless usa function calling | env `PAPERLESS_AI_LLM_EXTRA_PARAMS={"reasoning_effort": "none"}` + ricreazione container |
| `TypeError: … unexpected keyword argument 'thinking'` | provato `{"thinking": {"type": "disabled"}}`: curl risponde 200 ma l'SDK OpenAI lo rifiuta | usare `reasoning_effort` (riga sopra) |
| Cambio di `LLM_EXTRA_PARAMS` senza effetto | si legge **solo dall'env**, non da `ApplicationConfiguration` | env + ricreazione container |
| `PermissionError: … llm_index/index.lock` | `docker exec` gira come root e lascia in `data/` file che il worker non può scrivere | comandi che toccano l'indice o salvano documenti: **`docker exec -u 1000`**; riportare a uid 1000 i file creati da root |
| L'AI inventa tag nuovi | tassonomia poco mappabile (es. materiale D&D) | workflow 2 con `ai_create_missing=false`; misurare su fatture/referti prima di estendere |
| Il workflow AI non scatta | trigger *consumo*: il testo non c'è ancora | trigger *documento aggiunto* |

## Archivio e import

| Sintomo | Causa | Fix |
|---|---|---|
| Titoli assegnati da shell tornano al nome file | consumo ancora in finalizzazione | riscrivere a coda vuota, poi `document_renamer` |
| Tutti i file si chiamano come la cartella | storage path scritto come nome cartella | usare il template `CARTELLA/{{ title }}` + `document_renamer` |
| Spazzatura `._*` importata | resource fork macOS finiti in `consume/` | escluderli nel trasferimento (`! -name '._*'`) |
| Documenti importati due volte | il watcher legge file ancora in scrittura | staging + `mv` atomico in `consume/`; dedup per SHA-256 |
| File Office non letti | mancano gotenberg e tika | aggiungerli alla compose (aperto) |
| `POST /api/documents/delete` = 403 con Token | DA VERIFICARE | `doc.delete()` (cestino) + `documents.tasks.empty_trash(doc_ids=[…])` da shell |

## Backup

| Sintomo | Causa | Fix |
|---|---|---|
| Upload su Google Drive in 500 | il remote `gdrive` usa il client_id condiviso di rclone, in dismissione | serve un client_id proprio in `rclone.conf`; oggi il backup va solo sul NAS |

## Comandi base

```
S=sq6zcxrys7rues1umt3pnnl5
# script Django dentro il container (-u 1000 se scrive in data/ o salva documenti)
ssh n8n 'cat > /home/mulo/mio.py' < mio.py
ssh n8n "sudo -n docker exec -u 1000 -i paperless-$S python3 manage.py shell < /home/mulo/mio.py"

# rinominare i file dopo modifiche ai percorsi (idempotente)
ssh n8n "sudo -n docker exec -u 1000 paperless-$S python3 manage.py document_renamer"

# ricostruire l'indice vettoriale (in background)
ssh n8n "sudo -n docker exec -u 1000 -d paperless-$S sh -c 'python3 manage.py document_llmindex rebuild > /tmp/llmindex.log 2>&1'"
ssh n8n "sudo -n docker exec -u 1000 paperless-$S tail -5 /tmp/llmindex.log"
```
