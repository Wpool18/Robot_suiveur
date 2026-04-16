# Guide de Câblage — Puissance & Commande

## Plan de reproduction sur Multisim

> **Objectif :** Reproduire étape par étape le circuit complet du robot suiveur de ligne dans NI Multisim.  
> Ce guide détaille chaque connexion, broche par broche, avec les noms de composants Multisim correspondants.

---

## Table des matières

1. [Composants à placer dans Multisim](#1-composants-à-placer-dans-multisim)
2. [Étape 1 — Alimentation](#2-étape-1--alimentation)
3. [Étape 2 — Capteurs IR (×2 canaux)](#3-étape-2--capteurs-ir-2-canaux)
4. [Étape 3 — Comparateur LM393](#4-étape-3--comparateur-lm393)
5. [Étape 4 — Pont en H L293D](#5-étape-4--pont-en-h-l293d)
6. [Étape 5 — Moteurs](#6-étape-5--moteurs)
7. [Netlist complète — Connexion par connexion](#7-netlist-complète--connexion-par-connexion)
8. [Vérifications avant simulation](#8-vérifications-avant-simulation)
9. [Paramètres de simulation](#9-paramètres-de-simulation)

---

## 1. Composants à placer dans Multisim

Place ces composants sur la feuille de schéma. Les noms entre parenthèses sont les références dans la bibliothèque Multisim.

### Sources d'alimentation

| Réf. schéma | Composant Multisim | Bibliothèque | Valeur |
|-------------|-------------------|--------------|--------|
| V_BAT | DC_POWER (source de tension DC) | Sources > POWER_SOURCES | **7.4V** |
| GND | GROUND | Sources > POWER_SOURCES | 0V |

### Régulateur

| Réf. schéma | Composant Multisim | Bibliothèque | Notes |
|-------------|-------------------|--------------|-------|
| U1 | **LM7805** | Power > Voltage Regulators | TO-220, 3 broches |
| C1 | Condensateur | Passives > Capacitor | **100µF**, polarisé |
| C2 | Condensateur | Passives > Capacitor | **100nF** |
| C3 | Condensateur | Passives > Capacitor | **100µF**, polarisé |
| C4 | Condensateur | Passives > Capacitor | **100nF** |

### Capteurs IR (simulation)

> **Note :** Le TCRT5000 n'existe pas nativement dans Multisim. On le simule avec des composants séparés.

| Réf. schéma | Composant Multisim | Bibliothèque | Valeur / Notes |
|-------------|-------------------|--------------|----------------|
| D1, D2 | LED (IR) | Diodes > LED | Représente la LED IR émettrice |
| R1, R2 | Résistance | Passives > Resistor | **220Ω** |
| Q1, Q2 | Phototransistor **NPN** | Transistors > BJT_NPN | Ou utiliser un **potentiomètre** pour simuler le signal capteur (voir section simulation) |
| R3, R4 | Résistance | Passives > Resistor | **10kΩ** (pull-up) |

### Réglage de seuil

| Réf. schéma | Composant Multisim | Bibliothèque | Valeur |
|-------------|-------------------|--------------|--------|
| POT1, POT2 | **POTENTIOMETER** | Passives > Potentiometer | **10kΩ** |

### Comparateur

| Réf. schéma | Composant Multisim | Bibliothèque | Notes |
|-------------|-------------------|--------------|-------|
| U2 | **LM393** | Analog > Comparators | DIP-8, 2 comparateurs |
| R5, R6 | Résistance | Passives > Resistor | **4.7kΩ** (pull-up sortie) |

### Pont en H

| Réf. schéma | Composant Multisim | Bibliothèque | Notes |
|-------------|-------------------|--------------|-------|
| U3 | **L293D** | Mixed > Drivers | DIP-16 |
| C5 | Condensateur | Passives > Capacitor | **100nF** (découplage VCC1) |
| C6 | Condensateur | Passives > Capacitor | **100nF** (découplage VS) |

### Moteurs

| Réf. schéma | Composant Multisim | Bibliothèque | Notes |
|-------------|-------------------|--------------|-------|
| M1 | DC_MOTOR | Electromechanical > Motor | Moteur gauche |
| M2 | DC_MOTOR | Electromechanical > Motor | Moteur droit |

### Indicateurs (optionnel)

| Réf. schéma | Composant Multisim | Bibliothèque | Valeur |
|-------------|-------------------|--------------|--------|
| LED1, LED2 | LED (rouge) | Diodes > LED | Témoin sortie LM393 |
| R7, R8 | Résistance | Passives > Resistor | **1kΩ** |

---

## 2. Étape 1 — Alimentation

### 2.1 Schéma de principe

```
  V_BAT (7.4V)
     │
     ├──── C1 (100µF) ──── GND        ← Filtrage entrée
     │
     ├──── C2 (100nF) ──── GND        ← Découplage HF entrée
     │
     └──── U1 (LM7805)
           │  IN (pin 1)  ← V_BAT
           │  GND (pin 2) ← GND
           │  OUT (pin 3) ← VCC_5V
           │
           ├──── C3 (100µF) ──── GND   ← Filtrage sortie
           │
           └──── C4 (100nF) ──── GND   ← Découplage HF sortie
```

### 2.2 Câblage broche par broche — LM7805 (U1)

| Broche LM7805 | Nom | Connexion |
|:-:|------|-----------|
| **1** | IN | V_BAT (7.4V) |
| **2** | GND | Masse commune (GND) |
| **3** | OUT | Rail **VCC_5V** (alimentation logique) |

### 2.3 Instructions Multisim

1. Placer **V_BAT** (source DC 7.4V) à gauche
2. Placer **U1** (LM7805) au centre
3. Relier pin 1 de U1 à la borne + de V_BAT
4. Relier pin 2 de U1 à GND
5. Placer C1 (100µF) entre pin 1 de U1 et GND — **polarité : + vers pin 1**
6. Placer C2 (100nF) en parallèle de C1
7. Nommer le fil en sortie de pin 3 : **VCC_5V** (clic droit → Net Name)
8. Placer C3 (100µF) entre VCC_5V et GND — **polarité : + vers VCC_5V**
9. Placer C4 (100nF) en parallèle de C3

### 2.4 Nœuds créés

| Nom du nœud (net) | Tension | Usage |
|--------------------|---------|-------|
| **V_BAT** | 7.4V | Alimentation moteurs (→ VS du L293D) |
| **VCC_5V** | 5.0V | Alimentation capteurs, LM393, logique L293D |
| **GND** | 0V | Masse commune |

---

## 3. Étape 2 — Capteurs IR (×2 canaux)

### 3.1 Canal Gauche — Schéma

```
    VCC_5V
      │
     [R1 = 220Ω]
      │
     LED IR (D1)          VCC_5V
      │                     │
     GND                  [R3 = 10kΩ]
                            │
                        ┌───┤──── CAPT_G (vers LM393 pin 3)
                        │   │
                   Phototransistor (Q1)
                        │
                       GND
```

### 3.2 Câblage broche par broche — Canal Gauche

| De | Vers | Composant / Fil |
|----|------|-----------------|
| VCC_5V | R1 (borne 1) | Fil |
| R1 (borne 2) | D1 anode | Fil |
| D1 cathode | GND | Fil |
| VCC_5V | R3 (borne 1) | Fil |
| R3 (borne 2) | Q1 collecteur | Fil — **Nommer ce nœud : CAPT_G** |
| Q1 émetteur | GND | Fil |

### 3.3 Canal Droit — Schéma (identique, références différentes)

| De | Vers | Composant / Fil |
|----|------|-----------------|
| VCC_5V | R2 (borne 1) | Fil |
| R2 (borne 2) | D2 anode | Fil |
| D2 cathode | GND | Fil |
| VCC_5V | R4 (borne 1) | Fil |
| R4 (borne 2) | Q2 collecteur | Fil — **Nommer ce nœud : CAPT_D** |
| Q2 émetteur | GND | Fil |

### 3.4 Seuil de comparaison (×2)

```
    VCC_5V
      │
    [POT1 = 10kΩ]
      │
      ├──── SEUIL_G (vers LM393 pin 2)
      │
     GND
```

| De | Vers | Composant / Fil |
|----|------|-----------------|
| VCC_5V | POT1 borne haute | Fil |
| POT1 borne basse | GND | Fil |
| POT1 curseur (wiper) | — | **Nommer : SEUIL_G** |
| VCC_5V | POT2 borne haute | Fil |
| POT2 borne basse | GND | Fil |
| POT2 curseur (wiper) | — | **Nommer : SEUIL_D** |

### 3.5 Astuce Multisim — Simuler le capteur IR

> Le TCRT5000 n'étant pas dans Multisim, **remplace le phototransistor (Q1, Q2) par un potentiomètre** pour simuler le signal capteur :

| Remplacement | Multisim | Explication |
|--------------|----------|-------------|
| Q1 → **POT_SIM_G** | Potentiomètre 10kΩ | Curseur = CAPT_G. Tourner le pot simule le passage blanc/noir |
| Q2 → **POT_SIM_D** | Potentiomètre 10kΩ | Curseur = CAPT_D |

Câblage du pot simulant le capteur :
- Borne haute → VCC_5V
- Borne basse → GND
- Curseur → CAPT_G (ou CAPT_D)

**Simulation :**
- Curseur en haut (~4V) = surface blanche (forte réflexion)
- Curseur en bas (~0.5V) = surface noire (faible réflexion)

---

## 4. Étape 3 — Comparateur LM393

### 4.1 Brochage du LM393 (U2)

```
        ┌───── U ─────┐
   OUT1 │1           8│ VCC
   IN1- │2           7│ OUT2
   IN1+ │3           6│ IN2-
   GND  │4           5│ IN2+
        └─────────────┘
```

### 4.2 Câblage broche par broche

| Broche | Nom | Connexion | Nœud |
|:------:|------|-----------|------|
| **1** | OUT1 | R5 (4.7kΩ) vers VCC_5V | **CMD_G** (sortie commande gauche) |
| **2** | IN1− | POT1 curseur | SEUIL_G |
| **3** | IN1+ | R3/Q1 jonction | CAPT_G |
| **4** | GND | Masse | GND |
| **5** | IN2+ | R4/Q2 jonction | CAPT_D |
| **6** | IN2− | POT2 curseur | SEUIL_D |
| **7** | OUT2 | R6 (4.7kΩ) vers VCC_5V | **CMD_D** (sortie commande droite) |
| **8** | VCC | Rail 5V | VCC_5V |

### 4.3 Résistances de pull-up (obligatoires)

```
    VCC_5V              VCC_5V
      │                   │
    [R5 = 4.7kΩ]       [R6 = 4.7kΩ]
      │                   │
    CMD_G (pin 1)       CMD_D (pin 7)
```

| De | Vers | Composant |
|----|------|-----------|
| VCC_5V | R5 (borne 1) | Fil |
| R5 (borne 2) | U2 pin 1 | Fil — **Nommer : CMD_G** |
| VCC_5V | R6 (borne 1) | Fil |
| R6 (borne 2) | U2 pin 7 | Fil — **Nommer : CMD_D** |

### 4.4 LED témoins (optionnel)

```
    CMD_G                CMD_D
      │                    │
    [R7 = 1kΩ]           [R8 = 1kΩ]
      │                    │
    LED1 (anode)         LED2 (anode)
      │                    │
     GND                  GND
```

### 4.5 Logique du comparateur

| CAPT vs SEUIL | Sortie OUT | CMD | Signification |
|:-:|:-:|:-:|---|
| CAPT > SEUIL (blanc) | Transistor interne OFF | **HIGH (5V)** via pull-up | Surface blanche → moteur ON |
| CAPT < SEUIL (noir) | Transistor interne ON | **LOW (0V)** | Surface noire → moteur OFF |

---

## 5. Étape 4 — Pont en H L293D

### 5.1 Brochage du L293D (U3)

```
              ┌───── U ─────┐
   EN1,2 (1) │1          16│ VCC1 (5V logique)
    IN1  (2) │2          15│ IN4
   OUT1  (3) │3          14│ OUT4
    GND  (4) │4          13│ GND
    GND  (5) │5          12│ GND
   OUT2  (6) │6          11│ OUT3
    IN2  (7) │7          10│ IN3
    VS   (8) │8           9│ EN3,4
 (V_moteur)  └─────────────┘
```

### 5.2 Câblage broche par broche

| Broche | Nom | Connexion | Nœud / Valeur |
|:------:|------|-----------|---------------|
| **1** | EN1,2 | Rail 5V | **VCC_5V** (moteurs toujours activés) |
| **2** | IN1 | Sortie comparateur 1 | **CMD_G** (pin 1 du LM393) |
| **3** | OUT1 | Moteur gauche borne A | MOT_G_A |
| **4** | GND | Masse | **GND** |
| **5** | GND | Masse | **GND** |
| **6** | OUT2 | Moteur gauche borne B | MOT_G_B |
| **7** | IN2 | Masse | **GND** (sens fixe : avant uniquement) |
| **8** | VS | Batterie directe | **V_BAT** (7.4V) |
| **9** | EN3,4 | Rail 5V | **VCC_5V** (moteurs toujours activés) |
| **10** | IN3 | Sortie comparateur 2 | **CMD_D** (pin 7 du LM393) |
| **11** | OUT3 | Moteur droit borne A | MOT_D_A |
| **12** | GND | Masse | **GND** |
| **13** | GND | Masse | **GND** |
| **14** | OUT4 | Moteur droit borne B | MOT_D_B |
| **15** | IN4 | Masse | **GND** (sens fixe : avant uniquement) |
| **16** | VCC1 | Rail 5V logique | **VCC_5V** |

### 5.3 Condensateurs de découplage L293D

| De | Vers | Composant |
|----|------|-----------|
| U3 pin 16 (VCC1) | GND | C5 (100nF) — au plus près des broches |
| U3 pin 8 (VS) | GND | C6 (100nF) — au plus près des broches |

### 5.4 Explication du câblage IN/EN

| Signal | Rôle | Câblage choisi | Effet |
|--------|------|----------------|-------|
| **EN1,2** | Active/désactive le demi-pont 1 (moteur G) | Relié à VCC_5V → **toujours actif** | Le moteur est commandé uniquement par IN1/IN2 |
| **IN1** | Commande haute du moteur G | **CMD_G** (sortie LM393) | HIGH = moteur tourne, LOW = moteur arrêté |
| **IN2** | Commande basse du moteur G | **GND** (fixé à 0) | Le moteur ne tourne que dans un sens |
| **EN3,4** | Active/désactive le demi-pont 2 (moteur D) | Relié à VCC_5V → **toujours actif** | Idem |
| **IN3** | Commande haute du moteur D | **CMD_D** (sortie LM393) | HIGH = moteur tourne, LOW = moteur arrêté |
| **IN4** | Commande basse du moteur D | **GND** (fixé à 0) | Sens fixe avant |

**Table de vérité résultante :**

| IN1 (CMD_G) | IN2 (GND) | Moteur Gauche |
|:-:|:-:|---|
| HIGH (5V) | LOW (0V) | **Tourne (avant)** |
| LOW (0V) | LOW (0V) | **Arrêté (roue libre)** |

| IN3 (CMD_D) | IN4 (GND) | Moteur Droit |
|:-:|:-:|---|
| HIGH (5V) | LOW (0V) | **Tourne (avant)** |
| LOW (0V) | LOW (0V) | **Arrêté (roue libre)** |

---

## 6. Étape 5 — Moteurs

### 6.1 Câblage

| De | Vers |
|----|------|
| U3 pin 3 (OUT1) | M1 borne + |
| U3 pin 6 (OUT2) | M1 borne − |
| U3 pin 11 (OUT3) | M2 borne + |
| U3 pin 14 (OUT4) | M2 borne − |

> **Dans Multisim :** utiliser le composant DC_MOTOR ou simplement une résistance de 10Ω + inductance 2mH en série pour simuler l'impédance d'un moteur DC.

### 6.2 Modèle simplifié du moteur (si DC_MOTOR indisponible)

```
    OUT1 ──── [R = 10Ω] ──── [L = 2mH] ──── OUT2
```

Cela simule :
- **R = 10Ω** : résistance de bobinage (typique moteur TT)
- **L = 2mH** : inductance de bobinage

---

## 7. Netlist complète — Connexion par connexion

### 7.1 Récapitulatif de tous les nœuds

| Nœud | Tension | Description |
|------|---------|-------------|
| V_BAT | 7.4V | Batterie, alim moteurs |
| VCC_5V | 5.0V | Sortie régulateur, alim logique |
| GND | 0V | Masse commune |
| CAPT_G | 0.5–4.5V | Signal capteur gauche (variable) |
| CAPT_D | 0.5–4.5V | Signal capteur droit (variable) |
| SEUIL_G | 0–5V | Seuil réglable gauche (~2.5V) |
| SEUIL_D | 0–5V | Seuil réglable droit (~2.5V) |
| CMD_G | 0V ou 5V | Commande moteur gauche (sortie LM393) |
| CMD_D | 0V ou 5V | Commande moteur droit (sortie LM393) |
| MOT_G_A | variable | Moteur gauche borne A |
| MOT_G_B | variable | Moteur gauche borne B |
| MOT_D_A | variable | Moteur droit borne A |
| MOT_D_B | variable | Moteur droit borne B |

### 7.2 Liste exhaustive des connexions (netlist)

Chaque ligne = un fil à tracer dans Multisim.

```
┌────┬─────────────────────────┬─────────────────────────────┐
│ #  │ DE                      │ VERS                        │
├────┼─────────────────────────┼─────────────────────────────┤
│    │ === ALIMENTATION ===    │                             │
│ 1  │ V_BAT (+)               │ U1 pin 1 (IN)              │
│ 2  │ V_BAT (+)               │ C1 (+)                     │
│ 3  │ V_BAT (+)               │ C2 (borne 1)               │
│ 4  │ V_BAT (+)               │ U3 pin 8 (VS)              │
│ 5  │ U1 pin 2 (GND)          │ GND                        │
│ 6  │ U1 pin 3 (OUT)          │ VCC_5V (rail)              │
│ 7  │ VCC_5V                  │ C3 (+)                     │
│ 8  │ VCC_5V                  │ C4 (borne 1)               │
│ 9  │ C1 (−)                  │ GND                        │
│ 10 │ C2 (borne 2)            │ GND                        │
│ 11 │ C3 (−)                  │ GND                        │
│ 12 │ C4 (borne 2)            │ GND                        │
├────┼─────────────────────────┼─────────────────────────────┤
│    │ === CAPTEUR GAUCHE ===  │                             │
│ 13 │ VCC_5V                  │ R1 (borne 1)               │
│ 14 │ R1 (borne 2)            │ D1 anode (LED IR G)        │
│ 15 │ D1 cathode              │ GND                        │
│ 16 │ VCC_5V                  │ R3 (borne 1)               │
│ 17 │ R3 (borne 2)            │ CAPT_G (nœud)              │
│ 18 │ CAPT_G                  │ Q1 collecteur (ou POT_SIM) │
│ 19 │ Q1 émetteur             │ GND                        │
├────┼─────────────────────────┼─────────────────────────────┤
│    │ === CAPTEUR DROIT ===   │                             │
│ 20 │ VCC_5V                  │ R2 (borne 1)               │
│ 21 │ R2 (borne 2)            │ D2 anode (LED IR D)        │
│ 22 │ D2 cathode              │ GND                        │
│ 23 │ VCC_5V                  │ R4 (borne 1)               │
│ 24 │ R4 (borne 2)            │ CAPT_D (nœud)              │
│ 25 │ CAPT_D                  │ Q2 collecteur (ou POT_SIM) │
│ 26 │ Q2 émetteur             │ GND                        │
├────┼─────────────────────────┼─────────────────────────────┤
│    │ === SEUILS ===          │                             │
│ 27 │ VCC_5V                  │ POT1 borne haute            │
│ 28 │ POT1 borne basse        │ GND                        │
│ 29 │ POT1 curseur            │ SEUIL_G                    │
│ 30 │ VCC_5V                  │ POT2 borne haute            │
│ 31 │ POT2 borne basse        │ GND                        │
│ 32 │ POT2 curseur            │ SEUIL_D                    │
├────┼─────────────────────────┼─────────────────────────────┤
│    │ === COMPARATEUR ===     │                             │
│ 33 │ VCC_5V                  │ U2 pin 8 (VCC)             │
│ 34 │ U2 pin 4                │ GND                        │
│ 35 │ CAPT_G                  │ U2 pin 3 (IN1+)            │
│ 36 │ SEUIL_G                 │ U2 pin 2 (IN1−)            │
│ 37 │ CAPT_D                  │ U2 pin 5 (IN2+)            │
│ 38 │ SEUIL_D                 │ U2 pin 6 (IN2−)            │
│ 39 │ VCC_5V                  │ R5 (borne 1)               │
│ 40 │ R5 (borne 2)            │ U2 pin 1 (OUT1) = CMD_G    │
│ 41 │ VCC_5V                  │ R6 (borne 1)               │
│ 42 │ R6 (borne 2)            │ U2 pin 7 (OUT2) = CMD_D    │
├────┼─────────────────────────┼─────────────────────────────┤
│    │ === PONT EN H ===       │                             │
│ 43 │ VCC_5V                  │ U3 pin 16 (VCC1)           │
│ 44 │ VCC_5V                  │ U3 pin 1 (EN1,2)           │
│ 45 │ VCC_5V                  │ U3 pin 9 (EN3,4)           │
│ 46 │ V_BAT                   │ U3 pin 8 (VS)              │
│ 47 │ U3 pin 4                │ GND                        │
│ 48 │ U3 pin 5                │ GND                        │
│ 49 │ U3 pin 12               │ GND                        │
│ 50 │ U3 pin 13               │ GND                        │
│ 51 │ CMD_G                   │ U3 pin 2 (IN1)             │
│ 52 │ GND                     │ U3 pin 7 (IN2)             │
│ 53 │ CMD_D                   │ U3 pin 10 (IN3)            │
│ 54 │ GND                     │ U3 pin 15 (IN4)            │
│ 55 │ U3 pin 16 (VCC1)        │ C5 (borne 1) → GND        │
│ 56 │ U3 pin 8 (VS)           │ C6 (borne 1) → GND        │
├────┼─────────────────────────┼─────────────────────────────┤
│    │ === MOTEURS ===         │                             │
│ 57 │ U3 pin 3 (OUT1)         │ M1 borne +                 │
│ 58 │ U3 pin 6 (OUT2)         │ M1 borne −                 │
│ 59 │ U3 pin 11 (OUT3)        │ M2 borne +                 │
│ 60 │ U3 pin 14 (OUT4)        │ M2 borne −                 │
├────┼─────────────────────────┼─────────────────────────────┤
│    │ === LED TÉMOINS ===     │ (optionnel)                 │
│ 61 │ CMD_G                   │ R7 (borne 1)               │
│ 62 │ R7 (borne 2)            │ LED1 anode                 │
│ 63 │ LED1 cathode             │ GND                        │
│ 64 │ CMD_D                   │ R8 (borne 1)               │
│ 65 │ R8 (borne 2)            │ LED2 anode                 │
│ 66 │ LED2 cathode             │ GND                        │
└────┴─────────────────────────┴─────────────────────────────┘
```

---

## 8. Vérifications avant simulation

### 8.1 Checklist de câblage

| # | Vérification | ✓ |
|---|-------------|---|
| 1 | **Toutes les masses (GND)** sont reliées ensemble (V_BAT, U1, U2, U3, capteurs) | ☐ |
| 2 | **U1 pin 2** (GND du régulateur) est bien connecté à GND | ☐ |
| 3 | **U2 pin 4** (GND du LM393) est connecté à GND | ☐ |
| 4 | **U2 pin 8** (VCC du LM393) est connecté à VCC_5V | ☐ |
| 5 | **U3 pins 4, 5, 12, 13** (les 4 GND du L293D) sont tous à GND | ☐ |
| 6 | **Pull-up R5 et R6** (4.7kΩ) sont bien entre VCC_5V et les sorties du LM393 | ☐ |
| 7 | **EN1,2 (pin 1)** et **EN3,4 (pin 9)** du L293D sont à VCC_5V | ☐ |
| 8 | **IN2 (pin 7)** et **IN4 (pin 15)** du L293D sont à GND (sens fixe) | ☐ |
| 9 | **VS (pin 8)** du L293D est à V_BAT (7.4V), pas à VCC_5V | ☐ |
| 10 | Condensateurs de découplage (C5, C6) placés au plus près de U3 | ☐ |

### 8.2 Erreurs fréquentes à éviter

| Erreur | Conséquence | Solution |
|--------|------------|---------|
| Oublier les pull-up R5/R6 | Sortie LM393 flottante → pas de commande | Toujours mettre 4.7kΩ vers VCC_5V |
| Inverser IN+ et IN− du LM393 | Logique inversée (moteur ON sur noir) | IN+ = capteur (pin 3/5), IN− = seuil (pin 2/6) |
| VS du L293D à 5V au lieu de V_BAT | Moteurs sous-alimentés | VS (pin 8) = V_BAT, VCC1 (pin 16) = 5V |
| Oublier les 4 GND du L293D | Surchauffe, pas de dissipation thermique | Les 4 pins GND servent aussi de dissipateur |
| Polarité inversée des condensateurs électrolytiques | Risque d'explosion du condensateur | + vers la tension la plus haute |

---

## 9. Paramètres de simulation

### 9.1 Configuration recommandée dans Multisim

| Paramètre | Valeur | Menu |
|-----------|--------|------|
| Type de simulation | **DC Operating Point** (pour vérifier les tensions) | Simulate > Analyses |
| Puis | **Transient Analysis** (pour voir le comportement dynamique) | Simulate > Analyses |
| Durée simulation | 100 ms – 1 s | Settings |
| Pas de temps max | 10 µs | Settings |

### 9.2 Points de mesure à vérifier

Placer des **sondes de tension** (probes) sur ces nœuds :

| Sonde | Nœud | Valeur attendue |
|-------|------|-----------------|
| Probe 1 | VCC_5V | **5.00V** stable |
| Probe 2 | CAPT_G | Variable (0.5–4.5V selon pot simulateur) |
| Probe 3 | CAPT_D | Variable (0.5–4.5V selon pot simulateur) |
| Probe 4 | SEUIL_G | ~2.5V (à ajuster) |
| Probe 5 | CMD_G | **0V ou 5V** (doit basculer quand CAPT_G croise SEUIL_G) |
| Probe 6 | CMD_D | **0V ou 5V** (doit basculer quand CAPT_D croise SEUIL_D) |
| Probe 7 | MOT_G_A | ~0V ou ~6V selon CMD_G |
| Probe 8 | MOT_D_A | ~0V ou ~6V selon CMD_D |

### 9.3 Scénarios de test

| Test | Action | Résultat attendu |
|------|--------|-----------------|
| **Robot centré** | POT_SIM_G et POT_SIM_D > SEUIL (~4V) | CMD_G = HIGH, CMD_D = HIGH → 2 moteurs ON |
| **Dérive gauche** | POT_SIM_G < SEUIL (~1V), POT_SIM_D > SEUIL | CMD_G = LOW, CMD_D = HIGH → M1 OFF, M2 ON |
| **Dérive droite** | POT_SIM_G > SEUIL, POT_SIM_D < SEUIL (~1V) | CMD_G = HIGH, CMD_D = LOW → M1 ON, M2 OFF |
| **Ligne perdue** | POT_SIM_G et POT_SIM_D < SEUIL (~1V) | CMD_G = LOW, CMD_D = LOW → 2 moteurs OFF |

---

## Schéma global — Vue d'ensemble

```
                                    VCC_5V (5V)
                                       │
    V_BAT ──┬── U1 (LM7805) ──────────┤
    (7.4V)  │                          │
            │   ┌──────────────────────┼──────────────────────────┐
            │   │                      │                          │
            │   │  ┌────────┐     ┌────┴────┐     ┌────────────┐ │
            │   │  │ R1=220Ω│     │R3=10kΩ  │     │ POT1 10kΩ  │ │
            │   │  │   │    │     │   │     │     │   │        │ │
            │   │  │  D1    │     │ CAPT_G──┼─────┼───┤(+) U2  │ │
            │   │  │   │    │     │   │     │     │   │ LM393  │ │
            │   │  │  GND   │     │  Q1/POT │  SEUIL──┤(−)     │ │
            │   │  │        │     │   │     │     │   │   │    │ │
            │   │  └────────┘     │  GND    │     │  GND  │    │ │
            │   │                 └─────────┘     │    CMD_G   │ │
            │   │                                 │       │    │ │
            │   │  (idem canal D → CAPT_D, SEUIL_D, CMD_D)    │ │
            │   │                                 │       │    │ │
            │   │        ┌────────────────────────┘       │    │ │
            │   │        │  R5=4.7kΩ (pull-up)            │    │ │
            │   │        │  R6=4.7kΩ (pull-up)            │    │ │
            │   │        └────────────────────────────────┘    │ │
            │   │                                              │ │
            │   │        ┌─────────────────────────────────────┘ │
            │   │        │                                       │
            │   │   ┌────┴─────────────┐                         │
            │   │   │     U3 (L293D)   │                         │
            │   │   │                  │                         │
            │   │   │ IN1=CMD_G  OUT1──┤── M1 (moteur G)         │
            │   │   │ IN2=GND    OUT2──┘                         │
            │   │   │ IN3=CMD_D  OUT3──┤── M2 (moteur D)         │
            │   │   │ IN4=GND    OUT4──┘                         │
            │   │   │                  │                         │
            ├───┼───┤ VS = V_BAT       │                         │
            │   └───┤ VCC1 = VCC_5V    │                         │
            │       │ EN1,2 = VCC_5V   │                         │
            │       │ EN3,4 = VCC_5V   │                         │
            │       └──────────────────┘                         │
            │                                                    │
           GND ──────────────────────────────────────────────── GND
```

---

*Guide de câblage Multisim — Projet "Le Suiveur de Ligne Pur Hard"*
