# Sound_Bath_Assistant# Sound Therapy Assistant — Roadmap Projet

> Document de cadrage révisé — Mars 2026
> Statut : Pré-développement

---

## 1. Contexte & Profil Utilisateur

### Qui est l'utilisateur ?
- Praticien(ne) de sonothérapie
- Utilise principalement des **bols tibétains** et **bols en cristal**
- Instruments secondaires : flûtes, guitare, wayunki, koshi, tingshas
- Possède **10 à 25 instruments**
- Prépare ses séances **avant** la session (pas en live)
- Utilise un **smartphone** (iOS et Android)

### Problème principal à résoudre
Aujourd'hui, le praticien :
1. Frappe un bol
2. Utilise un tuner générique pour mesurer la fréquence
3. Note le résultat manuellement
4. Compare mentalement les fréquences entre bols
5. Essaie d'estimer le battement binaural à l'oreille

**Ce workflow est lent, imprécis et frustrant.**

### Ce que l'app doit faire
Automatiser cette chaîne : **mesurer → stocker → comparer → recommander**

---

## 2. Architecture Produit — Les 3 Piliers

Le projet est découpé en **3 piliers fonctionnels** par ordre de priorité :

### Pilier 1 — Mesure & Pairing (MVP)
> *Le cœur de l'app. Sans ça, rien d'autre n'a de sens.*

| Fonction | Description |
|----------|-------------|
| Frequency Analyzer | Frappe un bol → affiche fréquence, note, octave, cents, confiance |
| Bowl Library | Enregistre chaque instrument avec sa fréquence mesurée |
| Bowl Pairing | Sélectionne 2 bols → calcule le beat → classifie en bande cérébrale |
| Pairing Suggestions | À partir d'un bol, suggère les meilleurs partenaires dans ta collection |

### Pilier 2 — Exploration Harmonique (V2)
> *Pour approfondir la compréhension musicale et thérapeutique.*

| Fonction | Description |
|----------|-------------|
| Harmonic Suggestions | Suggère des intervalles compatibles (quinte, tierce, etc.) |
| Scale Explorer | Bibliothèque de gammes pertinentes pour la sonothérapie |
| Scale Playback | Écouter les gammes (synthèse sine wave) |

### Pilier 3 — Session Design (V3)
> *Pour structurer des sound baths complets.*

| Fonction | Description |
|----------|-------------|
| Session Builder | Créer une timeline de séance avec des phases |
| Session Templates | Modèles prédéfinis (relaxation, ancrage, méditation profonde) |
| Session History | Historique et notes post-séance |

---

## 3. MVP — Spécifications Détaillées

### 3.1 Frequency Analyzer

**Input** : signal microphone du smartphone

**Output** :
```
Fréquence :  349.2 Hz
Note :       F4
Déviation :  +8 cents
Confiance :  92%
```

**Défis techniques spécifiques aux bols chantants** :

| Défi | Explication | Solution envisagée |
|------|-------------|-------------------|
| Harmoniques dominantes | Un bol tibétain peut avoir une harmonique plus forte que sa fondamentale | Analyse multi-pitch : détecter les N pics les plus forts, identifier la fondamentale par analyse des rapports harmoniques |
| Sustain long | Le son dure 10-30 secondes avec une fréquence qui évolue | Analyse en fenêtres glissantes avec moyennage |
| Bruit ambiant | Préparation dans un espace pas toujours silencieux | Seuil de confiance minimum + indication visuelle "trop de bruit" |
| Battements internes | Certains bols produisent des battements avec eux-mêmes | Détection et affichage des fréquences multiples |

**Algorithme recommandé** :
- Approche hybride : FFT pour l'analyse spectrale globale + autocorrélation/YIN pour affiner la fondamentale
- Résolution cible : ±1 Hz (suffisant pour les calculs de battements binauraux)
- Taux de rafraîchissement : 5-10 Hz (préparation, pas du live)

**UX** :
- Écran unique, bouton "Mesurer" proéminent
- Affichage en gros de la note et fréquence
- Jauge visuelle pour les cents (flat ← → sharp)
- Bouton "Sauvegarder dans ma collection"

### 3.2 Bowl Library

**Modèle de données révisé** :

```
Bowl {
  id              : UUID
  name            : string        — ex: "Grand bol tibétain doré"
  type            : enum          — tibetan | crystal | gong | tuning_fork | other
  material        : string        — ex: "7 métaux", "Quartz", "Bronze"
  size            : number (cm)   — diamètre
  note            : string        — ex: "F4"
  frequency       : number (Hz)   — fondamentale mesurée
  overtones       : number[] (Hz) — harmoniques détectées (optionnel)
  cents_deviation : number        — écart en cents par rapport à la note pure
  chakra          : string?       — association chakra (optionnel)
  photo           : image?        — photo de l'instrument (optionnel)
  notes           : string        — notes libres du praticien
  created_at      : datetime
  updated_at      : datetime
}
```

**Pourquoi ce modèle est meilleur que l'original** :
- `type` avec enum → permet de filtrer et d'adapter l'UI
- `overtones` → crucial pour les bols tibétains qui ont des harmoniques complexes
- `cents_deviation` → savoir si un bol est "juste" ou décalé
- `chakra` → demande fréquente des praticiens (même si subjectif)
- `photo` → reconnaître visuellement ses bols rapidement
- `notes` → champ libre pour le praticien

**UX** :
- Liste scrollable avec tri par note / type / nom
- Fiche instrument avec toutes les infos
- Bouton "Mesurer à nouveau" depuis la fiche
- Recherche rapide

### 3.3 Bowl Pairing & Binaural Beat Calculator

**C'est LA fonctionnalité prioritaire.**

**Mode 1 — Comparaison manuelle** :
Sélectionner 2 bols de sa collection → afficher le résultat

```
Bol A :  Crystal F4  — 349 Hz
Bol B :  Tibétain F4 — 355 Hz
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Battement :  6 Hz
Bande :      Theta (4-8 Hz)
Effet :      Relaxation profonde / méditation
```

**Mode 2 — Matrice de compatibilité** :
Vue tableau montrant TOUS les pairings possibles dans la collection.

```
              Crystal F4   Tibétain C3   Crystal A4   ...
Crystal F4      —           Delta(3Hz)   Beta(15Hz)
Tibétain C3   Delta(3Hz)      —          Alpha(9Hz)
Crystal A4    Beta(15Hz)   Alpha(9Hz)       —
```

Avec code couleur :
- 🟣 Delta (0.5-4 Hz) — sommeil profond
- 🔵 Theta (4-8 Hz) — méditation
- 🟢 Alpha (8-12 Hz) — relaxation
- 🟡 Beta (12-30 Hz) — attention
- 🔴 Gamma (30-100 Hz) — stimulation cognitive

**Mode 3 — Recommandation intelligente** :
"Je veux un effet Theta" → l'app montre les paires de bols qui produisent 4-8 Hz.

**UX** :
- Accès rapide depuis la fiche d'un bol : "Comparer avec..."
- Matrice accessible depuis la bibliothèque
- Filtrage par bande cérébrale cible

### 3.4 Calibration & Gestion du Bruit

Fonctionnalité souvent oubliée, mais essentielle :

- **Calibration micro** : test au démarrage avec un son de référence (optionnel)
- **Détection bruit ambiant** : mesure du niveau de bruit avant analyse → alerte si trop élevé
- **Seuil de confiance** : si la confiance < 70%, afficher "Résultat incertain — réessayez dans un endroit plus calme"

---

## 4. Choix Techniques

### Framework Mobile

**Recommandation : React Native avec Expo**

| Critère | React Native + Expo | Flutter |
|---------|-------------------|---------|
| Cross-platform | ✅ iOS + Android | ✅ iOS + Android |
| Écosystème audio | ✅ expo-av, react-native-audio-api | ⚠️ Moins mature |
| Modules natifs | ✅ Expo Modules API pour le DSP | ✅ Platform channels |
| Familiarité dev JS/TS | ✅ Probable pour un dev web | ⚠️ Dart à apprendre |
| Claude Code compat | ✅ Excellent support JS/TS | ✅ Bon support Dart |
| Hot reload | ✅ | ✅ |

**Attention** : le traitement audio temps réel (FFT, pitch detection) devra probablement passer par un **module natif** (Swift/Kotlin) pour les performances. React Native sert de shell UI, pas de moteur DSP.

### Architecture Audio

```
┌─────────────────────────────────────────────┐
│              React Native UI                 │
│         (écrans, navigation, state)          │
├─────────────────────────────────────────────┤
│           JS Bridge / Expo Module            │
├─────────────────────────────────────────────┤
│         Module Audio Natif                   │
│   ┌──────────┐  ┌────────────┐             │
│   │ Capture   │→│ FFT        │             │
│   │ Micro     │  │ + YIN      │             │
│   └──────────┘  └──────┬─────┘             │
│                        ↓                    │
│              Pitch Detection                │
│              Multi-harmonique               │
│                        ↓                    │
│              {freq, note, cents,            │
│               confidence, overtones}         │
└─────────────────────────────────────────────┘
```

### Stockage Local

**SQLite** (via expo-sqlite) pour :
- Collection de bols
- Pas de backend nécessaire → 100% offline

### Bibliothèques Envisagées

| Besoin | Option |
|--------|--------|
| FFT natif iOS | Accelerate / vDSP |
| FFT natif Android | TarsosDSP ou implémentation custom |
| Audio capture | expo-av ou react-native-audio-api |
| Base de données | expo-sqlite |
| Synthèse audio (gammes V2) | Web Audio API ou Tone.js |
| Navigation | expo-router |
| UI | React Native + bibliothèque minimaliste (nativewind ou custom) |

---

## 5. Gammes — Scope Réduit pour la Sonothérapie

Le document original listait 30+ gammes. C'est trop. Voici un scope adapté aux praticiens :

### Gammes Essentielles (MVP V2)

| Gamme | Pourquoi elle est pertinente |
|-------|------------------------------|
| Pentatonique majeure | La gamme "universelle" de la relaxation — pas de tension |
| Pentatonique mineure | Introspection, douceur mélancolique |
| Gamme majeure (Ionien) | Référence de base |
| Dorien | Souvent utilisé en musique méditative |
| Aeolien (mineur naturel) | Profondeur émotionnelle |
| Phrygien | Atmosphère méditative orientale |

### Gammes Secondaires (ajout ultérieur)

| Gamme | Pourquoi |
|-------|----------|
| Hirajoshi | Sonorités japonaises zen |
| Phrygien dominant | Sonorités moyen-orientales |
| Gamme par tons | Effet planant, sans ancrage |
| In-Sen | Minimalisme japonais |

### À valider avec tes collègues praticiens
> "Quelles gammes utilisez-vous consciemment dans vos séances ?"
> Si la réponse est "aucune, je fais à l'oreille", alors la feature gammes est secondaire.

---

## 6. Roadmap par Phases

### Phase 0 — Prototype Technique (2-3 semaines)
**Objectif** : Valider que la détection de fréquence fonctionne correctement sur des bols chantants.

- [ ] Setup projet React Native + Expo
- [ ] Capture audio microphone
- [ ] Implémentation FFT basique (JS d'abord, natif si trop lent)
- [ ] Test sur bol tibétain réel → la fréquence affichée est-elle correcte ?
- [ ] Test sur bol cristal réel → idem
- [ ] Documenter les résultats et ajuster l'algorithme

**Livrable** : Un écran brut qui affiche la fréquence quand on frappe un bol.
**Critère de succès** : Précision ±2 Hz sur 5 bols différents.

> ⚠️ **C'est le risque technique #1 du projet.** Si la détection ne marche pas bien sur smartphone avec des bols réels, tout le reste est compromis. Ne pas avancer tant que cette phase n'est pas validée.

### Phase 1 — MVP (6-8 semaines)
**Objectif** : Une app utilisable pour préparer une séance.

- [ ] Frequency Analyzer complet (UI propre, confiance, sauvegarde)
- [ ] Bowl Library (CRUD, fiche instrument, photo)
- [ ] Bowl Pairing — comparaison de 2 bols
- [ ] Matrice de compatibilité
- [ ] Recommandation par bande cérébrale ("je veux du Theta")
- [ ] Détection bruit ambiant + seuil de confiance
- [ ] Design UI "calm & minimal" — dark mode
- [ ] Stockage SQLite local

**Livrable** : App installable sur iOS et Android, testable par 3-5 praticiens.
**Critère de succès** : Un praticien peut mesurer, stocker et comparer ses bols en < 10 secondes par action.

### Phase 2 — Exploration Harmonique (4-6 semaines)
**Objectif** : Aider à comprendre les relations musicales entre instruments.

- [ ] Harmonic suggestions (intervalles compatibles)
- [ ] Scale Explorer (bibliothèque réduite)
- [ ] Scale Playback (synthèse sine)
- [ ] Lien entre gammes et collection de bols ("tes bols forment une gamme pentatonique de C")

### Phase 3 — Session Builder (4-6 semaines)
**Objectif** : Structurer des séances complètes.

- [ ] Timeline de séance (phases, durées, instruments)
- [ ] Templates prédéfinis
- [ ] Notes post-séance
- [ ] Export / partage de séance

### Phase 4 — Polish & Distribution
- [ ] Tests utilisateurs élargis
- [ ] Onboarding / tutoriel
- [ ] App Store + Google Play
- [ ] Modèle économique (à décider après retours Phase 1)
- [ ] Landing page

---

## 7. Validation Terrain — Questions pour tes Collègues

Avant ou pendant la Phase 1, recueillir ces retours :

### Questions prioritaires
1. **"Comment choisis-tu quels bols jouer ensemble ?"** → Confirmer le besoin de pairing
2. **"Utilises-tu des gammes spécifiques consciemment ?"** → Valider/invalider le Pilier 2
3. **"Structures-tu tes séances à l'avance ou improvises-tu ?"** → Valider/invalider le Pilier 3
4. **"Quel outil utilises-tu aujourd'hui ?"** → Identifier la concurrence réelle
5. **"Serais-tu prêt(e) à payer pour cette app ? Combien ?"** → Orienter le modèle économique

### Format suggéré
- 5 entretiens de 20 minutes (en personne ou visio)
- Montrer le prototype Phase 0 pour recueillir des réactions concrètes
- Documenter les réponses pour alimenter les décisions produit

---

## 8. Ce Qui a Été Retiré du Document Original (et pourquoi)

| Élément retiré | Raison |
|----------------|--------|
| 25+ gammes exotiques (Enigmatic, Bebop, etc.) | Non pertinent pour la sonothérapie. Ajout possible plus tard si demandé |
| Choix "Flutter ou React Native" sans trancher | Décision prise : React Native + Expo |
| Audio preview avec "crystal bowl sample" | Complexe et non prioritaire — sine wave suffit pour V2 |
| Sound Bath Builder comme feature optionnelle | Repositionné en Phase 3 avec specs plus concrètes |
| Animation de pulsation binaurale | Gadget — à ajouter en polish si les praticiens le demandent |
| Modèle de données minimaliste | Remplacé par un modèle enrichi (type, overtones, chakra, photo) |

---

## 9. Risques Identifiés

| Risque | Impact | Mitigation |
|--------|--------|------------|
| Détection fréquence imprécise sur bols tibétains | Bloquant | Phase 0 dédiée — tester avant tout |
| Micro smartphone de mauvaise qualité | Élevé | Seuil de confiance + recommander micro externe en option |
| Trop de features → app jamais finie | Élevé | Roadmap stricte en phases — pas de Phase 2 avant retours Phase 1 |
| Pas de marché réel | Moyen | Validation terrain avec 5 praticiens avant Phase 1 complète |
| Performance FFT en JS | Moyen | Prévoir module natif Swift/Kotlin dès Phase 0 si nécessaire |

---

## 10. Prochaine Étape Immédiate

**→ Commencer la Phase 0 : prototype de détection de fréquence.**

1. Créer le projet React Native + Expo
2. Implémenter une capture micro basique
3. Appliquer un FFT sur le signal
4. Afficher la fréquence détectée
5. Tester avec tes propres bols
6. Itérer sur l'algorithme jusqu'à obtenir ±2 Hz de précision

C'est la validation technique qui débloque tout le reste.
