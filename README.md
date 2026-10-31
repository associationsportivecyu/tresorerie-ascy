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
5. Une saisie supprimée reste dans la feuille, avec la date de suppression. L'onglet **Historique** garde chaque action.

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
