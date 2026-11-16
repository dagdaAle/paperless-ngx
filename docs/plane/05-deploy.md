Stato: bozza generata da agente · Aggiornato: 2026-11-16 · Fonte: dagdaAle/paperless-ngx@2ce8ee998 (dev) + note vault server/Intecha-Docs-2026-10-27, server/Paperless-2026-10-13

# Deploy e operations

| | |
|---|---|
| Server | `n8n` (server di casa), raggiungibile via Tailscale |
| Progetto Coolify | `Paperless` → `fqjug4p6uas39fbog07kx4zg` |
| Servizio Coolify | `sq6zcxrys7rues1umt3pnnl5` |
| Container | `paperless-<servizio>`, `redis-<servizio>` |
| Cartella servizio | `/data/coolify/services/<servizio>/` (compose + `.env`) |
| Dominio | `paperless.intecha.dev` via Cloudflare Tunnel (`/etc/cloudflared/config.yml`) |
| Authentik | login locale di Paperless; un eventuale forward-auth davanti: DA VERIFICARE |
| Healthcheck | `docker inspect … --format '{{.State.Health.Status}}'` = `healthy`; `curl` su `/` = **302** (login) |

## Immagine: si costruisce sul server

`intecha/paperless-ngx:latest` **non è su un registry**. Una PR mergiata su `dev` non arriva in produzione finché non si ricostruisce l'immagine e si ricrea il container.

Comando di build esatto (cartella sul server, argomenti di `docker build`): **DA VERIFICARE**, non è documentato nelle note. Il Dockerfile è quello upstream, il fork non lo modifica.

Quale commit gira oggi in produzione: **DA VERIFICARE**. Fatti noti:
- 13/10 18:47: build dell'immagine con il branding (PR #1).
- 14/10 18:16: container ricreato; il template servito è **senza** `?v=` → la PR #3 non è in produzione.
- Se la PR #2 (logo) sia nell'immagine attuale: DA VERIFICARE.
- PR #3 e #4 sono aperte; dopo il loro merge va ricostruita l'immagine.

## Aggiornare (sequenza che funziona)

1. Merge della PR su `dev` (lo fa Ale, non un agente).
2. Build dell'immagine sul server con tag `intecha/paperless-ngx:latest`.
3. Ricreare il container dal compose:
   ```
   sudo -n bash -c 'cd /data/coolify/services/<servizio> && docker compose up -d --no-deps paperless'
   ```
4. Verifiche: health `healthy`, HTTP 302, controllo a vista di navbar, login e logo, con **Cmd+Shift+R** nel browser.

Cambiare una env: qui Coolify **non ricrea** il container al deploy. Scrivere il valore via API Coolify (`POST` nuova / `PATCH` esistente su `/api/v1/services/<servizio>/envs`, token: file protetto sul server), poi `docker compose up -d --no-deps paperless`, poi `printenv <VAR>` dentro il container.

## Backup

- `/etc/cron.d/paperless-backup`, ogni notte alle **03:30**.
- Script `/opt/paperless-backup/paperless-backup.sh`: copia SQLite consistente (API `backup`) + `data/` (senza `llm_index` e `index`, si ricostruiscono) + `media/` → **restic cifrato sul NAS** (`coolify-nas:coolify-backups/paperless-restic`).
- Retention: 7 giornalieri, 4 settimanali, 12 mensili. Esito in `/opt/paperless-backup/backup.log`.
- Primo snapshot 5/11/2026: 1,9 GiB, `restic check` pulito.
- Restore: stesso `docker run … restic/restic` dello script, `restore latest --host paperless --target …`.
- Off-site su Google Drive: **no** (vedi Runbook).

## Log

`docker logs paperless-<servizio>`; log applicativi in `data/log/` (standard Paperless).
