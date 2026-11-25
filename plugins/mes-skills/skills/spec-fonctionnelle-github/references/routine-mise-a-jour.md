# Routine de mise à jour

Une spec qui n'est pas maintenue perd sa valeur en quelques mois, et une spec regénérée de zéro à chaque fois perd l'historique des choix et le travail de vérification. La routine fait le milieu : à chaque échéance, elle lit ce qui a changé dans le code depuis la dernière révision et ne modifie que les sections concernées.

## 0. Confirmer le service et le compte GitHub

Avant toute chose, même si la spec à mettre à jour est désignée sans ambiguïté : quel service, et quel compte GitHub ? En déclenchement manuel, le demander à l'utilisateur en une question. En tâche planifiée, personne ne répond : le prompt de la tâche doit nommer les deux (voir le modèle en §7) ; s'il ne le fait pas, s'arrêter et le dire dans le résumé plutôt que de choisir un compte par défaut. Le compte conditionne l'accès au dépôt et le nom sous lequel la branche et la PR seront créées — une erreur ici est visible par toute l'équipe.

## 1. Lire l'en-tête `spec-meta`

Il contient tout ce qu'il faut : `repo`, `chemins`, `commit_reference`, `derniere_revision`, `cadence`, `destination`, `url_prod`. Si l'en-tête manque ou est incomplet (spec écrite à la main ou avec une ancienne version de la skill), le reconstituer avec l'utilisateur avant d'aller plus loin : sans `commit_reference`, il n'y a pas de diff possible et la seule option est une relecture complète.

## 2. Décider s'il est temps

| Cadence | La révision est due si… |
|---|---|
| `hebdo` | `derniere_revision` date de 7 jours ou plus |
| `mensuelle` | `derniere_revision` date de 28 jours ou plus |
| `trimestrielle` | `derniere_revision` date de 90 jours ou plus |
| `a-chaque-release` | un tag ou une release a été publié depuis `commit_reference` (`git tag --contains <sha>` ou l'API releases) |
| `manuelle` | jamais automatiquement ; seulement sur demande explicite |

Un déclenchement manuel (« mets à jour la spec maintenant ») passe outre la cadence. Un déclenchement planifié qui arrive trop tôt s'arrête en une phrase : « Spec révisée le JJ/MM, cadence mensuelle, prochaine révision à partir du JJ/MM ». C'est ce qui permet de programmer une tâche hebdomadaire unique pour plusieurs specs de cadences différentes.

## 3. Obtenir le diff

```bash
git clone --filter=blob:none https://github.com/<owner>/<repo>.git
cd <repo>
# s'assurer que le commit de référence est présent
git fetch --shallow-since="<derniere_revision moins 1 jour>" || git fetch --unshallow
git log --oneline --no-merges <commit_reference>..HEAD -- <chemin1> <chemin2>
git diff --stat <commit_reference>..HEAD -- <chemin1> <chemin2>
```

Si `commit_reference` est introuvable (historique réécrit, dépôt déplacé), le dire et proposer une relecture complète : ne pas inventer un diff.

## 4. Trier les changements

Lire les commits et les fichiers touchés, puis classer :

| Catégorie | Exemples | Effet sur la spec |
|---|---|---|
| **Fonctionnel** | nouvelle route ou page, champ ajouté ou retiré, règle de gestion modifiée (seuil, filtre, statut), flag activé ou désactivé, rôle ajouté, intégration ajoutée ou retirée, libellé changé | Modifier la section concernée ; entrée dans le journal |
| **Technique visible** | changement de stack, de mécanisme d'authentification, de modèle de données | Modifier les blocs `<details>` concernés ; entrée dans le journal si cela change un choix réalisé |
| **Technique invisible** | refactor, montée de version de dépendances, tests, CI, formatage | Rien, ou une ligne dans les points d'attention si cela réduit ou augmente une dette signalée |
| **Hors périmètre** | commits qui touchent le chemin mais pas le service (fichiers partagés sans effet) | Rien |

Le doute se tranche en lisant le diff du fichier, pas le message de commit. Les tests modifiés ou ajoutés sont souvent le moyen le plus rapide de comprendre une nouvelle règle de gestion.

## 5. Modifier en place

- Ouvrir la spec et modifier uniquement les sections touchées. Garder la structure, les captures, les blocs non concernés.
- Une fonctionnalité nouvelle prend une sous-section au bon endroit, au même format que les autres.
- Une fonctionnalité supprimée ne disparaît pas silencieusement : la retirer de « Fonctionnalités » et ajouter une ligne dans « Choix réalisés et limites connues » (« retirée le … », avec la raison si le commit la donne). Un lecteur qui l'a connue doit comprendre où elle est passée.
- Un flag qui change d'état passe d'une section à l'autre (« en préparation » ↔ « Fonctionnalités »).
- Les affirmations modifiées reprennent le statut `lu dans le code` : la vérification prod antérieure ne vaut plus pour elles. Si `url_prod` est renseignée et qu'un navigateur est disponible, proposer de revérifier ces points seulement (pas tout le service).
- Mettre à jour les captures uniquement si l'écran correspondant a changé.

## 6. Clore la révision

- `commit_reference` ← nouveau SHA de HEAD ; `derniere_revision` ← date du jour ; `derniere_verification_prod` si une vérification a eu lieu.
- Ajouter une ligne au journal des révisions : date, SHA court, « Mise à jour », et les changements fonctionnels en langage non technique (c'est cette colonne que les parties prenantes lisent pour savoir ce qui a changé sans relire la spec). S'il n'y a aucun changement fonctionnel, l'écrire : « Aucun changement fonctionnel ; N commits techniques ».
- Livrer comme en création : fichier sur une branche avec proposition de PR, ou écriture dans le dossier, ou mise à jour de la page Notion. Ne jamais pousser directement sur la branche principale ; respecter les règles de l'utilisateur sur le compte GitHub à utiliser.
- Résumer à l'utilisateur en trois lignes : commits analysés, changements fonctionnels retenus, prochaine échéance.

## 7. Modèle de prompt pour la tâche planifiée

À adapter (service, compte GitHub, repo, chemin de la spec, destination) et à créer avec l'outil de planification de la session, après avoir demandé à l'utilisateur quel service et quel compte la routine concerne. Chaque exécution démarre une session neuve : le prompt doit se suffire à lui-même.

```
Exécute la skill « spec-fonctionnelle-github » en mode mise à jour pour la
spécification suivante :

- Service : <nom du service> (chemins : <sous-dossiers>)
- Dépôt : <owner>/<repo>
- Compte GitHub à utiliser : <compte>
- Spécification : <chemin ou URL de la spec, ex. docs/specs/espace-partenaire.md
  sur la branche main, ou page Notion <id>>

Procédure : lis l'en-tête spec-meta de la spec, vérifie si la révision est due
selon sa cadence ; si oui, analyse les commits depuis commit_reference sur les
chemins déclarés, mets à jour les sections concernées en place, complète le
journal des révisions, et livre selon la destination déclarée (branche + PR,
jamais de push direct sur main). Si la révision n'est pas due ou qu'aucun
commit ne touche le périmètre, dis-le en une phrase et arrête-toi.

Ne saisis jamais d'identifiants ; si une vérification en production est
nécessaire et qu'une connexion est requise, note les points à vérifier dans
le résumé plutôt que de tenter de te connecter.

Termine par un résumé de trois lignes : commits analysés, changements
fonctionnels retenus, prochaine échéance.
```

Fréquence conseillée de la tâche : hebdomadaire, un matin de semaine hors heures de pointe. La skill s'arrête d'elle-même quand la cadence de la spec n'est pas atteinte, donc une seule tâche hebdomadaire peut porter une spec mensuelle et une spec trimestrielle (une tâche par spec, ou une tâche qui liste plusieurs specs).

## 8. Plusieurs specs, un même dépôt

Quand un monorepo porte plusieurs services spécifiés (espace partenaire, observatoire, API…), chaque spec a son propre en-tête et ses propres chemins. Un commit qui touche un dossier partagé apparaîtra dans le diff de plusieurs specs : c'est voulu. Le trier indépendamment pour chacune, car son effet fonctionnel peut différer d'un service à l'autre.
