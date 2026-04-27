# Synthèse exhaustive du projet — Denis · Force & Définition
*Générée le 13 avril 2026*

---

## 1. Contexte et objectifs

Le projet a été initié par Denis (dsm972@gmail.com) avec une demande claire : concevoir un programme d'entraînement personnel complet, **niveau avancé**, ciblant **force et définition musculaire** — sans prise de masse — et le rendre utilisable au quotidien via une application mobile.

Contraintes définies lors des échanges initiaux :

- Niveau : **avancé**
- Fréquence : **4 jours d'entraînement par semaine**
- Lieux : **salle de sport + maison** (flexibilité)
- Cardio : **HIIT + cardio intégré aux séances** (pas de cardio long/aérobique)
- Durée du programme : **1 mois (4 semaines)**
- Philosophie : intensité maximale, développement du cardio, définition — jamais de prise de masse

---

## 2. Livrables produits

### 2.1 `Programme_Force_Definition_Denis.docx`
Document Word complet (~25 Ko) généré avec la bibliothèque Node.js `docx`. Contenu :

- Page de couverture personnalisée
- Section principes (méthodologie Nassim Sahili / Fitmass)
- 4 fiches de séances détaillées avec tableaux (exercice · séries · reps · tempo · repos)
- Calendrier de progression sur 4 semaines (+5 % de charge par semaine)
- Section nutrition (principes généraux force/définition)

### 2.2 `app_entrainement.html`
Application web mobile complète (~70 Ko, fichier HTML unique). C'est le livrable principal.

---

## 3. Programme d'entraînement

### Structure hebdomadaire

| Jour | Séance | Groupes musculaires | Durée | Lieu |
|------|--------|---------------------|-------|------|
| Lundi | **PUSH** 💪 | Poitrine · Épaules · Triceps | ~70 min | Salle |
| Mardi | **PULL** 🔥 | Dos · Biceps · Trapèzes | ~70 min | Salle/Maison |
| Mercredi | Repos actif 🚶 | Marche · Étirements · Mobilité | 30 min | — |
| Jeudi | **JAMBES + CORE** 🦵 | Quadriceps · Ischios · Fessiers · Mollets | ~80 min | Salle |
| Vendredi | Repos total 😴 | Récupération | — | — |
| Samedi | **FULL BODY** ⚡ | Polyarticulaires complets | ~75 min | Salle/Maison |
| Dimanche | Repos total 😴 | Récupération | — | — |

### Exercices par séance (27 exercices au total)

**PUSH — Lundi** (7 exercices + HIIT 15 min)
1. Développé couché à la barre — 5×4-5 reps — tempo 4-0-1-0 — 3 min repos *(priorité force)*
2. Développé incliné aux haltères 70° — 4×8 — 3-1-1-0 — 90 s
3A. Développé militaire debout — 4×6 — 3-0-1-0 *(superset)*
3B. Élévations latérales haltères — 4×12 — 2-0-1-1 — 90 s
4A. Dips lestés — 3×8-10 — 3-0-1-0 *(superset)*
4B. Écarté poulie basse — 3×12 — 2-1-1-1 — 75 s
5. Extensions triceps poulie corde — 3×12 — 2-1-1-0 — 60 s
→ HIIT : 30s/15s × 10 rounds — Burpees, Mountain Climbers, Box Jumps, Battle Ropes

**PULL — Mardi** (7 exercices + HIIT 12 min)
1. Tractions lestées — 5×5-6 — 3-0-1-1 — 3 min *(exercice roi)*
2. Rowing buste penché à la barre — 4×8 — 3-0-1-1 — 90 s
3A. Tirage horizontal poulie basse — 4×10 — 2-1-1-0 *(superset)*
3B. Face Pulls poulie haute — 4×15 — 2-1-1-1 — 75 s
4A. Curl Larry Scott barre EZ — 4×8 — 3-2-1-0 *(superset)*
4B. Curl incliné haltères — 4×10 — 3-1-1-0 — 90 s
5. Shrugs haltères — 3×15 — 1-2-1-0 — 60 s
→ HIIT : 4 rounds — Pull-ups, Corde à sauter 45s, Renegade Rows

**JAMBES + CORE — Jeudi** (7 exercices + 4 exercices core + HIIT 15 min)
1. Squat barre libre — 5×4-5 — 4-1-1-0 — 3 min *(priorité force)*
2. Soulevé de terre roumain — 4×8 — 3-1-1-0 — 2 min
3. Presse à cuisses pieds hauts — 4×10 — 3-0-1-0 — 90 s
4A. Fentes marchées haltères — 3×12/jambe *(superset)*
4B. Leg Curl allongé — 3×12 — 75 s
5. Extensions de jambes — 3×15 — 2-0-1-1 — 60 s
6. Mollets debout lestés — 4×15 — 2-2-1-0 — 45 s
Core : Relevé de jambes suspendu · Gainage dynamique · V-Up · Russian Twists
→ HIIT : Vélo 30s sprint/30s récup × 8 rounds

**FULL BODY — Samedi** (6 exercices + HIIT 20 min)
1. Soulevé de terre barre — 5×3-4 — 3-0-1-0 — 4 min *(force absolue maximale)*
2A. Développé couché haltères — 4×8 *(superset)*
2B. Tractions prise supination — 4×8 — 2 min
3A. Squat Gobelet haltère lourd — 3×12 *(superset)*
3B. Pompes explosives (Clap Push-ups) — 3×10
4. Turkish Get-Up — 3×5/côté — 90 s
→ HIIT métabolique : 6 ex × 40s/20s — 3 rounds — Burpees, KB Swings, Mountain Climbers, Box Jumps, Dips, Sprint

### Méthodologie appliquée

Issue des travaux de **Nassim Sahili (Fitmass)** :

- **Tempo à 4 chiffres** : contraction excentrique — pause basse — contraction concentrique — pause haute. Ex : `4-0-1-0` = 4 s en descendant, 0 s en bas, 1 s en montant, 0 s en haut.
- **Supersets numérotés** (3A/3B) : enchaînement sans repos entre deux exercices antagonistes ou complémentaires, pour densifier l'entraînement et maintenir le cardio.
- **Principe HIIT < 15 min** : les séquences cardio à haute intensité sont courtes et finales, pour ne pas empiéter sur la récupération neuromusculaire (anti-catabolique).
- **Progression linéaire** : +5 % de charge chaque semaine sur 4 semaines.

---

## 4. Application mobile — Architecture technique

### Stack technologique

- **Langage** : HTML5/CSS3/JavaScript vanilla (zéro dépendance externe)
- **Format** : fichier unique `.html` — PWA-ready (meta tags Apple Mobile Web App, theme-color)
- **Responsive** : max-width 430 px, centré, optimisé mobile
- **Design** : thème sombre (dark mode natif)
- **Persistance** : `localStorage` — données survivent entre sessions

### Design system (CSS custom properties)
```
--orange: #E8500A      → accent principal, PUSH
--bg: #141414          → fond global
--card: #1E1E1E        → cartes
--green: #2DBD6E       → validations
--blue: #3A8EE8        → PULL, maison
--purple: #9B3AE8      → FULL BODY, tempos
--muted: #888          → textes secondaires
```

### Navigation
4 onglets en barre fixe basse :
- **Accueil** — tableau de bord du jour
- **Entraînement** — accès aux séances
- **Progression** — historique & logs
- **Minuteur** — minuteur repos & HIIT

### Fonctionnalités implémentées

**Accueil dynamique**
- Bannière "Séance du jour" calculée automatiquement selon le jour de la semaine
- Raccourci direct vers la séance courante
- Streak d'entraînement (jours consécutifs)

**Séances**
- Fiche complète de chaque séance avec tous les exercices
- Suivi des séries : boutons numérotés par série, cochage individuel (set 1 / set 2 / set 3…)
- Auto-check de l'exercice quand toutes les séries sont complétées
- Barre de progression générale de la séance
- Labels supersets visuels ("⚡ SUPERSET — Enchaîner sans repos")
- Indication salle/maison par code couleur (orange = salle, bleu = maison)
- Log de charge : saisie du poids utilisé pour chaque exercice, sauvegardé en `localStorage`
- Bouton de fin de séance avec animation de validation

**GIFs démonstratifs**
- Chargement automatique d'un GIF animé par exercice depuis l'API ExerciseDB (`exercisedb.dev`)
- Dictionnaire de 27 traductions français → anglais (termes de recherche API)
- Fallback CORS via proxy `allorigins.win` si le fetch direct échoue
- Timeout de 6 s (via `AbortController` — compatible tous navigateurs)
- GIF cliquable → modal plein écran avec agrandissement
- En cas d'échec : bouton "▶ Voir sur YouTube" avec recherche automatique
- Chargement asynchrone décalé (300 ms entre exercices) pour ne pas saturer l'API

**Minuteur repos**
- Minuteur modal avec compte à rebours SVG animé (cercle avec `stroke-dashoffset`)
- Sons de fin de countdown via `AudioContext` (bip synthétique, sans fichier audio externe)
- Préréglages rapides : 45 s, 60 s, 90 s, 2 min, 3 min
- +30 s / +60 s en cours de décompte
- Passe-partout (utilisable depuis n'importe quelle page)

**Timer HIIT**
- Deux modes : Repos · HIIT
- Configuration personnalisable : temps de travail, temps de repos, nombre de rounds
- Présets enregistrés : Tabata (20/10), Standard (30/30), Intensif (40/20)
- Affichage temps restant dans la phase en cours + numéro de round
- Indicateur phase : TRAVAIL (vert) / REPOS (bleu)
- Sons distincts au changement de phase

**Progression & historique**
- Historique de toutes les séances complétées (date + séance)
- Log de charges par exercice — évolution semaine par semaine
- Expandable par séance

---

## 5. Problèmes rencontrés et solutions

### Bug 1 — Erreur de syntaxe JavaScript (`\!`)
**Cause** : lors de l'injection du code GIF via un script Python exécuté depuis un heredoc bash, le shell a échappé tous les `!` en `\!`. Le code JS contenait donc `\!==`, `\!res.ok`, `\!area`, `\!w`, etc.
**Symptôme** : `Uncaught SyntaxError: Invalid or unexpected token` au chargement.
**Fix** : remplacement de tous les `\!` par `!` via `chr(92)+chr(33)` en Python (pour contourner le même problème d'échappement dans le shell).

### Bug 2 — `gifAreaHtml is not defined`
**Cause** : l'injection automatique du système de GIFs avait placé la référence `${gifAreaHtml}` dans le template literal de `makeExCard()`, mais avait omis de déclarer la variable elle-même dans la fonction.
**Fix** : ajout de la ligne dans `makeExCard()`, juste avant `card.innerHTML = \`...\`` :
```javascript
const gifAreaHtml = `<div class="ex-gif-area"><div class="gif-loading" id="gif-area-${workoutId}-${exIdx}"></div></div>`;
```

### Bug 3 — `AbortSignal.timeout()` non supporté
**Cause** : API trop récente, non disponible sur Safari et anciens navigateurs.
**Fix** : remplacé par un pattern `AbortController` + `setTimeout` + `clearTimeout`.

### Contrainte réseau sandbox
L'API ExerciseDB est inaccessible depuis le sandbox Linux (liste d'autorisation réseau restrictive). Les tests de connectivité retournent HTTP 000. Ceci est une limitation de l'environnement de développement — l'API fonctionne normalement depuis le navigateur de l'utilisateur.

---

## 6. État actuel du projet

| Composant | Statut |
|-----------|--------|
| Document Word `.docx` | ✅ Complet |
| Application HTML — Structure & navigation | ✅ Complet |
| Application HTML — Suivi séances & sets | ✅ Complet |
| Application HTML — Minuteur repos | ✅ Complet |
| Application HTML — Timer HIIT | ✅ Complet |
| Application HTML — Log de progression | ✅ Complet |
| Application HTML — Système GIFs | ✅ Code en place, dépend de l'accès réseau |
| Bug syntaxe `\!` | ✅ Corrigé |
| Bug `gifAreaHtml` non défini | ✅ Corrigé |
| Bug `AbortSignal.timeout` | ✅ Corrigé |
| Test GIFs en conditions réelles | ⏳ À valider par Denis en ouvrant l'app |

---

## 7. Ressources présentes dans le dossier

En dehors des livrables, le dossier `Musculation` contient une bibliothèque de référence :

- Frédéric Delavier — *Guide de musculation des bras*
- Frédéric Delavier — *La Méthode Delavier* Vol 1, 2, 3
- *Gainage et musculation profonde*
- *Guide des Mouvements de Musculation*
- *Je pratique la musculation — Du débutant au pratiquant confirmé*
- *La bible de la musculation au poids de corps* Tome 1 & 2
- *Méthode de Musculation — Optimisation Turbo*
- *Méthode de musculation — 110 exercices sans matériel*
- *Musculation du Paresseux*
- **Nassim Sahili — Fitmass Basic/Advanced/Elite — Perte de graisse** *(source principale du programme)*

---

## 8. Prochaines étapes suggérées

1. **Tester les GIFs** : ouvrir `app_entrainement.html` dans un navigateur avec accès internet et vérifier que les GIFs se chargent pour chaque exercice.
2. **Ajout d'une 4e semaine de progression** : le programme est décrit sur 4 semaines dans le `.docx` mais l'app affiche toujours le même programme — une évolution possible serait d'intégrer la notion de semaine courante avec les charges cibles adaptées.
3. **Export / partage** : bouton pour exporter l'historique de progression en CSV ou PDF.
4. **Mode hors-ligne** : transformer en vraie PWA (Service Worker + manifest.json) pour installation sur écran d'accueil et usage sans connexion.
