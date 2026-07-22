# proto-registry

SSOT des contrats gRPC de Volontariapp. `.proto` = source de vérité, jamais modifiés
depuis les microservices.

## Structure de proto/volontariapp/

- `common/` : `geo.proto`, `pagination.proto`, `timestamp.proto` — types partagés
  (pas de service, que des messages).
- `{post,user,social,event}/` : un domaine = 5 fichiers CQRS
  - `<domain>.proto` — entité pure (ex: `Event`, `User`)
  - `<domain>.command.proto` — requêtes d'écriture (Create/Update/Delete)
  - `<domain>.query.proto` — requêtes de lecture (Get/List)
  - `<domain>.responses.proto` — DTO de réponse
  - `<domain>.services.proto` — définitions RPC gRPC
- `post/` a en plus `comment.proto`, `comment.command.proto`, `comment.query.proto`,
  `comment.responses.proto` (pas de `comment.services.proto` : les RPC comment sont
  exposés via `post.services.proto`).

Ne jamais éditer les modules par ailleurs — nouveau champ/message va dans le fichier
du bon type (entity/command/query/responses/services) du domaine concerné.

## Génération de code

Un seul plugin configuré dans `buf.gen.yaml` : `ts_proto` (nom local
`protoc-gen-ts_proto`), sortie dans `gen/ts/`, options clés : `nestJs=true`,
`outputClientImpl=false`, `useDate=false`, `addMetadata=true`.

```bash
buf generate   # régénère gen/ts/ à partir de proto/
buf lint       # vérifie les conventions (voir buf.yaml)
buf breaking --against '.git#branch=main'   # détecte les ruptures de compat
```

Les artefacts générés sont ensuite publiés vers `npm-packages`
(`@volontariapp/contracts`, `@volontariapp/contracts-nest`) via CI, pas de publication
manuelle depuis ce repo.

## Lint (buf.yaml)

`lint.use: STANDARD` avec exceptions explicites :
`PACKAGE_VERSION_SUFFIX`, `FILE_LOWER_SNAKE_CASE`, `RPC_REQUEST_STANDARD_NAME`,
`RPC_RESPONSE_STANDARD_NAME` désactivées — donc pas besoin de suffixer les packages
par une version, les noms de fichiers multi-mots (`event.command.proto`) sont
acceptés, et les requêtes/réponses RPC n'ont pas à suivre le nommage standard
buf (`MethodNameRequest`/`Response`).

## Workflow de changement de contrat

Voir `.agents/skills/global/proto-contract-evolution/SKILL.md` pour l'ordre de
rollout (proto-registry -> npm-packages -> microservices) et l'usage de
`buf breaking`.

## 🚀 RTK - Rust Token Killer (Optimized)
All shell commands (`git`, `npm`, `jest`, etc.) are automatically proxied via `rtk` for 80% token savings.
- **Direct Usage:** `rtk gain` (analytics), `rtk discover` (missed savings).
- **Files:** Use `rtk read <file>`, `rtk ls`, `rtk find`, `rtk grep` for compressed agent output.

## graphify

This project has a knowledge graph at graphify-out/ with god nodes, community structure, and cross-file relationships.

Rules:
- For codebase questions, first run `graphify query "<question>"` when graphify-out/graph.json exists. Use `graphify path "<A>" "<B>"` for relationships and `graphify explain "<concept>"` for focused concepts. These return a scoped subgraph, usually much smaller than GRAPH_REPORT.md or raw grep output.
- If graphify-out/wiki/index.md exists, use it for broad navigation instead of raw source browsing.
- Read graphify-out/GRAPH_REPORT.md only for broad architecture review or when query/path/explain do not surface enough context.
- After modifying code, run `graphify update .` to keep the graph current (AST-only, no API cost).
