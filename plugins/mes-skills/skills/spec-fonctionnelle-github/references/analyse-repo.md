# Grille d'exploration d'un dépôt

Objectif : comprendre un service par son code en un temps raisonnable, sans lire le dépôt de bout en bout, et en gardant pour chaque constat la trace du fichier qui le fonde.

## Ordre d'exploration

Partir de l'extérieur (ce que le service expose) et remonter vers l'intérieur (comment il le fait). Cet ordre produit naturellement une spec orientée usage plutôt qu'une description d'arborescence.

1. **Ce que le dépôt dit de lui-même** — `README`, `docs/`, `CONTRIBUTING`, `ADR/` ou `decisions/`, `CHANGELOG`. Lire en premier : c'est là que les choix sont parfois déjà expliqués. Noter les écarts avec le code quand on les rencontre ensuite (un README vieillit).
2. **Le périmètre** — dans un monorepo, identifier le ou les sous-dossiers du service et les dossiers partagés dont il dépend (`shared/`, `packages/`, `libs/`). Le périmètre déclaré dans l'en-tête `spec-meta` doit couvrir tout ce dont un changement affecterait le service, sinon la routine de mise à jour ratera des commits.
3. **Les points d'entrée** — ce que le service expose au monde :
   - application web : routes / pages (`app/`, `pages/`, `routes/`, `urls.py`, `router.*`), navigation, menus ;
   - API : définitions d'endpoints, schémas OpenAPI, contrôleurs, handlers ;
   - batch / pipeline : commandes, crons, DAG, modèles dbt, `justfile` / `Makefile` ;
   - CLI : commandes et sous-commandes.
   Chaque point d'entrée est un candidat à une sous-section « Fonctionnalité ».
4. **Les écrans ou les contrats** — pour chaque point d'entrée, ce que l'utilisateur voit ou reçoit : composants de page, formulaires, tableaux (colonnes, filtres, tri, pagination), messages d'erreur, schémas de réponse.
5. **Les rôles et droits** — où sont définis les rôles, où est décidé ce qui est visible, comment un rôle est obtenu. Chercher : `role`, `permission`, `can`, `isAdmin`, `scope`, middleware d'auth, gardes de route.
6. **Les règles de gestion** — c'est le cœur de la valeur de la spec. Elles se cachent dans : les validations de formulaire, les `if` des services métier, les requêtes SQL (filtres, exclusions, jointures), les constantes nommées, les tests (souvent la meilleure source : un test décrit un comportement attendu en une phrase). Les formuler en français, pas en code.
7. **Le modèle de données** — interfaces TypeScript, schémas Pydantic/Zod, modèles ORM, migrations, DDL. Ne retenir que les entités et champs qui ont un sens métier.
8. **Les intégrations** — appels à des API externes, services d'identité, files de messages, stockage, analytics. Noter ce qui se passe quand l'intégration est indisponible, si le code le dit.
9. **La configuration et les flags** — `.env.example`, `config/`, `features.ts`, variables `NEXT_PUBLIC_*`, `settings.py`, `values.yaml`. Un flag révèle une fonctionnalité en préparation ou retirée ; le noter avec son nom exact, et chercher s'il est résolu au build ou à l'exécution (cela change ce que la vérification prod peut prouver).
10. **L'historique** — `git log --oneline -- <chemin>` sur les derniers mois, et `git log -S "<nom de flag>"` pour dater l'apparition d'une fonctionnalité. Les messages de commit et les PR expliquent parfois un choix que le code ne commente pas.

## Distinguer le vivant du mort

Un dépôt contient toujours du code qui ne sert plus. Avant de documenter une fonctionnalité, vérifier qu'elle est reliée : la route est-elle référencée dans la navigation ? Le composant est-il importé ? La commande est-elle appelée par un cron ou un `justfile` ? Un flag désactivé, une route orpheline ou un module non importé vont dans la section « Fonctionnalités en préparation ou désactivées », pas dans « Fonctionnalités ».

## Indices selon la stack

| Stack | Où chercher en priorité |
|---|---|
| Next.js / React | `app/**/page.tsx` ou `pages/`, `layout.tsx`, `middleware.ts`, `config/`, `interfaces/` ou `types/`, hooks d'auth |
| Django / Wagtail | `urls.py`, `views.py`, `models.py`, `admin.py`, `settings/`, `templates/`, migrations, `wagtail_hooks.py` |
| FastAPI / Flask | routeurs, schémas Pydantic, dépendances d'auth, `alembic/` |
| Node / Express / Nest | `routes/`, `controllers/`, `services/`, `guards/`, `dto/` |
| dbt / pipeline data | `models/` par couche, `macros/`, `seeds/`, `schema.yml` (descriptions de colonnes), `README` du projet, `justfile` |
| Infra | `docker-compose`, `helm/`, `.github/workflows/` — pour les environnements et le déploiement, pas pour la spec fonctionnelle, sauf si cela révèle des flags par environnement |

## Ce qu'on note au fil de l'eau

Tenir un fichier de travail `notes-exploration.md` (hors livrable) avec trois colonnes : constat, source (`fichier:ligne`), statut (`code` / `à vérifier en prod` / `déduit`). Les blocs techniques de la spec se rédigent presque directement depuis ce fichier, et la vérification prod part de la colonne « à vérifier ».

## Signaux d'alerte

- Un README qui décrit une fonctionnalité introuvable dans le code : la signaler comme écart dans « Choix réalisés et limites connues ».
- Deux implémentations du même concept (ancienne et nouvelle) : chercher laquelle est reliée ; documenter l'autre comme dette.
- Des règles de gestion dupliquées entre front et back avec des valeurs différentes : c'est une information importante pour les tech, à mettre en « Points d'attention ».
- Des données personnelles en dur dans le code ou les fixtures : ne jamais les reproduire dans la spec.
