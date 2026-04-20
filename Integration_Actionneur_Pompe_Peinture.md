# Intégration d'un Actionneur — Pompe à Peinture

## Module complémentaire au Robot Suiveur de Ligne "Pur Hard"

> **Concept :** Doter le robot suiveur de ligne d'un actionneur capable de **redéposer de la peinture** lorsqu'il détecte que la ligne au sol s'est effacée (zone usée, manquante ou décolorée).
> **Contrainte fondamentale :** Le module reste **100 % analogique** — aucune ligne de code, aucun microcontrôleur. La décision d'activer la pompe est prise par une chaîne capteur → comparateur → temporisateur analogique → étage de puissance.

> Ce document est **strictement additif** : il ne modifie ni le câblage, ni le BOM, ni la logique des documents existants (`Dossier_Technique_Suiveur_de_Ligne.md`, `Justification_Choix_Composants.md`, `Guide_Cablage_Multisim.md`). Il décrit un **étage indépendant** qui se greffe sur l'alimentation et partage uniquement la masse commune.

---

## Table des matières

1. [Cahier des charges du module actionneur](#1-cahier-des-charges-du-module-actionneur)
2. [Synoptique du module](#2-synoptique-du-module)
3. [Étage de détection — Capteur IR central](#3-étage-de-détection--capteur-ir-central)
4. [Étage de comparaison & inversion logique](#4-étage-de-comparaison--inversion-logique)
5. [Filtre anti-faux-déclenchement (RC + hystérésis)](#5-filtre-anti-faux-déclenchement-rc--hystérésis)
6. [Temporisation analogique — NE555 monostable](#6-temporisation-analogique--ne555-monostable)
7. [Étage de puissance — MOSFET + pompe](#7-étage-de-puissance--mosfet--pompe)
8. [Choix de la pompe](#8-choix-de-la-pompe)
9. [Schéma électrique complet du module](#9-schéma-électrique-complet-du-module)
10. [Nomenclature additive (BOM module)](#10-nomenclature-additive-bom-module)
11. [Calculs de dimensionnement](#11-calculs-de-dimensionnement)
12. [Calibration et procédure d'essai](#12-calibration-et-procédure-dessai)
13. [Analyse des cas de fonctionnement](#13-analyse-des-cas-de-fonctionnement)
14. [Sécurité & points de vigilance](#14-sécurité--points-de-vigilance)
15. [Améliorations possibles](#15-améliorations-possibles)

---

## 1. Cahier des charges du module actionneur

| Exigence | Spécification |
|----------|---------------|
| **Fonction principale** | Déposer une dose de peinture sur le sol lorsque la ligne est manquante sous le robot |
| **Technologie** | 100 % analogique (composants discrets + CI analogiques type LM393, NE555) |
| **Détection** | Capteur IR central (3ᵉ TCRT5000), indépendant des deux capteurs de suivi |
| **Anti-rebond** | Filtrage RC + hystérésis pour éviter une activation sur micro-irrégularité |
| **Temporisation** | Impulsion de durée **fixe** (~300–800 ms) par détection — évite que la pompe tourne en continu |
| **Puissance pompe** | Pompe 6 V DC, courant nominal ≤ 500 mA |
| **Réversibilité** | Aucun effet sur le suivi de ligne en cas de panne du module |
| **Alimentation** | Partage la batterie LiPo 2S 7.4 V existante (rail $V_{bat}$) et le rail logique 5 V (LM7805) |

---

## 2. Synoptique du module

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐    ┌──────────────────┐    ┌──────────┐
│ Capteur IR      │───▶│ Comparateur LM393│───▶│ Filtre RC +     │───▶│ NE555 monostable │───▶│ N-MOSFET │───▶ POMPE
│ central         │    │ (logique inversée)│    │ hystérésis      │    │ T_on ≈ 500 ms    │    │ IRLZ44N  │
│ (TCRT5000 #3)   │    │                  │    │                 │    │                  │    │ + diode  │
└─────────────────┘    └──────────────────┘    └─────────────────┘    └──────────────────┘    └──────────┘
         ▲                      ▲                                                                    │
         │                      │                                                                    ▼
   Réflexion sol         Pot. seuil 10 kΩ                                                       Vbat (7.4V)
   (noir = ligne         (réglage indépendant
    présente,            du seuil de
    blanc = ligne        "ligne effacée")
    absente)
```

**Principe en une phrase :** un 3ᵉ capteur IR placé entre les deux capteurs de suivi surveille en permanence le centre de la ligne ; lorsqu'il détecte du blanc pendant plus de quelques dizaines de millisecondes, un monostable 555 déclenche une impulsion de tension calibrée qui actionne la pompe via un MOSFET de puissance.

---

## 3. Étage de détection — Capteur IR central

### 3.1 Position mécanique

```
       VUE DE DESSOUS DU CHÂSSIS (avant du robot)
   ┌──────────────────────────────────────────────┐
   │                                              │
   │   [Capteur G]      [Capteur C]    [Capteur D]│
   │      ◯ ◉             ◯ ◉             ◯ ◉    │
   │      │                │                │     │
   │   Suivi G        Détection         Suivi D   │
   │  (existant)     "ligne effacée"   (existant) │
   │                  (NOUVEAU)                   │
   │                                              │
   │     ════════════ LIGNE NOIRE ══════════      │
   │                                              │
   └──────────────────────────────────────────────┘
                       AVANT
```

- Le capteur central est aligné **sur l'axe longitudinal** du robot, exactement au-dessus de l'endroit où la ligne devrait passer.
- Hauteur sol : **5 mm** (identique aux capteurs de suivi).
- Espacement par rapport aux capteurs G/D : **10–12 mm** de chaque côté.

### 3.2 Câblage du capteur

Schéma identique aux capteurs de suivi existants (cf. `Dossier_Technique_Suiveur_de_Ligne.md` §2.2) :

```
            VCC (+5V)                          VCC (+5V)
              │                                  │
            [220 Ω] R_LED_C                    [10 kΩ] R_pull_C
              │                                  │
          LED IR ── (faisceau IR vers sol) ─▶ Phototransistor
              │                                  │
             GND                                 ├─────▶ V_capt_C  (vers entrée + du LM393)
                                                 │
                                                Émetteur ── GND
```

| Surface sous le capteur central | $V_{capt\_C}$ |
|---------------------------------|---------------|
| **Ligne noire présente** (état nominal) | **0.3 – 1.0 V** |
| **Ligne effacée / blanche** (à détecter !) | **3.5 – 4.5 V** |

> La logique est l'**inverse** de ce que l'on souhaite : quand la ligne est *présente*, $V_{capt\_C}$ est *bas*. Il faudra donc **inverser** l'information pour obtenir un signal "haut = pompe à activer".

---

## 4. Étage de comparaison & inversion logique

L'inversion est obtenue **gratuitement** en câblant le comparateur LM393 avec :
- $V_{capt\_C}$ sur l'entrée **inverseuse** (−)
- $V_{seuil\_pompe}$ sur l'entrée **non-inverseuse** (+)

### 4.1 Schéma

```
                          VCC (+5V)
                            │
                          [4.7 kΩ] R_pull-up
                            │
   V_seuil_pompe ──▶(+)     │
                       ┌────┴────┐
                       │  LM393  │── V_inv ──▶ vers filtre RC (étage 5)
                       │ (canal) │
   V_capt_C    ──▶(−)  └─────────┘
```

> Le LM393 utilisé peut être un **3ᵉ canal** d'un boîtier LM324 (4 AOP), ou bien un **2ᵉ boîtier LM393** dédié au module pompe. Le BOM ci-dessous prévoit un **second LM393** pour garantir l'indépendance totale entre suivi et actionneur.

### 4.2 Fonctionnement

$$V_{inv} = \begin{cases} V_{CC} \quad (\approx 5\,V) & \text{si } V_{capt\_C} < V_{seuil\_pompe} \quad\text{(ligne PRÉSENTE → pas de pompe)} \\ 0\,V & \text{si } V_{capt\_C} > V_{seuil\_pompe} \quad\text{(ligne EFFACÉE → }\textbf{déclenchement}\text{)}\end{cases}$$

**Attention :** la sortie est *active basse* (passe à 0 V quand il faut activer la pompe). On en tient compte dans le déclenchement du 555 (entrée TRIG active basse — voir §6).

### 4.3 Réglage du seuil

Diviseur de tension par **potentiomètre 10 kΩ multitour** dédié au module :

```
        VCC (+5V)
           │
        [POT 10 kΩ multitour]
           │
           ├──────▶ V_seuil_pompe (vers entrée + du LM393)
           │
          GND
```

Valeur typique recherchée : $V_{seuil\_pompe} \approx 2.5\,V$, **identique au seuil des capteurs de suivi** (cohérence optique). Le potentiomètre indépendant permet toutefois un ajustement plus permissif (par exemple 3.0 V) pour ne déclencher que sur une zone *vraiment* effacée et non sur une simple bavure.

---

## 5. Filtre anti-faux-déclenchement (RC + hystérésis)

### 5.1 Pourquoi ce filtre ?

Sans filtrage, la pompe se déclencherait :
- Sur chaque **micro-poussière** ou rayure blanche traversée à grande vitesse,
- Lors de chaque **virage serré** où le capteur central peut momentanément quitter la ligne,
- Sur les **ondulations** de signal liées à la lumière ambiante ou aux vibrations.

### 5.2 Hystérésis sur le LM393

Ajout d'une résistance de **rétroaction positive** $R_{hyst} = 1\,M\Omega$ entre la sortie du LM393 et son entrée (+) :

```
                  R_hyst (1 MΩ)
        ┌──────────────────────────────┐
        │                              │
   V_seuil_pompe ──[10 kΩ]──▶(+)       │
                                  ┌────┴────┐
                                  │  LM393  │── V_inv ──▶
                                  │         │
   V_capt_C       ─────────▶(−)   └─────────┘
```

Largeur d'hystérésis :

$$\Delta V = V_{CC} \times \frac{R_1}{R_1 + R_{hyst}} = 5 \times \frac{10\,000}{10\,000 + 1\,000\,000} \approx 50\,mV$$

→ Élimine les oscillations parasites au franchissement du seuil.

### 5.3 Filtre RC passe-bas en aval

```
   V_inv ──[R_f = 100 kΩ]──┬──▶ V_filt  (vers TRIG du NE555)
                           │
                       [C_f = 1 µF]
                           │
                          GND
```

Constante de temps :

$$\tau = R_f \times C_f = 100\,k\Omega \times 1\,\mu F = 100\,ms$$

Le filtre impose à la condition "ligne effacée" de durer au moins **~70 ms** (≈ 0.7 τ) avant de pouvoir déclencher le 555. À une vitesse d'avance typique du robot (~0.2 m/s), cela correspond à une zone effacée de **≥ 14 mm** — cohérent avec un défaut réel de la piste, et bien supérieur à un simple grain de poussière.

> **Note polarité :** comme la sortie du LM393 est active basse, le condensateur se *décharge* lors du déclenchement. Le NE555 utilisera sa broche TRIG (active basse), parfaitement adaptée à cette polarité — aucune inversion supplémentaire nécessaire.

---

## 6. Temporisation analogique — NE555 monostable

### 6.1 Rôle

Garantir que **chaque détection** produit une impulsion de durée **fixe et reproductible** envoyée à la pompe, indépendamment de la durée pendant laquelle le capteur reste sur la zone effacée. Cela évite :
- Que la pompe tourne en continu sur une longue zone effacée → flaque de peinture,
- Que la pompe ne fasse qu'un "clic" trop bref pour amorcer correctement.

### 6.2 Schéma monostable (configuration standard)

```
            VCC (+5V)
              │
              ├─────[R_t = 470 kΩ]─────┐
              │                        │
              │   ┌─────────────┐      │
              ├──▶│ 4 RESET     │      │
              │   │             │      │
              ├──▶│ 8 VCC       │      │
              │   │           7 ├──────┤
              │   │  NE555    6 ├──────┤
              │   │           5 ├─[10nF]─ GND  (CTRL — découplage)
              │   │           2 ├──◀── V_filt   (TRIG, actif bas)
              │   │           3 ├──────▶ V_555  (vers grille MOSFET)
              │   │           1 ├── GND │
              │   └─────────────┘      │
              │                    [C_t = 1 µF]
              │                        │
              │                       GND
```

### 6.3 Calcul de la durée d'impulsion

Formule monostable 555 :

$$T_{on} = 1.1 \times R_t \times C_t = 1.1 \times 470\,k\Omega \times 1\,\mu F \approx 517\,ms$$

→ Chaque détection produit **~0.5 s** d'activation pompe. À ajuster avec $R_t$ selon le débit de la pompe et la quantité de peinture désirée.

| $R_t$ | $C_t$ | $T_{on}$ |
|-------|-------|----------|
| 220 kΩ | 1 µF | ~240 ms |
| **470 kΩ** | **1 µF** | **~517 ms** ← valeur retenue |
| 1 MΩ | 1 µF | ~1.1 s |
| 470 kΩ | 2.2 µF | ~1.14 s |

### 6.4 Ré-armement

Après $T_{on}$, la sortie OUT (broche 3) retombe à 0 V. Une nouvelle impulsion ne peut être déclenchée qu'après nouveau **front descendant** sur TRIG (broche 2). Cela impose une distance minimale entre deux dépôts de peinture, équivalente à la distance parcourue pendant $T_{on}$, soit ~10 cm à 0.2 m/s — comportement souhaité.

---

## 7. Étage de puissance — MOSFET + pompe

### 7.1 Schéma

```
                                V_bat (7.4 V)
                                    │
                                    ├────────┐
                                    │        │
                                    │     ┌──┴──┐
                                    │     │     │
                                    │     │POMPE│  (charge inductive)
                                    │     │ DC  │
                                    │     │     │
                                    │     └──┬──┘
                                    │        │
                            ┌───────┴──┐     │
                  D_FW ──▶ │1N5819    │     │           Drain
                  (cathode)│Schottky  │     ├──────────────┐
                           │          │     │              │
                  (anode) ◀┘          │     │           ┌──┴──┐
                                      │     │           │ Q1  │
                                      └─────┘           │ N-  │
                                                        │MOSFET│  IRLZ44N
                                                        │     │
                                  V_555 ──[R_g=220Ω]──▶│Gate │
                                                        │     │
                                  R_pd = 10 kΩ          │Source
                                  (V_555 ↔ GND)         │  │
                                                        └──┬──┘
                                                           │
                                                          GND
```

### 7.2 Rôles des composants

| Composant | Rôle |
|-----------|------|
| **Q1 — IRLZ44N** | MOSFET N-channel **logic-level** ($V_{GS(th)} \approx 1\text{–}2\,V$), pleinement saturé à $V_{GS} = 5\,V$. Supporte 47 A, $R_{DS(on)} = 22\,m\Omega$ — surdimensionné pour 500 mA. |
| **R_g = 220 Ω** | Résistance de grille — limite le courant de charge de la capacité de grille (commutation propre, pas de dépassement). |
| **R_pd = 10 kΩ** | Pull-down de grille — garantit que le MOSFET reste OFF si la sortie du 555 est flottante (ex : démarrage). |
| **D_FW — 1N5819** | Diode de roue libre Schottky en antiparallèle sur la pompe — absorbe la surtension inductive lors de la coupure ($V_{spike}$ pourrait dépasser plusieurs dizaines de volts sans elle). |

### 7.3 Vérification

- $V_{GS}$ disponible : 5 V (sortie du 555 → ~4.5 V réel, compatible logic-level) ✓
- $I_D$ requis : ~500 mA, $I_D$ max : 47 A → marge ×90 ✓
- Dissipation MOSFET : $P = R_{DS(on)} \times I^2 = 0.022 \times 0.5^2 = 5.5\,mW$ → négligeable, pas de dissipateur. ✓

---

## 8. Choix de la pompe

### 8.1 Comparatif des options

| Type | Référence typique | Tension | Débit | Avantages | Inconvénients |
|------|-------------------|---------|-------|-----------|---------------|
| **Micro-pompe péristaltique DC** | Kamoer KPP-B04 / Adafruit #1150 | 6 V | 100 ml/min | Précise, pas d'amorçage, dose reproductible | Plus chère (~15–25€) |
| **Pompe à diaphragme miniature** | R385 | 6–12 V | 1.5–2 L/min | Bon marché (~5€), débit élevé | Bruyante, débit difficile à doser |
| **Pompe centrifuge submersible** | (générique aquarium) | 3–6 V | variable | Très bon marché | Doit être amorcée, débit imprécis |
| **Électrovanne + réservoir gravitaire** | Solénoïde 6 V + tube | 6 V | dépend hauteur | Très simple, pas de moteur | Dépend de l'orientation, peut couler |

### 8.2 Recommandation

→ **Micro-pompe péristaltique 6 V** (type Kamoer KPP ou équivalente Adafruit).

**Justification :**
1. **Dose reproductible** : un tour de moteur = un volume défini, donc $T_{on} = 0.5\,s$ délivre toujours la même quantité de peinture (~0.8 ml).
2. **Pas d'amorçage** : la pompe peut rester à sec entre deux activations sans désamorçage.
3. **Pas de retour** : le tube reste plein, pas de gouttes parasites entre activations.
4. **Compatible avec une peinture liquide** (acrylique diluée, encre) — pas de pièces métalliques en contact avec le fluide.
5. **Tension nominale 6 V** compatible avec la batterie LiPo 2S après chute du MOSFET ($V_{moteur} \approx V_{bat} - 0.01\,V \approx 7.4\,V$ → légèrement au-dessus du nominal, durée d'impulsion limitée à 0.5 s = pas de surchauffe).

> **Alternative économique :** la **R385** (5€) reste un excellent choix de prototypage si la précision du dosage n'est pas critique.

### 8.3 Réservoir et tuyauterie

```
   ┌─────────────┐
   │  Réservoir   │  (flacon 30–50 ml, monté sur le châssis)
   │  peinture    │
   └──────┬──────┘
          │ tube silicone Ø 3 mm
          │
       ┌──▼──┐
       │POMPE│
       └──┬──┘
          │ tube silicone Ø 3 mm
          │
          ▼
      [Buse de dépôt — orientée vers le sol, juste derrière le capteur central]
```

---

## 9. Schéma électrique complet du module

```
   VCC=+5V                                                                                   V_bat=+7.4V
     │                                                                                            │
     │                                                                                            │
   ┌─┴─┐                                                                                          │
   │220│ R_LED_C                                                                                  │
   └─┬─┘                                                                                          │
     │                                                                                            │
   ─LED IR─    (faisceau vers sol)                                                                │
     │                                                                                            │
    GND                                                                                           │
                                                                                                  │
   VCC                                                                                            │
     │                                                                                            │
   ┌─┴──┐                                                                                         │
   │10kΩ│ R_pull_C                                                                                │
   └─┬──┘                                                                                         │
     │                                                                                            │
   PHOTOTRANSISTOR ── V_capt_C ─────────────▶ (−) entrée LM393 (canal pompe)                      │
     │                                              │                                             │
    GND                            VCC ──[POT 10kΩ]── V_seuil_pompe ──▶ (+) entrée LM393         │
                                       │                                  │                      │
                                      GND                                 │                      │
                                                                ┌─────────┴────────┐              │
                                                                │  LM393 #2 canal A│              │
                                                                │  + R_pull-up 4.7kΩ vers VCC    │
                                                                │  + R_hyst 1MΩ (réact. positive)│
                                                                └────────┬─────────┘              │
                                                                         │ V_inv (actif bas)      │
                                                            R_f=100kΩ ──┤                         │
                                                                         │                        │
                                                                  C_f=1µF── GND                   │
                                                                         │                        │
                                                                         ▼ V_filt                  │
                                                                ┌────────┴────────┐                │
                                                                │   NE555         │                │
                                                                │  TRIG (broche 2)│                │
                                                                │  R_t=470kΩ→VCC  │                │
                                                                │  C_t=1µF→GND    │                │
                                                                │  T_on ≈ 517 ms  │                │
                                                                │  OUT (broche 3) │                │
                                                                └────────┬────────┘                │
                                                                         │ V_555                   │
                                                                  R_g=220Ω                         │
                                                                         │                         │
                                                                         ▼                         │
                                                                       Gate                        │
                                                              ┌─────────┴────────┐                 │
                                                              │  Q1  IRLZ44N     │                 │
                                                  R_pd=10kΩ ──┤  Drain ──────────┼─────┐           │
                                                  (vers GND)  │  Source ── GND   │     │           │
                                                              └──────────────────┘     │           │
                                                                                       ▼           │
                                                                                  ┌────┴────┐      │
                                                                                  │ POMPE DC │     │
                                                                                  │  6 V     │     │
                                                                                  │  ~500 mA │     │
                                                                                  └────┬────┘      │
                                                                                       │           │
                                                                          1N5819 ── ◀─┤           │
                                                                       (anode→drain,   │           │
                                                                        cathode→V_bat) ├───────────┘
                                                                                       │
                                                                                       ▼
                                                                                     V_bat
```

---

## 10. Nomenclature additive (BOM module)

> **Tous les composants ci-dessous sont en SUPPLÉMENT** du BOM principal. Aucun composant existant n'est retiré ni remplacé.

### 10.1 Détection & traitement du signal

| # | Composant | Référence | Quantité | Rôle |
|---|-----------|-----------|----------|------|
| A1 | Capteur IR central | TCRT5000 | 1 | Détection ligne effacée |
| A2 | Résistance limit. LED IR | 220 Ω, 1/4 W | 1 | Limitation courant LED IR centrale |
| A3 | Résistance pull-up phototr. | 10 kΩ, 1/4 W | 1 | Conversion courant → tension |
| A4 | Comparateur dédié module | LM393 | 1 | Comparateur "ligne effacée" |
| A5 | Résistance pull-up sortie | 4.7 kΩ, 1/4 W | 1 | Pull-up sortie collecteur ouvert LM393 |
| A6 | Résistance hystérésis | 1 MΩ, 1/4 W | 1 | Réaction positive anti-oscillation |
| A7 | Potentiomètre seuil pompe | 10 kΩ multitour | 1 | Réglage indépendant du seuil |
| A8 | Résistance filtre RC | 100 kΩ, 1/4 W | 1 | Constante de temps anti-rebond |
| A9 | Condensateur filtre RC | 1 µF céramique | 1 | Constante de temps anti-rebond |

### 10.2 Temporisation

| # | Composant | Référence | Quantité | Rôle |
|---|-----------|-----------|----------|------|
| A10 | Timer monostable | NE555 (DIP-8) | 1 | Génération impulsion fixe ~500 ms |
| A11 | Résistance temporisation | 470 kΩ, 1/4 W | 1 | $R_t$ du monostable |
| A12 | Condensateur temporisation | 1 µF céramique X7R | 1 | $C_t$ du monostable |
| A13 | Condensateur découplage CTRL | 10 nF céramique | 1 | Découplage broche 5 du 555 |
| A14 | Condensateur découplage VCC 555 | 100 nF céramique | 1 | Découplage alimentation 555 |

### 10.3 Étage de puissance

| # | Composant | Référence | Quantité | Rôle |
|---|-----------|-----------|----------|------|
| A15 | MOSFET N logic-level | IRLZ44N (TO-220) | 1 | Commutation pompe |
| A16 | Résistance de grille | 220 Ω, 1/4 W | 1 | Limitation courant grille |
| A17 | Résistance pull-down grille | 10 kΩ, 1/4 W | 1 | État OFF par défaut |
| A18 | Diode de roue libre | 1N5819 Schottky | 1 | Protection surtension inductive pompe |

### 10.4 Actionneur & fluidique

| # | Composant | Référence | Quantité | Rôle |
|---|-----------|-----------|----------|------|
| A19 | Pompe DC | Pompe péristaltique 6 V (Kamoer KPP-B04) ou R385 | 1 | Actionneur |
| A20 | Réservoir | Flacon 30–50 ml avec bouchon percé | 1 | Stockage peinture |
| A21 | Tube silicone Ø 3 mm | Longueur ~30 cm | 1 | Acheminement fluide |
| A22 | Buse de sortie | Embout métallique Ø 1.5 mm | 1 | Dépôt précis |
| A23 | Colliers / supports | Brides plastiques | 4 | Fixation tubes et réservoir |

### 10.5 Coût estimatif additif

| Poste | Coût indicatif |
|-------|----------------|
| Composants électroniques (A1–A18) | ~6 € |
| Pompe péristaltique (A19) | ~18 € |
| Fluidique & support (A20–A23) | ~4 € |
| **Total module pompe** | **~28 €** |

---

## 11. Calculs de dimensionnement

### 11.1 LED IR centrale (A2)

Identique au calcul du document principal :

$$R_{LED} = \frac{V_{CC} - V_F}{I_F} = \frac{5 - 1.2}{0.020} = 190\,\Omega \rightarrow 220\,\Omega \text{ normalisée}$$

$$P_R = R \cdot I^2 = 220 \cdot (0.0173)^2 = 66\,mW \ll 250\,mW \quad ✓$$

### 11.2 Hystérésis du LM393 (A6)

$$\Delta V_{hyst} = V_{CC} \cdot \frac{R_3}{R_3 + R_{hyst}} = 5 \cdot \frac{10\,000}{1\,010\,000} \approx 49.5\,mV$$

Suffisant pour étouffer le bruit du phototransistor (typiquement ±10 mV).

### 11.3 Filtre RC anti-rebond (A8 + A9)

$$\tau = R_f \cdot C_f = 10^5 \cdot 10^{-6} = 100\,ms$$

Délai effectif de déclenchement (chute à ~33 % du seuil 555 fixé à $V_{CC}/3$) :

$$t_{déclenche} \approx \tau \cdot \ln\left(\frac{V_{CC}}{V_{CC}/3}\right) = 0.1 \cdot \ln(3) \approx 110\,ms$$

→ **Une zone effacée de moins de 22 mm à 0.2 m/s ne déclenchera PAS la pompe** : exigence respectée.

### 11.4 Monostable NE555 (A11 + A12)

$$T_{on} = 1.1 \cdot R_t \cdot C_t = 1.1 \cdot 470 \cdot 10^3 \cdot 10^{-6} = 517\,ms$$

À 0.2 m/s, dépôt sur ~10 cm de longueur de piste — cohérent avec un trait de marquage.

### 11.5 Étage de puissance — IRLZ44N (A15)

Vérification de la dissipation à $I_D = 500\,mA$ :

$$P_{cond} = R_{DS(on)} \cdot I_D^2 = 0.022 \cdot 0.25 = 5.5\,mW$$

$$P_{comm} \approx \frac{1}{2} \cdot V_{DS} \cdot I_D \cdot (t_r + t_f) \cdot f \approx 0\,W \quad (\text{commutation à } 1\,Hz \text{ max})$$

→ Boîtier TO-220 sans dissipateur largement suffisant.

### 11.6 Bilan de consommation additionnel

| Bloc | Courant moyen | Courant pic |
|------|---------------|-------------|
| LED IR centrale + photo + LM393 + 555 | ~10 mA continu | 10 mA |
| Pompe (active 0.5 s sur ~10 s typique) | ~25 mA moyen | 500 mA pic |
| **Total module** | **~35 mA moyen** | **510 mA pic** |

Impact sur l'autonomie : la consommation moyenne du robot passe de ~850 mA à **~885 mA** (+4 %). Autonomie LiPo 1500 mAh : **~1h 40 → ~1h 35** (impact négligeable).

### 11.7 Vérification du LM7805

Le module ajoute ~10 mA sur le rail 5 V. Total rail logique : ~60 mA → bien en dessous des 1000 mA du LM7805.

$$P_{diss,LM7805} = (7.4 - 5) \cdot 0.060 = 144\,mW$$

Élévation thermique (TO-220 sans dissipateur) : ~9 °C → reste froid.

---

## 12. Calibration et procédure d'essai

### 12.1 Test à sec (sans peinture, sans tube)

| Étape | Action | Résultat attendu |
|-------|--------|------------------|
| 1 | Alimenter le robot, déconnecter la pompe | Aucune pompe alimentée |
| 2 | Placer le robot sur la **ligne noire**, capteur central au-dessus du noir | $V_{capt\_C} \approx 0.5\,V$ (multimètre sur sortie phototransistor central) |
| 3 | Mesurer $V_{seuil\_pompe}$ | Ajuster pot. à **2.5 V** |
| 4 | Mesurer sortie LM393 (V_inv) | Doit être **~5 V** (HIGH = pas de déclenchement) |
| 5 | Glisser une feuille blanche sous le capteur central pendant **1 s** | V_inv → 0 V, puis sortie 555 (broche 3) → 5 V pendant ~500 ms |
| 6 | Mesurer la durée d'impulsion à l'oscilloscope ou avec une LED témoin sur la sortie 555 | $T_{on}$ entre **400 et 600 ms** |
| 7 | Glisser la feuille pendant **50 ms seulement** (rapide) | Aucune impulsion 555 (filtre RC anti-rebond efficace) |

### 12.2 Test avec pompe (sans peinture)

| Étape | Action | Résultat attendu |
|-------|--------|------------------|
| 1 | Reconnecter la pompe (à sec) | — |
| 2 | Répéter le test 5 ci-dessus | La pompe tourne pendant ~0.5 s puis s'arrête |
| 3 | Vérifier l'absence de surchauffe MOSFET au toucher après 10 cycles | Tiède au plus |

### 12.3 Test avec peinture

| Étape | Action | Résultat attendu |
|-------|--------|------------------|
| 1 | Remplir le réservoir avec de la peinture acrylique diluée à 50 % | — |
| 2 | Amorcer la pompe (1–2 cycles à vide pour remplir le tube) | Goutte au bout de la buse |
| 3 | Tracer une ligne noire sur papier blanc avec une **interruption de 30 mm** | Référence de test |
| 4 | Faire passer le robot sur la ligne | Dépôt de peinture localisé sur l'interruption |

### 12.4 Réglages fins

| Symptôme | Réglage |
|----------|---------|
| Pompe se déclenche trop facilement (faux positifs) | Augmenter $V_{seuil\_pompe}$ vers 3.0 V |
| Pompe ne se déclenche jamais | Diminuer $V_{seuil\_pompe}$ vers 2.0 V ; vérifier alignement capteur central |
| Dépôt trop court / trop bref | Augmenter $R_t$ (1 MΩ → ~1.1 s) |
| Dépôt trop long / flaque | Diminuer $R_t$ (220 kΩ → ~0.24 s) ou diluer la peinture |
| Déclenchement sur petits défauts | Augmenter $C_f$ (2.2 µF → délai ~240 ms) |

---

## 13. Analyse des cas de fonctionnement

### 13.1 Tableau de vérité combiné (suivi + pompe)

| Capt. G | Capt. C | Capt. D | Action moteurs | Action pompe |
|:-:|:-:|:-:|---|---|
| Blanc | **Noir** | Blanc | **Tout droit** | OFF (ligne OK sous le centre) |
| Noir | Noir | Blanc | Tourne gauche | OFF |
| Blanc | Noir | Noir | Tourne droite | OFF |
| Noir | Noir | Noir | Arrêt (intersection) | OFF |
| Blanc | **Blanc** | Blanc | Arrêt (ligne perdue) | **ON** ← *cas cible : ligne effacée droit devant* |
| Noir | **Blanc** | Blanc | Tourne gauche | **ON** (ligne effacée mais robot peut suivre via capt. G) |
| Blanc | **Blanc** | Noir | Tourne droite | **ON** (ligne effacée mais robot peut suivre via capt. D) |

> **Comportement émergent intéressant :** quand le robot perd complètement la ligne (ligne 100 % effacée sur une zone), les moteurs s'arrêtent (cas existant) **et** la pompe dépose une dose au point d'arrêt. Si la zone effacée est plus courte que ~10 cm, les capteurs G et D restent actifs et le robot continue d'avancer **tout en repeignant** la zone effacée — fonction "réparation à la volée".

### 13.2 Chronogramme d'un passage sur une zone effacée

```
                        Zone effacée (50 mm) à 0.2 m/s → 250 ms sous capteur central
                        ◄──────────────────────────────────────►
V_capt_C  ____________█████████████████████████____________________  (0.5V→4V→0.5V)
                      │                          │
V_inv     ▔▔▔▔▔▔▔▔▔▔▔▔▔__________________________▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔  (5V→0V→5V, actif bas)
                          │
V_filt    ▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔╲___________________╱▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔  (chute progressive ~110ms)
                                │
TRIG 555  (passage sous V_CC/3 à t≈110 ms après début de la zone effacée)
                                │
OUT 555   _________________________█████████████████████████____  ← impulsion 517 ms
                                   │                          │
POMPE     OFF                      ON ──────────────────────  OFF
                                   ▲
                                   └─ Dépôt commence ~110 ms après l'entrée dans la zone
                                      → bavure/peinture déposée sur la zone effacée + ~5 cm après
```

### 13.3 Robustesse au démarrage

- À la mise sous tension, $C_f$ se charge à $V_{CC}$ via $R_f$ (le LM393 est HIGH si la ligne est présente sous le capteur central) → TRIG du 555 reste à HIGH → **pas de déclenchement parasite au démarrage** ✓
- Si le robot démarre **hors ligne** (capteur central voit blanc), V_inv = 0 V immédiatement → le filtre RC se décharge en ~110 ms → **une impulsion** est envoyée au démarrage. C'est acceptable (le robot est de toute façon arrêté faute de signal de suivi, et une dose de peinture marque le point de départ).
- La résistance $R_{pd}$ = 10 kΩ sur la grille du MOSFET garantit que la pompe **reste OFF** pendant les ~50 ms d'établissement des alimentations.

---

## 14. Sécurité & points de vigilance

### 14.1 Sécurité électrique

| Risque | Mitigation |
|--------|------------|
| Surtension inductive de la pompe → destruction du MOSFET | Diode 1N5819 Schottky en antiparallèle (A18) |
| Courant de fuite grille → MOSFET en état indéterminé | Pull-down 10 kΩ (A17) |
| Sortie 555 flottante au démarrage → activation parasite | Découplage 100 nF + pull-down grille |
| Court-circuit de la pompe (peinture coagulée) | MOSFET surdimensionné (47 A vs 0.5 A) ; envisager fusible 1 A en série avec la pompe |

### 14.2 Sécurité fluidique

| Risque | Mitigation |
|--------|------------|
| Peinture qui sèche dans le tube | Rincer avec eau après chaque session ; utiliser peinture acrylique soluble à l'eau |
| Fuite sur l'électronique | Réservoir et tubes **sous** le niveau de la carte électronique ; carte sur entretoises hautes |
| Renversement du robot → peinture sur l'utilisateur | Bouchon de réservoir étanche ; volume limité (50 ml max) |

### 14.3 Point de vigilance majeur — Indépendance fonctionnelle

> **Le module pompe ne doit jamais perturber le suivi de ligne.** Vérifier :
> 1. Le capteur central est positionné à au moins **10 mm** de chaque capteur de suivi pour éviter toute diaphonie optique.
> 2. Le rail 5 V est partagé : ajouter un **condensateur 100 µF** en sortie du LM7805 (déjà prévu dans le BOM principal, position §6.2) pour absorber les pics de courant lors des activations pompe.
> 3. Le rail $V_{bat}$ alimente la pompe **directement** depuis la batterie via une piste dédiée — ne **PAS** passer par le rail moteur du L293D (sinon les à-coups pompe perturberaient la commande moteur).

### 14.4 En cas de panne du module

Le module est **strictement passif vis-à-vis du suivi** : un débranchement du capteur central, du LM393 #2, du 555, du MOSFET ou de la pompe **n'a aucun effet** sur le comportement du robot suiveur. Le robot continue à suivre la ligne normalement, simplement sans dépôt de peinture.

---

## 15. Améliorations possibles

### 15.1 Modulation du débit (purement analogique)

Remplacer la sortie tout-ou-rien du 555 par un **555 en astable PWM** dont le rapport cyclique varie en fonction de la « blancheur » détectée — la pompe pulse plus fort si la zone est très effacée que si elle est juste défraîchie. Réalisable avec un AOP différentiel (LM358) qui module la résistance équivalente d'entrée du 555.

### 15.2 Détection couleur (RGB)

Remplacer le TCRT5000 central par un capteur couleur analogique (TCS3200 — sortie en fréquence convertible en tension via 555 démodulateur) pour ne déposer la peinture **que** si la couleur détectée s'écarte de la couleur d'origine de la ligne. Reste réalisable sans MCU.

### 15.3 Mémoire de zone (latch RC long)

Ajouter un second 555 monostable cascadé avec $T = 5\,s$ qui **inhibe** le premier 555 après chaque déclenchement, pour éviter de redéposer sur la même zone si le robot fait demi-tour. Réalisable avec deux 555 et une porte AND analogique (transistor BJT).

### 15.4 Compteur de doses

Ajouter un compteur analogique (suite de bascules monostable) pour stopper le robot après N dépôts (réservoir vide). Réalisable avec un CD4017 (compteur décimal) — composant logique mais non programmable, donc compatible avec la philosophie "pur hard".

### 15.5 Indicateur visuel

LED rouge sur la sortie du 555 (broche 3) avec résistance 1 kΩ : signale visuellement chaque activation de la pompe — utile pour le débogage et la démonstration.

```
   Sortie 555 (broche 3) ──[1kΩ]──▶│LED rouge│──▶ GND
```

---

## Résumé exécutif

Ce module ajoute au robot suiveur de ligne une fonction d'**actionneur de redéposition de peinture** entièrement analogique, structurée en 5 étages : **détection** (capteur IR central), **comparaison** (LM393 avec hystérésis et inversion logique), **filtrage** (RC passe-bas anti-rebond), **temporisation** (NE555 monostable ~500 ms) et **puissance** (MOSFET IRLZ44N + pompe péristaltique 6 V). Le module est **strictement additif**, partage uniquement les rails d'alimentation existants, n'altère en rien le comportement du suivi de ligne, et reste **fidèle au concept "Pur Hard"** — zéro ligne de code, zéro microcontrôleur. Surcoût matériel : **~28 €**. Impact sur l'autonomie : **< 5 %**.
