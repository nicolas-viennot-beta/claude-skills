# spec-fonctionnelle-github

Skill Claude pour rédiger et maintenir la **spécification fonctionnelle d'un service à partir de son dépôt GitHub**, à double niveau de lecture : un corps accessible aux non-tech, des blocs dépliables détaillés pour les développeurs et les PM.

Générique : fonctionne pour une application web, une API, un back-office, un pipeline de données. Le dépôt, le service, la destination de la spec et la cadence de mise à jour sont des paramètres.

## Ce que produit la skill

Un fichier Markdown structuré (voir `assets/spec-template.md`) :

- un en-tête `spec-meta` (repo, chemins, commit de référence, cadence, destination) qui rend la mise à jour incrémentale possible ;
- une légende de provenance pour chaque affirmation : `lu dans le code` / `vérifié en prod` / `déduit` ;
- des sections fonctionnelles (pour qui, accès, fonctionnalités avec règles de gestion, rôles et droits, choix réalisés, fonctionnalités désactivées) ;
- des blocs `<details>` techniques (routes, modèle de données, configuration, points d'attention), rendus nativement par GitHub, GitLab, VS Code et la plupart des wikis ;
- un journal des révisions lisible par des parties prenantes.

## Deux modes

| Mode | Déclencheur | Ce qui se passe |
|---|---|---|
| **Création** | « fais la spec de l'espace partenaire à partir de `owner/repo` » | Cadrage (5 paramètres), exploration guidée du dépôt, rédaction depuis le gabarit, vérification prod optionnelle, livraison |
| **Mise à jour** | « mets à jour la spec de X » ou tâche planifiée | Lecture de l'en-tête, contrôle de la cadence, `git log` depuis le commit de référence, tri des changements, modification en place, journal |

## Routine

La cadence (`hebdo`, `mensuelle`, `trimestrielle`, `a-chaque-release`, `manuelle`) est propre à chaque spec et stockée dans son en-tête. Une tâche planifiée hebdomadaire peut donc porter plusieurs specs de cadences différentes : la skill s'arrête d'elle-même quand ce n'est pas l'heure. Le modèle de prompt de la tâche est dans `references/routine-mise-a-jour.md`.

## Structure

```
spec-fonctionnelle-github/
├── SKILL.md                          # instructions principales
├── assets/
│   └── spec-template.md              # gabarit de la spec produite
├── references/
│   ├── analyse-repo.md               # grille d'exploration d'un dépôt
│   ├── verification-prod.md          # protocole de vérification en production, règles sur les captures
│   └── routine-mise-a-jour.md        # mise à jour incrémentale, cadence, prompt de tâche planifiée
└── evals/                            # cas de test (prompts)
```

## Installation

- **Claude Code / marketplace de plugins** : copier le dossier dans le répertoire des skills du plugin (ex. `plugins/mes-skills/spec-fonctionnelle-github/`) et pousser.
- **Claude.ai** : importer le fichier `.skill` packagé, ou le `SKILL.md` seul si les ressources annexes ne sont pas nécessaires.

## Garde-fous

- Aucune donnée personnelle dans la spec ni dans les captures.
- L'utilisateur se connecte lui-même aux services authentifiés ; la skill ne saisit jamais d'identifiants.
- Jamais de push direct sur la branche principale : branche + proposition de PR.
- Les écarts entre code et production sont documentés comme des faits, pas corrigés.

## Licence

MIT.
