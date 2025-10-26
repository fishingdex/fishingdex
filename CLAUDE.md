# CLAUDE.md - Guide de développement FishingDex

## Vue d'ensemble du projet

**FishingDex** est une application mobile de compagnonnage pour les pêcheurs, construite avec React Native/Expo. Ce dépôt Git contient actuellement le **site web marketing** (landing page et documentation légale) pour l'application mobile principale.

### Informations clés
- **Type de projet** : Site web statique (HTML/CSS/JS) + Documentation d'application mobile (React Native)
- **Statut** : En développement actif ("Coming Soon")
- **Plateformes cibles** : iOS et Android (React Native) + Web (GitHub Pages)
- **Langue principale** : Français (avec support anglais prévu)
- **Licence** : MIT
- **Contact** : contact@fishingdex.app | support@fishingdex.fr

---

## Structure du projet

```
/home/user/fishingdex/
├── README.md                    # Documentation principale du projet (168 lignes)
├── index.html                   # Landing page marketing (108 lignes)
├── privacy-policy.html          # Politique de confidentialité RGPD (508 lignes)
├── terms-of-service.html        # Conditions d'utilisation (354 lignes)
├── styles.css                   # Styles CSS unifiés (503 lignes)
├── CLAUDE.md                    # Ce fichier - Guide développeur
└── .git/                        # Contrôle de version Git
```

**Total** : ~1,641 lignes de code/contenu (hors CLAUDE.md)

---

## Architecture de l'application mobile

### Technologies utilisées

**Frontend Mobile (Application principale - non dans ce repo)**
- **React Native** - Framework cross-platform
- **Expo** - Plateforme de développement et déploiement
- **TypeScript** - Langage avec typage statique
- **AsyncStorage/MMKV** - Stockage local des préférences
- **SQLite** - Base de données locale

**Backend & Services**
- **Supabase** - Backend cloud (PostgreSQL, Auth, Real-time)
  - Serveurs EU (Irlande) - Conforme RGPD
  - Chiffrement TLS 1.3 (transit) + AES-256 (repos)
  - Row-Level Security (RLS) activée
- **OpenMeteo API** - Données météorologiques et indices de pêche
- **Apple Sign In** - Authentification iOS native
- **Google OAuth** - Authentification Android

**Web (Ce repository)**
- HTML5 sémantique
- CSS3 moderne (variables CSS, dark mode)
- JavaScript vanilla (interactions simples)
- GitHub Pages (hébergement)

### Modèle de données

#### 1. Profil utilisateur (Optionnel)
```typescript
{
  email?: string,          // Email optionnel
  username: string,        // Nom d'utilisateur
  avatar?: string,         // URL de l'avatar
  theme: 'light' | 'dark', // Préférence de thème
  language: 'fr' | 'en',   // Langue d'interface
  units: 'metric' | 'imperial', // Système de mesure
  notifications: boolean   // Préférences de notifications
}
```

#### 2. Enregistrement de capture
```typescript
{
  id: string,
  speciesId: string,       // Référence à l'espèce
  weight?: number,         // Poids (optionnel)
  length?: number,         // Longueur (optionnelle)
  photos: string[],        // URLs ou chemins locaux
  location?: {             // GPS optionnel
    lat: number,
    lng: number
  },
  datetime: Date,          // Date et heure de capture
  notes?: string,          // Notes personnelles
  weather?: WeatherData    // Conditions météo au moment de capture
}
```

#### 3. Espèce de poisson
```typescript
{
  id: string,
  name: string,            // Nom commun
  scientificName: string,  // Nom scientifique
  description: string,     // Description
  photos: string[],        // Images de référence
  habitat: string,         // Informations d'habitat
  tips?: string            // Conseils de pêche
}
```

#### 4. Spot de pêche favori
```typescript
{
  id: string,
  name: string,            // Nom du spot
  coordinates: {           // Coordonnées GPS
    lat: number,
    lng: number
  },
  notes?: string,          // Notes et observations
  access?: string,         // Informations d'accès
  catches: string[]        // IDs des captures à ce spot
}
```

#### 5. Conditions météorologiques
```typescript
{
  location: { lat: number, lng: number },
  temperature: number,     // Température air
  feelsLike: number,       // Température ressentie
  wind: {
    speed: number,
    direction: number
  },
  humidity: number,        // Taux d'humidité
  precipitation: number,   // Probabilité de pluie
  fishingQuality: number   // Indice qualité de pêche (calculé)
}
```

---

## Fonctionnalités principales

### 1. Encyclopédie de poissons
- Base de données de 100+ espèces
- Photos et descriptions détaillées
- Aide à l'identification
- Système de favoris

### 2. Journal de captures
- Capture photo (appareil ou galerie)
- Enregistrement espèce, poids, taille
- GPS automatique (si autorisé)
- Horodatage
- Notes personnelles
- Métadonnées météo

### 3. Tableau de bord
- Statistiques personnalisées
  - Nombre total de prises
  - Espèces capturées
  - Records personnels
- Graphiques d'évolution
- Météo en temps réel
- Suggestions de spots

### 4. Gestion des spots
- Sauvegarde illimitée
- Coordonnées GPS précises
- Notes d'accès
- Historique des captures par spot
- Accès rapide

### 5. Météo et conditions
- Prévisions 7 jours
- Température (air + ressenti)
- Vent (vitesse + direction)
- Humidité
- Précipitations
- **Indice de qualité de pêche** (AI)
- Phases lunaires
- Prédictions de marée
- Meilleures heures de pêche

### 6. Personnalisation
- Thème sombre/clair
- Multilingue (FR/EN)
- Unités métriques/impériales
- Gestion des notifications
- Paramètres de confidentialité

### 7. Synchronisation cloud (optionnelle)
- Création de compte optionnelle
- Sync multi-appareils
- Sauvegarde cloud
- Période de récupération 30 jours après suppression

---

## Architecture et stratégies

### Approche "Offline-First"

L'application fonctionne en mode **local-first** :
- **Par défaut** : Tout stocké localement (SQLite, AsyncStorage)
- **Optionnel** : Synchronisation cloud via Supabase
- Aucun compte requis pour utiliser l'app
- Les fonctionnalités principales marchent sans connexion

### Flux de données

```
┌─────────────────────────────────────────┐
│        Application Mobile               │
│     (React Native + Expo)               │
└─────────────┬───────────────────────────┘
              │
    ┌─────────┴─────────┐
    │                   │
    ▼                   ▼
┌───────────┐     ┌──────────────┐
│  Local    │     │  Cloud Sync  │
│  Storage  │     │  (Optionnel) │
├───────────┤     ├──────────────┤
│ SQLite    │     │  Supabase    │
│ AsyncStore│     │  PostgreSQL  │
│ Photos    │     │  Auth        │
└───────────┘     │  Real-time   │
                  └──────┬───────┘
                         │
                    ┌────┴─────┐
                    │          │
                    ▼          ▼
              ┌─────────┐  ┌─────────┐
              │OpenMeteo│  │  OAuth  │
              │   API   │  │ (Apple/ │
              │ Weather │  │ Google) │
              └─────────┘  └─────────┘
```

### Gestion des permissions

| Permission     | Utilisation                  | Obligatoire | Timing           |
|----------------|------------------------------|-------------|------------------|
| Localisation   | GPS spots, météo             | Non         | À la demande     |
| Appareil photo | Capturer photos de prises    | Non         | À la demande     |
| Galerie photos | Sélectionner photos existantes| Non         | À la demande     |
| Stockage       | Sauvegarder données locales  | Non         | Premier lancement|

**Principe** : Demande au moment de l'utilisation, jamais en arrière-plan.

---

## Conformité et sécurité

### RGPD & CCPA

**Droits utilisateurs implémentés** :
- ✅ Droit d'accès (via paramètres app)
- ✅ Droit de rectification (édition dans paramètres)
- ✅ Droit à l'effacement (suppression compte + 30j récup)
- ✅ Droit à la portabilité (export JSON)
- ✅ Droit d'opposition (désactivation compte optionnel)

**Autorité de contrôle** : CNIL (France)

**Responsable de traitement** :
- Entité : FishingDex, France
- Contact : support@fishingdex.fr
- DPO : Non spécifié (peut ne pas s'appliquer selon taille)

### Sécurité

**Mesures techniques** :
- HTTPS/TLS 1.3 pour toutes communications
- Hachage bcrypt des mots de passe
- Chiffrement AES-256 au repos (cloud)
- Row-Level Security (RLS) Supabase
- Certification SOC 2 de l'infrastructure
- Monitoring 24/7 des accès non autorisés

**Confidentialité** :
- Pas de tracking publicitaire
- Pas de cookies marketing
- Pas de géolocalisation en arrière-plan
- Pas d'analytics individuels (seulement agrégés/anonymisés)
- Données sur l'appareil par défaut

**Rétention des données** :
- Compte actif : Tant que le compte existe
- Après suppression : 30 jours (récupération)
- Rapports de crash : 90 jours max
- Analytics anonymisées : 90 jours max

**Notification de violation** : 72h (conformité RGPD)

---

## Guide de développement

### Workflow Git

**Branches** :
```
main                     → Production (GitHub Pages)
development              → Testing/staging
claude/*                 → Feature branches (développement avec Claude)
```

**Branche actuelle** : `claude/create-project-documentation-011CUWgAk5wdjiydMaovw3m1`

**Commits récents** :
```
ad466b3 (HEAD) Update data controller name in privacy policy
bde1892        Update publisher name in privacy policy
d7b1241        Update contact email in footer
1623e12        Update footer contact and GitHub links
d919143        Add landing page and legal pages
```

### Conventions de commit

Utiliser des messages clairs et descriptifs :
```
✅ Bon : "Update contact email in footer"
✅ Bon : "Add landing page and legal pages"
❌ Mauvais : "fix stuff"
❌ Mauvais : "update"
```

### Standards de code

**HTML** :
- Utiliser HTML5 sémantique (`<header>`, `<section>`, `<article>`, `<footer>`)
- Attributs `lang` appropriés
- Meta tags pour responsive et SEO
- Accessibilité (attributs `alt`, rôles ARIA si nécessaire)

**CSS** :
- Variables CSS pour les couleurs et thèmes
- Mobile-first (responsive design)
- Support dark mode via variables CSS
- Classes descriptives (BEM-like si applicable)
- Commentaires pour sections majeures

**JavaScript** :
- Vanilla JS pour interactions simples
- Pas de dépendances lourdes
- Code commenté si logique complexe

### Variables CSS importantes

Le fichier `styles.css` utilise des variables CSS pour faciliter la maintenance :

```css
:root {
  --primary-color: #...;
  --background-color: #...;
  --text-color: #...;
  /* Vérifier styles.css pour la liste complète */
}

/* Support dark mode via media query */
@media (prefers-color-scheme: dark) {
  :root {
    /* Variables redéfinies pour dark mode */
  }
}
```

### Routes et pages

**Pages actuelles** :
- `/` → `index.html` - Landing page principale
- `/privacy-policy.html` → Politique de confidentialité
- `/terms-of-service.html` → Conditions d'utilisation

**Pages mobiles inférées** (dans l'app React Native) :
- `/home` ou `/dashboard` - Tableau de bord
- `/encyclopedia` - Encyclopédie poissons
- `/catches` - Journal de captures
- `/spots` - Spots favoris
- `/weather` - Météo et conditions
- `/profile` - Profil utilisateur

---

## Déploiement

### Web (GitHub Pages)

**Processus actuel** :
1. Push sur `main` → Auto-deploy GitHub Pages
2. Pas de build process nécessaire (HTML/CSS/JS statiques)
3. URL : https://fishingdex.github.io/fishingdex/ (ou domaine custom)

### Mobile (À venir)

**iOS** :
- Déploiement via Expo EAS Build
- Publication App Store (à venir)

**Android** :
- Déploiement via Expo EAS Build
- Publication Google Play (à venir)

---

## Points d'attention importants

### 1. Conformité légale

⚠️ **CRITIQUE** : Les fichiers `privacy-policy.html` et `terms-of-service.html` sont des documents **légaux**.

**Lors de modifications** :
- Vérifier la conformité RGPD/CCPA
- Mettre à jour la date de "Dernière mise à jour"
- Notifier les utilisateurs si changements substantiels
- Archiver les versions précédentes si nécessaire

### 2. Informations de contact

**Emails actuels** :
- Contact général : `contact@fishingdex.app`
- Support : `support@fishingdex.fr`

**Localisation** :
- Ces emails apparaissent dans plusieurs fichiers
- En cas de changement : rechercher et remplacer dans tous les fichiers

### 3. Liens GitHub

**Repository** : `https://github.com/fishingdex/fishingdex`

Utilisé dans :
- `index.html` (footer)
- `README.md` (liens issues/discussions)
- `privacy-policy.html`
- `terms-of-service.html`

### 4. Statut "Coming Soon"

L'application est en développement actif. Badges actuels :
```markdown
[![iOS](https://img.shields.io/badge/iOS-Bientôt-999999...)]
[![Android](https://img.shields.io/badge/Android-Bientôt-999999...)]
```

**Quand l'app sera publiée** :
- Mettre à jour les badges
- Ajouter liens vers App Store / Google Play
- Mettre à jour la section "Téléchargement" dans index.html et README.md

### 5. Captures d'écran

Section "Aperçu de l'application" dans README.md et index.html :
```html
> _Captures d'écran à venir_
```

**À faire ultérieurement** :
- Ajouter vraies captures d'écran de l'app mobile
- Créer dossier `/assets/screenshots/` si nécessaire
- Optimiser images pour le web (compression, formats modernes)

### 6. Multilingue

**État actuel** :
- Contenu principalement en français
- Mention "multilingue FR/EN" dans features
- Pas encore de version anglaise complète des pages web

**Pour support anglais** :
- Considérer structure `/en/` pour pages anglaises
- Ou détection langue navigateur + redirection
- Ou sélecteur de langue dans header

---

## Tâches courantes

### Modifier le texte de la landing page

```bash
# Éditer le fichier
vim index.html

# Vérifier changements
git diff index.html

# Commit
git add index.html
git commit -m "Update landing page copy"

# Push vers la branche actuelle
git push -u origin claude/create-project-documentation-011CUWgAk5wdjiydMaovw3m1
```

### Modifier les styles

```bash
# Éditer les styles
vim styles.css

# Tester dans navigateur (ouvrir index.html)

# Commit
git add styles.css
git commit -m "Update CSS styles for dark mode"
git push -u origin <branch-name>
```

### Mettre à jour la politique de confidentialité

⚠️ **ATTENTION** : Document légal sensible

```bash
# Éditer
vim privacy-policy.html

# Mettre à jour la date "Dernière mise à jour"
# Vérifier conformité RGPD

# Commit avec message descriptif
git add privacy-policy.html
git commit -m "Update privacy policy: [décrire le changement]"
git push -u origin <branch-name>
```

### Ajouter une nouvelle page

```bash
# Créer nouvelle page HTML
touch nouvelle-page.html

# Copier structure depuis index.html
# Adapter le contenu

# Ajouter lien dans navigation (si applicable)
# Ajouter dans README.md si nécessaire

# Commit
git add nouvelle-page.html
git commit -m "Add [nom page] page"
git push -u origin <branch-name>
```

---

## FAQ pour développeurs IA (Claude)

### Puis-je créer de nouveaux fichiers ?

✅ Oui, mais **préférer éditer les fichiers existants** autant que possible.
- Nouveau fichier HTML : OK si nouvelle page nécessaire
- Nouveau fichier CSS : ❌ Utiliser `styles.css` existant
- Nouveau fichier JS : OK si fonctionnalité complexe nécessaire
- Fichiers légaux : ⚠️ Demander confirmation avant création

### Comment gérer les branches ?

La branche actuelle est automatiquement spécifiée :
```
claude/create-project-documentation-011CUWgAk5wdjiydMaovw3m1
```

**Workflow** :
1. Développer sur cette branche
2. Commit régulièrement avec messages clairs
3. Push vers cette branche : `git push -u origin <branch-name>`
4. Créer PR vers `main` quand prêt

### Quels fichiers ne jamais modifier automatiquement ?

🔒 **Fichiers sensibles** :
- `privacy-policy.html` - Document légal
- `terms-of-service.html` - Document légal
- `.git/` - Ne jamais toucher manuellement

**Demander confirmation** avant modifications majeures de ces fichiers.

### Comment tester les changements ?

**Pour le web** :
1. Ouvrir `index.html` dans un navigateur
2. Tester responsive (DevTools mobile view)
3. Tester dark mode (DevTools préférences système)
4. Valider HTML (validator.w3.org si possible)
5. Vérifier tous les liens

**Pour l'app mobile** (non dans ce repo) :
- Utiliser Expo Go
- Tests iOS Simulator / Android Emulator

### Y a-t-il des dépendances à installer ?

**Pour ce repository (web)** : ❌ Non
- HTML/CSS/JS vanilla
- Pas de npm, pas de build process
- Prêt à déployer tel quel

**Pour l'app React Native** : ✅ Oui
- Node.js / npm
- Expo CLI
- React Native dependencies
- (Mais ce code n'est pas dans ce repository)

### Où trouver plus d'informations ?

- **README.md** : Documentation utilisateur et features
- **privacy-policy.html** : Détails techniques sur les données et services
- **terms-of-service.html** : Conditions d'utilisation et responsabilités
- **Ce fichier (CLAUDE.md)** : Guide développeur complet

---

## Ressources externes

### Documentation technique

- **React Native** : https://reactnative.dev/docs/getting-started
- **Expo** : https://docs.expo.dev/
- **Supabase** : https://supabase.com/docs
- **OpenMeteo API** : https://open-meteo.com/en/docs

### Conformité et légal

- **RGPD** : https://www.cnil.fr/fr/reglement-europeen-protection-donnees
- **CCPA** : https://oag.ca.gov/privacy/ccpa
- **CNIL** (Autorité française) : https://www.cnil.fr/

### Outils utiles

- **W3C Validator** : https://validator.w3.org/
- **GitHub Pages** : https://pages.github.com/
- **Expo Status** : https://status.expo.dev/
- **Supabase Status** : https://status.supabase.com/

---

## Changelog de ce document

| Date       | Version | Changements                              |
|------------|---------|------------------------------------------|
| 2025-10-26 | 1.0.0   | Création initiale du CLAUDE.md           |

---

## Contact projet

Pour toute question sur ce projet :

- **Email général** : contact@fishingdex.app
- **Support technique** : support@fishingdex.fr
- **GitHub Issues** : https://github.com/fishingdex/fishingdex/issues
- **GitHub Discussions** : https://github.com/fishingdex/fishingdex/discussions

---

**Dernière mise à jour** : 26 octobre 2025
**Mainteneur** : Équipe FishingDex
**Licence** : MIT
