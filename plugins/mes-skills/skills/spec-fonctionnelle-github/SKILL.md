---
name: spec-fonctionnelle-github
description: Rédige ou met à jour la spécification fonctionnelle d'un service (espace partenaire, observatoire, API, back-office, site…) à partir de son dépôt GitHub — en demandant toujours, avant de lancer une rédaction ou une routine, quel service est visé et quel compte GitHub utiliser —, à double niveau de lecture — accessible aux non-tech, avec des sections dépliables détaillées pour les tech et PM. Utiliser dès que l'utilisateur veut documenter, spécifier, décrire ou faire découvrir un service existant depuis son code, demande « une spec », « une doc fonctionnelle », « les grands principes de X », « comment marche X d'après le repo », veut onboarder quelqu'un sur un service, ou veut mettre à jour / rafraîchir une spec existante. Couvre aussi la routine de mise à jour périodique (diff des commits depuis la dernière révision) et la vérification optionnelle sur le site en production.
---

# Spécification fonctionnelle depuis GitHub

Produire, à partir du code d'un dépôt, un document qu'un nouvel arrivant peut lire pour comprendre un service : ce qu'il fait, pour qui, les choix qui ont été faits, comment il est structuré. Le document se lit à deux niveaux : le corps est accessible à quelqu'un qui ne lit pas de code ; les détails techniques sont dans des blocs dépliables que les tech et les PM ouvrent quand ils en ont besoin.

La skill fonctionne en deux modes :

- **Création** — première spec d'un service.
- **Mise à jour** — une spec existe déjà (reconnaissable à son en-tête `spec-meta`) ; on ne réécrit pas, on fait évoluer depuis le dernier commit de référence.

## Pourquoi ce format

Une spec écrite d'après le code a une qualité que n'ont ni les PRD ni les docs rédigées de mémoire : elle décrit ce qui *existe*, pas ce qui était prévu. C'est précieux pour l'onboarding, les arbitrages produit et les échanges avec les parties prenantes — à condition que le document reste honnête sur ce qu'il a vérifié et ce qu'il a déduit, et qu'il ne vieillisse pas. D'où trois règles qui traversent toute la skill :

1. **Dire d'où vient chaque affirmation** : lu dans le code, vérifié en production, ou déduit. Un lecteur non-tech doit pouvoir faire confiance au corps du texte ; un tech doit pouvoir retrouver le fichier.
2. **Séparer le quoi du comment** : le corps répond « que fait le service, pour qui, pourquoi ce choix » ; les blocs `<details>` répondent « où dans le code, avec quelles structures, quelles limites ».
3. **Rendre la mise à jour mécanique** : l'en-tête de la spec enregistre le commit de référence et la cadence, pour que la prochaine révision soit un diff, pas une relecture complète.

## Étape 0 — Cadrer (avant toute lecture de code)

Deux questions se posent **toujours**, avant toute rédaction et avant tout lancement ou création de routine, même si la conversation semble déjà y répondre, même en mode mise à jour :

1. **Quel service ?** — nom du service et sous-dossier(s) du dépôt. Un même repo (surtout un monorepo) porte plusieurs services ; se tromper de périmètre produit une spec hors sujet ou une routine qui surveille les mauvais chemins.
2. **Quel compte GitHub ?** — l'utilisateur peut disposer de plusieurs comptes (personnel, professionnel). Le compte détermine l'accès au dépôt, le nom sous lequel une branche ou une PR est créée, et le dépôt où la spec est hébergée. Ne jamais le déduire d'une session précédente ni d'un en-tête `spec-meta` : le confirmer explicitement.

Les poser ensemble, en une seule fois (AskUserQuestion si disponible), avec ceux des paramètres ci-dessous qui ne sont pas encore connus. Pour une mise à jour, les autres paramètres viennent de l'en-tête `spec-meta` de la spec existante et ne sont pas redemandés ; seuls le service et le compte GitHub sont reconfirmés.

| Paramètre | Rôle | Défaut si l'utilisateur ne tranche pas |
|---|---|---|
| `service` | Nom lisible + sous-dossier(s) concerné(s) dans le repo | **Toujours demandé** — proposer les candidats lus dans l'arborescence si l'utilisateur hésite |
| `compte_github` | Compte GitHub à utiliser pour lire le dépôt et, le cas échéant, écrire (branche, PR, repo de specs) | **Toujours demandé** — aucun défaut |
| `repo` | Dépôt GitHub (`owner/repo`) | — (indispensable) |
| `public_cible` | Qui lira en priorité (parties prenantes, nouvel arrivant tech, PM…) | Mixte, double niveau |
| `destination` | Où vit la spec : `docs/specs/<service>.md` dans le repo produit, un repo de specs séparé, un dossier local, Notion | Repo produit si accès en écriture, sinon dossier local |
| `cadence` | `hebdo` / `mensuelle` / `trimestrielle` / `a-chaque-release` / `manuelle` | `mensuelle` |
| `url_prod` | URL du service en production, pour la vérification optionnelle | Aucune (vérification sautée) |

Le choix de la destination mérite une phrase d'explication à l'utilisateur la première fois : une spec stockée à côté du code est versionnée avec lui, relue en PR, et la routine peut s'appuyer sur `git log` depuis le SHA de référence. Hors du repo produit, la routine reste possible mais la spec et le code peuvent diverger sans que personne le voie dans une PR.

## Étape 1 — Obtenir et explorer le dépôt

Cloner en surface (`git clone --depth 50` suffit pour une création ; pour une mise à jour, cloner assez profond pour atteindre le commit de référence, ou utiliser `git fetch --shallow-since`). Si un outil d'accès au dépôt est disponible dans la session (par exemple un connecteur GitHub ou `add_repo`), l'utiliser pour les dépôts privés plutôt que de demander un token.

Ensuite suivre **`references/analyse-repo.md`** : il liste, par couche, ce qu'il faut aller chercher (points d'entrée, routes et écrans, modèle de données, rôles et droits, feature flags, intégrations, configuration) et comment distinguer le code vivant du code mort. Ne pas lire le repo linéairement : partir des routes ou des commandes exposées et remonter vers les données.

Tenir au fil de l'exploration une liste « affirmation → source (fichier:ligne) » : elle alimente les blocs techniques et évite d'écrire de mémoire.

## Étape 2 — Rédiger la spec

Utiliser le gabarit **`assets/spec-template.md`** tel quel pour la structure ; adapter les titres de sections fonctionnelles au service (un service sans rôles n'a pas de section « Rôles et droits », un batch n'a pas d'« Écrans »).

Principes d'écriture, par ordre d'importance :

- **Le corps se lit sans ouvrir un seul bloc dépliable.** Test : masquer tous les `<details>` et relire ; si une idée manque, elle était au mauvais endroit.
- **Une fonctionnalité = ce qu'elle permet, à qui, dans quel but, puis ses règles de gestion.** Les règles de gestion (seuils, exclusions, statuts, cas particuliers) sont souvent ce que les parties prenantes cherchent et ce que le code seul documente : les formuler en phrases, pas en pseudo-code.
- **Les choix réalisés ont leur section.** Un choix d'architecture, de périmètre ou de simplification (« pas de base de données propre », « authentification déléguée à X », « fonctionnalité Y développée mais désactivée ») se note avec sa conséquence pour l'utilisateur, et, quand le code ou l'historique git le laisse voir, la raison.
- **Les blocs `<details>` sont techniques, pas verbeux.** Chemin des fichiers, interfaces ou schémas de données, noms des variables de configuration, points d'attention (dette, fragilité, dépendance). Un tableau vaut mieux qu'un paragraphe.
- **Marquer le statut de chaque affirmation sensible** avec la légende du gabarit : `lu dans le code` / `vérifié en prod le JJ/MM/AAAA` / `déduit`. Une fonctionnalité présente dans le code mais derrière un flag désactivé est une information en soi : la décrire comme telle plutôt que l'omettre ou la présenter comme active.
- **Diagrammes** : un diagramme Mermaid pour le modèle de rôles ou le flux principal aide presque toujours ; au-delà de deux, ils cessent d'être lus.
- **Aucune donnée personnelle** : pas de nom, d'adresse e-mail, d'identifiant d'utilisateur réel, ni dans le texte ni dans les captures. Les données « métier » publiques (noms d'organisations, SIRET, intitulés de campagnes publiques) sont acceptables.

Écrire en français, au présent, sans jargon dans le corps ; expliquer un sigle à sa première occurrence.

## Étape 3 — Vérifier sur la production (optionnel, recommandé)

Si `url_prod` est fournie et qu'un navigateur est disponible, suivre **`references/verification-prod.md`**. L'objectif n'est pas de tout re-tester mais de confirmer ce que le code ne prouve pas : les flags réellement actifs, les libellés d'écran, les colonnes affichées, les comportements côté client (état perdu au rechargement, redirections). Chaque point vérifié prend la mention `vérifié en prod`.

Si la vérification révèle un écart avec le code (fonctionnalité présente dans le code mais absente en prod, ou l'inverse), le documenter comme un écart, pas comme une erreur à corriger : c'est une information utile pour le lecteur.

L'utilisateur se connecte lui-même aux services qui demandent une authentification ; ne jamais saisir d'identifiants.

## Étape 4 — Renseigner l'en-tête et livrer

Compléter le bloc `spec-meta` en tête du gabarit : repo, service, chemins, `commit_reference` (SHA complet de HEAD au moment de la rédaction), `derniere_revision`, `cadence`, `url_prod`, `destination`, et la liste des vérifications prod effectuées. C'est ce bloc qui rend la routine possible.

Ajouter la première entrée du **journal des révisions** en fin de document.

Livrer selon la destination : écrire le fichier dans le dépôt (sur une branche, proposer la PR ; ne jamais pousser directement sur la branche principale), ou dans le dossier indiqué, ou créer la page Notion. Avant toute écriture sur GitHub, respecter les règles de l'utilisateur sur le choix du compte si une skill de garde existe pour cela.

Résumer à l'utilisateur en quelques lignes : ce qui a été couvert, ce qui n'a pas pu être vérifié, et la prochaine échéance de mise à jour.

## Mode mise à jour

Déclenché manuellement (« mets à jour la spec de X ») ou par une tâche planifiée. Suivre **`references/routine-mise-a-jour.md`**. En résumé :

0. Confirmer le service et le compte GitHub (Étape 0). En déclenchement manuel, les demander ; en tâche planifiée, le prompt de la tâche doit les nommer tous les deux, sinon s'arrêter en le signalant plutôt que de deviner.
1. Lire l'en-tête `spec-meta` de la spec existante.
2. Vérifier la cadence : si la dernière révision est plus récente que la cadence ne l'exige et qu'aucun déclencheur manuel ne force, s'arrêter en le disant.
3. `git log <commit_reference>..HEAD -- <chemins>` : lister les commits qui touchent le périmètre du service.
4. Trier : ce qui change le fonctionnel (nouvelle route, nouveau champ, nouvelle règle, flag activé/désactivé, suppression), ce qui change seulement la technique (refactor, dépendances), ce qui est hors sujet.
5. Modifier les sections concernées **en place** ; ne pas regénérer le document. Marquer les affirmations touchées comme `lu dans le code` jusqu'à nouvelle vérification prod.
6. Mettre à jour `commit_reference`, `derniere_revision`, ajouter une entrée au journal des révisions avec la liste des changements fonctionnels en langage non technique.
7. Livrer comme en création (PR, fichier, page). Si aucun commit ne touche le périmètre, le noter dans le journal avec le nouveau SHA et s'arrêter : une spec « à jour, rien à signaler » est un résultat valable.

## Mettre en place la routine

Quand l'utilisateur veut automatiser, demander d'abord quel service et quel compte GitHub la routine concerne (Étape 0), puis créer une tâche planifiée (avec l'outil de planification de la session) dont le prompt est autonome : il nomme le service, le compte GitHub, le repo, le chemin de la spec et demande d'exécuter cette skill en mode mise à jour. Un modèle de prompt prêt à adapter est dans `references/routine-mise-a-jour.md`. Une tâche par spec, dont la fréquence d'exécution est au moins aussi fréquente que la cadence de la spec (la skill elle-même s'arrête si ce n'est pas l'heure, donc une tâche hebdomadaire peut porter une spec mensuelle).

## Ressources

- `assets/spec-template.md` — gabarit complet de la spec, avec en-tête `spec-meta`, légende des statuts, sections et blocs dépliables.
- `references/analyse-repo.md` — grille d'exploration d'un dépôt, par couche, avec les indices à chercher selon la stack.
- `references/verification-prod.md` — protocole de vérification sur le site en production et règles sur les captures.
- `references/routine-mise-a-jour.md` — procédure de mise à jour incrémentale, gestion de la cadence, modèle de prompt pour la tâche planifiée.
