---
name: compte-rendu-reunion
description: "Génère un compte rendu de réunion (décisions, actions, points en suspens, prochaine réunion) à partir d'une transcription ou de notes brutes, avec un relevé de décisions prêt à envoyer par email — pas pour des synthèses d'interviews de recherche utilisateur."
---

# Compte rendu de réunion

Ce skill transforme une transcription (audio retranscrit, notes de réunion prises en direct, échanges bruts) en un compte rendu structuré, clair et exploitable.

À utiliser uniquement pour des réunions de travail classiques (points d'avancement, comités, réunions de décision, réunions internes ou avec partenaires). Ne pas l'utiliser pour synthétiser des entretiens de recherche utilisateur (interviews, tests utilisateurs) : ces synthèses suivent une structure différente (verbatims, insights, besoins) et relèvent du skill `synthese-recherche-utilisateur`.

## Règle impérative : en cas de doute, demander plutôt que supposer

Cette règle prime sur toutes les autres. Ne jamais deviner, reformuler à l'instinct, ou combler un flou en fabriquant une interprétation plausible. Un compte rendu incomplet mais fiable vaut mieux qu'un compte rendu complet mais inventé. Les cas qui déclenchent une question :

- **Terme ou sigle ambigu ou inconnu** : acronyme, nom de projet, jargon métier ou technique non explicité dans la source, ou qui pourrait avoir plusieurs sens.
- **Nom de personne incertain** : orthographe douteuse, homonymie, prénom seul, ou attribution incertaine d'une prise de parole, d'une action ou d'une décision.
- **Conclusion ou décision floue** : la transcription laisse penser qu'un accord a été pris, mais la formulation pourrait aussi bien être une proposition non tranchée. Ne jamais trancher soi-même une ambiguïté réelle du débat.
- **Contradiction dans la source** : deux passages qui se contredisent sur un fait, un chiffre, une échéance ou une décision.
- **Métadonnées manquantes** : participants, date ou objet de la réunion non identifiables.

## Entrée

La donnée de base peut être :
- une transcription automatique (souvent désordonnée, avec hésitations, répétitions, tours de parole mal identifiés) ;
- des notes de réunion prises à la volée (phrases courtes, abréviations, style télégraphique) ;
- un mélange des deux, ou plusieurs fichiers à consolider.

Si la source ressemble davantage à un entretien de recherche utilisateur (une personne interviewée, questions/réponses, objectif de recueillir des besoins ou retours d'usage plutôt que de prendre des décisions collectives), signaler que ce n'est probablement pas le bon format et demander confirmation avant de continuer.

## Un seul aller-retour avant le livrable

Pour ne pas multiplier les échanges, regrouper en **une seule question** (AskUserQuestion si disponible), avant de rédiger :

1. **Le titre proposé**, au format `AAAA-MM-JJ - [Objet de la réunion]`, où la date est celle du jour (date de rédaction, fournie par l'environnement, jamais devinée) et l'objet un intitulé court tiré de la source (ex. « 2026-09-09 - Comité de pilotage »). Si la réunion a eu lieu à une autre date identifiable dans la source, proposer les deux options (date de la réunion / date de rédaction).
2. **Tous les points à clarifier** relevés par la règle impérative, chacun avec le passage source concerné.

Attendre la réponse, puis rédiger. Si la source ne soulève aucun doute, la question ne porte que sur le titre. Si l'utilisateur a indiqué qu'il n'est pas disponible (tâche planifiée, « fais au mieux »), rédiger avec le titre proposé, marquer chaque point douteux `[à confirmer]` dans le corps, et ne pas produire le bloc email (voir plus bas).

## Sortie

Un document markdown structuré, dans les sections suivantes, toujours dans cet ordre :

```markdown
# [Titre validé]

**Date :** [date de la réunion]
**Participants :** [liste des personnes présentes]
**Absents excusés :** [si mentionné, sinon omettre la ligne]

## Ordre du jour

- [point 1]
- [point 2]

## Points discutés

### [Nom du point 1]
[Synthèse de la discussion — pas un mot à mot, mais les arguments, positions et contexte utiles pour quelqu'un qui n'était pas présent. Factuel et neutre : rapporter ce qui a été dit, pas une opinion sur le fond.]

### [Nom du point 2]

## Décisions prises

- [Décision 1 — formulée de façon actionnable et non ambiguë]

## Actions à mener

| Action | Responsable | Échéance |
|---|---|---|
| [action] | [personne ou équipe] | [date ou « non précisée »] |

## Points en suspens / à trancher

- [Question ouverte, désaccord non résolu, risque identifié, sujet reporté]

## Prochaine réunion

- **Date :** [date ou créneau évoqué, sinon « non fixée »]
- **Points reportés / à mettre à l'ordre du jour :** [liste, sinon « aucun »]

---

## Synthèse à envoyer par email

[Voir ci-dessous]
```

Si une section n'a aucun contenu (aucune action, aucun point en suspens), la garder avec la mention « Aucun » plutôt que de la supprimer : cela confirme que le point a été considéré et pas oublié.

## Synthèse email — relevé de décisions

À la toute fin du document, après un séparateur `---`, un bloc court, autonome et prêt à copier-coller tel quel dans un email. Il doit pouvoir être lu et compris sans le reste du compte rendu.

```markdown
Objet : Relevé de décisions — [Objet de la réunion] — [date]

Bonjour à toutes et tous,

Voici le relevé de décisions de la réunion du [date] sur [objet].

Décisions prises :
- [décision 1]
- [décision 2]

Actions à mener :
- [action] — [responsable] — échéance : [date ou « non précisée »]

Points en suspens :
- [point 1, si applicable — sinon omettre cette sous-partie entièrement]

Prochaine réunion : [date ou « à fixer »]

Le compte rendu complet est disponible en pièce jointe / [lien].

N'hésitez pas à revenir vers moi en cas de question ou de désaccord sur ce qui précède.

Cordialement,
```

Règles pour ce bloc :
- Reprendre uniquement les décisions et actions déjà validées dans le corps du compte rendu ; ne rien y ajouter, ne rien reformuler différemment.
- Rester très court : une ligne par décision ou action, sans développement.
- Omettre la sous-partie « Points en suspens » si elle est vide (dans un email, on va à l'essentiel ; dans le corps du CR, on garde « Aucun »).
- Ne jamais produire ce bloc tant que le compte rendu comporte des points `[à confirmer]` : le relevé de décisions envoyé par email doit être fiable à 100 %. Le remplacer alors par une ligne « *Relevé email à produire une fois les points à confirmer levés.* »

## Règles de rédaction

- Rédiger en français, sauf si la source est dans une autre langue et que l'utilisateur ne précise rien : demander alors la langue de sortie.
- Ne jamais inventer un nom, une échéance ou un chiffre absent de la source. Une action sans responsable ou sans date identifiable est marquée « non précisé ».
- Distinguer clairement une **décision** (quelque chose a été tranché), une **action** (quelque chose reste à faire, avec un porteur) et un **point en suspens** (rien n'a été tranché, ou un désaccord subsiste).
- Condenser : supprimer hésitations, digressions, redites. Garder la substance et les décisions, pas le fil exact de la conversation.
- Laisser de côté ce qui n'a pas sa place dans un document qui circule : apartés personnels, remarques sur des absents, informations privées évoquées en passant.
- Adapter le niveau de formalisme au contexte identifiable (réunion interne d'équipe vs réunion avec un partenaire externe) : factuel dans les deux cas, mais un compte rendu externe demande plus de contexte explicite pour un lecteur qui n'a pas celui du projet.

## Livraison

Produire le compte rendu comme fichier markdown (.md) dont le nom reprend le titre validé (ex. `2026-09-09-comite-pilotage.md`) et le livrer selon les règles de l'environnement (SendUserFile, dossier connecté, ou le format demandé par l'utilisateur : Word, Notion, Docs…). Terminer par une phrase : nombre de décisions, d'actions, de points en suspens, et la liste des `[à confirmer]` s'il en reste.
