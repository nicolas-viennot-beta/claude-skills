---
name: transcription-recherche-utilisateur
description: "Nettoyer une transcription d'interview de recherche utilisateur (audio à transcrire en local, ou notes déjà saisies en .csv ou tout autre format) : corrections de surface, tours de parole identifiés, correction via un anti-dictionnaire métier au format partagé, pseudonymisation optionnelle, en conservant le type de fichier et la structure d'origine — pas pour des comptes rendus de réunion."
---

# Nettoyage de transcription — recherche utilisateur

Première étape de la chaîne de recherche utilisateur, avant la synthèse (`synthese-recherche-utilisateur`). Ce skill s'applique uniquement aux **interviews de recherche utilisateur** (entretiens avec des usagers, agents, partenaires, menés dans une démarche de recherche) — jamais à des comptes rendus de réunion interne ou de point projet, qui relèvent de `compte-rendu-reunion`.

Il fusionne deux étapes : ① la mise au propre d'une transcription ou d'une prise de notes, ② la correction via l'anti-dictionnaire métier (noms propres, acronymes, jargon). Il ne résume pas et ne restructure pas : c'est le rôle de l'étape suivante.

## 1. Identifier le type d'entrée

- **Audio brut** (.mp3, .m4a, .wav…) : il faut d'abord produire une transcription.
- **Texte déjà transcrit ou notes** : .csv, .docx, .txt, .md, ou tout autre format contenant du texte d'entretien.

Si le format ou le contenu n'est pas clair (colonnes d'un .csv ambiguës, plusieurs entretiens mélangés, doute sur le fait qu'il s'agisse bien d'une interview et non d'une réunion), demander une clarification rapide avant de commencer.

## 2. Transcrire l'audio si nécessaire — en local uniquement

Un enregistrement d'entretien contient la voix et les propos d'une personne identifiable. Il ne quitte pas l'environnement de travail : **pas d'envoi à un service de transcription externe** (API cloud, site web) sans accord explicite de l'utilisateur, donné en connaissance de cause.

1. Vérifier l'outillage local (`faster-whisper` de préférence, sinon `openai-whisper`) ; l'installer si absent (pip avec `--break-system-packages`, dans les règles réseau de l'environnement). Pour du français, modèle `medium` au minimum, `large-v3` si la machine le permet ; forcer la langue (`language="fr"`) pour éviter les bascules.
2. Produire une transcription brute complète, avec horodatage par segment, et avec identification des locuteurs si l'outil le permet.
3. **Sauvegarder la transcription brute telle quelle** dans un fichier séparé (`<nom>_transcription_brute.txt`) avant toute correction : c'est la trace de référence en cas de doute.
4. Continuer avec cette transcription brute comme entrée des étapes suivantes.

Si aucun outil local n'est disponible et ne peut pas être installé, le dire clairement plutôt que d'improviser une fausse transcription ou de proposer un service externe par défaut.

## 3. Anti-dictionnaire métier

L'anti-dictionnaire est une liste vivante de termes récurrents mal reconnus ou mal orthographiés par la transcription automatique : noms propres (personnes, communes, organisations), acronymes, jargon métier. C'est une ressource partagée entre tous les entretiens d'un même projet : elle doit être retrouvée et enrichie d'un entretien à l'autre, jamais recréée.

### Format

Un fichier CSV, séparateur `;`, encodage UTF-8, une ligne par terme :

```csv
terme_correct;graphies_erronees;type;note
Mon Diag'Mob;mon diagnostique mob|mon diag mob|mondiagmob;produit;nom du service, apostrophe droite
DGITM;DGITN|DG ITM|déjà item;acronyme;Direction générale des infrastructures, des transports et des mobilités
Mérignac;Mérinac|mais rignac;lieu;commune
```

- `type` ∈ `personne`, `organisation`, `lieu`, `acronyme`, `produit`, `jargon`.
- `graphies_erronees` : toutes les variantes rencontrées, séparées par `|`, pour que la correction soit automatisable.
- Un même format pour tous les projets : on peut fusionner, comparer ou réutiliser dans un autre outil.

### Localiser ou créer

1. **Identifier le projet concerné.** Si ce n'est pas évident (nom du fichier, dossier source, conversation), demander à l'utilisateur à quel projet rattacher l'entretien : c'est ce qui dit où chercher l'anti-dictionnaire et où le ranger.
2. **Chercher un anti-dictionnaire existant** pour ce projet : à proximité des fichiers sources (`anti-dictionnaire.csv` dans le même dossier ou un dossier de référence, quand un dossier de l'ordinateur est connecté) ; à l'emplacement que l'utilisateur a déjà indiqué (mémoire, conversation précédente) ; à défaut, lui demander s'il a gardé le fichier livré lors d'un nettoyage précédent.
3. **Trois cas :**
   - **Il existe** → l'utiliser et le mettre à jour (nouvelles entrées), ne pas en créer un second.
   - **Il n'existe pas et un dossier de projet est identifiable** → en créer un au format ci-dessus, l'enregistrer à côté des fichiers sources, le signaler.
   - **Aucun projet identifiable** (fichier isolé, pas de dossier connecté, projet non précisé même après avoir demandé) → ne rien enregistrer par défaut ; livrer l'anti-dictionnaire comme fichier au même titre que la transcription nettoyée, en signalant qu'il n'est rattaché à aucun projet.

### Appliquer et enrichir

- Appliquer les corrections connues.
- Quand un terme qui ressemble à un nom propre, un acronyme ou du jargon est repéré mais absent du dictionnaire, l'ajouter avec la ou les graphies rencontrées. En cas de doute sur l'orthographe correcte (surtout les noms de personnes), demander plutôt que deviner ; à défaut de confirmation, laisser le terme tel quel et le lister dans le résumé final.

## 4. Nettoyage de base (sans restructurer ni résumer)

Corrections de surface uniquement, en préservant le fond et la structure du propos :

- Ponctuation et casse.
- Suppression des hésitations et tics de langage purs (« euh », « du coup » répété, bégaiements de mots), sans supprimer de contenu substantiel, même redondant en apparence.
- Correction des erreurs de transcription évidentes (mots mal reconnus, homophones).
- **Tours de parole** : quand ils sont discernables, les étiqueter de façon constante (`Enquêteur :` / `Répondant :`, ou `R1 :`, `R2 :` s'il y a plusieurs répondants). Un tour de parole incertain est marqué `[locuteur ?]` plutôt qu'attribué au hasard. Sans cette étape, la synthèse en aval mélange questions et réponses.
- **Horodatages et métadonnées** intacts : ils servent à retrouver le passage dans l'audio.
- Pas de reformulation du sens, pas de résumé, pas de réorganisation par thème.

## 5. Pseudonymisation (proposée, jamais imposée)

Au moment de livrer, proposer en une ligne de pseudonymiser le fichier nettoyé : nom du répondant et des tiers cités remplacés par un code (`R1`, `Tiers-A`), le reste inchangé. Si l'utilisateur accepte :

- écrire la correspondance (code, nom, rôle) dans un fichier séparé `correspondance_<nom>.csv`, livré à part et jamais inséré dans la transcription ;
- la transcription brute (section 2) reste non pseudonymisée : c'est la référence ; la pseudonymisation ne s'applique qu'à la version `_clean`.

Quand la pseudonymisation est faite ici, toute la chaîne aval (synthèse, présentation) travaille sur des données déjà sûres.

## 6. Conserver le format et la structure d'origine

Le fichier de sortie est du même type que l'original et garde la même structure :

- **.csv** : mêmes colonnes ; ne nettoyer que les colonnes textuelles, laisser intactes les colonnes non textuelles (horodatages, identifiants, métadonnées).
- **.docx / .txt / .md / autre** : même mise en forme générale (titres, paragraphes, sauts de ligne significatifs), seul le texte est corrigé.
- Ne jamais écraser le fichier source : écrire dans un nouveau fichier (suffixe `_clean`, ou sous-dossier dédié), à côté de l'original.

## 7. Livraison

Suivre les règles de livraison de l'environnement (SendUserFile, écriture dans le dossier connecté si applicable) pour le fichier nettoyé, la transcription brute le cas échéant, et la table de correspondance si pseudonymisation.

Pour l'anti-dictionnaire : s'il a été enregistré dans un dossier de projet, indiquer le chemin et ce qui a été ajouté ; sinon, le livrer comme fichier.

Terminer par un résumé bref : nombre de corrections appliquées, nouveaux termes ajoutés (et où le dictionnaire est rangé), termes restés incertains, tours de parole marqués `[locuteur ?]`, et tout passage qui mériterait une relecture humaine.
