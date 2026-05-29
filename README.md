# Adaptateur Piano vers Guitare

> Spécifications et méthodologie pour la conversion d'une partition piano en tablature guitare.

**Auteur :** Jean-Pierre Mallet ([@Samydidiyork](https://github.com/Samydidiyork))
**Licence :** Apache 2.0

---

## 1. Description du projet

Ce projet a pour but de formaliser les règles d'arrangement nécessaires à la conversion d'une partition de piano (format MIDI/MusicXML) vers une partition de guitare (tablatures).

La difficulté majeure traitée ici est la **réduction polyphonique** : transformer un instrument capable de jouer 10 notes simultanément (piano) en un instrument limité par le nombre de cordes et l'ergonomie du manche (guitare).

---

## 2. Objectifs techniques

- **Structuration des données** — Définir les règles de priorité des notes (conserver la mélodie principale, gérer les basses, simplifier les accords).
- **Formalisation des contraintes** — Établir une base théorique sur les doigtés possibles sur un manche de guitare standard.
- **Partage communautaire** — Offrir une méthodologie claire pour qu'un développeur puisse automatiser cette conversion sans compromis sur la qualité musicale.

---

## 3. Logique d'arrangement (Spécifications)

### 3.1 Gestion des couches (Layers)

| Couche | Source | Rôle |
|--------|--------|------|
| Mélodie | Main droite — note la plus haute | Priorité absolue, toujours conservée |
| Basse | Main gauche — note la plus basse | Priorité absolue, toujours conservée |
| Harmonie | Notes intermédiaires des deux mains | Réduction selon les règles ci-dessous |

### 3.2 Réduction harmonique

Règles de priorité pour la simplification des accords :

| Priorité | Intervalle | Décision |
|----------|-----------|----------|
| ✅ Conserver | Fondamentale (basse) | Toujours |
| ✅ Conserver | Mélodie (note la plus haute) | Toujours |
| ✅ Conserver | Tierce (3e mineure ou majeure) | Toujours — définit le caractère majeur/mineur |
| ❌ Sacrifier | Quinte (5e juste) | En priorité — harmoniquement neutre |
| ❌ Sacrifier | Neuvième (2e / 9e) | En priorité |
| ⚠️ Si surcharge | Autres notes | Si l'accord dépasse 6 notes |

**Règle spéciale — chevauchement :**
Si la mélodie (main droite) se retrouve au même niveau ou en dessous de la basse (main gauche) après fusion des deux mains, la mélodie est remontée d'une octave.

**Exemple de réduction :**
```
Cmaj9 (Do Mi Sol Si Ré) — 5 notes
        ↓
Do Mi Si — 3 notes
(fondamentale + tierce + septième majeure)
Sol et Ré sacrifiés
```

### 3.3 Optimisation des positions

Accordage standard de référence :

| Corde | Note | MIDI |
|-------|------|------|
| 6 (grave) | Mi2 | 40 |
| 5 | La2 | 45 |
| 4 | Ré3 | 50 |a
| 3 | Sol3 | 55 |
| 2 | Si3 | 59 |
| 1 (chanterelle) | Mi4 | 64 |

Règle d'assignation : pour chaque note conservée, choisir la corde qui donne la **case la plus basse** (position la plus naturelle sur le manche).

### 3.4 Règle de proximité de position

Pour assurer la jouabilité enchaînée des accords et des notes, chaque position sur le manche doit rester dans un **rayon de 4 cases maximum** autour de la position précédente.

| Situation | Action |
|-----------|--------|
| Note suivante dans un rayon ≤ 4 cases | Conserver la position |
| Note suivante hors du rayon (> 4 cases) | Chercher une position alternative sur une autre corde |
| Aucune alternative disponible | Autoriser le déplacement, signaler le saut de position |

**Exemple :**
```
Accord 1 : position de référence case 3
           → Accord 2 autorisé entre case 0 et case 7 (±4)
           → Accord 2 hors de cette plage = chercher un renversement
```

**Principe :** Un renversement d'accord (même notes, ordre différent) permet souvent de rester dans la zone de confort sans quitter la position de la main. Cette règle favorise le legato naturel et évite les sauts de position inutiles.

---

## 4. Format de la portée intermédiaire

Le concept central du projet est l'insertion d'une **portée tablature entre la clef de Sol et la clef de Fa** de la partition piano :

```
┌──────────────────────────────────┐
│  Clef de SOL  (main droite)      │
├──────────────────────────────────┤
│  ──── TABLATURE GUITARE ────     │  ← synthèse des deux mains
├──────────────────────────────────┤
│  Clef de FA   (main gauche)      │
└──────────────────────────────────┘
```

Cette portée intermédiaire permet au guitariste de lire simultanément la source piano et son adaptation guitare.

---

## 5. Cas particuliers documentés

- **Quinte altérée (♭5 ou ♯5)** → ne pas sacrifier, elle définit le caractère de l'accord
- **Accord de substitution** → si un accord reste injouable après réduction, proposer l'accord le plus proche par mouvement chromatique minimal
- **Notes hors tessiture** (< Mi2 ou > Mi4 en position standard) → transposer d'une octave
- **Saut de position** (> 4 cases entre deux accords) → proposer un renversement ou une position alternative sur une corde différente

---

## 6. Comment contribuer

- **Issues** — Documentez les cas où la conversion échoue (accord trop large, tessiture impossible, etc.)
- **Documentation** — Proposez des améliorations sur les règles d'arrangement
- **Code** — Si vous êtes développeur, reprenez ces spécifications pour créer un moteur de conversion en Python, JavaScript, ou comme plugin MuseScore

---

## 7. Licence

Ce projet est mis à disposition sous la **Licence Apache 2.0**.
Vous êtes libre d'utiliser, de modifier et de distribuer ce travail, sous réserve de conserver les notices de droit d'auteur.

Voir le fichier [LICENSE](LICENSE) pour le texte complet.
