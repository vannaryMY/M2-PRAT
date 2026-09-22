# Gestion automatisée des projets PRAT

Ce dépôt héberge le catalogue des projets PRAT publié via **GitHub Pages**.  
L'ajout de nouveaux sujets est entièrement automatisé dès la soumission du formulaire Google Forms par les encadrants.

---

## 1. Vue d'ensemble du workflow
```
[Google Form]
      │
      ▼ (Soumission)
[Google Sheet]
      │
      ▼ (Déclencheur automatique Apps Script)
[Google Apps Script]
      │
      ├─► Génération et archivage d'une fiche PDF sur Google Drive
      │
      └─► Requêtes HTTP vers l'API REST de GitHub :
            1. Création de la page HTML dédiée du sujet
            2. Mise à jour de la liste dans index.html
      │
      ▼ (Nouveau commit)
[GitHub Pages]
      │ (Build automatique « dynamic »)
      ▼
Site web public mis à jour en direct
```
---

## 2. Composants techniques

### A. Google Form & Google Sheet
* Le formulaire collecte les propositions (titre, encadrant, e-mail, description, etc.).
* Les réponses s'inscrivent dans une feuille Google Sheet liée.

### B. Google Apps Script (`onFormSubmit`)
Accessible directement via le menu **Extensions > Apps Script** du Google Sheet.  
À chaque nouvelle ligne enregistrée dans le tableur, la fonction `onFormSubmit` :
1. Crée un document Google Doc temporaire, le convertit en PDF et l'enregistre dans le dossier Google Drive dédié (`DriveApp`).
2. Nettoie le titre du projet pour forger un nom de fichier sécurisé (ex. `Registration_of_serial_cut_tissue.html`).
3. Compile la page HTML individuelle du sujet selon le gabarit de mise en page.
4. Envoie le fichier sur le dépôt GitHub via un appel API `PUT /repos/{owner}/{repo}/contents/{filename}`.
5. Récupère le contenu de `index.html` via l'API, insère le lien du nouveau sujet juste avant la balise fermante `</ul>`, et committe la mise à jour.

### C. Authentification GitHub
Pour écrire sur ce dépôt sans intervention humaine, le script utilise un **Personal Access Token (PAT)** GitHub avec la permission `repo` (ou `contents:write`).  
Ce token est injecté dans les en-têtes HTTP de chaque requête (`Authorization: Bearer <TOKEN>`). Tous les ajouts apparaissent donc comme committés par le détenteur du token.

### D. Déploiement GitHub Pages
Chaque commit poussé par le script déclenche automatiquement l'action interne de build GitHub Pages (*Triggered via dynamic*). La version en ligne du site est actualisée en moins de deux minutes.

---

## 3. Maintenance et points d'attention

* **Expiration du Personal Access Token (PAT) :**  
  Si le site cesse de s'actualiser malgré les envois de formulaires, vérifiez la date d'expiration du token GitHub. S'il a expiré, régénérez-en un depuis **Settings > Developer settings > Personal access tokens** et collez-le dans la variable `GITHUB_TOKEN` du script Apps Script.
* **Changement d'année universitaire / Nouveau dépôt :**  
  Pour réutiliser ce système l'année prochaine :
  1. Créer le nouveau dépôt avec un fichier `index.html` initial (contenant `<ul></ul>`).
  2. Activer GitHub Pages dans **Settings > Pages** du nouveau dépôt.
  3. Mettre à jour les variables `REPO_NAME` (et `TARGET_BRANCH` si nécessaire) en tête du Google Apps Script.
* **Logs et débogage :**  
  En cas d'erreur de publication, l'historique complet des exécutions et les codes retour de l'API GitHub sont consultables dans le menu **Exécutions** d'Apps Script.

## Contact

Si problèmes me contacter : 
- Théo HARDY : [theo.hardy@edu.devinci.fr](mailto:theo.hardy@edu.devinci.fr)