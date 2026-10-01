# CLAUDE.md — google-knowledge-graph-mcp

## Zweck
MCP-Server für die kostenlose Google Knowledge Graph Search API: Entitäten per Name suchen oder per MID nachschlagen (Entitäts-/Schema-Arbeit im SEO). Fremdprojekt von `houtini-ai` (v1.0.7), lokal mit einem eigenen Fix.

## Aufbau
- Einstieg: `src/index.ts` — Low-Level-`Server`, Tool-Liste im `ListToolsRequestSchema`-Handler, Verzweigung per `if (name === …)` im `CallToolRequestSchema`-Handler
- API-Client: `src/client.ts` — `searchEntities()`, `lookupEntities()`, Key-Prüfung (warnt bei Gemini-Key `AQ.`)
- `src/version.ts` — Version aus `package.json` für den Handshake
- Herkunft/Git:
  - Remotes: `origin` = `houtini-ai/google-knowledge-graph-mcp` (Upstream), `fork` = `david-wulf/google-knowledge-graph-mcp`
  - Eigene Anpassung: `2897460` — `ids`, `languages`, `types` als wiederholte Query-Parameter statt kommagetrennt (Lookup mit >1 MID lief sonst auf 400). Branch `fix/repeated-query-params`, gepusht nach `fork`
  - Upstream ziehen: `git fetch origin && git merge origin/main` (auf dem Arbeitsbranch)

## Tools
- `search_knowledge_graph` — Suche nach Name/Thema, Filter `types`, `languages`, `limit` (lesend)
- `lookup_knowledge_graph_entities` — Nachschlagen per MID (`/m/…`, `/g/…`) (lesend)

## Lokal starten & prüfen
```bash
npm install
npm run build        # tsc -> dist/
npm run type-check
npm start            # node dist/index.js
```
Keine Tests im Repo. Schnelltest z. B. mit `npx @modelcontextprotocol/inspector node dist/index.js` (Key muss im Env stehen).

## Einbindung in Claude
`~/.claude.json` (User-Scope) als `google-knowledge-graph`: `node C:\Users\david\.claude\infisical-run.mjs … --alias Google-API-Key:GOOGLE_KNOWLEDGE_GRAPH_API_KEY -- node …\google-knowledge-graph-mcp\dist\index.js`. Nach Änderungen `npm run build`, Claude neu starten.

## Konfiguration
- `GOOGLE_KNOWLEDGE_GRAPH_API_KEY` (Fallback `GOOGLE_CLOUD_API_KEY`) — Google-Cloud-Key (`AIza…`) mit aktivierter Knowledge Graph Search API; kommt aus Infisical

## Neues Tool hinzufügen
1. API-Funktion in `src/client.ts` (Query-Listen als wiederholte Parameter, siehe Fix)
2. Tool-Objekt mit `inputSchema` in das Array im `ListToolsRequestSchema`-Handler
3. `if (name === '<tool>')`-Zweig im `CallToolRequestSchema`-Handler; Antwort als JSON-Text, Fehler landen im gemeinsamen `catch` mit `isError: true`

## Grenzen
- Key nie in Code, Logs oder Commits; Diagnose nur auf stderr (stdout = MCP-Protokoll).
- `.github/workflows/publish-mcp.yml` und `prepublishOnly` gehören dem Upstream — nicht veröffentlichen.

## Offen
- Ob der Fix als PR an `houtini-ai` gegangen ist, steht nicht im Repo; der Arbeitsbranch ist nicht `main`.
