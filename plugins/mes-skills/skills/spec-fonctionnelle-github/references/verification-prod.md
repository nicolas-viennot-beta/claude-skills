# Vérification sur le site en production

Le code dit ce qui est *possible* ; la production dit ce qui est *actif*. Un flag résolu au build, un environnement configuré différemment, un déploiement en retard sur la branche principale : autant de raisons pour lesquelles la spec doit distinguer ce qui a été lu de ce qui a été vu. Cette étape est optionnelle, mais elle transforme une spec plausible en spec fiable.

## Préalables

- Une `url_prod` fournie par l'utilisateur et un navigateur disponible dans la session (navigateur intégré ou extension). Charger la skill du navigateur concerné avant la première action.
- L'utilisateur se connecte lui-même si le service exige une authentification. Ne jamais saisir d'identifiants, de codes ou de mots de passe ; dire simplement à l'utilisateur quand la page de connexion est affichée et attendre.
- Préférer la lecture du texte et de la structure de la page (`get_page_text`, `read_page`, ou équivalent) aux captures pour vérifier des libellés et des colonnes ; réserver les captures aux illustrations destinées à la spec.

## Quoi vérifier, par priorité

1. **Les flags et fonctionnalités optionnelles** repérés dans le code. Pour chacun : la fonctionnalité est-elle visible ? L'accès direct par URL est-il bloqué ou redirigé ? Un formulaire propose-t-il les champs attendus ? C'est l'information la plus précieuse, car impossible à déduire du code seul quand le flag est résolu au build.
2. **L'arborescence réelle** : menus, onglets, pages atteignables, et leur variation selon le rôle (si un mécanisme de simulation ou plusieurs comptes de test existent).
3. **Les libellés et colonnes** des écrans principaux : titres d'onglets, en-têtes de tableaux, intitulés de boutons. La spec reprend les libellés tels qu'affichés, pas les noms de variables.
4. **Les comportements côté client** qui ne se voient pas dans le code : un état perdu au rechargement, une redirection, un message d'erreur, un délai.
5. **Les écarts** : toute différence entre le code et la prod se note comme un fait (« présent dans le code au commit X, absent en prod le JJ/MM ») dans la section « Choix réalisés et limites connues » ou « Fonctionnalités en préparation ou désactivées ».

Ne pas chercher à tester les parcours d'écriture (création, modification, suppression) au-delà de l'ouverture du formulaire : annuler systématiquement, ne jamais valider. Une spec n'a pas besoin de créer un enregistrement pour décrire un formulaire.

## Captures d'écran

Une capture sert à montrer la structure d'un écran, pas son contenu. Avant d'en garder une :

- aucune donnée personnelle : ni nom de personne, ni adresse e-mail, ni identifiant individuel, ni l'adresse de l'utilisateur connecté affichée dans un en-tête. En cas de doute, choisir un autre écran, filtrer la liste sur une valeur neutre, ou ne pas mettre de capture ;
- les données d'organisations publiques (noms d'opérateurs, de collectivités, SIRET, intitulés de dispositifs publics) sont acceptables ;
- nommer les fichiers de façon stable (`images/01-accueil.jpg`, `images/02-export.jpg`) pour que la routine de mise à jour puisse les remplacer sans toucher au texte ;
- une capture par fonctionnalité majeure au plus ; la page d'accueil et un écran représentatif de chaque grande section suffisent généralement.

Si une capture déjà prise contient une donnée personnelle, la supprimer et le dire à l'utilisateur plutôt que de la masquer par retouche : une retouche se rate, une suppression non.

## Tracer les vérifications

Pour chaque point vérifié, la spec porte `vérifié en prod le JJ/MM/AAAA` et l'en-tête `spec-meta` enregistre `derniere_verification_prod`. Tenir aussi, dans le résumé livré à l'utilisateur, la liste courte de ce qui a été vérifié et de ce qui ne l'a pas pu (page inaccessible avec le compte disponible, fonctionnalité réservée à un autre rôle, etc.), pour que la prochaine révision sache où reprendre.

## Quand s'arrêter

Si le navigateur ne répond plus, si une page demande une action que seul l'utilisateur peut faire (connexion, captcha, acceptation de conditions), ou si trois tentatives sur un même élément échouent : s'arrêter, dire ce qui a été tenté et demander comment continuer. Une spec partiellement vérifiée, honnête sur ses limites, vaut mieux qu'une session de navigation qui tourne en rond.
