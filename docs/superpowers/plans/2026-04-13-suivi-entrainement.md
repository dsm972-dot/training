# Suivi Entraînement Avancé — Plan d'Implémentation

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Ajouter 8 fonctionnalités de suivi avancé à `app_entrainement.html` : saisie poids+reps par série (modal slide-up), semaine S1-S4 auto, timer repos auto, graphiques de progression, records personnels, résumé de fin de séance, suivi corporel.

**Architecture:** Application HTML mono-fichier. Toutes les modifications s'appliquent à `app_entrainement.html` — CSS injecté dans le `<style>` existant, HTML ajouté dans `<div id="app">`, JS ajouté/modifié dans le `<script>` existant. Données persistées en `localStorage`.

**Tech Stack:** HTML5/CSS3/JavaScript vanilla, SVG natif pour les graphiques, localStorage pour la persistance — zéro dépendance externe.

---

## Structure des modifications

| Section | Fichier | Nature |
|---------|---------|--------|
| CSS (8 blocs) | `app_entrainement.html` | Insertions dans `<style>` |
| HTML (5 éléments) | `app_entrainement.html` | Modal série, pastille timer, page résumé, page paramètres, page summary |
| JS — State | `app_entrainement.html` | 7 nouvelles variables après le bloc STATE existant |
| JS — Helpers | `app_entrainement.html` | 8 nouvelles fonctions utilitaires |
| JS — Modifié | `app_entrainement.html` | `makeExCard`, `renderSeance`, `completeSession`, `renderHome`, `renderProgress` |
| JS — Nouveau | `app_entrainement.html` | 13 nouvelles fonctions |

---

## Task 1 — Nouvelles variables d'état et fonctions utilitaires

**Files:**
- Modify: `app_entrainement.html` — bloc `// STATE` (~ligne 646) et après `renderProgress()`

### Étape 1 — Ajouter les nouvelles variables d'état

- [ ] Dans `app_entrainement.html`, trouver le commentaire `// ── TIMER MODE ──` (ligne ~664) et ajouter **après** `let timerMode = 'rest';` :

```js
// ── TRACKING AVANCÉ STATE ──
let setLog = JSON.parse(localStorage.getItem('setLog') || '{}');
let sessionLog = JSON.parse(localStorage.getItem('sessionLog') || '{}');
let bodyLog = JSON.parse(localStorage.getItem('bodyLog') || '[]');
let programStart = localStorage.getItem('programStart') || null;
let currentWeek = localStorage.getItem('currentWeek') ? parseInt(localStorage.getItem('currentWeek')) : null;
let sessionStart = null;       // timestamp début séance courante
let sessionPRs = [];           // PRs accumulés pendant la séance courante
let sessionMaxByEx = {};       // { exName: { kg, reps } } — meilleure série par exercice en cours
let autoRestInterval = null;   // interval pastille flottante
let autoRestSec = 0;
let autoRestTotal = 0;
```

### Étape 2 — Ajouter les fonctions utilitaires

- [ ] Trouver la ligne `function fmtSec(s)` (~ligne 1335) et ajouter **avant** :

```js
// ══════════════════════════════════════════════
// HELPERS TRACKING AVANCÉ
// ══════════════════════════════════════════════

function getCurrentWeek() {
  if (currentWeek !== null) return Math.min(4, Math.max(1, currentWeek));
  if (!programStart) return 1;
  const diffDays = Math.floor((Date.now() - new Date(programStart)) / 86400000);
  return Math.min(4, Math.max(1, Math.ceil((diffDays + 1) / 7)));
}

function getWeekCoeff(week) {
  return [1.00, 1.05, 1.10, 1.15][(week || 1) - 1] || 1.00;
}

function getWeekLabel(week) {
  return ['', '75% 1RM', '+5%', '+5% · +1 série', '85-90% 1RM'][week] || '';
}

function calcVolume(workoutId) {
  return Object.entries(setLog)
    .filter(([k]) => k.startsWith(workoutId + '_'))
    .reduce((sum, [, v]) => sum + (v.kg * v.reps), 0);
}

function detectPR(exName, kg, reps) {
  const best = Object.values(sessionLog)
    .map(s => s.maxByEx?.[exName])
    .filter(Boolean)
    .reduce((max, v) => Math.max(max, (v.kg || 0) * (v.reps || 0)), 0);
  return (kg * reps) > best;
}

function getRefKgForEx(exName) {
  // Charge de référence S1 = avg kg des séries loggées dans sessions S1
  const s1sessions = Object.entries(sessionLog)
    .filter(([, s]) => s.week === 1)
    .map(([, s]) => s.maxByEx?.[exName]?.kg)
    .filter(Boolean);
  if (!s1sessions.length) return null;
  return s1sessions.reduce((a, b) => a + b, 0) / s1sessions.length;
}

function setWeekOverride(w) {
  if (currentWeek === w) {
    currentWeek = null;
    localStorage.removeItem('currentWeek');
  } else {
    currentWeek = w;
    localStorage.setItem('currentWeek', w);
  }
  renderHome();
}

function saveProgramStart(dateStr) {
  programStart = dateStr;
  localStorage.setItem('programStart', dateStr);
  currentWeek = null;
  localStorage.removeItem('currentWeek');
}
```

### Étape 3 — Vérification

- [ ] Ouvrir `app_entrainement.html` dans Chrome
- [ ] Ouvrir DevTools → Console
- [ ] Taper `getCurrentWeek()` → doit retourner `1` (pas de programStart)
- [ ] Taper `calcVolume('lundi')` → doit retourner `0`
- [ ] Taper `detectPR('Développé couché à la barre', 100, 5)` → doit retourner `true`
- [ ] Aucune erreur dans la console

### Étape 4 — Commit

```bash
git add app_entrainement.html
git commit -m "feat: add tracking state variables and utility helpers"
```

---

## Task 2 — Modal slide-up de saisie par série

**Files:**
- Modify: `app_entrainement.html` — CSS, HTML (modal), JS (`makeExCard`, `toggleSet`)

### Étape 1 — CSS du modal

- [ ] Trouver `/* ─── TIMER PAGE ───*/` dans le CSS et ajouter **avant** :

```css
/* ─── SET MODAL (slide-up) ─── */
#set-modal {
  position: fixed; bottom: 0; left: 50%; transform: translateX(-50%) translateY(100%);
  width: 100%; max-width: 430px; background: #1E1E1E;
  border-radius: 20px 20px 0 0; padding: 20px 20px calc(20px + env(safe-area-inset-bottom));
  z-index: 300; transition: transform .3s cubic-bezier(.25,.8,.25,1);
  box-shadow: 0 -8px 40px rgba(0,0,0,.6);
}
#set-modal.show { transform: translateX(-50%) translateY(0); }
.sm-backdrop {
  position: fixed; inset: 0; background: rgba(0,0,0,.5);
  z-index: 299; display: none;
}
.sm-backdrop.show { display: block; }
.sm-handle { width: 40px; height: 4px; background: #333; border-radius: 2px; margin: 0 auto 16px; }
.sm-header { font-size: 11px; font-weight: 800; color: var(--muted); text-transform: uppercase; letter-spacing: .5px; margin-bottom: 4px; }
.sm-prev { font-size: 12px; color: #555; margin-bottom: 14px; min-height: 16px; }
.sm-inputs { display: flex; gap: 10px; align-items: center; margin-bottom: 14px; }
.sm-field { display: flex; flex-direction: column; align-items: center; gap: 4px; }
.sm-label { font-size: 10px; color: var(--muted); font-weight: 700; text-transform: uppercase; }
.sm-input {
  width: 90px; background: #252525; border: 1.5px solid #444;
  border-radius: 10px; color: var(--text); font-size: 22px; font-weight: 800;
  text-align: center; padding: 10px 0; -moz-appearance: textfield;
}
.sm-input:focus { outline: none; border-color: var(--orange); }
.sm-input::-webkit-outer-spin-button, .sm-input::-webkit-inner-spin-button { -webkit-appearance: none; }
.sm-sep { font-size: 20px; color: #444; font-weight: 700; margin-top: 14px; }
.sm-confirm-btn {
  width: 100%; background: var(--orange); color: #fff; border: none;
  border-radius: 12px; padding: 14px; font-size: 16px; font-weight: 800; cursor: pointer;
}
.sm-confirm-btn:active { opacity: .85; }
.sm-pr-flash {
  display: none; text-align: center; color: var(--green);
  font-size: 15px; font-weight: 800; padding: 8px 0; animation: prPop .4s ease;
}
@keyframes prPop { from { transform: scale(.8); opacity: 0; } to { transform: scale(1); opacity: 1; } }
```

### Étape 2 — HTML du modal

- [ ] Trouver `<!-- ═══════════════ REST MODAL ═══════════════ -->` et ajouter **avant** :

```html
<!-- ═══════════════ SET MODAL ═══════════════ -->
<div class="sm-backdrop" id="sm-backdrop" onclick="closeSetModalCancel()"></div>
<div id="set-modal">
  <div class="sm-handle"></div>
  <div class="sm-pr-flash" id="sm-pr-flash">🏆 Nouveau record !</div>
  <div class="sm-header" id="sm-header">Série 1</div>
  <div class="sm-prev" id="sm-prev"></div>
  <div class="sm-inputs">
    <div class="sm-field">
      <span class="sm-label">Poids (kg)</span>
      <input class="sm-input" id="sm-kg" type="number" step="0.5" min="0" inputmode="decimal">
    </div>
    <span class="sm-sep">×</span>
    <div class="sm-field">
      <span class="sm-label">Reps</span>
      <input class="sm-input" id="sm-reps" type="number" step="1" min="1" inputmode="numeric">
    </div>
  </div>
  <button class="sm-confirm-btn" id="sm-confirm-btn" onclick="confirmSet()">✓ Valider la série</button>
</div>
```

### Étape 3 — Variables de contexte du modal

- [ ] Dans le bloc `// ── TRACKING AVANCÉ STATE ──` (Task 1), ajouter à la fin :

```js
let _smCtx = null; // { workoutId, exIdx, setIdx } — contexte du modal ouvert
```

### Étape 4 — Fonctions du modal

- [ ] Trouver `function openSetModal` — elle n'existe pas encore. Ajouter **après** la fonction `checkExercise()` (~ligne 1011) :

```js
function openSetModal(workoutId, exIdx, setIdx) {
  const w = PROGRAM[workoutId];
  const ex = w.exercises[exIdx];
  _smCtx = { workoutId, exIdx, setIdx };

  // En-tête
  document.getElementById('sm-header').textContent = `Série ${setIdx + 1} / ${ex.sets} · ${ex.name}`;

  // Rappel série précédente
  const prevKey = `${workoutId}_${exIdx}_${setIdx - 1}`;
  const prev = setLog[prevKey];
  document.getElementById('sm-prev').textContent = prev
    ? `Série ${setIdx} : ${prev.kg} kg × ${prev.reps} reps`
    : (setIdx === 0 ? 'Première série' : '');

  // Pré-remplir kg avec dernière valeur connue pour cet exercice
  const allKeys = Object.keys(setLog).filter(k => k.startsWith(`${workoutId}_${exIdx}_`));
  const lastKg = allKeys.length
    ? setLog[allKeys[allKeys.length - 1]].kg
    : (getRefKgForEx(ex.name) || '');
  document.getElementById('sm-kg').value = lastKg;

  // Pré-remplir reps avec cible programme (premier chiffre de "4-5" ou valeur directe)
  const targetReps = parseInt(ex.reps) || parseInt(ex.reps.split('-')[0]) || '';
  document.getElementById('sm-reps').value = targetReps;

  document.getElementById('sm-pr-flash').style.display = 'none';
  document.getElementById('sm-backdrop').classList.add('show');
  document.getElementById('set-modal').classList.add('show');

  // Focus automatique sur le champ kg
  setTimeout(() => document.getElementById('sm-kg').focus(), 350);
}

function closeSetModalCancel() {
  document.getElementById('sm-backdrop').classList.remove('show');
  document.getElementById('set-modal').classList.remove('show');
  _smCtx = null;
}

function confirmSet() {
  if (!_smCtx) return;
  const { workoutId, exIdx, setIdx } = _smCtx;
  const kg = parseFloat(document.getElementById('sm-kg').value) || 0;
  const reps = parseInt(document.getElementById('sm-reps').value) || 0;
  if (kg <= 0 || reps <= 0) { alert('Saisis le poids et les reps.'); return; }

  const w = PROGRAM[workoutId];
  const ex = w.exercises[exIdx];
  const key = `${workoutId}_${exIdx}_${setIdx}`;

  // Enregistrer dans setLog
  setLog[key] = { kg, reps };
  localStorage.setItem('setLog', JSON.stringify(setLog));

  // Mettre à jour le meilleur de la session
  if (!sessionMaxByEx[ex.name] || kg * reps > sessionMaxByEx[ex.name].kg * sessionMaxByEx[ex.name].reps) {
    sessionMaxByEx[ex.name] = { kg, reps };
  }

  // Détection PR
  const isPR = detectPR(ex.name, kg, reps);
  if (isPR) {
    sessionPRs.push(`${ex.name}: ${kg}kg×${reps}`);
    const flash = document.getElementById('sm-pr-flash');
    flash.style.display = 'block';
  }

  // Mettre à jour le bouton de série
  const btn = document.getElementById(`set-btn-${workoutId}-${exIdx}-${setIdx}`);
  if (btn) {
    btn.classList.add('done');
    btn.innerHTML = `<span>${setIdx + 1}</span><span class="set-kg-label">${kg}kg</span>`;
  }

  // Marquer dans setsDone pour la barre de progression
  if (!setsDone[workoutId]) setsDone[workoutId] = {};
  setsDone[workoutId][key] = true;
  updateProgressBar(workoutId);

  // Auto-check exercice si toutes les séries sont faites
  let allDone = true;
  for (let s = 0; s < ex.sets; s++) {
    if (!setsDone[workoutId][`${workoutId}_${exIdx}_${s}`]) { allDone = false; break; }
  }
  const check = document.getElementById(`check-${workoutId}-${exIdx}`);
  if (check) {
    check.classList.toggle('checked', allDone);
    document.getElementById(`ex-${workoutId}-${exIdx}`)?.classList.toggle('done-card', allDone);
  }

  // Démarrer auto-timer repos
  if (ex.rest > 0) startAutoRestTimer(ex.rest);

  // Fermer modal (avec délai si PR pour laisser voir le flash)
  const delay = isPR ? 1500 : 400;
  setTimeout(() => {
    document.getElementById('sm-backdrop').classList.remove('show');
    document.getElementById('set-modal').classList.remove('show');
    _smCtx = null;
  }, delay);
}
```

### Étape 5 — Modifier `makeExCard()` pour utiliser le modal

- [ ] Dans `makeExCard()` (~ligne 916), remplacer la génération des boutons de série :

Trouver :
```js
  let setsHtml = '';
  for (let s = 0; s < ex.sets; s++) {
    const key = `${workoutId}_${exIdx}_${s}`;
    const done = !!setsDone[workoutId]?.[key];
    setsHtml += `<button class="set-btn${done?' done':''}" onclick="toggleSet('${workoutId}',${exIdx},${s},this)">${s+1}</button>`;
  }
```

Remplacer par :
```js
  let setsHtml = '';
  for (let s = 0; s < ex.sets; s++) {
    const key = `${workoutId}_${exIdx}_${s}`;
    const done = !!setsDone[workoutId]?.[key];
    const logged = setLog[key];
    const btnContent = logged
      ? `<span>${s+1}</span><span class="set-kg-label">${logged.kg}kg</span>`
      : `${s+1}`;
    setsHtml += `<button class="set-btn${done?' done':''}" id="set-btn-${workoutId}-${exIdx}-${s}" onclick="openSetModal('${workoutId}',${exIdx},${s})">${btnContent}</button>`;
  }
```

### Étape 6 — CSS du label kg sur les boutons

- [ ] Dans le CSS, trouver `.set-btn` et ajouter **après** son bloc :

```css
.set-btn { flex-direction: column; gap: 1px; line-height: 1.1; }
.set-kg-label { font-size: 9px; font-weight: 700; opacity: .9; }
```

### Étape 7 — Vérification

- [ ] Ouvrir l'app → aller sur la séance PUSH (Lundi)
- [ ] Taper sur le bouton "1" de l'exercice Développé couché → modal doit remonter du bas
- [ ] Vérifier : champ kg pré-rempli, champ reps pré-rempli avec 4 (cible "4-5")
- [ ] Saisir 100 kg × 5 reps → "✓ Valider" → modal se ferme, bouton passe en orange avec "100kg"
- [ ] Taper hors du modal → modal se ferme sans valider
- [ ] Recharger la page → poids "100kg" toujours visible sur le bouton de série (persisté)

### Étape 8 — Commit

```bash
git add app_entrainement.html
git commit -m "feat: add per-set modal with weight+reps logging and PR detection"
```

---

## Task 3 — Pastille timer repos auto-démarrage

**Files:**
- Modify: `app_entrainement.html` — CSS, HTML, JS

### Étape 1 — CSS de la pastille flottante

- [ ] Ajouter dans le CSS (avant `/* ─── TIMER PAGE ───*/`) :

```css
/* ─── AUTO REST BUBBLE ─── */
#auto-rest-bubble {
  position: fixed; bottom: calc(var(--nav-h) + 12px); right: 12px;
  width: 62px; height: 62px; border-radius: 50%;
  background: #1E1E1E; border: 2px solid var(--orange);
  display: none; align-items: center; justify-content: center; flex-direction: column;
  z-index: 200; cursor: pointer; box-shadow: 0 4px 16px rgba(0,0,0,.5);
  transition: transform .2s;
}
#auto-rest-bubble.visible { display: flex; }
#auto-rest-bubble:active { transform: scale(.93); }
.arb-time { font-size: 13px; font-weight: 800; color: var(--text); line-height: 1; }
.arb-label { font-size: 9px; color: var(--muted); margin-top: 2px; }
.arb-ring {
  position: absolute; top: 0; left: 0; width: 100%; height: 100%;
  transform: rotate(-90deg);
}
.arb-ring-track { fill: none; stroke: #333; stroke-width: 3; }
.arb-ring-prog { fill: none; stroke: var(--orange); stroke-width: 3; stroke-linecap: round; transition: stroke-dashoffset .9s linear; }
```

### Étape 2 — HTML de la pastille

- [ ] Trouver `<!-- ═══════════════ SET MODAL ═══════════════ -->` et ajouter **avant** :

```html
<!-- ═══════════════ AUTO REST BUBBLE ═══════════════ -->
<div id="auto-rest-bubble" onclick="bubbleTapped()">
  <svg class="arb-ring" viewBox="0 0 62 62">
    <circle class="arb-ring-track" cx="31" cy="31" r="27"/>
    <circle class="arb-ring-prog" id="arb-prog" cx="31" cy="31" r="27"
      stroke-dasharray="169.6" stroke-dashoffset="0"/>
  </svg>
  <span class="arb-time" id="arb-time">1:30</span>
  <span class="arb-label">REPOS</span>
</div>
```

### Étape 3 — Fonctions de la pastille

- [ ] Ajouter juste après les fonctions `openSetModal / closeSetModalCancel / confirmSet` de la Task 2 :

```js
function startAutoRestTimer(sec) {
  clearInterval(autoRestInterval);
  autoRestSec = sec;
  autoRestTotal = sec;
  const bubble = document.getElementById('auto-rest-bubble');
  bubble.classList.add('visible');
  updateArbDisplay();
  autoRestInterval = setInterval(() => {
    autoRestSec--;
    if (autoRestSec <= 0) {
      beep();
      stopAutoRestTimer();
      return;
    }
    updateArbDisplay();
  }, 1000);
}

function stopAutoRestTimer() {
  clearInterval(autoRestInterval);
  autoRestSec = 0;
  document.getElementById('auto-rest-bubble').classList.remove('visible');
}

function updateArbDisplay() {
  document.getElementById('arb-time').textContent = fmtSec(autoRestSec);
  const pct = autoRestTotal > 0 ? autoRestSec / autoRestTotal : 0;
  const circ = 169.6;
  document.getElementById('arb-prog').style.strokeDashoffset = circ * (1 - pct);
}

function bubbleTapped() {
  // Synchronise le timer modal existant avec le décompte en cours
  clearInterval(autoRestInterval);
  startModalRest(autoRestSec, 'Prochain exercice');
  stopAutoRestTimer();
}
```

### Étape 4 — Vérification

- [ ] Valider une série avec 90s de repos configuré → pastille orange apparaît en bas à droite
- [ ] Pastille affiche le décompte qui tourne
- [ ] Taper la pastille → ouvre le timer modal existant synchronisé
- [ ] Attendre la fin → bip + pastille disparaît
- [ ] Valider une nouvelle série avant la fin → timer repart depuis la nouvelle durée

### Étape 5 — Commit

```bash
git add app_entrainement.html
git commit -m "feat: add floating auto-rest timer bubble after set validation"
```

---

## Task 4 — Suivi semaine S1→S4 + Page Paramètres

**Files:**
- Modify: `app_entrainement.html` — CSS, HTML (page settings), JS (`renderHome`, `makeExCard`, `showPage`)

### Étape 1 — CSS de la bannière semaine et de la page paramètres

- [ ] Ajouter dans le CSS (avant `/* ─── TIMER PAGE ───*/`) :

```css
/* ─── WEEK BANNER ─── */
.week-banner { background: var(--card); border-radius: var(--radius); padding: 12px 14px; margin-bottom: 14px; display: flex; align-items: center; justify-content: space-between; gap: 10px; }
.wb-left { display: flex; flex-direction: column; gap: 2px; }
.wb-label { font-size: 10px; color: var(--muted); font-weight: 700; text-transform: uppercase; letter-spacing: .4px; }
.wb-title { font-size: 15px; font-weight: 800; color: var(--orange); }
.wb-sub { font-size: 11px; color: var(--muted); }
.week-pills { display: flex; gap: 5px; }
.week-pill { padding: 5px 9px; border-radius: 8px; border: 1.5px solid #2a2a2a; background: var(--card2); color: var(--muted); font-size: 11px; font-weight: 800; cursor: pointer; transition: all .15s; }
.week-pill.active { border-color: var(--orange); color: var(--orange); background: rgba(232,80,10,.1); }

/* Badge cible semaine sur carte exercice */
.week-target-badge { font-size: 10px; color: var(--orange); font-weight: 700; background: rgba(232,80,10,.1); border-radius: 4px; padding: 1px 5px; margin-left: 6px; }

/* ─── PAGE PARAMÈTRES ─── */
#page-settings { background: var(--bg); }
.settings-header { display: flex; align-items: center; gap: 12px; padding: 16px; border-bottom: 1px solid #2a2a2a; }
.settings-back { background: none; border: none; color: var(--text); font-size: 20px; cursor: pointer; padding: 4px; }
.settings-title { font-size: 18px; font-weight: 800; }
.settings-group { margin: 20px 16px 0; }
.settings-group-label { font-size: 11px; font-weight: 800; color: var(--muted); text-transform: uppercase; letter-spacing: .5px; margin-bottom: 8px; }
.settings-row { background: var(--card); border-radius: var(--radius); padding: 14px 16px; margin-bottom: 8px; }
.settings-row-label { font-size: 14px; font-weight: 700; margin-bottom: 8px; }
.settings-date-input { width: 100%; background: #252525; border: 1.5px solid #444; border-radius: 10px; color: var(--text); font-size: 15px; padding: 10px 12px; box-sizing: border-box; }
.settings-date-input:focus { outline: none; border-color: var(--orange); }
.settings-save-btn { width: 100%; background: var(--orange); color: #fff; border: none; border-radius: 10px; padding: 12px; font-size: 14px; font-weight: 800; margin-top: 10px; cursor: pointer; }
.settings-reset-btn { width: 100%; background: transparent; color: var(--muted); border: 1.5px solid #333; border-radius: 10px; padding: 10px; font-size: 13px; font-weight: 700; margin-top: 6px; cursor: pointer; }
.settings-gear { background: none; border: none; color: var(--muted); font-size: 20px; cursor: pointer; padding: 4px; }
```

### Étape 2 — Ajouter la page Paramètres dans le HTML

- [ ] Trouver `<!-- ═══════════════ BOTTOM NAV ═══════════════ -->` et ajouter **avant** :

```html
<!-- ═══════════════ PAGE PARAMÈTRES ═══════════════ -->
<div id="page-settings" class="page">
  <div class="settings-header">
    <button class="settings-back" onclick="showPage('home')">←</button>
    <span class="settings-title">⚙️ Paramètres</span>
  </div>
  <div class="settings-group">
    <div class="settings-group-label">Programme</div>
    <div class="settings-row">
      <div class="settings-row-label">Date de début du programme</div>
      <input class="settings-date-input" id="settings-program-start" type="date"
        placeholder="AAAA-MM-JJ">
      <button class="settings-save-btn" onclick="saveSettingsProgramStart()">Enregistrer</button>
      <button class="settings-reset-btn" onclick="resetProgram()">Réinitialiser le programme</button>
    </div>
  </div>
  <div class="settings-group">
    <div class="settings-group-label">Données</div>
    <div class="settings-row">
      <div class="settings-row-label">Effacer les données de poids</div>
      <button class="settings-reset-btn" onclick="if(confirm('Effacer tous les logs de poids ?')){setLog={};sessionLog={};localStorage.removeItem(\'setLog\');localStorage.removeItem(\'sessionLog\');alert(\'Effacé.\');}">Effacer setLog + sessionLog</button>
    </div>
  </div>
</div>
```

### Étape 3 — Bouton ⚙️ sur la page d'accueil

- [ ] Dans le HTML de `#page-home`, trouver la ligne `<div class="page-title">` (dans la page home) et la modifier pour ajouter l'icône engrenage. Trouver :

```html
      <div class="page-title" id="home-greeting">Bonjour Denis 👊</div>
```

Remplacer par :

```html
      <div style="display:flex;align-items:center;justify-content:space-between">
        <div class="page-title" id="home-greeting">Bonjour Denis 👊</div>
        <button class="settings-gear" onclick="showPage('settings')">⚙️</button>
      </div>
```

### Étape 4 — Enregistrer `settings` dans `showPage()`

- [ ] Dans `showPage(name)` (~ligne 669), trouver la ligne `if (name === 'progress') renderProgress();` et ajouter après :

```js
  if (name === 'settings') renderSettingsPage();
```

### Étape 5 — Fonction `renderSettingsPage()`

- [ ] Ajouter après `setWeekOverride()` (Task 1) :

```js
function renderSettingsPage() {
  const inp = document.getElementById('settings-program-start');
  if (inp && programStart) inp.value = programStart;
}

function saveSettingsProgramStart() {
  const val = document.getElementById('settings-program-start').value;
  if (!val) return;
  saveProgramStart(val);
  alert('Date de départ enregistrée. Semaine recalculée automatiquement.');
  showPage('home');
}

function resetProgram() {
  if (!confirm('Réinitialiser la date de départ et la semaine ?')) return;
  programStart = null;
  currentWeek = null;
  localStorage.removeItem('programStart');
  localStorage.removeItem('currentWeek');
  alert('Programme réinitialisé.');
  showPage('home');
}
```

### Étape 6 — Bandeau semaine sur l'accueil

- [ ] Dans `renderHome()` (~ligne 710), trouver `// Today banner` et ajouter **avant** :

```js
  // Week banner
  const week = getCurrentWeek();
  const weekBanner = document.getElementById('week-banner');
  if (weekBanner) {
    weekBanner.innerHTML = `
      <div class="wb-left">
        <span class="wb-label">Programme en cours</span>
        <span class="wb-title">Semaine ${week} ${programStart ? '✓' : ''}</span>
        <span class="wb-sub">${getWeekLabel(week)}</span>
      </div>
      <div class="week-pills">
        ${[1,2,3,4].map(w => `<button class="week-pill${week===w?' active':''}" onclick="setWeekOverride(${w})">S${w}</button>`).join('')}
      </div>
    `;
  }
```

- [ ] Dans le HTML de `#page-home`, trouver `<div id="today-banner"` et ajouter **avant** :

```html
      <div class="week-banner" id="week-banner"></div>
```

### Étape 7 — Badge cible semaine sur les cartes exercice

- [ ] Dans `makeExCard()` (~ligne 916), trouver :

```js
  const noteHtml = ex.note ? `<div class="badge orange">★ ${ex.note}</div>` : '';
```

Remplacer par :

```js
  const noteHtml = ex.note ? `<div class="badge orange">★ ${ex.note}</div>` : '';
  const week = getCurrentWeek();
  const refKg = getRefKgForEx(ex.name);
  const targetKg = refKg ? Math.round(refKg * getWeekCoeff(week) * 2) / 2 : null;
  const weekBadge = (week > 1 && targetKg) ? `<span class="week-target-badge">Cible S${week} : ~${targetKg} kg</span>` : '';
```

- [ ] Dans le template de `makeExCard()`, trouver `<div class="ex-name">${ex.name}</div>` et remplacer par :

```js
  `<div class="ex-name">${ex.name}${weekBadge}</div>`
```

### Étape 8 — Vérification

- [ ] Page accueil : bandeau "Semaine 1" visible avec pills S1/S2/S3/S4
- [ ] Taper S2 → bandeau passe à "Semaine 2", pill S2 active
- [ ] Taper S2 à nouveau → retour auto (null → semaine 1 si pas de programStart)
- [ ] Bouton ⚙️ → page Paramètres
- [ ] Saisir une date → enregistrer → retour accueil → semaine recalculée
- [ ] Sur séance PUSH avec données S1 enregistrées → badge "Cible S2 : ~105 kg" sous le nom

### Étape 9 — Commit

```bash
git add app_entrainement.html
git commit -m "feat: add S1-S4 week tracker with auto-calculation and settings page"
```

---

## Task 5 — Écran de résumé de fin de séance

**Files:**
- Modify: `app_entrainement.html` — CSS, HTML (page summary), JS (`renderSeance`, `completeSession`)

### Étape 1 — CSS de la page résumé

- [ ] Ajouter dans le CSS (avant `/* ─── TIMER PAGE ───*/`) :

```css
/* ─── SESSION SUMMARY ─── */
#page-summary { justify-content: flex-start; }
.summary-confetti { height: 5px; background: linear-gradient(90deg,#E8500A,#9B3AE8,#2DBD6E,#E8500A); background-size: 200%; animation: confetti 2s linear infinite; }
@keyframes confetti { to { background-position: 200%; } }
.summary-hero { text-align: center; padding: 28px 16px 20px; }
.summary-emoji { font-size: 48px; display: block; margin-bottom: 8px; }
.summary-name { font-size: 22px; font-weight: 800; }
.summary-meta { font-size: 13px; color: var(--muted); margin-top: 4px; }
.summary-stats { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; padding: 0 16px; margin-bottom: 16px; }
.summary-stat { background: var(--card); border-radius: var(--radius); padding: 16px; text-align: center; }
.summary-stat-val { font-size: 24px; font-weight: 800; color: var(--orange); }
.summary-stat-label { font-size: 11px; color: var(--muted); margin-top: 3px; }
.summary-prs { margin: 0 16px 16px; background: rgba(45,189,110,.08); border: 1px solid rgba(45,189,110,.25); border-radius: var(--radius); padding: 12px 14px; }
.summary-prs-title { font-size: 11px; font-weight: 800; color: var(--green); text-transform: uppercase; letter-spacing: .4px; margin-bottom: 8px; }
.summary-pr-item { font-size: 13px; color: var(--text); padding: 3px 0; }
.summary-rating { padding: 0 16px 12px; }
.summary-rating-label { font-size: 13px; font-weight: 700; text-align: center; margin-bottom: 10px; }
.stars-row { display: flex; gap: 8px; justify-content: center; margin-bottom: 12px; }
.star-btn { font-size: 30px; cursor: pointer; opacity: .3; transition: opacity .15s, transform .15s; background: none; border: none; padding: 0; }
.star-btn.active { opacity: 1; transform: scale(1.15); }
.summary-note { width: 100%; background: var(--card); border: 1.5px solid #333; border-radius: 10px; color: var(--text); font-size: 14px; font-family: inherit; padding: 12px 14px; box-sizing: border-box; resize: none; }
.summary-note:focus { outline: none; border-color: var(--orange); }
.summary-save-btn { display: block; width: calc(100% - 32px); margin: 14px 16px; background: var(--green); color: #fff; border: none; border-radius: var(--radius); padding: 16px; font-size: 16px; font-weight: 800; cursor: pointer; }
```

### Étape 2 — HTML de la page résumé

- [ ] Trouver `<!-- ═══════════════ PAGE PARAMÈTRES ═══════════════ -->` et ajouter **avant** :

```html
<!-- ═══════════════ PAGE RÉSUMÉ SÉANCE ═══════════════ -->
<div id="page-summary" class="page">
  <div class="summary-confetti"></div>
  <div class="summary-hero">
    <span class="summary-emoji" id="summary-emoji">💪</span>
    <div class="summary-name" id="summary-name">Séance terminée !</div>
    <div class="summary-meta" id="summary-meta"></div>
  </div>
  <div class="summary-stats">
    <div class="summary-stat">
      <div class="summary-stat-val" id="summary-volume">0</div>
      <div class="summary-stat-label">kg soulevés</div>
    </div>
    <div class="summary-stat">
      <div class="summary-stat-val" id="summary-sets">0/0</div>
      <div class="summary-stat-label">séries ✓</div>
    </div>
  </div>
  <div class="summary-prs" id="summary-prs-block" style="display:none">
    <div class="summary-prs-title">🏆 Records du jour</div>
    <div id="summary-prs-list"></div>
  </div>
  <div class="summary-rating">
    <div class="summary-rating-label">Comment tu te sens ?</div>
    <div class="stars-row" id="stars-row">
      <button class="star-btn" data-val="1" onclick="setRating(1)">⭐</button>
      <button class="star-btn" data-val="2" onclick="setRating(2)">⭐</button>
      <button class="star-btn" data-val="3" onclick="setRating(3)">⭐</button>
      <button class="star-btn" data-val="4" onclick="setRating(4)">⭐</button>
      <button class="star-btn" data-val="5" onclick="setRating(5)">⭐</button>
    </div>
    <textarea class="summary-note" id="summary-note" rows="2" placeholder="Note rapide… (optionnel)"></textarea>
  </div>
  <button class="summary-save-btn" onclick="saveSessionSummary()">✅ Enregistrer la séance</button>
</div>
```

### Étape 3 — Variable rating courante

- [ ] Ajouter à la fin du bloc `// ── TRACKING AVANCÉ STATE ──` (Task 1) :

```js
let _currentRating = 0;
let _summaryWorkoutId = null;
```

### Étape 4 — Fonctions de la page résumé

- [ ] Ajouter après `resetProgram()` (Task 4) :

```js
function showSessionSummary(id) {
  _summaryWorkoutId = id;
  _currentRating = 0;

  const w = PROGRAM[id];
  const durationMin = sessionStart ? Math.round((Date.now() - sessionStart) / 60000) : 0;
  const volume = calcVolume(id);

  // Compter séries complétées
  let totalSets = 0, doneSets = 0;
  w.exercises.forEach((ex, exIdx) => {
    for (let s = 0; s < ex.sets; s++) {
      totalSets++;
      if (setsDone[id]?.[`${id}_${exIdx}_${s}`]) doneSets++;
    }
  });

  // Remplir le résumé
  document.getElementById('summary-emoji').textContent = w.icon;
  document.getElementById('summary-name').textContent = `${w.name} terminée !`;
  document.getElementById('summary-meta').textContent = `${new Date().toLocaleDateString('fr-FR',{weekday:'long',day:'numeric',month:'long'})}${durationMin ? ' · ' + durationMin + ' min' : ''}`;
  document.getElementById('summary-volume').textContent = volume > 0 ? volume.toLocaleString('fr-FR') : '—';
  document.getElementById('summary-sets').textContent = `${doneSets}/${totalSets}`;

  // PRs
  const prsBlock = document.getElementById('summary-prs-block');
  const prsList = document.getElementById('summary-prs-list');
  if (sessionPRs.length > 0) {
    prsBlock.style.display = '';
    prsList.innerHTML = sessionPRs.map(p => `<div class="summary-pr-item">🏆 ${p}</div>`).join('');
  } else {
    prsBlock.style.display = 'none';
  }

  // Reset étoiles
  document.querySelectorAll('.star-btn').forEach(b => b.classList.remove('active'));
  document.getElementById('summary-note').value = '';

  // Afficher la page
  document.querySelectorAll('.page').forEach(p => p.classList.remove('active'));
  document.querySelectorAll('.nav-btn').forEach(b => b.classList.remove('active'));
  document.getElementById('page-summary').classList.add('active');
  currentPage = 'summary';
}

function setRating(val) {
  _currentRating = val;
  document.querySelectorAll('.star-btn').forEach(b => {
    b.classList.toggle('active', parseInt(b.dataset.val) <= val);
  });
}

function saveSessionSummary() {
  const id = _summaryWorkoutId;
  if (!id) return;

  const today = new Date().toISOString().slice(0, 10);
  const sessionKey = `${today}_${id}`;
  const durationMin = sessionStart ? Math.round((Date.now() - sessionStart) / 60000) : 0;

  // Sauvegarder dans completedSessions (compatibilité existante)
  completedSessions.push({ id, date: today, time: Date.now() });
  localStorage.setItem('completedSessions', JSON.stringify(completedSessions));

  // Sauvegarder dans sessionLog (nouveau)
  sessionLog[sessionKey] = {
    duration: durationMin,
    volume: calcVolume(id),
    rating: _currentRating,
    note: document.getElementById('summary-note').value.trim(),
    prs: sessionPRs.slice(),
    week: getCurrentWeek(),
    maxByEx: { ...sessionMaxByEx }
  };
  localStorage.setItem('sessionLog', JSON.stringify(sessionLog));

  // Réinitialiser état session
  sessionStart = null;
  sessionPRs = [];
  sessionMaxByEx = {};
  setLog = {};
  localStorage.removeItem('setLog');

  // Retour accueil
  goBack();
  renderHome();
}
```

### Étape 5 — Modifier `renderSeance()` et `completeSession()`

- [ ] Dans `renderSeance(id)` (~ligne 805), trouver `if (!setsDone[id]) setsDone[id] = {};` et ajouter **après** :

```js
  sessionStart = sessionStart || Date.now();
  sessionPRs = [];
  sessionMaxByEx = {};
```

- [ ] Trouver la fonction `completeSession(id)` (~ligne 1026) et la remplacer entièrement par :

```js
function completeSession(id) {
  showSessionSummary(id);
}
```

### Étape 6 — Vérification

- [ ] Faire une séance complète : valider quelques séries avec des poids
- [ ] Appuyer "Marquer la séance complète" → page résumé s'affiche
- [ ] Vérifier : tonnage calculé, séries comptées, PRs listés si présents
- [ ] Taper 4 étoiles + ajouter une note
- [ ] Appuyer "Enregistrer" → retour accueil, séance dans l'historique
- [ ] Recharger → séance bien persistée

### Étape 7 — Commit

```bash
git add app_entrainement.html
git commit -m "feat: add full session summary screen with volume, PRs, rating and note"
```

---

## Task 6 — Onglet Progrès : 3 sous-tabs + historique enrichi

**Files:**
- Modify: `app_entrainement.html` — CSS, HTML (`#page-progress`), JS (`renderProgress`)

### Étape 1 — CSS des sous-tabs

- [ ] Ajouter dans le CSS (avant `/* ─── TIMER PAGE ───*/`) :

```css
/* ─── PROGRESS SUB-TABS ─── */
.progress-tabs { display: flex; gap: 6px; margin-bottom: 16px; }
.ptab { flex: 1; padding: 8px 4px; border-radius: 10px; border: 1.5px solid #2a2a2a; background: var(--card); color: var(--muted); font-size: 12px; font-weight: 700; cursor: pointer; text-align: center; transition: all .15s; }
.ptab.active { border-color: var(--orange); color: var(--orange); background: rgba(232,80,10,.1); }
.session-card-enriched { background: var(--card); border-radius: var(--radius); padding: 12px 14px; margin-bottom: 8px; cursor: pointer; }
.sce-row1 { display: flex; align-items: center; gap: 10px; }
.sce-icon { font-size: 20px; }
.sce-info { flex: 1; }
.sce-name { font-size: 14px; font-weight: 700; }
.sce-meta { font-size: 11px; color: var(--muted); margin-top: 2px; }
.sce-badges { display: flex; gap: 4px; flex-wrap: wrap; margin-top: 6px; }
.sce-badge { font-size: 10px; font-weight: 700; padding: 2px 7px; border-radius: 5px; }
.sce-badge-vol { background: rgba(232,80,10,.15); color: var(--orange); }
.sce-badge-pr { background: rgba(45,189,110,.15); color: var(--green); }
.sce-badge-stars { background: rgba(255,193,7,.12); color: #FFC107; }
.sce-note { font-size: 12px; color: var(--muted); margin-top: 8px; padding-top: 8px; border-top: 1px solid #2a2a2a; display: none; font-style: italic; }
.session-card-enriched.open .sce-note { display: block; }
```

### Étape 2 — Modifier le HTML de `#page-progress`

- [ ] Trouver dans le HTML :

```html
      <h3>Plan de progression — 4 semaines</h3>
      <div id="prog-plan-section"></div>

      <h3>Séances complétées</h3>
      <div id="completed-sessions-list"></div>

      <h3>Charges enregistrées</h3>
      <div id="weight-log-list"></div>
```

Remplacer par :

```html
      <h3>Plan de progression — 4 semaines</h3>
      <div id="prog-plan-section"></div>

      <div class="progress-tabs">
        <button class="ptab active" id="ptab-sessions" onclick="switchProgressTab('sessions')">📅 Séances</button>
        <button class="ptab" id="ptab-curves" onclick="switchProgressTab('curves')">📈 Courbes</button>
        <button class="ptab" id="ptab-body" onclick="switchProgressTab('body')">⚖️ Corps</button>
      </div>
      <div id="ptab-content-sessions"></div>
      <div id="ptab-content-curves" style="display:none"></div>
      <div id="ptab-content-body" style="display:none"></div>
```

### Étape 3 — Réécrire `renderProgress()` et ajouter les sous-rendus

- [ ] Remplacer la fonction `renderProgress()` entière par :

```js
function renderProgress() {
  // Progression plan (inchangé)
  const progSection = document.getElementById('prog-plan-section');
  if (progSection) {
    progSection.innerHTML = `
      <div class="card" style="padding:0;overflow:hidden">
        <table class="prog-week-table">
          <thead><tr><th>Semaine</th><th>Charge</th><th>Séries</th><th>Repos</th><th>Objectif</th></tr></thead>
          <tbody>
            <tr><td class="prog-week-label">S1</td><td>75% 1RM</td><td>Comme indiqué</td><td>Normal</td><td>Tempos + sensations</td></tr>
            <tr><td class="prog-week-label">S2</td><td>+5%</td><td>Comme indiqué</td><td>Normal</td><td>Échec sur dernière série</td></tr>
            <tr><td class="prog-week-label">S3</td><td>+5%</td><td>+1 série/ex</td><td>−15 s</td><td>Techniques d'intensification</td></tr>
            <tr><td class="prog-week-label">S4</td><td>85–90% 1RM</td><td>+1 série/ex</td><td>−15 s</td><td>Supersets + échec systématique</td></tr>
          </tbody>
        </table>
      </div>
      <div class="prog-tech-list">
        <div class="prog-tech-item"><span class="prog-tech-name">Rest-Pause</span><span class="prog-tech-desc">Atteindre l'échec, 15 s de repos, reprendre 2-3 reps. → Développé couché, Tractions</span></div>
        <div class="prog-tech-item"><span class="prog-tech-name">Dropset</span><span class="prog-tech-desc">À l'échec, réduire charge de 20 % et reprendre immédiatement. → Curls, Extensions triceps, Leg Curl</span></div>
        <div class="prog-tech-item"><span class="prog-tech-name">Pause Squat</span><span class="prog-tech-desc">2 secondes d'arrêt en bas du squat. → Squat, Leg Press</span></div>
        <div class="prog-tech-item"><span class="prog-tech-name">Excentrique ×</span><span class="prog-tech-desc">Allonger la descente à 5-6 secondes en S4 sur tous les exercices.</span></div>
      </div>
    `;
  }
  switchProgressTab('sessions');
}

function switchProgressTab(tab) {
  ['sessions','curves','body'].forEach(t => {
    document.getElementById('ptab-' + t)?.classList.toggle('active', t === tab);
    const c = document.getElementById('ptab-content-' + t);
    if (c) c.style.display = t === tab ? '' : 'none';
  });
  if (tab === 'sessions') renderSessionsTab();
  if (tab === 'curves')   renderCurvesTab();
  if (tab === 'body')     renderBodyTab();
}

function renderSessionsTab() {
  const el = document.getElementById('ptab-content-sessions');
  if (!el) return;

  // Fusionner completedSessions et sessionLog
  const all = [...completedSessions].reverse().slice(0, 15);
  if (!all.length) {
    el.innerHTML = '<div class="empty-state"><div class="es-icon">🏋️</div><p>Aucune séance enregistrée.<br>Complète ta première séance !</p></div>';
    return;
  }
  el.innerHTML = all.map((s, i) => {
    const w = PROGRAM[s.id];
    const d = new Date(s.date);
    const label = d.toLocaleDateString('fr-FR', { weekday:'short', day:'numeric', month:'short' });
    const sKey = `${s.date}_${s.id}`;
    const sData = sessionLog[sKey] || {};
    const badges = [
      sData.volume ? `<span class="sce-badge sce-badge-vol">⚖️ ${(sData.volume).toLocaleString('fr-FR')} kg</span>` : '',
      sData.prs?.length ? `<span class="sce-badge sce-badge-pr">🏆 ${sData.prs.length} PR${sData.prs.length>1?'s':''}</span>` : '',
      sData.rating ? `<span class="sce-badge sce-badge-stars">${'⭐'.repeat(sData.rating)}</span>` : '',
    ].filter(Boolean).join('');
    const noteHtml = sData.note ? `<div class="sce-note">${sData.note}</div>` : '';
    return `<div class="session-card-enriched" onclick="this.classList.toggle('open')">
      <div class="sce-row1">
        <span class="sce-icon">${w?.icon||'💪'}</span>
        <div class="sce-info">
          <div class="sce-name">${w?.name||s.id}</div>
          <div class="sce-meta">${label}${sData.duration ? ' · ' + sData.duration + ' min' : ''}</div>
        </div>
        <div style="color:var(--green);font-size:14px;font-weight:700">✓</div>
      </div>
      ${badges ? `<div class="sce-badges">${badges}</div>` : ''}
      ${noteHtml}
    </div>`;
  }).join('');
}
```

### Étape 4 — Vérification

- [ ] Aller dans l'onglet Progrès → 3 tabs visibles : Séances / Courbes / Corps
- [ ] Tab Séances : séances existantes avec badges tonnage, PRs, étoiles
- [ ] Taper sur une séance avec note → expand → note visible
- [ ] Tabs Courbes et Corps : ne crashent pas (fonctions appelées, contenu vide)

### Étape 5 — Commit

```bash
git add app_entrainement.html
git commit -m "feat: redesign Progrès tab with 3 sub-tabs and enriched session history"
```

---

## Task 7 — Sub-tab Courbes (graphiques SVG)

**Files:**
- Modify: `app_entrainement.html` — CSS (chart), JS (`renderCurvesTab`, `drawChart`)

### Étape 1 — CSS des graphiques

- [ ] Ajouter dans le CSS (avant `/* ─── TIMER PAGE ───*/`) :

```css
/* ─── CHART ─── */
.chart-card { background: var(--card); border-radius: var(--radius); padding: 14px; margin-bottom: 12px; }
.chart-card-title { font-size: 13px; font-weight: 800; color: var(--orange); margin-bottom: 10px; display: flex; align-items: center; gap: 6px; }
.chart-empty { font-size: 12px; color: var(--muted); text-align: center; padding: 20px 0; }
```

### Étape 2 — Fonctions `drawChart` et `renderCurvesTab`

- [ ] Ajouter **après** `renderSessionsTab()` (Task 6) :

```js
function drawChart(containerId, points, color) {
  const el = document.getElementById(containerId);
  if (!el) return;
  if (points.length < 2) {
    el.innerHTML = '<div class="chart-empty">Complète au moins 2 séances pour voir la courbe</div>';
    return;
  }
  const W = 300, H = 90;
  const PAD = { top: 18, right: 12, bottom: 22, left: 34 };
  const kgs = points.map(p => p.kg);
  const minKg = Math.min(...kgs) * 0.95;
  const maxKg = Math.max(...kgs) * 1.02;
  const maxKgVal = Math.max(...kgs);
  const xS = i => PAD.left + (i / (points.length - 1)) * (W - PAD.left - PAD.right);
  const yS = kg => PAD.top + (1 - (kg - minKg) / (maxKg - minKg)) * (H - PAD.top - PAD.bottom);

  const path = points.map((p, i) => `${i===0?'M':'L'}${xS(i).toFixed(1)},${yS(p.kg).toFixed(1)}`).join(' ');

  const dots = points.map((p, i) => {
    const isPR = p.kg >= maxKgVal;
    const cx = xS(i).toFixed(1), cy = yS(p.kg).toFixed(1);
    const dateLbl = p.date.slice(5).replace('-','/');
    return `${isPR ? `<text x="${cx}" y="${(parseFloat(cy)-9).toFixed(1)}" fill="#2DBD6E" font-size="8" text-anchor="middle" font-weight="bold">PR</text>` : ''}
    <circle cx="${cx}" cy="${cy}" r="${isPR?5:3}" fill="${isPR?'#2DBD6E':color}"/>
    <text x="${cx}" y="${H-3}" fill="#555" font-size="8" text-anchor="middle">${dateLbl}</text>`;
  }).join('');

  const yMid = ((minKg + maxKg) / 2);
  const yLabels = [minKg, yMid, maxKg].map(kg => {
    const y = yS(kg);
    return `<text x="${PAD.left-3}" y="${y.toFixed(1)}" fill="#555" font-size="8" text-anchor="end" dominant-baseline="middle">${Math.round(kg)}</text>`;
  }).join('');

  el.innerHTML = `<svg viewBox="0 0 ${W} ${H}" width="100%" style="overflow:visible;display:block">
    <line x1="${PAD.left}" y1="${PAD.top}" x2="${PAD.left}" y2="${H-PAD.bottom}" stroke="#2a2a2a" stroke-width="1"/>
    <line x1="${PAD.left}" y1="${H-PAD.bottom}" x2="${W-PAD.right}" y2="${H-PAD.bottom}" stroke="#2a2a2a" stroke-width="1"/>
    ${yLabels}
    <path d="${path}" fill="none" stroke="${color}" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"/>
    ${dots}
  </svg>`;
}

function renderCurvesTab() {
  const el = document.getElementById('ptab-content-curves');
  if (!el) return;

  const KEY_EXERCISES = [
    { name: 'Développé couché à la barre',     color: '#E8500A', icon: '🏋' },
    { name: 'Squat barre libre',                color: '#2A7A2A', icon: '🦵' },
    { name: 'Soulevé de terre barre',           color: '#8A1A8A', icon: '⚡' },
    { name: 'Tractions lestées (ou Tirage Poitrine)', color: '#1A4D8A', icon: '🔥' },
    { name: 'Développé militaire debout',       color: '#E8500A', icon: '💪' },
    { name: 'Rowing buste penché à la barre',   color: '#1A4D8A', icon: '🏃' },
  ];

  // Construire les points depuis sessionLog
  el.innerHTML = KEY_EXERCISES.map((ex, idx) => {
    const points = Object.entries(sessionLog)
      .map(([key, s]) => ({ date: key.split('_')[0], kg: s.maxByEx?.[ex.name]?.kg || 0 }))
      .filter(p => p.kg > 0)
      .sort((a, b) => a.date.localeCompare(b.date));

    return `<div class="chart-card">
      <div class="chart-card-title">${ex.icon} ${ex.name.split(' ').slice(0,3).join(' ')}</div>
      <div id="chart-${idx}"></div>
    </div>`;
  }).join('');

  // Rendu SVG après injection HTML
  KEY_EXERCISES.forEach((ex, idx) => {
    const points = Object.entries(sessionLog)
      .map(([key, s]) => ({ date: key.split('_')[0], kg: s.maxByEx?.[ex.name]?.kg || 0 }))
      .filter(p => p.kg > 0)
      .sort((a, b) => a.date.localeCompare(b.date));
    drawChart(`chart-${idx}`, points, ex.color);
  });
}
```

### Étape 3 — Vérification

- [ ] Compléter 2+ séances avec des poids différents pour Développé couché
- [ ] Onglet Progrès → Courbes → graphique visible avec courbe orange
- [ ] Dernier point en vert si c'est un PR
- [ ] Exercices sans données : message "Complète au moins 2 séances"
- [ ] Pas d'erreur console

### Étape 4 — Commit

```bash
git add app_entrainement.html
git commit -m "feat: add SVG progression charts for 6 key exercises in Courbes tab"
```

---

## Task 8 — Sub-tab Corps (poids corporel & mensurations)

**Files:**
- Modify: `app_entrainement.html` — CSS, JS (`renderBodyTab`, `saveBodyEntry`)

### Étape 1 — CSS du tab Corps

- [ ] Ajouter dans le CSS (avant `/* ─── TIMER PAGE ───*/`) :

```css
/* ─── BODY TAB ─── */
.body-form { background: var(--card); border-radius: var(--radius); padding: 14px; margin-bottom: 14px; }
.body-form-title { font-size: 13px; font-weight: 800; margin-bottom: 12px; }
.body-fields { display: grid; grid-template-columns: 1fr 1fr; gap: 8px; margin-bottom: 12px; }
.body-field { display: flex; flex-direction: column; gap: 4px; }
.body-field label { font-size: 10px; color: var(--muted); font-weight: 700; text-transform: uppercase; }
.body-field input { background: #252525; border: 1.5px solid #333; border-radius: 8px; color: var(--text); font-size: 15px; padding: 8px 10px; text-align: center; }
.body-field input:focus { outline: none; border-color: var(--orange); }
.body-save-btn { width: 100%; background: var(--orange); color: #fff; border: none; border-radius: 10px; padding: 11px; font-size: 14px; font-weight: 800; cursor: pointer; }
.body-history { background: var(--card); border-radius: var(--radius); overflow: hidden; }
.body-history-row { display: flex; align-items: center; gap: 8px; padding: 10px 14px; border-bottom: 1px solid #2a2a2a; font-size: 13px; }
.body-history-row:last-child { border-bottom: none; }
.body-hist-date { color: var(--muted); font-size: 11px; min-width: 60px; }
.body-hist-val { font-weight: 800; color: var(--orange); }
.body-hist-extra { color: var(--muted); font-size: 11px; }
```

### Étape 2 — Fonctions du tab Corps

- [ ] Ajouter après `renderCurvesTab()` (Task 7) :

```js
function renderBodyTab() {
  const el = document.getElementById('ptab-content-body');
  if (!el) return;

  const recentHistory = [...bodyLog].reverse().slice(0, 10);
  const histHtml = recentHistory.length
    ? recentHistory.map(e => `
        <div class="body-history-row">
          <span class="body-hist-date">${e.date}</span>
          <span class="body-hist-val">${e.weight} kg</span>
          <span class="body-hist-extra">${[
            e.waist ? `taille ${e.waist}cm` : '',
            e.arm   ? `bras ${e.arm}cm`   : '',
            e.thigh ? `cuisse ${e.thigh}cm` : ''
          ].filter(Boolean).join(' · ')}</span>
        </div>`).join('')
    : '<div style="padding:14px;text-align:center;font-size:13px;color:var(--muted)">Aucune entrée</div>';

  el.innerHTML = `
    <div class="body-form">
      <div class="body-form-title">⚖️ Enregistrer aujourd'hui</div>
      <div class="body-fields">
        <div class="body-field" style="grid-column:1/-1">
          <label>Poids corporel (kg) *</label>
          <input id="bf-weight" type="number" step="0.1" min="30" max="200" placeholder="82.5" inputmode="decimal">
        </div>
        <div class="body-field">
          <label>Tour de taille (cm)</label>
          <input id="bf-waist" type="number" step="0.5" placeholder="84" inputmode="decimal">
        </div>
        <div class="body-field">
          <label>Bras fléchi (cm)</label>
          <input id="bf-arm" type="number" step="0.5" placeholder="40" inputmode="decimal">
        </div>
        <div class="body-field">
          <label>Cuisse (cm)</label>
          <input id="bf-thigh" type="number" step="0.5" placeholder="58" inputmode="decimal">
        </div>
      </div>
      <button class="body-save-btn" onclick="saveBodyEntry()">Enregistrer</button>
    </div>
    <h3 style="margin:0 0 8px">Historique</h3>
    <div class="body-history">${histHtml}</div>
    <div id="body-chart-wrap" style="margin-top:14px">
      <div class="chart-card">
        <div class="chart-card-title">⚖️ Poids corporel</div>
        <div id="body-weight-chart"></div>
      </div>
    </div>
  `;

  // Graphique poids corporel
  const weightPoints = [...bodyLog]
    .filter(e => e.weight > 0)
    .slice(-20)
    .map(e => ({ date: e.date, kg: e.weight }));
  drawChart('body-weight-chart', weightPoints, '#3A8EE8');
}

function saveBodyEntry() {
  const weight = parseFloat(document.getElementById('bf-weight')?.value) || 0;
  if (!weight) { alert('Le poids est obligatoire.'); return; }
  const entry = {
    date: new Date().toLocaleDateString('fr-FR', { day:'numeric', month:'short' }),
    weight,
    waist: parseFloat(document.getElementById('bf-waist')?.value) || null,
    arm:   parseFloat(document.getElementById('bf-arm')?.value)   || null,
    thigh: parseFloat(document.getElementById('bf-thigh')?.value) || null,
  };
  bodyLog.push(entry);
  localStorage.setItem('bodyLog', JSON.stringify(bodyLog));
  renderBodyTab();
}
```

### Étape 3 — Vérification

- [ ] Onglet Progrès → Corps → formulaire visible avec 4 champs
- [ ] Entrer 83.2 kg + taille 84 → Enregistrer → apparaît dans l'historique
- [ ] Entrer une 2ème valeur différente → graphique de poids affiché
- [ ] Recharger → données persistées

### Étape 4 — Commit final

```bash
git add app_entrainement.html
git commit -m "feat: add body weight and measurements tracking with chart in Corps tab"
```

---

## Auto-review du plan

**Couverture spec :**
- ✅ Suivi par série (modal slide-up) → Task 2
- ✅ Timer repos auto-démarrage → Task 3
- ✅ Semaine S1-S4 + date départ + override → Task 4
- ✅ Badge cible semaine sur exercices → Task 4 Étape 7
- ✅ Résumé fin de séance (volume, PRs, étoiles, note) → Task 5
- ✅ Détection PR (pendant saisie série) → Task 2 `detectPR`
- ✅ Graphiques Progrès > Courbes → Task 7
- ✅ Poids corporel + mensurations → Task 8
- ✅ Onglet Progrès 3 sous-tabs → Task 6
- ✅ Historique séances enrichi (durée, volume, stars) → Task 6

**Cohérence des types :**
- `setLog` clé format `workoutId_exIdx_setIdx` → cohérent Tasks 1, 2
- `sessionLog` clé format `YYYY-MM-DD_workoutId` → cohérent Tasks 1, 5, 7
- `sessionMaxByEx[exName] = { kg, reps }` → cohérent Tasks 1, 2, 5
- `detectPR(exName, kg, reps)` → signature cohérente Tasks 1, 2
- `drawChart(containerId, points, color)` avec `points = [{date, kg}]` → cohérent Tasks 7, 8
- `switchProgressTab(tab)` appelé depuis HTML et `renderProgress()` → cohérent Task 6

**Pas de placeholders :** aucun TBD, TODO ou "implémenter plus tard".
