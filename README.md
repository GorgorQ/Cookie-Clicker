# 🍪 Cookie Clicker - Vue 3 Edition

Un jeu de clicker moderne et interactif développé avec Vue 3, Vite et une interface utilisateur élégante. Cliquez sur le cookie pour accumuler des points et débloquer des améliorations !

## 🌐 Démo en Ligne

**🎮 Jouez maintenant :** [https://cookie-clicker-neon.vercel.app/](https://cookie-clicker-neon.vercel.app/)

*Testez l'application directement dans votre navigateur !*

## 🎮 Fonctionnalités

### 🎯 Jeu Principal
- **Cookie Clicker Interactif** : Cliquez sur le cookie géant pour gagner des cookies
- **Système CPS (Cookies Per Second)** : Génération automatique de cookies
- **Animations Fluides** : Effets visuels attrayants avec glow et transitions CSS

### 🔧 Système d'Améliorations
- **10 Types d'Améliorations** : Du simple Cursor aux Alchemy Labs
- **Progression Exponentielle** : Prix qui augmentent avec chaque achat
- **Calcul Automatique du CPS** : Mise à jour en temps réel de la production

### 🏆 Système d'Achievements
- **5 Achievements Débloquables** :
  - Premier Cookie (1 clic)
  - Cliqueur Novice (100 clics)
  - Grand-mère Adoptée (acheter 1 Grand-mère)
  - Industriel (acheter 10 améliorations)
  - Millionnaire (1M cookies)
- **Notifications Animées** : Alertes visuelles lors du déblocage

### 👥 Système Utilisateur
- **Authentification Complète** : Sign In / Sign Up
- **Sauvegarde Persistante** : LocalStorage + JSON utilisateurs
- **Mode Invité** : Jouer sans compte avec sauvegarde temporaire

### 💾 Gestion des Sauvegardes
- **Export/Import** : Sauvegardez vos progrès en fichier
- **Encodage Base64** : Sécurisation des données
- **Reset Complet** : Remise à zéro avec confirmation

### 📊 Classement
- **Ranking Dynamique** : Top players avec stats temps réel
- **Médailles** : 🥇🥈🥉 pour les meilleurs joueurs
- **Mise à jour Automatique** : Classement qui se met à jour en direct

### 🎨 Interface Utilisateur
- **Design Glassmorphism** : Interface moderne avec blur effects
- **Responsive Design** : Adapté à tous les écrans
- **Animations CSS** : Transitions fluides et effets visuels
- **Dark/Light Theme** : Adaptation automatique au thème système

## 🚀 Technologies Utilisées

- **Vue 3** (v3.5.18) - Framework JavaScript moderne
- **Vite** (v7.1.2) - Build tool ultra-rapide
- **CSS3** - Animations et effets visuels avancés
- **LocalStorage** - Persistance des données côté client
- **JSON** - Format de données pour les utilisateurs

## 📁 Structure du Projet

```
Cookie-Clicker/
├── src/
│   ├── components/
│   │   ├── Achievements.vue      # Système d'achievements
│   │   ├── Connection.vue        # Authentification utilisateur
│   │   ├── CookieClicker.vue     # Composant cookie principal
│   │   ├── DisplayName.vue       # Gestion nom d'affichage
│   │   ├── Options.vue           # Gestion sauvegardes
│   │   ├── Ranking.vue           # Classement joueurs
│   │   ├── UpgradeItem.vue       # Item d'amélioration
│   │   └── Upgrades.vue          # Liste des améliorations
│   ├── assets/
│   │   └── image.png            # Image du cookie
│   ├── App.vue                  # Composant principal
│   ├── main.js                  # Point d'entrée
│   └── style.css               # Styles globaux
├── public/
│   └── users.json              # Base de données utilisateurs
├── package.json
├── vite.config.js
└── index.html
```

## 🛠️ Installation

### Prérequis
- Node.js (v16 ou plus récent)
- npm ou yarn

### Installation
```bash
# Cloner le projet
git clone https://github.com/GorgorQ/Cookie-Clicker.git
cd Cookie-Clicker

# Installer les dépendances
npm install

# Lancer en mode développement
npm run dev

# Build pour production
npm run build

# Preview du build de production
npm run preview
```

## 🎯 Comment Jouer

1. **Démarrage** : Ouvrez l'application et commencez à cliquer sur le cookie
2. **Accumulation** : Chaque clic vous donne 1 cookie
3. **Améliorations** : Achetez des améliorations pour automatiser la production
4. **Achievements** : Débloquez des succès en atteignant certains objectifs
5. **Sauvegarde** : Connectez-vous pour sauvegarder votre progression
6. **Classement** : Comparez vos scores avec les autres joueurs

## 💡 Fonctionnalités Techniques

### 🔄 Système de Sauvegarde
- **Auto-save** : Sauvegarde automatique toutes les 10 secondes
- **Sauvegarde Manuel** : Avant fermeture de page
- **Multi-format** : LocalStorage + JSON + Export fichier

### ⚡ Performance
- **Optimisations Vue 3** : Composition API avec réactivité optimisée
- **Throttling** : Limitation des calculs intensifs
- **Lazy Loading** : Chargement optimisé des composants

### 🎨 Interface
- **Glassmorphism** : Effets de verre avec backdrop-filter
- **CSS Grid/Flexbox** : Layout responsive moderne
- **Animations CSS** : Keyframes et transitions optimisées

## 🔧 Configuration

### Variables d'Environnement
Aucune configuration spéciale requise. Le jeu fonctionne directement après l'installation.

### Base de Données Utilisateurs
Le fichier `public/users.json` contient les données des utilisateurs avec des comptes de test :
- **admin** / admin123 (1M cookies)
- **player1** / password (50K cookies)
- **CookieMaster** / cookies123 (15M+ cookies)


## 📄 License

Tous droits réservés. Ce projet est développé dans un cadre académique.

## 👨‍💻 Auteur

**GorgorQ** - [GitHub Profile](https://github.com/GorgorQ)

## 🎓 Projet Académique

Développé dans le cadre du cours "Interface and Designs" - Semestre 7 - EFREI


*Amusez-vous bien en cliquant ! 🍪*
