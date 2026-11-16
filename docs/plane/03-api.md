Stato: bozza generata da agente · Aggiornato: 2026-11-16 · Fonte: dagdaAle/paperless-ngx@2ce8ee998 (dev) + note vault server/Intecha-Docs-2026-10-27, server/Paperless-2026-10-13

# API e tool MCP

Il fork **non aggiunge endpoint**. La REST API è quella di Paperless-ngx: Documentazione ufficiale upstream: https://docs.paperless-ngx.com/api/
Schema OpenAPI sull'istanza: `/api/schema/view/` (standard Paperless).

## Auth

Login web via allauth (pagine rebrandizzate). Per l'API: token o sessione, come upstream.

Nota pratica: `POST /api/documents/delete` ha risposto **403** con `Token` auth in questa configurazione. Le cancellazioni in blocco sono state fatte da shell (vedi Runbook). Causa: DA VERIFICARE.

## Tool MCP

Nel gateway `mcp.intecha.dev` **non c'è** un server MCP per Intecha Docs (verificato sull'elenco tool il 16/11/2026). Le operazioni di amministrazione si fanno da `manage.py shell` nel container.
