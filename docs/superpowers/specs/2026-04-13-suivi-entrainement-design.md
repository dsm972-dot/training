# Design — Améliorations Suivi Entraînement
*Denis · Force & Définition — 13 avril 2026*

---

## Contexte

L'application `app_entrainement.html` suit déjà les séances complétées et permet d'enregistrer une charge par exercice. Ce design ajoute 8 fonctionnalités de suivi avancé choisies par Denis, organisées autour d'une décision fondamentale : **enregistrer le poids et les reps réels pour chaque série individuelle**.

---

## Décisions de design

| Question | Choix |
|----------|-------|
| Saisie des charges | Modal slide-up par série (poids + reps réels) |
| Suivi semaine S1→S4 | Auto depuis date de départ + override manuel |
| Graphiques | Dans l'onglet Progrès, sous-tabs internes |
| Fin de séance | Résumé complet (volume, PRs, étoiles, note) |

---

## Architecture des données

### Nouvelles clés `localStorage`

```js
// Session en cours uniquement (réinitialisé à chaque nouvelle séance du même workout)
setLog: {
  "lundi_0_0": { kg: 100, reps: 5 },   // workoutId_exIdx_setIdx
  "lundi_0_1": { kg: 100, reps: 4 },
  ...
}

// Journal historique enrichi (inclut les séries pour les courbes)
sessionLog: {
  "2026-04-13_lundi": {
    duration: 68,           // minutes
    volume: 6820,           // kg total (Σ kg×reps)
    rating: 4,              // étoiles 1-5
    note: "Séance intense", // texte libre optionnel
    prs: ["Développé couché: 105kg×5"],
    sets: {                 // snapshot des séries pour les courbes historiques
      "0_0": { kg: 100, reps: 5 },
      "0_1": { kg: 100, reps: 4 }
    }
  }
}

// Suivi corporel
bodyLog: [
  { date: "2026-04-13", weight: 82.5, waist: 84, arm: 40, thigh: 58 }
]

// Suivi de semaine
programStart: "2026-04-06"   // date lundi de départ du programme
currentWeek: null             // null = auto, 1-4 = override manuel
```

### Données dérivées (calculées à la volée, jamais stockées)

- **Volume** = Σ (kg × reps) sur toutes les séries d'une séance
- **PR** = max de `kg × reps` enregistré dans `setLog` pour un exercice
- **Semaine courante** = `Math.ceil((today − programStart) / 7)` si `currentWeek === null`, sinon `currentWeek`
- **Charge cible** = charge S1 × coefficient de semaine (`S1: ×1.00`, `S2: ×1.05`, `S3: ×1.10`, `S4: ×1.15`). Source : kg moyen des séries S1 dans `sessionLog`. Badge masqué si aucune session S1 enregistrée.
- **`sessionPRs`** = variable de session `let sessionPRs = []` (réinitialisée à `renderSeance()`), accumulée pendant la séance, vidée à `saveSessionSummary()`

---

## Fonctionnalités

### 1. Suivi par série — Modal slide-up

**Déclencheur :** tap sur un bouton de série numéroté.

**Comportement :**
- Le modal remonte du bas de l'écran (animation CSS `transform: translateY`)
- En-tête : `"Série N · [Nom de l'exercice]"`
- Rappel : `"Dernière fois : 100 kg × 5 reps"` (depuis `setLog`)
- Deux champs numériques :
  - **kg** : pré-rempli avec la dernière valeur de `setLog` pour cet exercice (ou 0 si première fois)
  - **reps** : pré-rempli avec la cible reps du programme (ex : `5` pour "4-5")
- Validation → `closeSetModal()` :
  1. Enregistre `setLog["workoutId_exIdx_setIdx"] = { kg, reps }`
  2. Met à jour le bouton de série : couleur de la séance + poids affiché (`100kg`)
  3. Si PR détecté → flash `"🏆 Nouveau record !"` en vert pendant 1.5s
  4. Auto-démarre le timer repos (durée définie sur l'exercice)
  5. Ferme le modal
- Fermeture sans validation : swipe bas ou tap hors du modal → série reste non validée

**Détection PR :**
```js
function detectPR(workoutId, exIdx, kg, reps) {
  const prefix = `${workoutId}_${exIdx}_`;
  const existing = Object.entries(setLog)
    .filter(([k]) => k.startsWith(prefix))
    .map(([, v]) => v.kg * v.reps);
  return (kg * reps) > Math.max(0, ...existing);
}
```

---

### 2. Semaine courante S1→S4

**Sur l'accueil :**
- Bandeau sous la date : `"Semaine 3 — +10% de charge · +1 série"`
- 4 pills `S1 / S2 / S3✓ / S4` tapables pour override manuel
- Tap sur une pill → `currentWeek = N` (null si tap sur la pill déjà active = retour auto)

**Dans chaque séance :**
- Sous le nom de chaque exercice principal : badge discret `"Cible S3 : ~105 kg"`
- Calcul : kg moyen des séries S1 (depuis `sessionLog`) × coefficient semaine
- Badge masqué si aucune session S1 enregistrée pour cet exercice (pas de valeur "0 kg" affichée)

**Page Paramètres (⚙️ sur l'accueil) :**
- Champ date de départ du programme → stocké dans `programStart`
- Bouton "Réinitialiser le programme" (remet `currentWeek` à null et efface `programStart`)

---

### 3. Timer repos auto-démarrage

- Après chaque validation de série → `startAutoRestTimer(ex.rest)` si `ex.rest > 0`
- Affichage : pastille circulaire flottante **bas-droite de l'écran** (position fixed, z-index élevé), non bloquante
- Contenu : décompte `MM:SS` + arc de progression SVG (identique au timer modal existant, format réduit)
- Tap sur la pastille → ouvre le modal de repos existant (synchronisé sur le même décompte)
- Fin du décompte → son existant (`beep()`) + disparition de la pastille
- Si une nouvelle série est validée avant la fin → repart avec la nouvelle durée

---

### 4. Onglet Progrès — 3 sous-tabs

#### Sub-tab "Séances" (existant, enrichi)
- Chaque entrée affiche : icône + nom séance + date + durée + volume + étoiles + badge PRs si applicable
- Clic sur une entrée → expand avec la note de ressenti

#### Sub-tab "Courbes" (nouveau)
- 6 exercices clés tracés : Développé couché, Squat, Soulevé de terre, Tractions lestées, Développé militaire, Rowing buste penché
- Graphique SVG natif par exercice (pas de librairie externe) :
  - Axe X : séances (dates)
  - Axe Y : charge max de la séance (kg max dans `setLog` pour cet exercice ce jour)
  - Ligne de progression + points interactifs
  - Points verts = PR
- Scroll vertical pour voir les 6 graphiques
- Si moins de 2 points de données : message `"Complète au moins 2 séances pour voir la courbe"`

#### Sub-tab "Corps" (nouveau)
- Formulaire d'entrée : poids corporel (kg) + 3 champs optionnels (tour de taille, bras fléchi, cuisse)
- Bouton "Enregistrer" → push dans `bodyLog`
- Graphique SVG du poids corporel sur les 30 derniers jours
- Tableau des 10 dernières entrées

---

### 5. Écran de fin de séance — Résumé complet

Remplace le bouton `"✅ Marquer la séance complète"`. Quand l'utilisateur tape ce bouton :

1. Calcule la durée (timestamp fin − timestamp début de séance, stocké au `renderSeance()`)
2. Calcule le volume total (`calcVolume(workoutId)`)
3. Collecte les PRs réalisés pendant la séance (accumulés dans un array `sessionPRs` pendant la séance)
4. Affiche la page de résumé :
   - Header animé : emoji séance + nom + durée
   - Deux stats : tonnage total en kg / nb séries complétées
   - Section PRs (masquée si aucun)
   - 5 étoiles tapables (ressenti)
   - Textarea optionnel pour la note libre
   - Bouton "Enregistrer" → `saveSessionSummary()`
5. `saveSessionSummary()` :
   - Enregistre dans `completedSessions` (existant)
   - Enregistre dans `sessionLog` (nouveau)
   - Retour à l'accueil

---

### 6. Records personnels (PRs) — Récapitulatif

- **Pendant la séance** : détectés dans `closeSetModal()` → flash dans le modal + accumulés dans `sessionPRs[]`
- **Fin de séance** : listés sur l'écran de résumé
- **Onglet Progrès > Courbes** : points verts sur les graphiques
- **Onglet Progrès > Séances** : badge `"🏆 2 PRs"` sur les cartes de séances concernées

---

## Changements dans le code existant

### Fonctions modifiées

| Fonction | Changement |
|----------|------------|
| `makeExCard()` | Boutons série → `onclick="openSetModal(...)"` au lieu de `toggleSet`. Badge cible semaine sous le nom. Affichage du poids logué sous chaque bouton validé. |
| `toggleSet()` | Supprimée, remplacée par `openSetModal()` + `closeSetModal()` |
| `completeSession()` | Enregistre le timestamp de début, redirige vers `showSessionSummary(id)` |
| `renderHome()` | Bandeau semaine + pills S1-S4 |
| `renderProgress()` | 3 sous-tabs + nouveau rendu pour chacun |
| `renderSeance()` | Stocke `sessionStart = Date.now()` à l'ouverture |

### Nouvelles fonctions

| Fonction | Rôle |
|----------|------|
| `openSetModal(wId, exIdx, setIdx)` | Affiche le modal, pré-remplit les champs |
| `closeSetModal(wId, exIdx, setIdx, kg, reps)` | Valide, enregistre, PR check, auto-timer |
| `detectPR(wId, exIdx, kg, reps)` | Retourne `true` si nouveau record |
| `calcVolume(wId)` | Calcule le tonnage total d'une séance depuis `setLog` |
| `getCurrentWeek()` | Retourne 1-4 depuis date ou override |
| `getWeekTarget(exName, week)` | Calcule la charge cible pour un exercice à une semaine donnée |
| `showSessionSummary(id)` | Affiche la page résumé fin de séance |
| `saveSessionSummary(id, rating, note)` | Enregistre dans `completedSessions` + `sessionLog` |
| `startAutoRestTimer(sec)` | Démarre la pastille flottante |
| `stopAutoRestTimer()` | Arrête et masque la pastille |
| `drawChart(containerId, points, color)` | Rendu SVG d'un graphique de progression |
| `renderCurvesTab()` | Rendu du sous-tab Courbes |
| `renderBodyTab()` | Rendu du sous-tab Corps |
| `saveBodyEntry(weight, waist, arm, thigh)` | Enregistre dans `bodyLog` |
| `renderSettingsPage()` | Page paramètres (date départ + reset) |

### Ce qui n'est pas modifié
Navigation 4 onglets, CSS design system, HIIT timer, minuteur repos modal, GIFs exercices, échauffements, conseils ⚡, fichiers PWA (`manifest.json`, `sw.js`, `generate-icons.html`)

---

## Hors périmètre

- Export CSV / PDF de l'historique
- Notifications push Android
- Synchronisation cloud
- Gestion multi-utilisateurs
