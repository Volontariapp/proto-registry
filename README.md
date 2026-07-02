# Proto Registry

## Project Overview & Value Proposition
Le dépôt `proto-registry` agit comme la Single Source of Truth (SSOT) pour l'ensemble des contrats d'interface de la plateforme Volontariapp. Son rôle central est de définir de manière formelle et agnostique la communication inter-services via gRPC et Protocol Buffers. 

La proposition de valeur de ce registre repose sur la prévention de la dérive des API (API Drift). En interdisant les modifications locales de contrats au sein des microservices individuels, l'architecture garantit une gouvernance stricte des schémas de données. Toute altération de payload, d'entité métier ou de service gRPC doit être introduite, validée et versionnée au sein de ce registre central.

## Key Features
- **Gouvernance Centralisée des Schémas** : Définition de toutes les structures de données, requêtes et réponses gRPC.
- **Ségrégation CQRS** : Séparation stricte des intentions de lecture (Queries) et d'écriture (Commands) dans les définitions de contrats.
- **Validation Statique Avancée** : Vérification systématique du linting et détection automatisée des ruptures de compatibilité ascendante (Breaking Changes).
- **Génération de Code Automatisée** : Transformation des fichiers `.proto` en interfaces TypeScript et décorateurs spécifiques au framework (NestJS) via `ts-proto`.
- **Distribution GitOps** : Publication automatisée des contrats compilés vers le registre de dépendances (`npm-packages`) via Pull Request.

## Tech Stack & Dependencies

| Technologie | Catégorie | Usage / Rôle |
| :--- | :--- | :--- |
| **Protocol Buffers (v3)** | Sérialisation | Langage de définition d'interface (IDL) pour les contrats de données. |
| **gRPC** | Transport | Framework RPC de haute performance pour la communication inter-services. |
| **Buf** | Outillage | CLI de linting, de vérification de compatibilité et de build de schémas protobuf. |
| **ts-proto** | Génération de Code | Plugin de compilation générant des interfaces TypeScript natives et compatibles NestJS. |
| **GitHub Actions** | CI/CD | Automatisation de la validation (CI) et de la propagation (CD) des contrats générés. |

## Getting Started

### Prérequis
- **Buf CLI** installé sur votre machine (`brew install bufbuild/buf/buf` ou via npm).
- Un environnement compatible UNIX (Linux/macOS) pour l'exécution des scripts de génération locaux.
- **Node.js** (>= 18.x) si vous souhaitez tester le cycle de compilation complet avec des dépendances locales.

### Installation et Configuration Locale
Clonez le dépôt et vérifiez l'installation de Buf :

```bash
git clone git@github.com:volontariapp/proto-registry.git
cd proto-registry
buf --version
```

Aucune variable d'environnement (`.env`) n'est requise pour l'exécution locale des outils de linting ou de génération de ce dépôt.

### Commandes de Build et Génération
Pour générer les interfaces TypeScript localement (les artefacts seront placés dans le répertoire `gen/ts`) :

```bash
buf generate
```

## Testing

La stratégie de test dans ce registre repose sur l'analyse statique des schémas et la vérification de la compatibilité des contrats.

### Linting des Contrats
Pour vérifier la conformité syntaxique et le respect des conventions de nommage :

```bash
buf lint
```

### Vérification de Rétrocompatibilité (Breaking Changes)
Pour s'assurer qu'aucune modification ne brise les contrats existants pour les clients en production, testez vos changements contre la branche principale :

```bash
buf breaking --against '.git#branch=main'
```

## CI/CD & Deployment

Le cycle de vie du déploiement est entièrement orchestré par GitHub Actions, implémentant une approche GitOps pour la distribution des contrats.

1. **Validation Continue (CI)** : À chaque Pull Request, les workflows exécutent `buf lint` et `buf breaking`.
2. **Synchronisation Automatisée (CD)** : Lors du merge sur la branche `main` (`sync-to-npm.yml`), le CI/CD génère les artefacts TypeScript.
3. **Propagation vers npm-packages** : Le pipeline ouvre automatiquement une Pull Request sur le monorepo `npm-packages` pour mettre à jour les paquets internes (`@volontariapp/contracts` et `@volontariapp/contracts-nest`).
4. **Plan de Remédiation** : En cas de désynchronisation entre le registre et le monorepo, le workflow `emergency-reset.yml` permet une regénération complète (clean state) pour restaurer l'intégrité de la distribution.
