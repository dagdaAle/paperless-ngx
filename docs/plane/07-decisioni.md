Stato: bozza generata da agente · Aggiornato: 2026-11-16 · Fonte: dagdaAle/paperless-ngx@2ce8ee998 (dev) + note vault server/Intecha-Docs-2026-10-27, server/Paperless-2026-10-13

# Decisioni (ADR)

## ADR-001 — Fork rebrandizzato invece del solo APP_TITLE / APP_LOGO
- Contesto: Paperless permette titolo e logo da configurazione, non palette, font, navbar e pagine di login.
- Decisione: fork `dagdaAle/paperless-ngx`; override in `src-ui/src/intecha.scss` caricato dopo `theme.scss`, template Django modificati.
- Alternative scartate: solo configurazione (branding parziale).
- Conseguenze: immagine da costruire in proprio; release upstream da riassorbire (ADR-004). Attribuzione "basato su Paperless-ngx" mantenuta (GPL-3.0).

## ADR-002 — Branch di default `dev`
- Contesto: l'upstream sviluppa su `dev`.
- Decisione: il fork usa `dev` come default; tutte le PR puntano lì.
- Conseguenze: `gh pr create --base dev`, mai `main`.

## ADR-003 — Immagine costruita sul server, nessun registry
- Contesto: un solo server, uso personale.
- Decisione: `intecha/paperless-ngx:latest` costruita localmente sul server.
- Alternative scartate: DA VERIFICARE (non documentate).
- Conseguenze: PR verde ≠ produzione; l'immagine può perdere il tag (vedi Runbook).

## ADR-004 — Sync con l'upstream via GitHub Action (PR #4, aperta)
- Decisione: workflow `intecha-sync-upstream.yml`, ogni giorno alle 05:17 UTC + avvio manuale. Se l'ultima release upstream non è in `dev`: merge su `sync/<tag>` e PR verso `dev`. Con conflitti il job fallisce e la mail di GitHub elenca i file (le issue del fork sono disattivate).
- Prerequisito: Settings → Actions → General → *Allow GitHub Actions to create and approve pull requests*.
- Conseguenze: dopo il merge della PR di sync resta manuale la build dell'immagine. Al 16/11 `upstream/dev` ha 203 commit non presenti nel fork. La CI della PR #4 era rossa al 14/10: causa DA VERIFICARE.

## ADR-005 — AI: suggerimenti in cloud, indice semantico in casa
- Contesto: DeepSeek costa poco ma **non espone `/embeddings`**.
- Decisione (di Ale): chat su DeepSeek `deepseek-flash`, embedding con Ollama `bge-m3` sulla GPU del server.
- Alternative scartate: tutto cloud (impossibile per gli embedding); tutto locale (motivazione non documentata: DA VERIFICARE).
- Conseguenze: il testo dei documenti va a DeepSeek per i suggerimenti; l'indice vettoriale resta in casa e non costa.

## ADR-006 — Workflow AI prudente
- Decisione: workflow 2 su *documento aggiunto*, `ai_create_missing=false`, `ai_overwrite_existing=false`. Il tag `da classificare` resta come coda di revisione.
- Motivo: sui primi documenti il modello inventava tag nuovi invece di usare quelli esistenti.

## ADR-007 — Posta: solo in entrata, label dedicata (proposta, non attuata)
- Proposta: regola quotidiana su una label/cartella dedicata (es. `Paperless`) + una passata una tantum sullo storico filtrata sugli allegati documento (`.pdf`, `.docx`, `.xlsx`, `.odt`). Prima esecuzione con la regola `enabled: false`.
- Caselle previste: due Gmail personali + caselle intecha su Aruba (`imaps.aruba.it:993`).
- Stato: bloccata su credenziali (app password Gmail, password IMAP Aruba → Vaultwarden, raccolta *Server intecha*) e sulla scelta del filtro.

## ADR-008 — Backup con restic sul NAS, non su Google Drive
- Motivo: il client_id condiviso di rclone dà 500 sugli upload grossi.
- Conseguenze: nessun off-site finché non c'è un client_id proprio.
