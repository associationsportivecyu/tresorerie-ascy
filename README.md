# Trésorerie AS CY

Site de trésorerie partagé de l'AS CY, utilisable depuis un téléphone ou un ordinateur.

- **`index.html`** : la page publiée avec GitHub Pages. Elle ne contient aucune donnée.
- **`apps-script/Code.gs`** : le code Google Apps Script qui vérifie la connexion et les droits, et lit ou écrit les données.
- **`test/`** : banc d'essai local, avec de faux services Google. Il n'est pas nécessaire en ligne.

## Fonctionnement

1. Chaque personne se connecte avec son compte Google. Seules les adresses de l'onglet **Accès** peuvent entrer.
2. Les saisies vont dans l'onglet **Saisies** de la Google Sheet « Trésorerie AS CY », et les justificatifs dans un dossier Drive privé.
3. Tout le monde voit toutes les saisies et leur auteur. Le site se met à jour toutes les minutes.
4. Un membre modifie ou supprime seulement ses saisies. Un admin peut tout corriger et gère les accès.
   Une saisie d'un membre reste « À valider » jusqu'à la validation d'un admin, et le redevient si le membre la modifie. Elle compte déjà dans les totaux.
   Dans l'onglet **Accès**, un trésorier (rôle admin) a tout. Pour un membre, des cases accordent les onglets Événements, Bilan et Banque, ainsi que la validation et la modification de toutes les saisies. Elles sont gardées dans la colonne E de l'onglet Accès. Masquer Bilan ou Banque ne cache pas les saisies, que tout le monde voit.
5. Une saisie supprimée reste dans la feuille, avec la date de suppression. L'onglet **Historique** garde chaque action.

## Agent des cotisations

La fonction `synchroniserCotisations` tourne chaque heure (déclencheur du projet Apps Script). Pour chaque inscrit dont le « Paiement vérifié » est à « Oui » dans la feuille d'inscriptions, elle ajoute une recette de 15 €, une seule fois. Si un paiement disparaît, elle ne supprime rien et ajoute une ligne « À vérifier » dans l'historique. Les réglages sont en haut de `Code.gs`.

Pour éviter le double comptage, le site refuse une cotisation saisie à la main avec le moyen HelloAsso, ainsi que le virement de HelloAsso vers la banque. Si une cotisation manuelle contient déjà le nom de l'étudiant, les agents n'ajoutent rien et notent « À vérifier » dans l'historique.
## Agent HelloAsso

La fonction `synchroniserHelloAsso` tourne chaque heure (déclencheur du projet Apps Script). Elle lit les paiements de l'association sur l'API HelloAsso et contrôle les saisies, sans rien ajouter ni modifier. C'est l'agent des cotisations qui crée les recettes. Elle envoie un seul mail à ceux qui peuvent valider, et ajoute une ligne « À vérifier » dans l'historique, si elle trouve un paiement sans saisie depuis 24 h, un montant différent, un paiement annulé ou remboursé, ou une saisie HelloAsso sans paiement. Tout va bien : aucun mail. Les paiements et les saisies sont rapprochés par le nom ou par l'adresse e-mail de l'inscription.

Pour la brancher, crée la clé API dans l'espace HelloAsso de l'association, puis ajoute `HELLOASSO_CLIENT_ID` et `HELLOASSO_CLIENT_SECRET` dans **Paramètres du projet → Propriétés du script**. Ne les écris jamais dans `Code.gs`. Sans ces deux propriétés, la fonction ne fait rien.

## Événements

L'onglet **Événements** du site remplit les colonnes A à D de l'onglet « Budget événements » du tableur « Saisies trésorerie AS CYU », jusqu'à la ligne 30, où s'arrêtent ses formules. Tout le monde peut ajouter un événement ; seul un admin le modifie ou le supprime. Une saisie compte pour l'événement choisi dans son formulaire ; sans choix, pour celui dont le mot-clé figure dans sa description. Le script recopie ce réel dans les colonnes J et K de « Budget événements », que lisent les formules F et G. Il recopie aussi les saisies actives dans l'onglet « Saisies du site » de ce tableur, source du Tableau de bord. Le site est la seule source : l'ancien Google Form n'entre plus dans les totaux.

## Saisies récurrentes et rappels

L'onglet **Récurrentes**, réservé aux trésoriers, crée une saisie par mois au jour choisi, depuis l'onglet « Récurrentes » de la Google Sheet. Une fois par semaine, chaque personne reçoit un mail pour ses dépenses sans justificatif, créées depuis 3 à 60 jours. Les deux passent par le déclencheur horaire de `synchroniserCotisations`.

## Tutoriel

Un guide s'ouvre à la prochaine connexion de chaque personne, une seule fois, avec seulement les étapes de ses droits. La colonne F de l'onglet **Accès** retient qui l'a vu, et le bouton **Aide** le rouvre.

## Notification

Chaque heure, `synchroniserCotisations` envoie un seul mail à ceux qui peuvent valider, s'il y a de nouvelles saisies à valider. Cet envoi utilise MailApp : après son ajout, le compte du script doit autoriser l'envoi de mails depuis l'éditeur avant le nouveau déploiement.

## Dossier de subvention

L'onglet **Subventions** suit l'autorisation Bilan et crée un PDF pour un financeur, à partir des saisies de la période ou d'un événement. Il fonctionne dans le navigateur : les informations saisies ne sont pas enregistrées, et les justificatifs restent à joindre à part.

## Lecture des tickets

Quand une photo est choisie pour une nouvelle saisie, Tesseract.js lit le ticket dans le navigateur et complète le montant, la date et la description restés vides. La photo n'est pas envoyée pour cette lecture, mais le lecteur et sa langue sont téléchargés depuis jsDelivr au premier usage.

## Journal de connexion

L'onglet **Historique** note chaque connexion Google, déconnexion et tentative refusée, avec le type d'appareil. Son filtre sépare les connexions des autres actions. Une session encore ouverte (6 h) ne crée pas de nouvelle ligne, et l'adresse IP n'est pas disponible dans Apps Script.

## Sauvegardes

Une fois par semaine, le passage horaire de `synchroniserCotisations` copie la Google Sheet et les événements dans le dossier Drive « Sauvegardes trésorerie AS CY », et garde les 8 dernières. Les justificatifs restent dans leur dossier Drive.

## Rapprochement bancaire

L'onglet **Banque**, réservé aux admins, lit un relevé CSV dans le navigateur, sans l'envoyer ni l'enregistrer. Il associe chaque opération à une saisie Carte, Virement ou Chèque de même montant, à 5 jours près. Le format réel des relevés de l'association n'a pas encore été essayé.

## Mise en place

1. Sur [console.cloud.google.com](https://console.cloud.google.com), avec le compte de l'association, crée un projet, configure l'écran de consentement en mode **Externe** et publie-le en production (sinon seuls les utilisateurs de test peuvent se connecter), puis crée un **ID client OAuth** de type **Application Web**. Dans « Origines JavaScript autorisées », ajoute `https://associationsportivecyu.github.io`.
2. Sur [script.google.com](https://script.google.com), crée un projet et colle `apps-script/Code.gs`. Renseigne `CLIENT_ID` en haut du fichier.
3. Lance la fonction `installer` une seule fois. Elle crée la Google Sheet et le dossier Drive, et ajoute le compte du script comme admin.
4. Va dans **Déployer → Nouveau déploiement → Application Web**, avec « Exécuter en tant que : Moi » et « Accès : Tout le monde ».
5. Copie l'URL obtenue dans `API_URL` et l'ID client dans `CLIENT_ID`, en haut du script de `index.html`.
6. Publie `index.html` et `logo.png` sur GitHub, puis active **Settings → Pages → Deploy from a branch → main / (root)**.

« Accès : Tout le monde » signifie seulement que la page peut joindre le script : chaque action vérifie la session et les droits.

Après toute modification de `Code.gs`, va dans **Déployer → Gérer les déploiements → Modifier → Nouvelle version**. L'URL reste la même.

## Tests

Dans PowerShell, depuis ce dossier :

```
powershell -ExecutionPolicy Bypass -File test/serveur.ps1
```

Ouvre `http://localhost:8765/`, puis lance `bancTests()` dans la console du navigateur.
