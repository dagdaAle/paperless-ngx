Stato: bozza generata da agente · Aggiornato: 2026-11-16 · Fonte: dagdaAle/paperless-ngx@2ce8ee998 (dev) + note vault server/Intecha-Docs-2026-10-27, server/Paperless-2026-10-13

# Sviluppo locale

Setup, comandi e test sono quelli upstream: https://docs.paperless-ngx.com/development/

## Specifico del fork

| | |
|---|---|
| Copia locale | `~/Progetti/paperless-ngx` |
| Remote | `origin` = `dagdaAle/paperless-ngx`, `upstream` = `paperless-ngx/paperless-ngx` |
| Branch base | `dev` |
| Flusso | branch `feat/`, `fix/`, `ci/`, `docs/` → PR su `dev`: `gh pr create --head <branch> --base dev --repo dagdaAle/paperless-ngx` |
| Merge e deploy | solo Ale |

## File del branding

- `src-ui/src/intecha.scss` — palette (petrolio `#1b6d7f`, navy `#121c21`, sabbia `#d9b382`, crema `#faf6ee`), font Cairo, navbar.
- `src-ui/src/styles.scss` (importa Cairo e `intecha`), `theme.scss`, `index.html`, `manifest.webmanifest`.
- Logo: `common/logo/logo.component.html`, `logo/brand-mark/brand-mark.component.html`, `templates/paperless-ngx/snippets/svg_logo.html`, `svg_leaf.html`.
- Pagine Django: `src/documents/static/base.css`, `templates/paperless-ngx/base.html`, `templates/index.html`, `templates/account/*`, `socialaccount/*`, `mfa/authenticate.html`.
- Dipendenza aggiunta: `@fontsource/cairo`.

Regola: toccare il meno possibile i file upstream. Ogni riga cambiata è un conflitto potenziale al sync.

## Prima di aprire una PR

- `ruff format` sui file Python (il 14/10 il Lint è fallito su `views.py` per questo).
- I test upstream restano verdi **senza modificarli** (es. `test_index` sui path dei bundle).
- Build frontend: comandi standard upstream in `src-ui/` (pnpm).
- CI: verificare per SHA, non con `gh run watch`.
- Dopo il merge la produzione non cambia finché non si ricostruisce l'immagine sul server.
