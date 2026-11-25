---
name: synthese-recherche-utilisateur
description: "Produire une synthèse de recherche utilisateur à partir de retours multi-répondants (export de questionnaire ou plusieurs entretiens compilés) : synthèse par répondant au format choisi à chaque appel, fiche de synthèse globale avec fréquences et verbatims, répondants pseudonymisés, questions fermées calculées en code. Jamais pour des comptes rendus de réunion."
---

# Synthèse de recherche utilisateur

Deuxième étape de la chaîne de recherche utilisateur, après le nettoyage de transcription (`transcription-recherche-utilisateur`). Ce skill s'applique à des fichiers **multi-répondants** : retours collectés via questionnaire (export contenant les réponses de plusieurs répondants) ou compilation de plusieurs entretiens ou notes dans un même fichier. Il ne s'applique jamais à des comptes rendus de réunion interne ou de point d'équipe, qui relèvent de `compte-rendu-reunion`.

La structure s'inspire des bonnes pratiques du secteur (affinity mapping, analyse thématique, enchaînement « constat → preuve → implication ») : chaque thème identifié est appuyé par au moins un verbatim, et chaque friction ou usage significatif est relié, quand c'est pertinent, à une opportunité concrète. L'objectif est de passer de « voici ce qui a été dit » à « voici ce qu'on peut en faire ».

Ce skill ne maintient pas de base agrégée entre les fichiers : chaque appel produit ses livrables de façon autonome.

Les fichiers reçus varient beaucoup d'un appel à l'autre (orientation du tableau, nombre d'onglets, champs déjà synthétisés ou non, présence de questions fermées…). Plutôt que de couvrir tous les cas à l'avance, ce skill privilégie le dialogue : **dès qu'une ambiguïté se présente, poser une question de clarification à ce moment-là, une seule à la fois, plutôt que de deviner ou de grouper plusieurs questions au début.** Avancer avec la réponse, et poser la suivante seulement quand une nouvelle ambiguïté apparaît (typiquement : le périmètre en section 1, puis le format de restitution en section 3).

## 1. Entrée et détection du cas

Accepter un fichier de retours multi-répondants : typiquement un .csv/.xlsx avec une ligne par répondant, mélangeant questions fermées et commentaires libres, ou plusieurs entretiens compilés dans un même document.

Si le fichier ne permet pas d'identifier le contexte (qui a répondu, quand, dans quel cadre), ou s'il n'est pas clair qu'il s'agit de recherche utilisateur et non d'une réunion, demander avant de deviner.

## 2. Protection des répondants

Les retours utilisateurs contiennent souvent des noms, des e-mails, des fonctions précises, parfois des propos sur des tiers. Les livrables sont faits pour circuler ; les données brutes, non.

- **Pseudonymiser par défaut** : les répondants sont désignés R1, R2… dans tous les livrables. Aucun nom de personne privée, aucune adresse e-mail, aucun identifiant individuel n'y figure, sauf demande explicite de l'utilisateur. Le profil (rôle, type de structure, territoire) reste, car c'est ce qui donne du sens aux constats — à condition qu'il ne permette pas à lui seul de reconnaître la personne (un « directeur mobilité d'une métropole de 300 000 habitants » est identifiable ; « cadre d'une grande collectivité » ne l'est pas).
- **Table de correspondance** : si l'utilisateur a besoin de retrouver qui est R3, produire un fichier séparé `correspondance_repondants.csv` (code, identifiant source) livré à part et jamais inséré dans la synthèse.
- **Verbatims** : vérifier que la citation ne nomme pas un tiers ni ne révèle une information privée ; sinon, la tronquer ou en choisir une autre.
- Les noms d'organisations publiques et de dispositifs publics ne sont pas concernés.

## 3. Méthode de synthèse

1. **Coder le matériau** : relire les réponses et repérer les fragments pertinents, en les étiquetant par catégorie (usage, friction, positif, interrogation, opportunité potentielle).
2. **Regrouper par thème** (affinity mapping) : rassembler les fragments qui parlent du même sujet plutôt que de suivre l'ordre du fichier.
3. **Ne pas sur-interpréter une source isolée** : un thème repéré chez un seul répondant est une observation à confirmer, pas un pattern. La fréquence de mention (nombre et proportion de répondants concernés) donne un premier niveau de validation. En dessous d'environ 5 répondants, la fiche globale le dit explicitement en tête : les fréquences sont indicatives et ne permettent pas de hiérarchiser.
4. **Calculer, ne pas estimer** : dès qu'il y a des questions fermées (notes, échelles, choix), les agrégats (moyennes, médianes, répartitions en %) sont calculés en code (pandas ou équivalent), jamais de tête. Les fréquences de mention des thèmes qualitatifs sont comptées de la même façon à partir du codage, pas évaluées à l'œil. C'est le premier endroit où une synthèse se trompe, et une erreur de chiffre discrédite tout le reste.
5. **Rester factuel** : distinguer ce que la personne a dit (verbatim) de ce que Claude en déduit (interprétation), et attribuer clairement les deux.
6. **Associer systématiquement le verbatim** à chaque friction, point positif et interrogation : jamais de constat sans la citation exacte qui l'appuie, dans la synthèse par répondant comme dans la fiche globale.

## 4. Livrables

Pas de fiche par répondant au format imposé. Avant de produire la synthèse par répondant, **proposer 2 ou 3 options de restitution adaptées au fichier reçu** (nombre de colonnes existantes, questions fermées, taille, lisibilité si on ajoute des colonnes…) avec une recommandation argumentée, et attendre le choix. Exemples d'options : compléter le document source par des colonnes de synthèse à la suite de l'existant, sans le réorganiser ; produire un document séparé avec un tableau par répondant ; produire une fiche courte par répondant regroupées dans un seul document. Ce choix est redemandé à chaque appel, car le format source varie d'un fichier à l'autre.

Une fois l'option choisie :

1. **Synthèse par répondant** selon l'option retenue. Chaque friction, point positif ou interrogation est accompagné de son verbatim, dans des colonnes appariées (`frictions` / `frictions_verbatim`, etc.) ou toute présentation qui garde le verbatim collé au constat qu'il illustre — jamais une colonne « verbatim » générique. Si l'option consiste à compléter le document source, écrire dans un nouveau fichier (suffixe `_complete`), sans écraser l'original.
2. **Questions fermées** : agrégées en code (section 3, point 4) et versées dans la fiche globale (« Chiffres clés »).
3. **Fiche de synthèse globale** (structure en section 5) : chaque constat indique sa fréquence de mention (« 6/12 répondants ») et son verbatim.

## 5. Structure de la fiche de synthèse globale

```markdown
# Synthèse globale — [N répondants] — [date / période]

[Si N < 5 : avertissement sur le caractère indicatif des fréquences.]

## Contexte
- Profil des répondants (rôle, type de structure, territoire…), sans identité
- Date/période, cadre de la collecte (projet, objectif de la recherche, nombre de répondants, taux de réponse si connu)

## Chiffres clés (uniquement si questions fermées)
Agrégats calculés : moyennes, médianes, répartition par choix. Indiquer le nombre de répondants à chaque question (les non-réponses changent les pourcentages).

## Résumé (3-5 lignes)
Ce qu'il faut retenir en une lecture rapide.

## Usages dominants
Comment les répondants utilisent (ou non) le produit ou service aujourd'hui — habitudes, contournements, fréquence.
- Constat (fréquence) — *« verbatim »* (R#)

## Points de friction
Ce qui bloque, ralentit ou frustre.
- Friction (fréquence) — *« verbatim »* (R#) → impact pour l'utilisateur

## Points positifs
Ce qui fonctionne bien et doit être préservé.
- Point positif (fréquence) — *« verbatim »* (R#)

## Interrogations exprimées
Doutes, questions ou incompréhensions formulés par les répondants (pas déduits).
- Interrogation (fréquence) — *« verbatim »* (R#)

## Verbatims marquants
Les 3 à 5 citations les plus fortes, pensées pour être réutilisées telles quelles dans une présentation (vérifiées sans donnée identifiante).

## Opportunités
Pistes d'action, chacune reliée explicitement à la friction ou à l'usage dont elle découle, et marquée comme piste à valider, pas comme décision. Quand les données le permettent, indiquer la fréquence de la friction d'origine pour aider à prioriser.

## Tags / thèmes
Mots-clés normalisés, cohérents d'un fichier à l'autre, pour faciliter un rapprochement manuel futur.
```

Garder ces six dimensions (usages, friction, positif, interrogations, verbatims, opportunités) sauf demande explicite. Pour frictions, points positifs et interrogations, le verbatim est obligatoire.

## 6. Livraison

Livrer la synthèse par répondant selon l'option choisie, la fiche de synthèse globale, et le cas échéant la table de correspondance à part. Respecter les règles de livraison de l'environnement (SendUserFile, dossier connecté). Terminer par un court récapitulatif : nombre de répondants, de frictions, de points positifs et d'opportunités identifiés, et le script ou la méthode utilisée pour les chiffres si des questions fermées ont été agrégées.
