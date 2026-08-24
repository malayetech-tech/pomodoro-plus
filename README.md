# Pomodoro+ 🍅

Une application de minuteur Pomodoro en un seul fichier, en français. Tout — HTML, CSS et JavaScript — vit dans [index.html](index.html). Pas de build, pas de dépendances, pas de suite de tests : il suffit d'ouvrir le fichier dans un navigateur.

## Utilisation

- **Lancer l'app** : ouvrir `index.html` directement dans un navigateur, ou servir le dossier (par ex. `npx serve .` ou `python -m http.server`) si vous avez besoin d'une origine autre que `file://` (nécessaire pour un comportement cohérent de l'API Notifications selon les navigateurs).
- **Modifier** : éditer `index.html` et recharger le navigateur — il n'y a ni compilation ni rechargement à chaud.

## Fonctionnalités

- Trois modes : Focus (25 min), Pause courte (5 min), Pause longue (15 min).
- Anneau de progression SVG animé, couleur d'accent qui change selon le mode actif.
- Liste de tâches avec compteur de pomodoros par tâche.
- Statistiques : sessions du jour, série de jours consécutifs, total.
- Pause longue automatique toutes les 4 sessions Focus complétées dans la journée.
- Chime sonore (Web Audio API) et notification navigateur à la fin d'une session.
- Toutes les données restent dans le navigateur (`localStorage`), aucun backend.

## Architecture

Tout est dans un seul fichier, organisé de haut en bas :

1. **`<style>`** — thème via des variables CSS personnalisées sur `:root` (`--bg`, `--accent`, `--focus`/`--short`/`--long`, etc.), avec une variante `prefers-color-scheme: dark`. Chaque mode du minuteur définit aussi une variable `--mode-color` (en JS) qui se propage aux boutons et à l'anneau de progression.
2. **`<body>`** — structure HTML statique (sélecteur de mode, anneau SVG, ligne de statistiques, liste de tâches) que le JS remplit ; pas de templates côté client.
3. **`<script>`** — une seule IIFE (`(function () { "use strict"; ... })()`) contenant toute la logique. Points clés :
   - **`state`** — un objet unique qui contient tout : `mode` courant (`focus`/`short`/`long`), `remaining` en secondes, `running`, `endAt` (timestamp absolu pour survivre au throttling/passage en arrière-plan de l'onglet), `tasks[]`, `activeTaskId`, et `stats` (`total` + compteurs par jour dans `dates`). Chargé via `load()` et persisté via `save()` dans `localStorage` sous la clé `"pomodoroPlusState"`.
   - **Boucle de rendu** — chaque mutation de state est suivie d'un appel à `render()` (ou aux fonctions plus ciblées `renderStats()`/`renderTasks()`), qui redérive tout le DOM à partir de `state`. Pas de diffing : `renderTasks()` vide et reconstruit `#taskList` à chaque appel.
   - **Mécanique du minuteur** — `startTick()` fixe `state.endAt = Date.now() + remaining*1000` et interroge toutes les 250 ms via `tick()`, en recalculant `remaining` à partir de l'horloge murale plutôt qu'en décrémentant un compteur, pour rester correct même onglet throttlé ou en arrière-plan. Au chargement, une session en cours (`state.running && state.endAt`) est reprise ou complétée selon le temps écoulé.
   - **Fin de session** (`completeSession()`) — à la fin d'une session `focus`, incrémente le compteur du jour et le total, crédite la tâche active, et passe automatiquement en pause `long` toutes les 4 sessions focus complétées dans la journée, sinon en pause `short`. La fin d'une pause ramène toujours en `focus`.
   - **Retours utilisateur** — `playChime()` synthétise un carillon à trois notes avec la Web Audio API (aucun fichier audio), et `notify()` déclenche une notification navigateur si la permission a déjà été accordée ; la permission est demandée au premier clic sur le bouton de démarrage.

## Conventions

- Aucun framework ni bibliothèque externe — API DOM natives uniquement.
- Les textes de l'interface sont en français.
- Toute la persistance est côté client (`localStorage`) ; pas de backend.
