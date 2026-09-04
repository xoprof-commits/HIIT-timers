# Minuteur Sport

Application de minuteur d'entraînement par intervalles : créez des presets personnalisés (préparation, échauffement, exercices avec séries/récupération, retour au calme) puis lancez l'entraînement avec décompte visuel et signaux sonores.

## Fichiers du projet

- `index.html` — l'application complète (HTML/CSS/JS, aucune dépendance externe)
- `manifest.json` — permet l'installation sur l'écran d'accueil Android
- `sw.js` — fait fonctionner l'application hors-ligne
- `icon-192.png`, `icon-512.png` — icônes de l'application

Tout est autonome : pas de serveur, pas de connexion internet requise après la première installation.

## Tester rapidement sur votre téléphone (sans conversion)

1. Copiez le dossier complet sur votre téléphone (ou hébergez-le, voir plus bas), puis ouvrez `index.html` dans Chrome.
2. Menu Chrome (⋮) → **Ajouter à l'écran d'accueil**.
3. L'app s'ouvre alors en plein écran comme une application native, avec son icône.

⚠️ Pour que l'installation et le mode hors-ligne fonctionnent correctement, les fichiers doivent être servis via **http(s)://**, pas ouverts directement en `file://`. Le plus simple est d'utiliser un hébergement gratuit (voir ci-dessous) ou un petit serveur local :
```
cd minuteur-sport
python3 -m http.server 8080
```
puis ouvrir `http://<IP-du-PC>:8080` depuis le téléphone (même réseau Wi-Fi).

## Obtenir un vrai fichier `.apk`

La génération d'un `.apk` nécessite le SDK Android, qui n'est pas disponible dans cet environnement. La méthode la plus simple et gratuite :

### Étape 1 — Héberger les fichiers en ligne
Déposez le contenu de ce dossier sur un hébergement gratuit, par exemple :
- **GitHub Pages** (créez un dépôt, activez Pages)
- **Netlify Drop** : [app.netlify.com/drop](https://app.netlify.com/drop) — glissez-déposez le dossier, un lien est généré instantanément

### Étape 2 — Générer l'APK avec PWABuilder
1. Allez sur [www.pwabuilder.com](https://www.pwabuilder.com)
2. Collez l'URL de votre app hébergée
3. Cliquez sur **Start** puis **Package for stores** → choisissez **Android**
4. Téléchargez le `.apk` (ou `.aab` pour le Play Store) généré

### Étape 3 — Installer sur votre téléphone
Transférez le `.apk` sur votre téléphone Android et ouvrez-le (autorisez l'installation depuis des sources inconnues si demandé).

## Fonctionnement de l'app

1. **Accueil** : liste de vos presets, bouton **+** pour en créer un nouveau.
2. **Création — étape 1** : cochez Préparation / Échauffement / Récupération entre exercices / Retour au calme, indiquez le nombre d'exercices, et choisissez l'**ambiance sonore** (Zen ou Intense — bouton ▶ pour écouter avant de choisir).
3. **Configuration détaillée** : molettes heures/minutes/secondes pour chaque durée, puis question **Temps fixe ?** (OUI = un seul réglage travail/récupération/tours pour tous les exercices, NON = réglages propres à chaque exercice), puis nom du preset.
4. **Détail du preset** : résumé de toutes les phases, ambiance sonore choisie, bouton **Démarrer**.
5. **Entraînement** : anneau de décompte coloré par phase, signal sonore et vibration à chaque transition, écran maintenu allumé pendant la séance.

### Ambiances sonores

Les deux ambiances sont toujours précédées des **3 tics d'avertissement** dans les 3 dernières secondes de chaque phase :

- **Zen 🧘** — un bong/cloche chaude et posée (idéal yoga, étirements, retour au calme)
- **Intense 🚨** — un buzzer plein spectre : bruit blanc (toutes les fréquences audio, 20 Hz–20 kHz) superposé à des harmoniques graves/médiums/aiguës, le tout passé dans un compresseur/limiteur qui pousse le niveau au maximum acceptable par les haut-parleurs du téléphone sans distorsion (idéal HIIT, cross-training)

Le choix se fait par preset (chaque entraînement garde son ambiance), avec un bouton d'écoute pour tester avant de valider.

### Économie d'écran pendant l'entraînement

Un bouton **🔋 Éco** est disponible en haut de l'écran d'entraînement (activé par défaut, désactivable à tout moment) :
- Le maintien d'éveil (Wake Lock) empêche toujours la mise en veille complète du téléphone pendant la séance.
- Après **3 secondes sans toucher l'écran**, un voile sombre s'affiche pour réduire la luminosité perçue et la consommation (particulièrement efficace sur écran OLED). Il disparaît instantanément au moindre contact avec l'écran.

⚠️ **Limite technique importante** : il n'existe aucune API web permettant à un site ou une application web (même packagée en `.apk` via PWABuilder, sans code natif) d'agir sur la luminosité réelle du rétroéclairage. Le voile sombre est donc la meilleure approximation possible dans ce cadre. Une vraie baisse de la luminosité matérielle nécessiterait de reconstruire l'app avec un wrapper natif (par exemple Capacitor + un plugin de luminosité), ce qui implique Android Studio et sort du périmètre d'un simple export PWABuilder — n'hésitez pas à demander si vous souhaitez explorer cette voie.

## Notes techniques

- Les presets sont enregistrés localement sur l'appareil (`localStorage`) — ils restent disponibles hors-ligne et d'une session à l'autre une fois l'app installée. *(Dans un simple aperçu de fichier, comme le rendu intégré de Claude.ai, cette sauvegarde peut ne pas persister d'une session à l'autre — c'est normal, tout fonctionne pleinement une fois l'app installée sur le téléphone.)*
- Les signaux sonores utilisent l'API Web Audio (aucun fichier audio externe), donc aucun problème de connexion. Tous les sons passent par un bus commun avec un `DynamicsCompressorNode` qui normalise/limite le niveau final.
- Le verrouillage d'écran (Wake Lock API) empêche la mise en veille pendant l'entraînement si le téléphone le supporte ; le mode Éco y ajoute un voile d'assombrissement logiciel après 3s d'inactivité tactile.
