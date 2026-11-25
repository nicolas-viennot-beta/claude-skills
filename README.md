# claude-skills

Marketplace de plugins Claude Code hébergeant mes skills : produit, documentation et recherche utilisateur pour des services numériques publics.

## Installation

Dans Claude Code :

```
/plugin marketplace add nicolas-viennot-beta/claude-skills
/plugin install mes-skills@claude-skills
```

Chaque skill est aussi utilisable seule : copier son dossier dans le répertoire de skills de Claude, ou importer son `SKILL.md` dans Claude.ai.

## Skills

| Skill | Rôle |
|---|---|
| [`spec-fonctionnelle-github`](plugins/mes-skills/skills/spec-fonctionnelle-github/) | Rédige et maintient la spécification fonctionnelle d'un service à partir de son dépôt GitHub — double niveau de lecture (non-tech + blocs dépliables tech/PM), routine de mise à jour par diff de commits, vérification prod optionnelle |
| [`compte-rendu-reunion`](plugins/mes-skills/skills/compte-rendu-reunion/) | Transforme une transcription ou des notes de réunion en compte rendu structuré (décisions, actions, points en suspens, prochaine réunion) avec un relevé de décisions prêt à envoyer par email |
| [`transcription-recherche-utilisateur`](plugins/mes-skills/skills/transcription-recherche-utilisateur/) | Nettoie une transcription d'entretien de recherche utilisateur : transcription audio en local, corrections de surface, tours de parole, anti-dictionnaire métier partagé, pseudonymisation optionnelle |
| [`synthese-recherche-utilisateur`](plugins/mes-skills/skills/synthese-recherche-utilisateur/) | Produit une synthèse de recherche utilisateur multi-répondants : synthèse par répondant, fiche globale avec fréquences et verbatims, répondants pseudonymisés, chiffres calculés en code |

## Quelle skill pour quel document ?

```
Vous avez…                                  → Skill
──────────────────────────────────────────────────────────────────────
un enregistrement ou des notes              → transcription-recherche-utilisateur
  d'UN entretien de recherche utilisateur       (nettoyage, pas de synthèse)
                                                        │
                                                        ▼
plusieurs entretiens nettoyés, ou l'export  → synthese-recherche-utilisateur
  d'un questionnaire                            (fiche globale + par répondant)

une transcription ou des notes              → compte-rendu-reunion
  de RÉUNION (décisions à prendre)

un dépôt GitHub et un service à documenter  → spec-fonctionnelle-github
```

La différence entre entretien et réunion : dans un entretien, une personne est interrogée pour recueillir ses usages et besoins ; dans une réunion, plusieurs personnes décident. Les deux premières skills refusent les réunions, la troisième refuse les entretiens.

## Garde-fous communs

- Aucune donnée personnelle dans les livrables destinés à circuler : répondants pseudonymisés, captures sans nom ni e-mail.
- Les enregistrements audio sont transcrits en local, jamais envoyés à un service externe sans accord explicite.
- En cas de doute (terme, nom, décision floue), la skill demande plutôt que de supposer.
- Jamais de push direct sur une branche principale : branche + proposition de PR.

## Structure

```
claude-skills/
├── .claude-plugin/marketplace.json
└── plugins/mes-skills/
    ├── .claude-plugin/plugin.json
    └── skills/
        ├── spec-fonctionnelle-github/      (SKILL.md, assets/, references/, evals/)
        ├── compte-rendu-reunion/
        ├── transcription-recherche-utilisateur/
        └── synthese-recherche-utilisateur/
```

## Licence

MIT.
