# Documentation Technique - CloudDrive Pro

## 1. Présentation Générale

**CloudDrive Pro** est une application web de stockage de fichiers cloud développée avec Node.js/Express et une interface utilisateur moderne. L'application permet aux utilisateurs d'uploader, visualiser, gérer et supprimer des fichiers via une interface intuitive.

### Caractéristiques principales :
- Upload de fichiers avec drag & drop
- Visualisation des fichiers stockés
- Suppression sécurisée de fichiers
- Interface responsive avec thème clair/sombre
- Prévisualisation d'images et PDF
- Statistiques de stockage
- Recherche et tri des fichiers

## 2. Architecture Technique

### Stack Technologique
- **Backend** : Node.js + Express.js
- **Frontend** : HTML5, CSS3, JavaScript vanilla
- **Template Engine** : EJS (Embedded JavaScript)
- **Middleware** : Multer (gestion des uploads)
- **Styling** : Bootstrap 5.3 + CSS personnalisé
- **Bibliothèques UI** :
  - SweetAlert2 (alertes modernes)
  - Font Awesome (icônes)
  - Google Fonts (Poppins, Inter)

### Structure des dossiers
```
project/
├── app.js                 # Serveur principal
├── views/
│   └── index.ejs         # Template principal
├── filestorage/          # Dossier de stockage des fichiers
├── package.json          # Dépendances du projet
└── README.md            # Documentation
```

## 3. Documentation du Backend (`app.js`)

### 3.1 Configuration du Serveur

```javascript
const express = require('express');
const multer = require('multer');
const path = require('path');
const fs = require('fs');

const app = express();
const port = 3000;
```

- **Express** : Framework web pour Node.js
- **Multer** : Middleware pour la gestion des fichiers
- **Path/FS** : Modules Node.js pour la gestion des chemins et fichiers

### 3.2 Configuration de Multer

```javascript
const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, 'filestorage/');  // Dossier de destination
  },
  filename: (req, file, cb) => {
    const fileName = `${Date.now()}-${file.originalname}`;  // Nom unique
    cb(null, fileName);
  },
});

const upload = multer({ storage });
```

- Les fichiers sont stockés dans `filestorage/`
- Les noms des fichiers sont préfixés par un timestamp pour éviter les conflits

### 3.3 Routes API

#### GET `/`
- **Description** : Page d'accueil
- **Réponse** : Rend le template `index.ejs`

#### POST `/upload`
- **Description** : Upload un fichier
- **Middleware** : `upload.single('file')` - gère l'upload d'un seul fichier
- **Redirection** : Vers la page d'accueil après l'upload

#### DELETE `/delete/:fileName`
- **Description** : Supprime un fichier
- **Paramètres** : `fileName` - nom du fichier à supprimer
- **Fonctionnement** :
  - Vérifie l'existence du fichier
  - Supprime le fichier avec `fs.unlinkSync()`
  - Retourne un message de confirmation ou d'erreur

#### GET `/view`
- **Description** : Liste tous les fichiers
- **Réponse** : JSON contenant la liste des fichiers
  ```json
  { "files": ["timestamp-filename.ext", ...] }
  ```

### 3.4 Configuration Statique
```javascript
app.use('/uploads', express.static(path.join(__dirname, 'filestorage')));
```
- Les fichiers sont accessibles via `/uploads/filename`

## 4. Documentation du Frontend (`index.ejs`)

### 4.1 Structure HTML

#### En-tête
- Titre "CloudDrive Pro" avec statistiques en temps réel
- Bouton de changement de thème

#### Section Upload (Colonne gauche)
- Zone de drag & drop
- Bouton de parcours des fichiers
- Statistiques de stockage (images, documents, autres)

#### Section Gestion (Colonne droite)
- Liste des fichiers avec prévisualisation
- Outils de recherche et tri
- Zone de suppression rapide
- Pagination (10 fichiers par page)

### 4.2 Fonctionnalités JavaScript

#### Gestion des Fichiers
```javascript
// Chargement des fichiers
async function loadFiles() {
  const response = await fetch('/view');
  const data = await response.json();
  allFiles = data.files || [];
  updateStatistics();
  renderFileList();
}

// Upload avec drag & drop
function setupDragAndDrop() {
  // Événements dragover, dragleave, drop
}

// Suppression avec confirmation
async function deleteSingleFile(fileName) {
  await Swal.fire({...}); // Confirmation SweetAlert2
  await fetch(`/delete/${fileName}`, { method: 'DELETE' });
}
```

#### Système de Thème
```javascript
function toggleTheme() {
  const html = document.documentElement;
  const currentTheme = html.getAttribute('data-bs-theme');
  const newTheme = currentTheme === 'dark' ? 'light' : 'dark';
  html.setAttribute('data-bs-theme', newTheme);
  localStorage.setItem('theme', newTheme);
}
```

#### Pagination et Recherche
- Affichage de 10 fichiers par page
- Recherche en temps réel
- Tri alphabétique (ascendant/descendant)

### 4.3 Design et UI/UX

#### Thème Sombre/Clair
- Basé sur l'attribut `data-bs-theme` de Bootstrap
- Transition fluide entre les thèmes
- Préférence sauvegardée dans `localStorage`

#### Composants Visuels
- **Cartes** : Effets d'ombre et transitions hover
- **Fichiers** : Icônes colorées par type de fichier
- **Upload Zone** : Feedback visuel pendant le drag & drop
- **Alertes** : Notifications toast avec SweetAlert2

#### Responsive Design
- Adapté pour mobile et desktop
- Grille Bootstrap pour la mise en page
- Menu d'actions adaptatif

## 5. Installation et Déploiement

### 5.1 Prérequis
- Node.js (version 14 ou supérieure)
- npm ou yarn

### 5.2 Installation
```bash
# Cloner le projet
git clone [repository-url]
cd clouddrive-pro

# Installer les dépendances
npm install express ejs multer

# Lancer le serveur
node app.js
```

### 5.3 Configuration
- Port par défaut : 3000
- Dossier de stockage : `./filestorage/`
- Taille maximale des fichiers : Gérée par Multer (défaut : pas de limite)

### 5.4 Accès
- Application : http://localhost:3000
- Fichiers : http://localhost:3000/uploads/[filename]

## 6. Sécurité et Bonnes Pratiques

### Points Forts
1. **Noms de fichiers uniques** : Préfixés par timestamp
2. **Validation côté serveur** : Vérification de l'existence des fichiers avant suppression
3. **Sanitisation** : Encodage URI pour les noms de fichiers
4. **Gestion d'erreurs** : Try/catch sur les opérations asynchrones

### Recommandations d'Amélioration
1. **Authentification** : Ajouter un système de connexion
2. **Limites de taille** : Configurer Multer pour limiter la taille des fichiers
3. **Types de fichiers** : Valider les extensions autorisées
4. **HTTPS** : Utiliser SSL en production
5. **Sauvegarde** : Mettre en place un système de backup

## 7. API Reference

### Endpoints

| Méthode | Endpoint | Description | Paramètres |
|---------|----------|-------------|------------|
| GET | `/` | Page d'accueil | Aucun |
| POST | `/upload` | Upload fichier | `file` (form-data) |
| DELETE | `/delete/:fileName` | Supprimer fichier | `fileName` (URL) |
| GET | `/view` | Lister fichiers | Aucun |

### Exemples d'utilisation

#### Upload avec cURL
```bash
curl -X POST -F "file=@monfichier.pdf" http://localhost:3000/upload
```

#### Suppression avec cURL
```bash
curl -X DELETE http://localhost:3000/delete/1634567890000-monfichier.pdf
```

## 8. Dépannage

### Problèmes Courants

1. **Dossier de stockage inexistant**
   ```bash
   mkdir filestorage
   ```

2. **Permissions d'écriture**
   ```bash
   chmod 755 filestorage
   ```

3. **Port déjà utilisé**
   ```javascript
   // Modifier le port dans app.js
   const port = process.env.PORT || 3001;
   ```

4. **Fichiers trop volumineux**
   ```javascript
   // Ajouter dans la configuration Multer
   const upload = multer({
     storage: storage,
     limits: { fileSize: 10 * 1024 * 1024 } // 10MB
   });
   ```

## 9. Licence et Crédits

- **Framework** : Express.js (MIT)
- **UI Components** : Bootstrap 5.3 (MIT)
- **Icons** : Font Awesome (CC BY 4.0)
- **Alerts** : SweetAlert2 (MIT)

## 10. Support et Contact

Pour les questions techniques :
- Consulter les issues GitHub
- Référence à la documentation Express/Multer
- Vérifier les logs du serveur Node.js

---

*Documentation générée pour CloudDrive Pro - Version 1.0.0*
