# Démarche de Conception Pas à Pas — Robot Suiveur de Ligne "Pur Hard"

## Carnet de bord du concepteur

> Ce document reprend **l'ensemble du projet** dans l'ordre chronologique réel où un concepteur le ferait, depuis la feuille blanche. À chaque étape, on part d'un besoin, on liste les options envisagées, on calcule, on tranche, et on en déduit le composant à câbler. Le but n'est pas de justifier *a posteriori* un BOM figé, mais de **revivre la démarche**.

> Le document est organisé en **deux parties** :
> - **Partie I — Conception du robot suiveur de ligne** (étapes I.0 à I.18) : le robot complet sans actionneur, depuis le cahier des charges jusqu'à la mise en service.
> - **Partie II — Intégration du 3ᵉ actionneur (pompe à peinture)** (étapes II.0 à II.20 + bonus B1 à B12) : on reprend une feuille blanche et on **ajoute** la fonction "redéposition de peinture" sans rien casser de la Partie I.

---

# 📚 Table des matières générale

## Partie I — Robot Suiveur de Ligne (sans actionneur)

- [Étape I.0 — Cahier des charges du robot](#étape-i0--cahier-des-charges-du-robot)
- [Étape I.1 — Philosophie : pourquoi 100 % analogique ?](#étape-i1--philosophie--pourquoi-100--analogique-)
- [Étape I.2 — Décomposition fonctionnelle en blocs](#étape-i2--décomposition-fonctionnelle-en-blocs)
- [Étape I.3 — Choix du type de capteur de ligne](#étape-i3--choix-du-type-de-capteur-de-ligne)
- [Étape I.4 — Géométrie : combien de capteurs et comment les espacer ?](#étape-i4--géométrie--combien-de-capteurs-et-comment-les-espacer-)
- [Étape I.5 — Câblage des capteurs : LED IR + phototransistor](#étape-i5--câblage-des-capteurs--led-ir--phototransistor)
- [Étape I.6 — Mesure de référence et choix du seuil de détection](#étape-i6--mesure-de-référence-et-choix-du-seuil-de-détection)
- [Étape I.7 — Choix du comparateur](#étape-i7--choix-du-comparateur)
- [Étape I.8 — Câblage du comparateur : choix des polarités](#étape-i8--câblage-du-comparateur--choix-des-polarités)
- [Étape I.9 — Pull-up de la sortie collecteur ouvert](#étape-i9--pull-up-de-la-sortie-collecteur-ouvert)
- [Étape I.10 — Anti-oscillation : hystérésis sur le comparateur](#étape-i10--anti-oscillation--hystérésis-sur-le-comparateur)
- [Étape I.11 — Choix du type de moteur de propulsion](#étape-i11--choix-du-type-de-moteur-de-propulsion)
- [Étape I.12 — Pourquoi un pont en H ? Choix du driver](#étape-i12--pourquoi-un-pont-en-h--choix-du-driver)
- [Étape I.13 — Câblage L293D : du comparateur au moteur](#étape-i13--câblage-l293d--du-comparateur-au-moteur)
- [Étape I.14 — Architecture d'alimentation : 2 rails séparés](#étape-i14--architecture-dalimentation--2-rails-séparés)
- [Étape I.15 — Choix de la batterie](#étape-i15--choix-de-la-batterie)
- [Étape I.16 — Châssis, roues et roue folle](#étape-i16--châssis-roues-et-roue-folle)
- [Étape I.17 — Bilan énergétique du robot](#étape-i17--bilan-énergétique-du-robot)
- [Étape I.18 — Validation finale : table de vérité et tests](#étape-i18--validation-finale--table-de-vérité-et-tests)
- [Synthèse Partie I](#synthèse-partie-i)

## Partie II — Module Actionneur Pompe à Peinture

- [Étape II.0 — Cahier des charges du module pompe](#étape-ii0--cahier-des-charges-du-module-pompe)
- [Étape II.1 — Choix de la philosophie : analogique ou logique programmée ?](#étape-ii1--choix-de-la-philosophie--analogique-ou-logique-programmée-)
- [Étape II.2 — Décomposition fonctionnelle en blocs](#étape-ii2--décomposition-fonctionnelle-en-blocs)
- [Étape II.3 — Choix du capteur de détection](#étape-ii3--choix-du-capteur-de-détection)
- [Étape II.4 — Position mécanique du capteur](#étape-ii4--position-mécanique-du-capteur)
- [Étape II.5 — Câblage du capteur : LED IR + phototransistor](#étape-ii5--câblage-du-capteur--led-ir--phototransistor)
- [Étape II.6 — Mesure de référence et choix du seuil](#étape-ii6--mesure-de-référence-et-choix-du-seuil)
- [Étape II.7 — Choix du comparateur](#étape-ii7--choix-du-comparateur)
- [Étape II.8 — Câblage du comparateur : sens et inversion logique](#étape-ii8--câblage-du-comparateur--sens-et-inversion-logique)
- [Étape II.9 — Pull-up de la sortie collecteur ouvert](#étape-ii9--pull-up-de-la-sortie-collecteur-ouvert)
- [Étape II.10 — Anti-oscillation : ajout de l'hystérésis](#étape-ii10--anti-oscillation--ajout-de-lhystérésis)
- [Étape II.11 — Anti-faux-déclenchement : filtre RC passe-bas](#étape-ii11--anti-faux-déclenchement--filtre-rc-passe-bas)
- [Étape II.12 — Choix du temporisateur : pourquoi un monostable ?](#étape-ii12--choix-du-temporisateur--pourquoi-un-monostable-)
- [Étape II.13 — Calcul de la durée d'impulsion](#étape-ii13--calcul-de-la-durée-dimpulsion)
- [Étape II.14 — Choix de la pompe](#étape-ii14--choix-de-la-pompe)
- [Étape II.15 — Choix du transistor de puissance](#étape-ii15--choix-du-transistor-de-puissance)
- [Étape II.16 — Sécurisation de la grille du MOSFET](#étape-ii16--sécurisation-de-la-grille-du-mosfet)
- [Étape II.17 — Protection contre la surtension inductive](#étape-ii17--protection-contre-la-surtension-inductive)
- [Étape II.18 — Architecture d'alimentation : ne rien casser](#étape-ii18--architecture-dalimentation--ne-rien-casser)
- [Étape II.19 — Bilan énergétique et impact sur l'autonomie](#étape-ii19--bilan-énergétique-et-impact-sur-lautonomie)
- [Étape II.20 — Vérification finale : qu'est-ce qui peut mal se passer ?](#étape-ii20--vérification-finale--quest-ce-qui-peut-mal-se-passer-)
- [Synthèse Partie II](#synthèse-partie-ii)

### 🎁 Bonus — Intégration physique de la pompe sur le robot existant

- [B1 — État des lieux avant intégration](#b1--état-des-lieux-avant-intégration)
- [B2 — Préparation et test des composants neufs](#b2--préparation-et-test-des-composants-neufs-avant-montage)
- [B3 — Stratégie de montage : du plus passif au plus actif](#b3--stratégie-de-montage--du-plus-passif-au-plus-actif)
- [B4 — Câblage du capteur central](#b4--sous-bloc-1--câblage-du-capteur-central)
- [B5 — Câblage du comparateur LM393 #2](#b5--sous-bloc-2--câblage-du-comparateur-lm393-2)
- [B6 — Filtre RC + monostable NE555](#b6--sous-bloc-3--filtre-rc--monostable-ne555)
- [B7 — Étage de puissance MOSFET (à vide)](#b7--sous-bloc-4--étage-de-puissance-mosfet-à-vide)
- [B8 — Branchement de la pompe (à sec)](#b8--sous-bloc-5--branchement-de-la-pompe-à-sec)
- [B9 — Intégration mécanique du circuit fluidique](#b9--sous-bloc-6--intégration-mécanique-du-circuit-fluidique)
- [B10 — Test sur piste réelle](#b10--test-sur-piste-réelle)
- [B11 — Mise en service finale](#b11--mise-en-service-finale-et-procédures-dutilisation)
- [B12 — Synthèse de l'intégration en une page](#b12--synthèse-de-lintégration-en-une-page)

---

# 🤖 PARTIE I — Conception du Robot Suiveur de Ligne

> **Point de départ :** une feuille blanche. Aucun composant n'est encore choisi. On part juste d'une idée : *"Je veux un robot qui suit tout seul une ligne noire tracée au sol, sans microcontrôleur."*

---

## Étape I.0 — Cahier des charges du robot

**Question de départ :** « Comment faire avancer un robot le long d'une ligne, sans aucune programmation ? »

**Réflexion :**
- Il faut **voir** la ligne (capteur optique).
- Il faut **décider** où elle est par rapport au robot (à gauche ? à droite ? sous lui ?).
- Il faut **réagir** : tourner les bonnes roues pour rester centré.
- Le tout sans une ligne de code.

J'écris mes contraintes :

| # | Contrainte | Origine |
|---|------------|---------|
| C1 | Suivre une ligne noire (18 mm) sur fond blanc | Sujet |
| C2 | Vitesse modérée (~0.2 m/s) | Sécurité + temps de réponse capteurs |
| C3 | 100 % analogique, zéro MCU, zéro code | Philosophie pédagogique du projet |
| C4 | Autonomie ≥ 1 h sur batterie embarquée | Démonstration |
| C5 | Châssis simple, prototype sur breadboard | Reproductibilité |
| C6 | Coût raisonnable (< 50 €) | Budget étudiant |

À ce stade : **aucun composant choisi**. On comprend le problème avant de tirer un fil.

---

## Étape I.1 — Philosophie : pourquoi 100 % analogique ?

C'est la question structurante qui détermine **toute** l'architecture suivante.

| Option | Avantages | Inconvénients | Verdict |
|--------|-----------|---------------|---------|
| **Microcontrôleur** (Arduino) | Flexible, facile à modifier, capteurs ADC précis | Nécessite firmware → viole C3 | ❌ |
| **Logique combinatoire** (74HC série) | Sans code, prédictible | Lourd pour une simple comparaison | ⚠️ Surdimensionné |
| **100 % analogique** (capteur → comparateur → puissance) | Simple, robuste, pédagogique, **temps de réponse < 20 µs** | Peu flexible (réglage par potentiomètre uniquement) | ✅ **Retenu** |

→ **Décision :** chaîne analogique pure. Pas un seul composant programmable dans tout le robot.

**Conséquence forte :** tout le « cerveau » du robot tient dans **un comparateur** qui décide, par capteur, si on est sur du noir ou sur du blanc. La régulation se fait par la **physique** : les bons moteurs s'allument, les mauvais s'éteignent.

---

## Étape I.2 — Décomposition fonctionnelle en blocs

Avant de tirer un fil, je dessine la chaîne de traitement la plus simple :

```
   [Voir la ligne] → [Décider] → [Commander] → [Avancer]
```

Traduction électronique :

| Verbe | Bloc | Signal d'entrée | Signal de sortie |
|-------|------|-----------------|------------------|
| Voir | Capteur opto | Réflectivité (lumière IR) | Tension analogique 0–5 V |
| Décider | Comparateur | 2 tensions | Logique 0/5 V |
| Commander | Pont en H | Logique | Courant moteur |
| Avancer | Moteur DC + roue | Courant | Mouvement mécanique |

**Règle que je me fixe :** je conçois **bloc par bloc, dans l'ordre du flux**, et je ne passe au bloc suivant qu'une fois le précédent figé. Un bloc ne peut pas être dimensionné sans connaître ce qu'il doit *fournir* au suivant.

---

## Étape I.3 — Choix du type de capteur de ligne

### Besoin

Distinguer **noir** (ligne) de **blanc** (sol nu) en un point précis du sol, sans contact.

### Options

| Capteur | Principe | Sensibilité ambiante | Coût | Verdict |
|---------|----------|----------------------|------|---------|
| Photorésistance LDR | Lumière visible | **Très** sensible | Bas | ❌ Inutilisable en éclairage variable |
| Capteur ultrason | Distance | Pas pour couleur | Moyen | ❌ Mauvais principe physique |
| Capteur couleur (TCS3200) | RGB par filtres | Faible | Moyen | Surdimensionné, sortie en fréquence |
| **Paire IR TCRT5000** | Réflexion IR 950 nm | **Faible** (filtre IR) | Bas (~0.50 €) | ✅ **Choix idéal** |

### Décision

→ **TCRT5000** : émetteur IR + phototransistor récepteur dans le même boîtier 5×4 mm, distance optimale 2–15 mm, longueur d'onde 950 nm peu perturbée par la lumière visible ambiante.

**Pourquoi pas une LED visible + LDR ?** Parce qu'un projecteur, un néon ou un rayon de soleil suffirait à fausser la mesure. L'IR à 950 nm est filtré naturellement par les TCRT5000 → robuste.

---

## Étape I.4 — Géométrie : combien de capteurs et comment les espacer ?

### Combien ?

| Nombre | Comportement possible | Complexité |
|--------|------------------------|------------|
| 1 | Ne sait que dire "je vois" / "je vois pas" → ne sait pas tourner | Trop simple |
| **2** | Sait dire si la ligne est à gauche ou à droite → **suffit pour suivre** | ✅ Minimum viable |
| 3+ | Anticipation des virages, traitement de croix | Bonus, pour Partie II |

→ **Décision :** **2 capteurs** (gauche G et droit D) en première intention.

### Espacement

```
                   ┌─── ligne noire 18 mm ───┐
                   │                          │
        [Capteur G]│                          │[Capteur D]
              │ <──── espacement E ────> │
                   │                          │
                   └──────────────────────────┘
```

**Règle :** l'espacement E doit être **légèrement supérieur à la largeur de la ligne**.
- Si E < largeur ligne → les deux capteurs peuvent voir noir simultanément en ligne droite → ambigu.
- Si E ≫ largeur ligne → la ligne disparaît entre les deux dans les virages serrés → perte de signal.

**Calcul :** ligne 18 mm → E = 20 à 25 mm. Je retiens **E = 20 mm**, soit 1 mm de marge de chaque côté de la ligne quand le robot est centré.

### Hauteur sol

Datasheet TCRT5000 : optimum à 5 mm. Trop haut → signal faible, trop bas → risque de toucher le sol → **5 mm** précisément.

---

## Étape I.5 — Câblage des capteurs : LED IR + phototransistor

Chaque capteur a deux moitiés indépendantes à câbler.

### Moitié émettrice (LED IR)

Loi d'Ohm :
$$R_{LED} = \frac{V_{CC} - V_F}{I_F} = \frac{5 - 1.2}{0.020} = 190\,\Omega$$

Valeur normalisée immédiatement supérieure : **220 Ω** (sécurité = courant légèrement réduit à 17.3 mA).

Vérification puissance : $P = R \cdot I^2 = 220 \cdot (0.0173)^2 = 66\,mW \ll 250\,mW$ → résistance 1/4 W. ✓

### Moitié réceptrice (phototransistor)

Le phototransistor convertit la lumière en **courant**. Pour le lire, on l'insère en série avec une **résistance de pull-up** vers $V_{CC}$, et on lit la tension au point milieu :

```
   VCC ──[R_pull]──┬── Collecteur photo
                   │
                  [V_capt]  (vers comparateur)
                   │
                Émetteur ── GND
```

**Choix de R_pull — démarche par essais (datasheet + mesure) :**

| $R_{pull}$ | $V$ blanc | $V$ noir | Plage utile | Verdict |
|------------|-----------|----------|-------------|---------|
| 1 kΩ | ~4.5 V | ~4.0 V | 0.5 V | ❌ Indiscriminable |
| **10 kΩ** | **~4.2 V** | **~0.5 V** | **3.7 V** | ✅ Excellent |
| 100 kΩ | ~4.8 V | ~0.01 V | Saturation, lent | ❌ |

→ **R_pull = 10 kΩ**.

### Bilan capteur (à dupliquer ×2 : G et D)

| Composant | Valeur | Rôle |
|-----------|--------|------|
| TCRT5000 | — | Capteur IR intégré |
| Résistance LED | 220 Ω 1/4 W | Limitation courant LED |
| Résistance pull-up | 10 kΩ 1/4 W | Conversion courant → tension |

---

## Étape I.6 — Mesure de référence et choix du seuil de détection

Avant de choisir le comparateur, je **mesure** ce que mes capteurs sortent réellement, multimètre en main, robot posé sur la piste.

| Situation | $V_{capt}$ mesuré |
|-----------|-------------------|
| Capteur sur ligne noire neuve | 0.5 V |
| Capteur sur sol blanc propre | 4.0 V |
| Capteur sur sol blanc avec poussière | 3.5 V |
| Capteur sur ligne noire usée | 1.5 V |

### Choix du seuil

Pour distinguer noir (≤ 1.5 V) de blanc (≥ 3.5 V), le seuil doit être **entre 1.5 V et 3.5 V**.

→ **$V_{seuil} = 2.5\,V$** (centre de la plage, soit $V_{CC}/2$).

### Réalisation : potentiomètre multitour

Je veux pouvoir **ajuster** le seuil sur le terrain (la piste vieillit, l'éclairage change). Diviseur ajustable :

```
   VCC ──[POT 10 kΩ multitour]── GND
                │
                ├──▶ V_seuil
```

**Pourquoi 10 kΩ ?** Même ordre de grandeur que la résistance de pull-up → impédances de source équivalentes, pas de déséquilibre.

**Pourquoi multitour ?** Réglage à ±50 mV près requis. Monotour donne ~15 mV/° ; multitour 10 tours donne ~1.4 mV/° → **10× plus précis**.

**Un potentiomètre par canal (×2)** : les deux capteurs ont des tolérances de fabrication différentes, on doit pouvoir compenser indépendamment.

---

## Étape I.7 — Choix du comparateur

### Besoin

Comparer 2 tensions analogiques et sortir un niveau logique tout-ou-rien (0 ou 5 V).

### Options

| CI | Type | $t_{réponse}$ | Sortie | Nb canaux | Verdict |
|----|------|---------------|--------|-----------|---------|
| **LM393** | **Comparateur dédié** | **1.3 µs** | Collecteur ouvert | 2 | ✅ **Choix #1** |
| LM358 | AOP en mode comparateur | 20 µs | Push-pull | 2 | Acceptable |
| LM324 | AOP en mode comparateur | 20 µs | Push-pull | 4 | Surdimensionné |

### Décision

→ **LM393**, 4 raisons :
1. **15× plus rapide** qu'un AOP — fronts propres, pas d'oscillation au franchissement.
2. **Conçu pour comparer** : pas de risque de saturation lente comme avec un AOP en boucle ouverte.
3. **2 canaux exactement** = 1 par capteur, pas de gaspillage.
4. **Sortie collecteur ouvert** = on choisit librement la tension de pull-up (utile si on veut des niveaux compatibles 5 V ou 12 V).

---

## Étape I.8 — Câblage du comparateur : choix des polarités

**La question :** quelle tension va sur (+), laquelle sur (−) ?

Ce que je veux :

| Situation sous capteur | $V_{capt}$ | Sortie souhaitée | Action sur le moteur opposé |
|------------------------|------------|------------------|------------------------------|
| **Blanc** (hors ligne) | haut (4 V) | **HIGH** | Moteur ON |
| **Noir** (sur ligne) | bas (0.5 V) | **LOW** | Moteur OFF |

→ Sortie HIGH quand $V_{capt}$ HIGH → $V_{capt}$ va sur **(+)**, $V_{seuil}$ va sur **(−)**.

```
   V_capt ──▶(+)
                  ┌─────────┐
                  │ LM393   │── V_out  →  vers entrée du L293D
   V_seuil ─▶(−)  └─────────┘
```

### Logique de commande différentielle

**Convention de lecture :** « Dérive à gauche » signifie que la ligne (qui était centrée sous le robot) se retrouve **sous le capteur gauche** — autrement dit, le robot perçoit la ligne partir vers sa gauche et doit la rattraper en virant **à gauche**.

**Convention de câblage adoptée :** **chaque capteur commande le moteur du MÊME côté** (mapping direct G→G, D→D).

| Cas | Capt. G | Capt. D | Sortie G | Sortie D | Moteur G | Moteur D | Trajectoire |
|-----|---------|---------|----------|----------|----------|----------|-------------|
| Centré | Blanc | Blanc | HIGH | HIGH | ON | ON | **Tout droit** |
| Ligne sous capt. G | Noir | Blanc | LOW | HIGH | OFF | ON | Roue G freine → robot pivote à gauche → recentre |
| Ligne sous capt. D | Blanc | Noir | HIGH | LOW | ON | OFF | Roue D freine → robot pivote à droite → recentre |
| Ligne perdue | Noir | Noir | LOW | LOW | OFF | OFF | Arrêt sécurité |

> **Pourquoi G commande G (et pas D) ?** Quand le capteur G voit du noir, c'est que la ligne s'est décalée sous lui ; pour que le robot suive, il doit pivoter **vers la gauche**. Avec un entraînement différentiel, on pivote à gauche en **ralentissant la roue gauche**. Donc la sortie G doit couper le moteur G. C'est **logique et symétrique** une fois la convention de lecture clarifiée.
>
> *Astuce mémo :* « capteur qui voit noir = roue qui s'arrête, du même côté. »

---

## Étape I.9 — Pull-up de la sortie collecteur ouvert

La sortie du LM393 ne sait que tirer vers GND. Pour avoir un état HIGH, résistance externe vers $V_{CC}$.

### Calcul

Contraintes :
1. Courant absorbé à l'état BAS doit rester < 16 mA (limite LM393) :
$$R > \frac{5}{0.016} = 312\,\Omega$$
2. Pas trop grand pour rester rapide (la capacité parasite ~10 pF se charge à travers R).

→ **R_pull = 4.7 kΩ**.

Vérification : $I_{sink} = 5/4700 = 1.06\,mA$ (×15 sous la limite) ✓
Vitesse : $\tau = 4700 \cdot 10^{-11} = 47\,ns$ (négligeable) ✓

À placer **×2** : un par sortie du LM393.

---

## Étape I.10 — Anti-oscillation : hystérésis sur le comparateur

### Problème

Si le capteur est exactement à la frontière noir/blanc, $V_{capt}$ vibre autour du seuil → la sortie oscille → **le moteur clignote** à haute fréquence → trajectoire en zigzag, voire blocage.

### Solution : rétroaction positive

Renvoyer une fraction de la sortie sur l'entrée (+) à travers une grosse résistance :

```
        R_hyst (1 MΩ)
   ┌─────────────────┐
   │                 │
   V_capt ─▶(+)      │
              ┌──────┴──┐
              │  LM393  │── V_out
   V_seuil ─▶(−)       │
              └─────────┘
```

### Calcul de la largeur d'hystérésis

$$\Delta V = V_{CC} \cdot \frac{R_{capt}}{R_{capt} + R_{hyst}} = 5 \cdot \frac{10\,000}{10\,000 + R_{hyst}}$$

Cible : 50 mV (suffisant pour étouffer le bruit ±10 mV, sans altérer la précision du seuil).

$$R_{hyst} = \frac{5 \cdot 10\,000}{0.05} - 10\,000 \approx 990\,k\Omega \rightarrow \textbf{1 MΩ}$$

À placer **×2**.

---

## Étape I.11 — Choix du type de moteur de propulsion

### Besoin

Faire avancer un châssis de ~300 g à ~0.2 m/s avec une commande simple TOR (tout-ou-rien).

### Options

| Type | Vitesse | Couple | Commande | Coût |
|------|---------|--------|----------|------|
| Servo modèle réduit | Moyenne | Élevé | PWM (signal 50 Hz) | Moyen |
| **Moteur DC + réducteur** | Adaptable | Bon | **Tension simple** | Bas |
| Moteur pas-à-pas | Précis | Faible | Séquence multi-phases | Élevé |
| Brushless | Très rapide | Élevé | ESC complexe | Élevé |

### Décision

→ **Moteur DC à balais avec réducteur**, type **"TT" jaune** (hobby) :
- Tension : 3–6 V
- Vitesse après réduction : ~200 RPM
- Couple : ~0.8 kg·cm
- Courant à vide : ~70 mA, en charge : ~250 mA, démarrage : ~600 mA pic
- Coût : ~2 € l'unité

**Pourquoi ce choix ?**
1. Commande **TOR directe** : on applique 5 V → ça tourne, on coupe → ça s'arrête. Pas besoin de PWM.
2. Réducteur intégré → couple suffisant pour pousser le robot.
3. Largement disponible, kits avec roues compatibles.

→ **2 moteurs** (G et D) en commande différentielle.

---

## Étape I.12 — Pourquoi un pont en H ? Choix du driver

### Pourquoi pas brancher le moteur directement à la sortie du LM393 ?

Sortie LM393 : ~1 mA max. Moteur en charge : 250 mA. **Facteur 250×** → impossible direct.

Il faut un **étage de puissance**. Et pas n'importe lequel : un **pont en H** pour pouvoir éventuellement inverser le sens (utile pour le freinage actif et les manœuvres serrées).

### Options de pont en H

| Solution | $I_{max}$ | Diodes intégrées | Complexité | Coût |
|----------|-----------|------------------|------------|------|
| 4 MOSFET discrets | >10 A | Non (à ajouter) | Élevée (+ driver de grille) | ~5 € |
| **L293D** | 600 mA / canal | **Oui** | DIP-16, simple | ~1.50 € |
| L298N | 2 A / canal | Non | Module, dissipateur requis | ~3 € |
| TB6612FNG | 1.2 A / canal | Oui | CMS | ~2 € |

### Décision

→ **L293D**, 5 raisons :
1. **Diodes de roue libre intégrées** : pas de composants supplémentaires pour gérer la surtension inductive des moteurs.
2. **600 mA continu** : marge ×1.5 sur les 400 mA des moteurs en charge.
3. **2 ponts en H dans un seul boîtier DIP-16** : un par moteur, parfait.
4. **Compatible 5 V logique** directement à partir du LM393.
5. **Protection thermique intégrée** en cas de blocage.

**Compromis accepté :** chute de tension de ~1.4 V dans les transistors Darlington internes → tension moteur réduite, mais reste suffisante (5 V → 3.6 V aux bornes du moteur, OK pour les TT).

---

## Étape I.13 — Câblage L293D : du comparateur au moteur

### Schéma de connexion

```
                       ┌───────────── L293D ─────────────┐
                       │                                 │
   LM393 OUT_G ────▶ │ IN1 (broche 2)         OUT1 (3) │────┐
                       │                                 │    │
              GND ──▶ │ IN2 (broche 7)         OUT2 (6) │────┤── MOTEUR GAUCHE
                       │                                 │
   LM393 OUT_D ────▶ │ IN3 (broche 10)        OUT3 (11)│────┐
                       │                                 │    │
              GND ──▶ │ IN4 (broche 15)        OUT4 (14)│────┤── MOTEUR DROIT
                       │                                 │
              VCC ──▶ │ EN1,2 (1) ; EN3,4 (9)           │
                       │                                 │
              VCC ──▶ │ Vss (16)  Vs (8)        ◀── V_bat (rail moteurs)
                       │                                 │
              GND ──▶ │ GND (4, 5, 12, 13)              │
                       └─────────────────────────────────┘
```

### Pourquoi seulement IN1, IN3 commandés ? (pas IN2, IN4)

On veut **un seul sens de rotation** (avant uniquement). En mettant IN2 et IN4 à GND en permanence :
- IN1 = HIGH → le moteur tourne en avant.
- IN1 = LOW → le moteur s'arrête (roue libre).

Pour un suiveur de ligne basique, le freinage actif n'est pas requis. Si on en voulait un jour, il suffirait de piloter IN2 et IN4 par les compléments des sorties LM393.

### Découplage

- **100 nF** sur chaque broche d'alimentation (Vss et Vs) au plus près du CI.
- **100 µF électrolytique** sur Vs pour absorber les à-coups de courant moteur.

---

## Étape I.14 — Architecture d'alimentation : 2 rails séparés

### Le piège à éviter

Si je branche tout sur la même tension régulée 5 V, à chaque démarrage moteur le rail s'effondre brièvement → **les capteurs et comparateurs voient un parasite** → faux déclenchements → le robot zigzague de manière incompréhensible.

### Solution : **séparer la logique et la puissance**

```
   ┌──────────────────────┐
   │  Batterie 7.4 V       │
   └─────────┬────────────┘
             │
     ┌───────┴───────┐
     │               │
  (rail puissance)  (rail logique via LM7805)
     │               │
   V_bat 7.4V       VCC 5V
   → L293D Vs       → LM393, capteurs, diviseurs de seuil
```

**Les deux rails partagent la masse commune** (sinon les comparaisons n'ont pas de référence), mais les **courants moteur** ne traversent PAS le LM7805.

### Choix du régulateur logique : LM7805

| Critère | LM7805 | AMS1117 (LDO) | Buck converter |
|---------|--------|----------------|----------------|
| $V_{dropout}$ | 2 V | 1.1 V | 0.3 V |
| Bruit de sortie | **Très faible** | Très faible | Moyen (switching) |
| Complexité | 3 composants | 3 composants | >6 composants |
| Coût | 0.30 € | 0.30 € | 2 € |

→ **LM7805**, pour son **bruit minimal** (capital pour les capteurs analogiques) et sa simplicité. La perte (~2.4 V dissipée en chaleur sur 50 mA = 120 mW) est négligeable.

### Vérification thermique du LM7805

$$P = (V_{in} - V_{out}) \cdot I = (7.4 - 5) \cdot 0.050 = 0.12\,W$$

TO-220 sans dissipateur supporte 1 W → **élévation 8 °C** → reste froid. ✓

> ⚠️ **Règle d'or :** ne JAMAIS faire passer le courant moteur par le LM7805. C'est la première erreur qui grille un régulateur sur un robot.

---

## Étape I.15 — Choix de la batterie

### Besoin

Alimenter ~850 mA crête pendant ≥ 1 heure → ≥ 850 mAh.

### Options

| Type | Tension | Capacité typique | Poids | Coût | Verdict |
|------|---------|------------------|-------|------|---------|
| 4×AA alcalines | 6 V | 2500 mAh | 95 g | 3 € | Simple mais lourd |
| 6×AA alcalines | 9 V | 2500 mAh | 140 g | 4 € | Trop lourd |
| 9V "PP3" | 9 V | 500 mAh | 45 g | 4 € | Capacité trop faible |
| **LiPo 2S 1500 mAh** | 7.4 V | 1500 mAh | 80 g | 12 € | ✅ **Choix optimal** |
| Li-ion 18650 ×2 | 7.4 V | 2500 mAh | 90 g | 8 € | Excellent si support dispo |

### Décision

→ **LiPo 2S 7.4 V 1500 mAh**, pour son **rapport poids/énergie** imbattable et sa tension parfaitement adaptée (LM7805 a besoin de 7 V min, 7.4 V est juste au-dessus).

**Vérification autonomie :**
$$t = \frac{1500\,mAh}{850\,mA} \approx 1\,h\,46 \quad \text{✓ (> 1 h CDC)}$$

⚠️ **Précautions LiPo :** chargeur balance obligatoire, ne jamais décharger sous 6.4 V (1S < 3.2 V), boîtier souple → risque de gonflement si maltraité. Acceptable au regard des bénéfices.

---

## Étape I.16 — Châssis, roues et roue folle

### Châssis

| Matériau | Avantages | Inconvénients | Verdict |
|----------|-----------|---------------|---------|
| Plexiglas 3 mm | Propre, transparent, découpe laser | Cassant | ✅ Si accès laser |
| MDF 3 mm | Bon marché, facile à percer | Sensible humidité | ✅ Bon prototype |
| PLA 3D | Forme libre | Imprimante requise | Bonus |

→ **MDF 3 mm** ou plexiglas, dimensions **150 × 120 mm**.

### Roues motrices

Diamètre : 65 mm (compromis vitesse/couple). À 200 RPM × π × 0.065 m = **0.68 m/s en pointe**, plafonné par la friction au sol et la chute L293D à ~**0.2–0.3 m/s** réel — conforme au CDC.

### Roue folle (pivot avant ou arrière)

Trois roues = configuration tricycle. La 3ᵉ "roue" doit être **omnidirectionnelle** :

| Type | Avantage |
|------|----------|
| **Bille castor** (Pololu) | Omnidirectionnel parfait, glisse |
| Roulette pivotante | Bon marché |

→ **Bille castor** à l'arrière (les capteurs sont à l'avant, donc la bille va à l'opposé pour équilibrer).

### Disposition

```
        VUE DE DESSUS
   ┌─────────────────────────┐
   │ [Capt. G]    [Capt. D]  │  ← Avant : capteurs à 5 mm du sol
   │                         │
   │  ●●● Roue G   Roue D ●●●│  ← Roues motrices
   │     ↑              ↑    │
   │   Moteur         Moteur │
   │                         │
   │   [Breadboard]          │
   │   [LM7805+L293D+LM393]  │
   │                         │
   │       [Batterie]        │
   │                         │
   │           ◯             │  ← Bille castor
   └─────────────────────────┘
        ARRIÈRE
```

---

## Étape I.17 — Bilan énergétique du robot

| Bloc | Courant moyen |
|------|---------------|
| 2 × LED IR (220 Ω) | 2 × 17.3 mA = 34.6 mA |
| 2 × phototransistor | ~2 mA |
| LM393 | 0.8 mA |
| 2 × pull-up sortie 4.7 kΩ (à l'état BAS, 50 % du temps) | 2 × 0.5 mA = 1 mA |
| 2 × diviseur seuil 10 kΩ | 2 × 0.5 mA = 1 mA |
| 2 × moteurs (charge moyenne) | 2 × 400 mA = 800 mA |
| **TOTAL** | **~840 mA** |

Autonomie LiPo 1500 mAh : $1500/840 \approx \mathbf{1\,h\,46}$ ✓ Conforme CDC.

---

## Étape I.18 — Validation finale : table de vérité et tests

### Table de vérité globale

| Capt. G voit | Capt. D voit | $V_{out\_G}$ | $V_{out\_D}$ | Mot. G | Mot. D | Action |
|:---:|:---:|:---:|:---:|:---:|:---:|---|
| Blanc | Blanc | HIGH | HIGH | ON | ON | **Tout droit** (centré) |
| Noir | Blanc | LOW | HIGH | OFF | ON | Tourne **gauche** (recentre) |
| Blanc | Noir | HIGH | LOW | ON | OFF | Tourne **droite** (recentre) |
| Noir | Noir | LOW | LOW | OFF | OFF | Arrêt (intersection / ligne perdue) |

### Procédure de mise en service

1. **Test à vide** : robot soulevé en l'air, vérifier que les deux moteurs tournent librement quand on présente une feuille blanche sous les capteurs.
2. **Calibration des seuils** : poser sur ligne, mesurer $V_{capt}$, ajuster chaque potentiomètre à $V_{seuil} = (V_{noir} + V_{blanc})/2 \approx 2.5\,V$.
3. **Test sur ligne droite** : doit avancer droit sans zigzaguer (si zigzags → réduire l'hystérésis ? non, le contraire : *augmenter* R_hyst — wait : pour augmenter l'hystérésis et stabiliser, *diminuer* R_hyst. Vérifier au cas par cas).
4. **Test virage 90°** : doit suivre. Si décroche → réduire la vitesse ou rapprocher les capteurs.
5. **Test ligne perdue** : doit s'arrêter (les deux moteurs OFF).

### Critères de réussite (PV de recette)

- [ ] Suit une ligne droite de 1 m sans dévier de plus de 5 mm.
- [ ] Suit un virage à rayon 200 mm sans perdre la ligne.
- [ ] S'arrête en moins de 50 ms si la ligne est masquée.
- [ ] Tient ≥ 1 h sur une charge complète.

---

## Synthèse Partie I

| # | Étape | Décision | Composant |
|---|-------|----------|-----------|
| I.0 | CDC | 6 contraintes | — |
| I.1 | Philosophie | 100 % analogique | — |
| I.2 | Décomposition | 4 blocs | — |
| I.3 | Capteur | TCRT5000 (IR robuste) | **TCRT5000 ×2** |
| I.4 | Géométrie | 2 capteurs, E = 20 mm, h = 5 mm | — |
| I.5a | LED IR | $R = 190 \Omega \rightarrow 220$ | **220 Ω ×2** |
| I.5b | Phototransistor | Pull-up donne 0.5–4 V | **10 kΩ ×2** |
| I.6 | Seuil | 2.5 V ajustable | **POT 10 kΩ multitour ×2** |
| I.7 | Comparateur | LM393 (rapide, 2 canaux) | **LM393 ×1** |
| I.8 | Polarité | V_capt sur (+), commande directe (G→G, D→D) | — |
| I.9 | Pull-up sortie | 4.7 kΩ → 1 mA | **4.7 kΩ ×2** |
| I.10 | Hystérésis | 50 mV → 1 MΩ | **1 MΩ ×2** |
| I.11 | Moteur | DC + réducteur (TT) | **Moteur TT ×2** |
| I.12 | Driver | L293D (intégré, sûr) | **L293D ×1** |
| I.13 | Câblage L293D | IN2/IN4 à GND | — |
| I.14 | Alimentation | 2 rails séparés via LM7805 | **LM7805 ×1** |
| I.15 | Batterie | LiPo 2S 1500 mAh | **LiPo 2S 1500 mAh ×1** |
| I.16 | Mécanique | MDF + bille castor + roues 65 mm | **Châssis + roues + bille** |
| I.17 | Énergie | 840 mA → 1 h 46 | OK |
| I.18 | Validation | Table de vérité + 5 tests | — |

### Ce que cette Partie I m'a appris

1. **Tout commence par le capteur.** Sans signal exploitable, aucune électronique ne sauve le robot.
2. **L'analogique pur est étonnamment puissant** : 1 CI logique (LM393) + 1 CI puissance (L293D) suffisent pour faire un robot autonome.
3. **La séparation des rails d'alimentation** est non-négociable dès qu'il y a des moteurs.
4. **L'hystérésis est obligatoire** dès qu'on compare deux signaux qui peuvent être proches.
5. **La géométrie mécanique influence l'électronique** : l'espacement des capteurs détermine la largeur de ligne acceptable, qui détermine la finesse du seuil requis.

> **Le robot suit maintenant la ligne.** Mais que se passe-t-il quand la ligne est *effacée* ? C'est l'objet de la Partie II.

---
---

# 🎨 PARTIE II — Intégration du 3ᵉ actionneur (Pompe à Peinture)

> **Point de départ :** le robot de la Partie I est terminé, monté, calibré, et il fonctionne. On veut maintenant lui donner une nouvelle capacité : **redéposer de la peinture** quand il rencontre une zone effacée.

> **Contrainte forte :** **NE RIEN MODIFIER** de la Partie I. Le module pompe est strictement additif. Si on le débranche, le robot doit continuer à fonctionner exactement comme avant.

---

## Étape II.0 — Cahier des charges du module pompe

**Question de départ :** « Mon robot suit une ligne noire. Que dois-je ajouter pour qu'il *répare* la ligne s'il la trouve effacée ? »

**Réflexion :**
- Il faut **détecter** que la ligne manque.
- Il faut **agir** : déposer de la peinture.
- Il faut **doser** : ni trop (flaque), ni trop peu (illisible).
- Il faut **ne rien casser** dans le robot existant.

J'écris noir sur blanc mes contraintes :

| # | Contrainte | Origine |
|---|------------|---------|
| C1 | Détection optique du sol | Cohérence avec le suivi |
| C2 | Décision automatique sans intervention | Robot autonome |
| C3 | Dose calibrée et reproductible | Qualité du marquage |
| C4 | Indépendance vis-à-vis du suivi de ligne | Sécurité fonctionnelle |
| C5 | Alimentation depuis la batterie existante | Pas de 2ᵉ batterie |
| C6 | Aucune ligne de code, aucun MCU | Philosophie du projet |

À ce stade, **aucun composant n'est encore choisi**. C'est volontaire : on ne dimensionne pas avant d'avoir compris.

---

## Étape II.1 — Choix de la philosophie : analogique ou logique programmée ?

Avant de choisir des composants, je tranche **la** question structurante.

### Options envisagées

| Option | Avantages | Inconvénients | Verdict |
|--------|-----------|---------------|---------|
| **Microcontrôleur** (Arduino Nano, ATtiny85…) | Logique flexible, seuils ajustables par soft, ajout facile de fonctions | Nécessite firmware, viole C6 | ❌ Rejeté |
| **Logique câblée (74HC + 555)** | Pas de code, modulaire, prédictible | Nombreux CI, plus de câblage | ⚠️ Possible mais lourd |
| **100 % analogique** (AOP/comparateur + 555) | Cohérent avec l'existant, simple, robuste | Moins flexible | ✅ **Retenu** |

→ **Décision :** chaîne analogique + un unique CI temporisateur (NE555 — qui reste un circuit analogique au sens où il n'exécute pas d'instructions).

---

## Étape II.2 — Décomposition fonctionnelle en blocs

Avant de tirer un seul fil, je dessine sur papier la chaîne de traitement la plus simple possible :

```
   [Voir le sol] → [Décider] → [Attendre/Doser] → [Faire couler la peinture]
```

Je traduis chaque verbe en bloc électronique :

| Verbe | Bloc électronique | Type de signal |
|-------|-------------------|----------------|
| Voir | Capteur opto | Tension analogique |
| Décider | Comparateur | Logique 0/5 V |
| Attendre/Doser | Temporisateur | Impulsion calibrée |
| Faire couler | Étage de puissance + actionneur | Courant fort |

**Pourquoi cet ordre ?** Parce que chaque bloc *consomme* la sortie du précédent. Concevoir dans cet ordre garantit que je connais la nature du signal d'entrée de chaque bloc avant de le dimensionner.

**Règle que je me fixe :** je conçois **bloc par bloc, dans l'ordre du flux**, et je ne passe au bloc suivant qu'une fois le précédent figé.

---

## Étape II.3 — Choix du capteur de détection

### Besoin

Mesurer la réflectivité d'un point précis du sol, distinguer **noir** (ligne) de **blanc** (sol nu).

### Options sur la table

| Capteur | Principe | Coût | Disponibilité | Compatibilité projet |
|---------|----------|------|---------------|----------------------|
| Photorésistance LDR | Lumière visible | Très bas | Forte | Sensible à l'ambiant |
| Capteur couleur TCS3200 | RGB par filtres | Moyen | Bonne | Sortie en fréquence (complexe) |
| **Paire IR TCRT5000** | Réflexion IR 950 nm | Bas (~0.50 €) | Excellente | **Identique à l'existant** |
| Lidar / ToF | Distance | Élevé | Bonne | Surdimensionné |

### Décision

→ **TCRT5000**, pour trois raisons cumulatives :
1. **Cohérence** : c'est exactement le même capteur que les deux capteurs de suivi. Une seule famille de composants à connaître, à stocker, à dépanner.
2. **Insensibilité à la lumière ambiante** (IR 950 nm vs visible) — déjà éprouvé sur le robot.
3. **Réponse rapide** (~15 µs) — le robot avance, on ne peut pas se permettre un capteur lent.

> **Leçon retenue :** quand un composant fonctionne déjà bien sur le projet, il faut une **excellente raison** d'en introduire un nouveau. La standardisation prime.

---

## Étape II.4 — Position mécanique du capteur

Avant le moindre câblage, je décide **où** physiquement le poser. Cette décision conditionne tout le reste.

### Hypothèse A — Capteur à l'arrière du robot

```
   [Capt. G] [Capt. D]
       suivi
                    [Capt. central]  ← derrière
                          ↓
                    [Buse pompe]
```

❌ Inconvénient : le robot doit *passer sur* la zone effacée avant de s'en rendre compte. Le dépôt est en retard.

### Hypothèse B — Capteur à l'avant, sur l'axe

```
                [Capt. central]  ← devant, sur l'axe
   [Capt. G]         ↓         [Capt. D]
       suivi    [Buse pompe]
```

✅ Avantage : la pompe dépose **au moment** où le capteur central voit du blanc → dépôt aligné avec la zone effacée.

### Décision

→ **Hypothèse B**, capteur central aligné sur l'axe longitudinal, **entre** les capteurs G/D existants, à 5 mm du sol (même hauteur que les autres). Buse de la pompe placée juste derrière, sur le même axe.

**Calcul de l'espacement :**
- Capteurs G/D existants : espacés de 20 mm.
- Capteur central : à équidistance, soit **10 mm de chaque côté**.
- Vérification : 10 mm > diamètre du boîtier TCRT5000 (5×4 mm) → pas de chevauchement physique. ✓
- Vérification optique : 10 mm > distance d'émission utile du TCRT5000 (≤ 5 mm en réflexion diffuse) → **pas de diaphonie** entre capteurs. ✓

---

## Étape II.5 — Câblage du capteur : LED IR + phototransistor

Le capteur a deux moitiés à câbler indépendamment.

### Moitié émettrice : la LED IR

**Loi d'Ohm appliquée à la LED :**

$$R_{LED} = \frac{V_{CC} - V_F}{I_F}$$

Données du datasheet TCRT5000 :
- $V_F = 1.2\,V$ (chute aux bornes de la LED IR)
- $I_F$ nominal = 20 mA, max 60 mA

$$R_{LED} = \frac{5 - 1.2}{0.020} = 190\,\Omega$$

**Choix de la valeur normalisée :** la valeur immédiatement supérieure dans la série E24 est **220 Ω**. On choisit *toujours* la valeur supérieure → courant réel un peu *inférieur* au nominal → marge de sécurité.

**Vérification du courant réel :**

$$I_F = \frac{5 - 1.2}{220} = 17.3\,mA$$

→ 17.3 mA, dans la zone d'efficacité maximale, et à 28 % du courant max. ✓

**Vérification de la puissance dissipée :**

$$P = R \cdot I^2 = 220 \cdot (0.0173)^2 = 66\,mW$$

Une résistance 1/4 W (250 mW) est largement suffisante (ratio ×3.8). ✓

### Moitié réceptrice : le phototransistor

Le phototransistor est un transistor NPN dont le courant de collecteur dépend de la lumière reçue. Pour transformer ce **courant** en **tension** mesurable, on insère une **résistance de pull-up** entre le collecteur et $V_{CC}$.

```
   VCC
    │
   [R_pull]
    │
    ├──────▶ V_capt  (sortie tension)
    │
   Collecteur (phototransistor)
   Émetteur ── GND
```

**Choix de la valeur de pull-up — démarche par essais :**

| $R_{pull}$ | $V$ sur blanc | $V$ sur noir | Plage utile |
|------------|---------------|--------------|-------------|
| 1 kΩ | ~4.5 V | ~4.0 V | 0.5 V (faible) |
| **10 kΩ** | **~4.2 V** | **~0.5 V** | **3.7 V** ✓ |
| 100 kΩ | ~4.8 V | ~0.01 V | saturation, lent |

→ **10 kΩ** est le compromis. C'est aussi la valeur déjà utilisée sur les autres capteurs du robot — cohérence préservée.

---

## Étape II.6 — Mesure de référence et choix du seuil

Maintenant que le capteur est câblé, **avant** de choisir le comparateur, je vais **réellement mesurer** ce qui sort.

### Démarche

Je mesure $V_{capt\_C}$ au multimètre dans 4 situations :

| Situation | $V_{capt\_C}$ mesuré (typique) |
|-----------|--------------------------------|
| Capteur sur ligne noire neuve | 0.5 V |
| Capteur sur sol blanc propre | 4.0 V |
| Capteur sur ligne effacée (gris) | 2.8 V |
| Capteur sur sol blanc avec poussière | 3.5 V |

### Choix du seuil

Pour distinguer "ligne présente" de "ligne effacée", le seuil doit être **entre 0.5 V et 2.8 V**.

Je choisis **2.5 V** comme valeur initiale, pour deux raisons :
1. Centrée entre noir (0.5 V) et le pire cas "ligne effacée" (2.8 V).
2. Égale à $V_{CC}/2$ → réalisable avec un simple diviseur 50/50.

### Réalisation du seuil

Pour pouvoir **ajuster** ce seuil sur le terrain (la peinture vieillit, les sols varient), je préfère un **potentiomètre** plutôt qu'un diviseur fixe.

```
   VCC
    │
  [POT 10 kΩ]
    │
    ├──▶ V_seuil_pompe
    │
   GND
```

**Pourquoi 10 kΩ ?**
- Même ordre de grandeur que la résistance de pull-up (10 kΩ) → impédance de source équivalente, pas de déséquilibre.
- Courant consommé : $I = 5/10\,000 = 0.5\,mA$ → négligeable.

**Pourquoi multitour ?** Parce que le seuil doit être réglé à ±50 mV près. Un potentiomètre monotour donne ~15 mV/degré ; un multitour 10 tours donne ~1.4 mV/degré → **réglage 10× plus fin**. Choix : **multitour 10 tours**.

---

## Étape II.7 — Choix du comparateur

À ce stade, j'ai deux tensions analogiques (V_capt_C et V_seuil_pompe) et je veux une **décision binaire**. C'est exactement le rôle d'un comparateur.

### Options

| Composant | Type | Temps de réponse | Sortie | Nb canaux | Verdict |
|-----------|------|------------------|--------|-----------|---------|
| **LM393** | Comparateur dédié | 1.3 µs | Collecteur ouvert | 2 | ✅ Choix #1 |
| LM358 | AOP en mode comparateur | ~20 µs | Push-pull | 2 | Acceptable |
| LM324 | AOP en mode comparateur | ~20 µs | Push-pull | 4 | Surdimensionné |
| Schmitt-trigger 74HC14 | Inverseur logique | Très rapide | Push-pull | 6 | Pas de réglage de seuil |

### Décision

→ **LM393**, pour 4 raisons :
1. **C'est le composant déjà utilisé** sur le robot pour la même fonction (cohérence).
2. **15× plus rapide** qu'un AOP en boucle ouverte → fronts propres, pas de zone grise.
3. **Sortie collecteur ouvert** → je peux choisir librement la tension à laquelle la sortie est tirée (utile si je veux 5 V ici).
4. **Pas d'oscillation** : un AOP sans rétroaction peut osciller au passage du seuil, un comparateur dédié non.

### Question importante : un nouveau LM393 ou réutiliser un canal libre ?

Le LM393 contient 2 comparateurs. Sur le robot existant, les deux canaux sont déjà utilisés (un par capteur de suivi). 

**Options :**
- (a) Remplacer le LM393 existant par un LM324 (4 canaux) → modifie un document existant. ❌
- (b) Ajouter un **second** boîtier LM393 dédié au module pompe → strictement additif. ✅

→ **Option (b)** retenue. Coût marginal (~0.30 €) pour préserver l'indépendance fonctionnelle (contrainte C4).

---

## Étape II.8 — Câblage du comparateur : sens et inversion logique

C'est le moment le plus subtil. Il faut décider quelle tension va sur l'entrée (+) et laquelle sur l'entrée (−).

### Rappel du fonctionnement

$$V_{out} = \begin{cases} V_{CC} & \text{si } V_{(+)} > V_{(-)} \\ 0 & \text{si } V_{(+)} < V_{(-)} \end{cases}$$

### Ce que je veux

| Situation | $V_{capt\_C}$ | Sortie souhaitée pour piloter la pompe |
|-----------|----------------|----------------------------------------|
| Ligne **présente** (noir) | bas (~0.5 V) | **0 V** (pompe OFF) |
| Ligne **effacée** (blanc) | haut (~4 V) | **5 V** (pompe ON) |

→ La sortie doit être **HAUTE** quand $V_{capt\_C}$ est **HAUT**. Donc $V_{capt\_C}$ doit aller sur l'entrée (+).

### Mais attendez — qu'est-ce qui suit le comparateur ?

Le bloc suivant sera un **NE555 monostable** (cf. étape II.12). Or l'entrée TRIG du 555 est **active basse** : le 555 se déclenche sur un **front descendant**. 

**Donc en réalité, je veux que la sortie du comparateur passe de HAUT à BAS quand la ligne est effacée**, pour générer un front descendant.

### Conclusion : on inverse

→ **Câblage final :**
- $V_{capt\_C}$ sur l'entrée **(−)**
- $V_{seuil\_pompe}$ sur l'entrée **(+)**

```
   V_seuil_pompe ──▶(+)
                       ┌─────────┐
                       │ LM393   │── V_inv
   V_capt_C    ──▶(−)  └─────────┘
```

Vérification :

| Situation | $V_{(+)}$ | $V_{(-)}$ | $V_{(+)} > V_{(-)}$ ? | $V_{inv}$ |
|-----------|-----------|-----------|------------------------|-----------|
| Ligne présente | 2.5 V | 0.5 V | OUI | **HIGH** ✓ |
| Ligne effacée | 2.5 V | 4.0 V | NON | **LOW** ✓ |

Front descendant produit lors du passage à "ligne effacée". 

> **Leçon retenue :** la polarité du bloc **suivant** détermine la polarité de sortie du bloc **courant**. Toujours concevoir en pensant à l'interface.

---

## Étape II.9 — Pull-up de la sortie collecteur ouvert

La sortie du LM393 ne peut **que tirer vers GND**. Pour avoir un état HAUT, il faut une résistance externe vers $V_{CC}$.

### Calcul

La sortie doit attaquer l'entrée TRIG du 555, qui consomme < 1 µA. Donc la résistance peut être quasiment quelconque. Les contraintes sont :

1. **Courant absorbé à l'état BAS** (par le LM393) :
$$I_{sink} = \frac{V_{CC}}{R_{pull}} \quad \text{doit rester} < 16\,mA \text{ (limite LM393)}$$
$$\Rightarrow R_{pull} > \frac{5}{0.016} = 312\,\Omega$$

2. **Vitesse de commutation** : la capacité parasite (~10 pF) charge à travers $R_{pull}$, constante de temps :
$$\tau = R \cdot C \quad \text{doit rester} \ll 1\,ms$$

### Choix : 4.7 kΩ

- $I_{sink} = 5/4700 = 1.06\,mA$ (×15 sous la limite) ✓
- $\tau = 4700 \cdot 10^{-11} = 47\,ns$ (négligeable) ✓
- Valeur standard, recommandée par le datasheet du LM393.

→ **Résistance 4.7 kΩ entre la sortie du LM393 et $V_{CC}$.**

---

## Étape II.10 — Anti-oscillation : ajout de l'hystérésis

### Le problème

Si le capteur central est exactement à la frontière noir/blanc, $V_{capt\_C}$ vibre autour du seuil → la sortie du comparateur **oscille** à la fréquence du bruit. Conséquence : la pompe se déclenche n'importe comment.

### La solution

Une **rétroaction positive** : on renvoie une petite fraction de la sortie sur l'entrée (+). Quand la sortie passe HIGH, le seuil monte légèrement → il faut "vraiment" descendre pour repasser LOW. Et inversement.

```
                  R_hyst
        ┌─────────────────────────────┐
        │                             │
   V_seuil ──[10 kΩ]──▶(+)            │
                                  ┌───┴─────┐
                                  │ LM393   │── V_inv
   V_capt_C       ────────▶(−)    └─────────┘
```

### Calcul de la largeur d'hystérésis

$$\Delta V = V_{CC} \cdot \frac{R_1}{R_1 + R_{hyst}}$$

Je veux $\Delta V \approx 50\,mV$ (suffisant pour étouffer le bruit ±10 mV du phototransistor, mais assez petit pour ne pas altérer le réglage du seuil).

$$50\,mV = 5\,V \cdot \frac{10\,000}{10\,000 + R_{hyst}} \Rightarrow R_{hyst} = \frac{5 \cdot 10\,000}{0.05} - 10\,000 = 990\,k\Omega$$

→ Valeur normalisée : **1 MΩ**, donnant $\Delta V = 49.5\,mV$. ✓

> **Leçon retenue :** l'hystérésis n'est pas optionnelle dès qu'on compare deux signaux qui peuvent être proches. C'est une règle, pas une option.

---

## Étape II.11 — Anti-faux-déclenchement : filtre RC passe-bas

L'hystérésis règle le bruit *électrique*. Mais il reste un autre problème : le robot peut traverser une **micro-rayure** blanche en quelques millisecondes, et déclencher la pompe pour rien.

### Cahier des charges du filtre

- À 0.2 m/s, je veux ignorer les zones effacées **< 20 mm**.
- 20 mm à 0.2 m/s = **100 ms**.
- Donc le filtre doit imposer une **durée minimale de "ligne effacée"** d'environ 100 ms avant déclenchement.

### Topologie : RC passe-bas en série

```
   V_inv ──[R_f]──┬──▶ V_filt
                  │
                [C_f]
                  │
                 GND
```

Constante de temps :
$$\tau = R_f \cdot C_f$$

Je veux $\tau \approx 100\,ms$.

### Choix des valeurs

Plusieurs combinaisons donnent $RC = 100\,ms$ :

| $R_f$ | $C_f$ | Remarque |
|-------|-------|----------|
| 10 kΩ | 10 µF | Cap. élevée → électrolytique → polarité à gérer |
| **100 kΩ** | **1 µF** | Cap. céramique non polarisée ✓ |
| 1 MΩ | 100 nF | Impédance trop élevée, sensible aux fuites |

→ **R_f = 100 kΩ, C_f = 1 µF céramique.**

### Vérification : seuil de déclenchement effectif

Le 555 déclenche quand TRIG passe sous $V_{CC}/3 \approx 1.67\,V$.

Décharge de C_f depuis 5 V vers 0 V à travers R_f :
$$V_{filt}(t) = 5 \cdot e^{-t/\tau}$$

Temps pour atteindre 1.67 V :
$$t = \tau \cdot \ln\left(\frac{5}{1.67}\right) = 100\,ms \cdot \ln(3) \approx 110\,ms$$

→ Une zone effacée de moins de **22 mm à 0.2 m/s** ne déclenche **pas** la pompe. ✓ Cahier des charges respecté.

---

## Étape II.12 — Choix du temporisateur : pourquoi un monostable ?

À cette étape, j'ai un signal "il y a une zone effacée durable". Que faire ?

### Option naïve : commander la pompe directement

Si je relie le filtre à la grille du MOSFET, **la pompe tourne tant que le capteur voit blanc**. 

❌ Problème : sur une longue zone effacée (50 cm), la pompe tourne 2.5 secondes → flaque. Sur une zone courte (5 cm), la pompe tourne 0.25 s → trop bref pour amorcer.

### Solution : impulsion calibrée — c'est exactement ce qu'est un monostable

Un **monostable** délivre une impulsion de **durée fixe** quel que soit la durée du déclencheur. Parfait.

### Quel monostable ?

| Solution | Avantages | Inconvénients |
|----------|-----------|---------------|
| **NE555 en monostable** | Composant universel, bon marché (0.40 €), facile, robuste | DIP-8 |
| Bascule monostable 74HC123 | Plus précis | CMS, plus rare |
| Réseau RC + transistor | Le plus simple | Forme d'onde imprécise |
| Microcontrôleur | Le plus précis | Viole C6 |

→ **NE555**, sans hésitation : c'est le composant analogique de temporisation par excellence.

---

## Étape II.13 — Calcul de la durée d'impulsion

### Formule du 555 monostable

$$T_{on} = 1.1 \cdot R_t \cdot C_t$$

### Cahier des charges sur la durée

Je veux que la pompe :
- Tourne assez longtemps pour amorcer le tube (>200 ms).
- Ne tourne pas trop longtemps pour ne pas faire de flaque (<1 s).
- À 0.2 m/s, dépose sur une longueur de 5 à 20 cm → soit 250 ms à 1 s.

→ Cible : **~500 ms**.

### Choix de R_t et C_t

$$T_{on} = 0.5\,s \Rightarrow R_t \cdot C_t = \frac{0.5}{1.1} = 0.455$$

Je préfère un C_t **petit et céramique** (stable, peu cher, pas de polarité) → $C_t = 1\,\mu F$.

$$R_t = \frac{0.455}{10^{-6}} = 455\,k\Omega$$

→ Valeur normalisée la plus proche : **470 kΩ**.

Vérification :
$$T_{on} = 1.1 \cdot 470\,000 \cdot 10^{-6} = 0.517\,s \approx 517\,ms \quad ✓$$

### Bonus : ajustabilité

Si je veux pouvoir tester d'autres durées sans dessouder, je peux remplacer R_t par un potentiomètre 1 MΩ + résistance fixe 100 kΩ en série, donnant une plage de 110 ms à 1.2 s. Pour un premier proto, je reste sur 470 kΩ fixe.

### Petits détails du 555 à ne pas oublier

| Broche | Composant | Rôle |
|--------|-----------|------|
| 5 (CTRL) | 10 nF vers GND | Découplage de la référence interne |
| 8 (VCC) | 100 nF vers GND | Découplage alimentation |
| 4 (RESET) | Relié à VCC | Empêche un reset inopiné |
| 6 et 7 | Reliés ensemble + à C_t et R_t | Configuration monostable standard |

---

## Étape II.14 — Choix de la pompe

J'ai retardé volontairement ce choix : je voulais d'abord savoir **comment** je commanderais la pompe avant de choisir laquelle.

### Critères

- Tension : compatible avec la batterie (7.4 V LiPo) → 6 V nominal acceptable.
- Courant : le moins possible pour ne pas surcharger la batterie partagée.
- Dose : reproductible.
- Compatibilité chimique : peinture acrylique (à base d'eau, légèrement abrasive).

### Comparatif

| Type | Référence | Tension | Débit | Reproductibilité | Prix |
|------|-----------|---------|-------|-------------------|------|
| **Péristaltique** | Kamoer KPP-B04 | 6 V | 100 ml/min | **Excellente** (volume = nb de tours) | 18 € |
| Diaphragme | R385 | 6–12 V | 1500 ml/min | Moyenne | 5 € |
| Centrifuge | Aquarium générique | 3–6 V | Variable | Faible (amorçage requis) | 3 € |
| Électrovanne + gravité | — | 6 V | Variable | Faible | 4 € |

### Décision

→ **Péristaltique Kamoer KPP-B04** (ou équivalent Adafruit #1150).

**Pourquoi accepter un coût ×4 vs la R385 ?**
1. La R385 a un débit **15× supérieur** à la péristaltique → 0.5 s de pompage = ~12 ml de peinture = grosse flaque.
2. La R385 nécessite un amorçage manuel.
3. La péristaltique ne touche pas le fluide (le tube est pincé) → durabilité supérieure, pas de coagulation dans le moteur.
4. Dose précise = un atout pour la qualité du marquage et pour économiser la peinture.

**Si budget critique** : R385 acceptable, mais alors raccourcir T_on à ~100 ms (R_t = 91 kΩ) pour limiter le débit.

### Données utiles pour la suite

- Tension nominale : 6 V
- Courant nominal : ~300 mA
- Courant pic au démarrage : ~500 mA
- Type : moteur DC à balais → **charge inductive** ⚠️

---

## Étape II.15 — Choix du transistor de puissance

### Pourquoi pas le L293D existant ?

Le L293D pilote déjà les deux moteurs de propulsion. Je pourrais utiliser un canal libre, mais :
- Le L293D n'a que 4 sorties, toutes utilisées (2 par moteur).
- Mutualiser ferait passer le courant pompe par le rail moteur → couplage parasite avec les moteurs (bruit, à-coups).
- Modifier le L293D = modifier un document existant → viole la consigne.

→ **Étage de puissance dédié.**

### MOSFET ou BJT ?

| Critère | MOSFET N | BJT NPN (TIP120) |
|---------|----------|------------------|
| Pertes en conduction | $R_{DS(on)} \cdot I^2$ | $V_{CE(sat)} \cdot I$ |
| Pour 0.5 A | 0.022 × 0.25 = **5.5 mW** | 1 V × 0.5 = **500 mW** |
| Commande | Tension (capacité de grille) | Courant |
| Logic-level dispo ? | Oui | N/A |

→ **MOSFET N**, écrasant pour la dissipation.

### Choix du modèle

Critères :
- $V_{GS(th)} < 3\,V$ pour être pleinement ouvert avec la sortie du 555 (~4.5 V) → "logic-level".
- $V_{DS}$ supportable > 12 V (marge sur 7.4 V batterie + spike inductif).
- $I_D$ supportable > 1 A (×2 le pic moteur).

| MOSFET | $V_{GS(th)}$ | $V_{DS}$ | $I_D$ | $R_{DS(on)}$ | Verdict |
|--------|--------------|----------|-------|---------------|---------|
| 2N7000 | 2 V | 60 V | 0.2 A | 1.2 Ω | $I_D$ trop faible ❌ |
| **IRLZ44N** | 1–2 V | 55 V | **47 A** | **22 mΩ** | ✅ Surdimensionné mais idéal |
| IRF540N | 4 V | 100 V | 33 A | 44 mΩ | Pas logic-level ❌ |
| AO3400 (CMS) | 1.3 V | 30 V | 5.7 A | 27 mΩ | Bien mais CMS |

→ **IRLZ44N** en boîtier TO-220, traversant, facile à câbler sur breadboard.

### Vérifications

- Dissipation conduction : $0.022 \cdot 0.5^2 = 5.5\,mW$ → boîtier reste froid, pas de dissipateur. ✓
- Marge en courant : 47 A / 0.5 A = **×94**. Inutile mais sans surcoût (le MOSFET coûte 0.80 €).

---

## Étape II.16 — Sécurisation de la grille du MOSFET

Un MOSFET sans précautions sur sa grille = un piège à pannes intermittentes.

### Problème 1 — Grille flottante au démarrage

Tant que le 555 ne s'est pas initialisé (~50 ms après mise sous tension), sa sortie peut être indéterminée → le MOSFET peut être *partiellement* passant → la pompe peut faire un sursaut.

**Solution : pull-down de grille**

```
   V_555 ──[R_g]──┬──▶ Gate
                  │
              [R_pd = 10 kΩ]
                  │
                 GND
```

Tant que V_555 est flottant, R_pd tire la grille à 0 V → MOSFET fermement OFF.

**Choix de R_pd = 10 kΩ :**
- Assez grand pour ne pas charger la sortie du 555 ($I = 5/10\,000 = 0.5\,mA$ ≪ 200 mA capable).
- Assez petit pour être robuste face aux courants de fuite (~µA).

### Problème 2 — Pic de courant de charge de la grille

La grille du MOSFET a une capacité (~1 nF). Si on l'attaque sans résistance, le 555 doit fournir un pic de courant énorme pendant quelques ns.

**Solution : résistance de grille en série**

$$I_{peak} = \frac{V_{CC}}{R_g}$$

Je veux $I_{peak} < 50\,mA$ (confortable pour le 555 qui peut fournir 200 mA mais autant rester loin de la limite).

$$R_g > \frac{5}{0.05} = 100\,\Omega$$

→ **R_g = 220 Ω** (valeur standard, légèrement supérieure).

Vérification : temps de commutation $\tau = R_g \cdot C_{gate} = 220 \cdot 10^{-9} = 220\,ns$ → bien plus rapide que les 500 ms d'impulsion. ✓

---

## Étape II.17 — Protection contre la surtension inductive

### Le problème (rappel)

La pompe est un moteur DC = inductance. Quand on coupe le courant brutalement, l'inductance s'oppose à la variation et génère :

$$V_{spike} = -L \cdot \frac{dI}{dt}$$

Avec $L \approx 1\,mH$, $\Delta I = 0.5\,A$ et $\Delta t \approx 100\,ns$ (commutation MOSFET) :

$$V_{spike} = 10^{-3} \cdot \frac{0.5}{10^{-7}} = 5000\,V$$

(en pratique limité par les capacités parasites à ~50–100 V, mais c'est déjà bien assez pour griller le MOSFET dont $V_{DS,max} = 55\,V$).

### Solution : diode de roue libre en antiparallèle

```
                V_bat
                  │
                  ├──── cathode
                  │     │
                  │   [D_FW] (anti-parallèle sur la pompe)
                  │     │
                  ├──── anode
                  │     │
              [POMPE]
                  │
                Drain MOSFET
```

Quand le MOSFET s'ouvre, le courant inductif continue de circuler **par la diode** au lieu de générer une surtension.

### Choix de la diode

Critères :
- Courant : > 0.5 A → toutes les diodes signal conviennent.
- Tension inverse : > 7.4 V → idem.
- **Vitesse** : doit s'allumer très vite (avant que le pic ne dépasse 55 V).

| Diode | Type | $V_F$ | $t_{rr}$ | Verdict |
|-------|------|-------|----------|---------|
| 1N4007 | Standard | 1 V | ~2 µs | Trop lent ❌ |
| 1N4148 | Signal rapide | 1 V | 4 ns | OK mais 200 mA seulement |
| **1N5819** | **Schottky** | **0.4 V** | < 10 ns | ✅ **Choix optimal** |

→ **1N5819 Schottky**, 1 A, ultra-rapide.

> **Erreur classique à éviter :** mettre une 1N4007 ici. Elle "marche" mais sa lenteur laisse passer le pic — on risque une dégradation du MOSFET sur le long terme.

---

## Étape II.18 — Architecture d'alimentation : ne rien casser

Je dois maintenant intégrer le module dans l'écosystème électrique du robot.

### Les rails disponibles sur le robot

| Rail | Tension | Source | Usage actuel |
|------|---------|--------|--------------|
| $V_{bat}$ | 7.4 V | LiPo direct | Moteurs (via L293D) |
| $V_{CC}$ logique | 5 V | LM7805 | Capteurs, LM393 |

### Où prendre quoi ?

| Bloc du module | Rail nécessaire | Justification |
|----------------|-----------------|---------------|
| LED IR centrale | 5 V | Identique aux autres LED IR |
| Phototransistor | 5 V | Idem |
| LM393 #2 | 5 V | Tension logique |
| Diviseur seuil | 5 V | Référence stable |
| NE555 | 5 V | $V_{CC}$ logique, plage 4.5–16 V |
| **Pompe** | $V_{bat}$ (7.4 V) | Évite de surcharger le LM7805 (1 A max) avec 500 mA pompe |

### Règle d'or : la pompe NE PASSE PAS par le LM7805

$$P_{LM7805} = (V_{in} - V_{out}) \cdot I_{out} = (7.4 - 5) \cdot I_{out}$$

Si je faisais passer la pompe (500 mA pic) par le LM7805 :
$$P = 2.4 \cdot 0.5 = 1.2\,W$$

Le LM7805 en TO-220 sans dissipateur supporte ~1 W → **destruction probable**.

→ La pompe est tirée **directement** de $V_{bat}$, en parallèle de l'alimentation moteur, **avec sa propre piste** depuis la batterie pour ne pas faire transiter ses à-coups par la masse moteur.

### Schéma de répartition

```
  Batterie LiPo 2S (7.4 V) ──┬─────▶ L293D (Vs) ────▶ Moteurs
                              ├─────▶ POMPE (via MOSFET du module)  ← NOUVEAU
                              │
                              └─────▶ LM7805 ────▶ +5V ──┬──▶ Capteurs (G, D, C)
                                                          ├──▶ LM393 #1 (suivi)
                                                          ├──▶ LM393 #2 (pompe)  ← NOUVEAU
                                                          └──▶ NE555  ← NOUVEAU
```

### Découplage supplémentaire à ne pas oublier

Lors de l'activation pompe, le rail $V_{bat}$ subit un à-coup. Pour éviter qu'il ne perturbe les autres consommateurs :
- **Condensateur 100 µF** entre $V_{bat}$ et GND, **au plus près de la pompe**.
- **Condensateur 100 nF** sur le VCC du NE555 (déjà compté dans le BOM).

---

## Étape II.19 — Bilan énergétique et impact sur l'autonomie

### Consommation détaillée du module

| Composant | $I$ continu | $I$ pic | Durée pic |
|-----------|-------------|---------|-----------|
| LED IR centrale (R = 220 Ω) | 17.3 mA | — | — |
| Phototransistor | ~1 mA | — | — |
| LM393 #2 | ~0.8 mA | — | — |
| Pull-up 4.7 kΩ (à l'état BAS) | 1.06 mA | — | — |
| Diviseur seuil 10 kΩ | 0.5 mA | — | — |
| NE555 | ~3 mA | — | — |
| Pompe | 0 (au repos) | **500 mA** | 0.5 s par activation |

### Hypothèse de fréquence d'activation

Sur un parcours typique de 5 m avec 3 zones effacées tous les 1.5 m, à 0.2 m/s :
- Durée totale parcours : 25 s
- Activations pompe : 3 × 0.5 s = 1.5 s
- **Rapport cyclique pompe : 6 %**
- Courant moyen pompe : $0.06 \cdot 500 = 30\,mA$

### Total module

$$I_{moy,module} = 17.3 + 1 + 0.8 + 1.06 + 0.5 + 3 + 30 = 53.7\,mA$$

### Impact sur l'autonomie globale

| Scénario | Conso totale moyenne | Autonomie LiPo 1500 mAh |
|----------|----------------------|--------------------------|
| Robot seul (cf. Partie I, étape I.17) | 840 mA | 1 h 46 min |
| Robot + module pompe | **894 mA** | **1 h 40 min** |
| **Perte d'autonomie** | — | **−6.4 %** |

→ Impact acceptable, dans la marge habituelle de variation des LiPo. ✓

---

## Étape II.20 — Vérification finale : qu'est-ce qui peut mal se passer ?

Avant de figer le design, je passe en revue les modes de défaillance.

### Check-list FMEA (Failure Mode and Effects Analysis) simplifiée

| # | Défaillance | Effet | Mitigation déjà prévue ? |
|---|-------------|-------|--------------------------|
| 1 | Capteur central débranché | Ligne "toujours effacée" → pompe en continu pendant 0.5 s puis se ré-arme jamais (TRIG reste bas) | ✅ Le 555 monostable ne se ré-arme que sur un nouveau front descendant. Pompe ne tourne qu'une fois. |
| 2 | LM393 #2 grillé | Sortie indéterminée | ✅ Pull-up 4.7 kΩ tire la sortie HIGH par défaut → comparateur lu comme "ligne présente" → pompe OFF |
| 3 | NE555 grillé (sortie en court à VCC) | Pompe en continu | ⚠️ Risque réel. **Atténuation :** ajouter un fusible 1 A en série avec la pompe (à intégrer en option) |
| 4 | MOSFET court-circuité (Drain-Source) | Pompe en continu jusqu'à vidange réservoir | Idem ↑ |
| 5 | Pompe bloquée mécaniquement (peinture sèche) | Surcourant prolongé sur MOSFET | ✅ MOSFET ×94 surdimensionné, pas de dégradation immédiate. **Recommandation :** rincer le tube après chaque session. |
| 6 | Diode 1N5819 inversée | Court-circuit permanent de $V_{bat}$ → fusible LiPo saute | ⚠️ Erreur de câblage. **Mitigation :** marquer clairement la cathode au montage (bague blanche). |
| 7 | Démarrage robot avec capteur sur blanc | Une impulsion parasite au boot | ✅ Acceptable (le robot est de toute façon arrêté faute de signal de suivi). |
| 8 | Vibrations détachent le potentiomètre | Seuil dérive | ✅ Multitour = vis bloquée mécaniquement. **Optionnel :** scellement à la cire après calibration. |

### Recommandations finales suite à FMEA

1. ✅ **Ajouter au schéma définitif :** un fusible polymère réarmable 1 A (PTC) en série avec la pompe. Coût marginal (~0.50 €), couvre les défauts #3 et #4.
2. ✅ **Procédure de montage :** vérifier l'orientation de la diode 1N5819 au multimètre (mode diode) avant mise sous tension.
3. ✅ **Procédure d'entretien :** rinçage du tube à l'eau claire après chaque utilisation.

---

## Synthèse Partie II

Voici, en une page, la séquence complète des décisions du module pompe, dans l'ordre :

| # | Étape | Décision | Composant déduit |
|---|-------|----------|------------------|
| II.0 | Cahier des charges | 6 contraintes posées | — |
| II.1 | Philosophie | 100 % analogique | — |
| II.2 | Décomposition | 4 blocs en série | — |
| II.3 | Capteur | TCRT5000 (cohérence) | **TCRT5000 ×1** |
| II.4 | Position | Centré, avant, sur l'axe | — |
| II.5a | Câblage LED | $R = (5-1.2)/0.020 = 190 \Omega$ | **220 Ω** |
| II.5b | Câblage photo | Pull-up donne 0.5–4 V | **10 kΩ** |
| II.6 | Mesure & seuil | $V_{seuil} \approx 2.5\,V$, ajustable | **POT 10 kΩ multitour** |
| II.7 | Comparateur | LM393 dédié (canal isolé) | **LM393 ×1** |
| II.8 | Câblage comparateur | $V_{capt}$ sur (−), seuil sur (+) → inversion | — |
| II.9 | Pull-up sortie | $R = 4.7\,k\Omega$ → 1 mA sink | **4.7 kΩ** |
| II.10 | Hystérésis | $\Delta V = 50\,mV \Rightarrow R = 1\,M\Omega$ | **1 MΩ** |
| II.11 | Filtre RC | $\tau = 100\,ms \Rightarrow$ R=100k, C=1µF | **100 kΩ + 1 µF** |
| II.12 | Temporisation | NE555 monostable | **NE555 ×1** |
| II.13 | Durée d'impulsion | $T_{on} = 1.1 \cdot 470k \cdot 1\mu = 517\,ms$ | **470 kΩ + 1 µF + 10 nF + 100 nF** |
| II.14 | Pompe | Péristaltique 6 V, dose précise | **Kamoer KPP-B04** |
| II.15 | Transistor | MOSFET logic-level, $R_{DS} \ll$ BJT | **IRLZ44N** |
| II.16 | Sécurité grille | Pull-down + résistance série | **220 Ω + 10 kΩ** |
| II.17 | Roue libre | Schottky rapide | **1N5819** |
| II.18 | Alimentation | Pompe sur $V_{bat}$ direct | Découplage **100 µF** |
| II.19 | Énergie | Impact = −6.4 % autonomie | OK |
| II.20 | FMEA | Ajout d'un PTC 1 A | **Fusible PTC 1 A** |

### Ce que cette Partie II m'a appris

1. **Ne jamais choisir un composant avant d'avoir compris ce qui l'entoure.** Chaque composant n'a de sens que dans son interface avec le suivant.
2. **Mesurer avant de calculer.** Le seuil de 2.5 V vient d'une mesure réelle, pas d'une supposition.
3. **La cohérence avec l'existant prime.** Réutiliser le TCRT5000, le LM393, le 5 V → moins de pièces, moins d'erreurs, plus de maintenabilité.
4. **L'inversion logique est gratuite si on la fait au bon endroit.** Câbler V_capt sur l'entrée (−) du comparateur évite un inverseur supplémentaire.
5. **Anticiper les pannes.** Une FMEA même sommaire fait apparaître des composants oubliés (le PTC ici).
6. **Standardiser les valeurs.** 10 kΩ, 100 nF, 1 µF, 4.7 kΩ : la moitié du BOM utilise des valeurs déjà présentes sur le robot. Pas de stock additionnel à gérer.

> **Mot de la fin :** la conception "pas à pas" prend plus de temps que sortir un schéma de catalogue, mais elle produit un circuit qui *tient debout* : chaque composant a une raison d'être, chaque valeur est justifiée, chaque choix peut être défendu en revue. C'est la différence entre **assembler** et **concevoir**.

---

# 🎁 BONUS — Intégration physique du 3ᵉ actionneur (la pompe) sur le robot existant

> **Pourquoi ce bonus ?** Les 20 étapes précédentes décrivent la **conception** du module. Mais entre un schéma validé sur papier et un robot qui marche, il y a tout le travail d'**intégration physique** : où poser le 555 sur la breadboard, par où passer les fils, comment fixer la pompe, dans quel ordre brancher pour ne rien griller. C'est ce que ce bonus détaille, **dans l'ordre exact où je le ferais à l'établi**.

> **Contexte :** le robot est déjà câblé et fonctionnel. Les deux moteurs tournent, les deux capteurs IR de suivi fonctionnent. Tout ce qui suit s'**ajoute** sans rien démonter de l'existant.

---

## B1 — État des lieux avant intégration

Avant de poser le moindre composant nouveau, je fais l'**inventaire** de ce qui existe et de ce qui est libre. Sans cet état des lieux, on cumule les erreurs.

### Inventaire de la breadboard existante

```
        +────────── Rails d'alimentation ──────────+
        │  + (rouge) = +5V (sortie LM7805)         │
        │  + (rouge bis, en bas) = Vbat (7.4V)     │
        │  − (bleu) = GND commune                  │
        +──────────────────────────────────────────+

        Zone A : Alimentation     [LM7805 + 100µF + 100nF]      → OCCUPÉE
        Zone B : Capteurs G+D     [TCRT5000 ×2 + résistances]   → OCCUPÉE
        Zone C : Comparateur #1   [LM393 + pots + pull-ups]     → OCCUPÉE
        Zone D : Pont en H        [L293D + condos]              → OCCUPÉE
        Zone E : LIBRE (~25 lignes disponibles à droite)        → DISPO ✓
```

→ **Décision :** tout le module pompe va dans la **Zone E**, à droite du L293D, avec ses propres rails locaux.

### Inventaire mécanique

| Élément du châssis | État | Utilisable pour le module ? |
|--------------------|------|------------------------------|
| Face avant inférieure | Capteurs G/D montés | OUI — il reste 10 mm au centre pour le 3ᵉ capteur |
| Face supérieure | Vide | OUI — pose du réservoir peinture |
| Côté arrière | Vide | OUI — passage de la buse |
| Espace sous le PCB | Câblage moteurs | NON (humidité peinture interdite) |

### Liste des outils sortis sur l'établi

Avant de toucher à quoi que ce soit, je sors :

- Multimètre (continuité + tension DC)
- Pince à dénuder
- Fer à souder + étain (pour le câble pompe uniquement, le reste reste sur breadboard)
- Tournevis multitour (pour les potentiomètres)
- Sachet zip pour ranger les composants neufs sortis de leur emballage

---

## B2 — Préparation et test des composants neufs (avant montage)

**Règle d'or :** je teste chaque composant **isolément** avant de l'insérer dans le circuit. Un composant mort identifié sur le circuit complet, c'est 30 min de débogage. Identifié seul, c'est 10 secondes.

### Test du TCRT5000 #3 (capteur central)

Sur une mini-breadboard de test, ou même simplement avec deux pinces crocodile :

```
   +5V ──[220Ω]── LED IR (anode) ── (cathode) ── GND     (vérifier qu'on voit ROUGE faible avec un appareil photo de smartphone : la caméra "voit" l'IR)

   +5V ──[10kΩ]──┬── Collecteur photo
                 │── Émetteur photo ── GND
                 │
                 └─▶ Multimètre (mesure tension)
```

| Test | Résultat attendu |
|------|------------------|
| Smartphone visant la LED IR | Petit point rose visible ✓ |
| V_collecteur main devant capteur | Chute de ~4 V à ~0.5 V quand on approche le doigt |
| V_collecteur sans réflexion | Reste à ~4.5 V |

Si l'un de ces tests échoue → composant défectueux, ne jamais l'insérer dans le circuit final.

### Test du LM393 #2

Branchement minimal :
```
   Pin 8 (VCC) ── +5V
   Pin 4 (GND) ── GND
   Pin 3 (IN+) ── +5V (via 10kΩ pour test)
   Pin 2 (IN−) ── GND (via 10kΩ pour test)
   Pin 1 (OUT) ── pull-up 4.7kΩ vers +5V + multimètre
```

→ Sortie doit être à **~5 V** (puisque IN+ > IN−). Inverser les entrées → sortie tombe à **~0 V**. Si non → CI mort.

### Test du NE555

Branchement astable rapide (juste pour confirmer qu'il vit) :
```
   Pin 1 ── GND
   Pin 8, 4 ── +5V
   Pin 2, 6 reliées
   Pin 7 ── via 10kΩ vers +5V
   Pin 2-6 ── via 10kΩ vers Pin 7, et via 10µF vers GND
   Pin 3 ── LED + 470Ω vers GND
```

→ La LED doit clignoter visiblement (~1 Hz). Si non → 555 mort.

### Test du MOSFET IRLZ44N (mode diode au multimètre)

| Test | Résultat attendu |
|------|------------------|
| Diode entre Drain (rouge) et Source (noir) | Bloqué (∞) |
| Diode entre Source (rouge) et Drain (noir) | ~0.5 V (diode body) |
| Toucher la grille avec le doigt après un test | Le MOSFET peut conduire transitoirement (charge statique) — normal |

### Test de la diode 1N5819

Mode diode au multimètre :
- Sens passant (rouge sur anode) : ~0.2 V
- Sens bloqué : ∞
- **Marquer la cathode au feutre indélébile** (bague blanche déjà présente, mais re-vérifier).

### Test à blanc de la pompe

⚠️ **Étape critique pour ne pas griller le MOSFET sur erreur de câblage de la pompe.**

Brancher la pompe **directement** sur la batterie LiPo (à travers un interrupteur manuel) pendant 1 seconde :
- Vérifier le sens de rotation (doit aspirer dans le bon sens — sinon inverser les fils).
- Mesurer le courant à l'ampèremètre en série : doit être **300–500 mA**.
- Si le courant dépasse 800 mA → mécanique grippée, ne pas continuer.

→ Une fois ce test passé, on connaît le sens correct des fils. **Marquer le fil "+" au feutre rouge.**

---

## B3 — Stratégie de montage : du plus passif au plus actif

**Règle d'or n°2 :** on monte dans l'ordre **du plus passif au plus actif**, et on **teste après chaque sous-bloc**. Ainsi, si quelque chose ne marche plus, c'est forcément lié au dernier bloc ajouté.

Ordre choisi :

```
   1. Capteur central (passif, sans interaction avec l'existant)
   2. Comparateur LM393 #2 (actif mais sortie déconnectée du reste)
   3. Filtre RC + 555 (chaîne de signal complète, sortie déconnectée du MOSFET)
   4. Étage de puissance MOSFET (sortie déconnectée de la pompe)
   5. Pompe (avec test à sec, sans peinture)
   6. Réservoir + tube + peinture (intégration mécanique finale)
```

Chaque étape se termine par un **point de contrôle** au multimètre.

---

## B4 — Sous-bloc 1 : Câblage du capteur central

### Position physique sur le châssis

```
         AVANT DU ROBOT (vue de dessous)
   ┌─────────────────────────────────────┐
   │                                     │
   │ [TCRT5000 G]   [TCRT5000 C]   [TCRT5000 D]
   │      ●               ◉                ●     ← ◉ = capteur NEUF à percer
   │                                     │
   │  (déjà         (à percer            (déjà
   │   monté)        et coller)           monté)
   │                                     │
   └─────────────────────────────────────┘
```

### Étapes physiques

1. **Tracer** au crayon le centre exact entre les deux capteurs existants.
2. **Percer** un trou Ø 5.5 mm dans la platine châssis (perceuse à main + foret bois ou alu selon matériau).
3. **Insérer** le TCRT5000 #3 par le dessous, le clipser ou le coller à la colle chaude (juste un petit point).
4. **Vérifier** la hauteur sol : le capteur doit dépasser de 5 mm au-dessous de la platine, comme les deux autres. Cale possible si nécessaire.
5. **Tirer 4 fils** (anode LED, cathode LED, collecteur photo, émetteur photo) **vers la breadboard** zone E, en passant **AU-DESSUS** du PCB (pour que la peinture éventuelle ne mouille pas les contacts).

### Câblage breadboard du capteur

```
   Ligne 1 zone E :  +5V ──[220Ω]── Anode LED IR
   Ligne 2 zone E :  Cathode LED IR ── GND

   Ligne 3 zone E :  +5V ──[10kΩ]── Collecteur photo  ── ➤ V_capt_C (ligne 3)
   Ligne 4 zone E :  Émetteur photo ── GND
```

### ✅ Point de contrôle 1

Multimètre sur V_capt_C :

| Position du robot | Tension attendue |
|-------------------|------------------|
| Robot soulevé en l'air | ~4.5 V |
| Robot posé sur feuille blanche | 3.8–4.3 V |
| Robot posé sur ligne noire | 0.4–0.8 V |

Si valeurs cohérentes → on continue. Sinon → vérifier l'orientation de la LED IR (anode = patte la plus longue) et la continuité des fils.

---

## B5 — Sous-bloc 2 : Câblage du comparateur LM393 #2

### Position sur la breadboard

LM393 inséré sur la zone E, à cheval sur le canyon central, **5 lignes après** le capteur.

```
   Pin 8 (VCC)  ─── +5V
   Pin 4 (GND)  ─── GND
   Pin 1 (OUT1) ─── ligne dédiée "V_inv" (à laisser flottante pour l'instant)
   Pin 2 (IN1−) ─── V_capt_C (déjà disponible)
   Pin 3 (IN1+) ─── ligne dédiée "V_seuil_pompe"
   Pins 5, 6, 7 ─── canal #2 NON UTILISÉ (laisser flottant ou tirer à GND)
```

### Câblage des éléments périphériques

1. **Diviseur de seuil :**
   ```
   +5V ── pin 1 du POT 10 kΩ multitour
   GND ── pin 3 du POT
   pin 2 (curseur) du POT ── Pin 3 du LM393 (IN1+)
   ```

2. **Pull-up de sortie :**
   ```
   Pin 1 du LM393 ──[4.7kΩ]── +5V
   ```

3. **Hystérésis :**
   ```
   Pin 1 du LM393 ──[1MΩ]── Pin 3 du LM393 (rétroaction positive)
   ```

### Réglage initial du potentiomètre

Avant de connecter quoi que ce soit en aval, **régler $V_{seuil\_pompe}$ à 2.5 V** :
- Multimètre entre Pin 3 du LM393 et GND.
- Tourner la vis du multitour jusqu'à lire **2.50 V** ± 50 mV.

### ✅ Point de contrôle 2

| Position du robot | V_capt_C | V_inv (Pin 1 LM393) |
|-------------------|----------|---------------------|
| Sur ligne noire | ~0.5 V | **~5 V** (HIGH) |
| Sur feuille blanche | ~4 V | **~0.1 V** (LOW) |

→ Si l'inversion fonctionne, le cœur logique est validé. ✓

> ⚠️ **Erreur fréquente à ce stade :** oublier le pull-up 4.7 kΩ. Sans lui, V_inv "flotte" à n'importe quoi. Symptôme : valeurs aléatoires au multimètre, qui changent quand on touche la sonde.

---

## B6 — Sous-bloc 3 : Filtre RC + monostable NE555

### Câblage du filtre RC

Sur 2 lignes adjacentes après la sortie LM393 :
```
   Pin 1 LM393 (V_inv) ──[100kΩ]── ligne "V_filt"
   Ligne "V_filt" ──[1µF céramique]── GND
```

### Câblage du NE555

NE555 inséré 5 lignes après le filtre :

| Broche 555 | Connexion |
|------------|-----------|
| 1 (GND) | GND |
| 2 (TRIG) | Ligne "V_filt" |
| 3 (OUT) | Ligne dédiée "V_555" (laisser flottante pour l'instant) |
| 4 (RESET) | +5V (relié directement) |
| 5 (CTRL) | ──[10nF]── GND |
| 6 (THRES) | Reliée à pin 7 |
| 7 (DISCH) | Reliée à pin 6 |
| 8 (VCC) | +5V (avec [100nF] de découplage vers GND au plus près) |

### Composants de temporisation

```
   +5V ──[470kΩ]── pin 7 (et donc pin 6)
   pin 6/7 ──[1µF céramique]── GND
```

### Témoin visuel temporaire (optionnel mais TRÈS utile pour la suite)

Sur la sortie pin 3, ajouter une LED rouge avec résistance 1 kΩ vers GND :
```
   Pin 3 du 555 ──[1kΩ]── Anode LED ── Cathode ── GND
```

→ Cette LED s'allumera ~500 ms à chaque déclenchement. **Indispensable** pour valider visuellement avant de brancher la pompe.

### ✅ Point de contrôle 3

1. Soulever le robot en l'air (capteur central voit "blanc lointain") → V_capt_C ≈ 4 V → V_inv = 0 → **LED rouge s'allume ~0.5 s puis s'éteint** ✓
2. Reposer le robot sur la ligne → V_capt_C ≈ 0.5 V → V_inv = 5 V → V_filt remonte → **rien ne se passe** ✓
3. Glisser brièvement (50 ms) une feuille blanche sous le capteur central → **rien** (filtre RC absorbe) ✓
4. Glisser une feuille blanche pendant 1 s → **flash de la LED ~0.5 s** ✓

> Si la LED reste allumée en permanence : le 555 oscille → vérifier le 10 nF sur pin 5 (CTRL).
> Si la LED ne s'allume jamais : vérifier que pin 4 (RESET) est bien à +5 V.

**À ce stade, toute la chaîne de décision est validée.** Reste à brancher l'actionneur.

---

## B7 — Sous-bloc 4 : Étage de puissance MOSFET (À VIDE)

> ⚠️ **Phase la plus délicate.** Une erreur ici peut détruire le MOSFET ou la batterie. On câble **sans la pompe**, on teste, **puis** on branche la pompe.

### Câblage du MOSFET sur breadboard

L'IRLZ44N est en boîtier TO-220 — il occupe 3 lignes adjacentes.

```
   Pin 1 (Gate)   ──[220Ω]── Pin 3 du 555 (V_555)
   Pin 1 (Gate)   ──[10kΩ]── GND  (pull-down)
   Pin 2 (Drain)  ── ligne dédiée "Drain_pump" (laisser flottante)
   Pin 3 (Source) ── GND (rail commun)
```

### Tirage du rail $V_{bat}$

Sur la zone E, je dédie 2 lignes au rail $V_{bat}$ :
- Tirer un fil **rouge épais (AWG 22 ou plus gros)** depuis le rail batterie principal jusqu'à ces 2 lignes.
- **Ne PAS** prendre $V_{bat}$ depuis le L293D — passer directement par le rail batterie.

### Condensateur de découplage local

```
   V_bat (zone E) ──[100µF électrolytique]── GND
```

→ Polarité : **patte longue (+) sur V_bat**, bande blanche (−) sur GND.

### Diode de roue libre 1N5819

Insérée entre Drain_pump et V_bat :
```
   Anode (côté sans bague) ── Drain_pump
   Cathode (bande blanche) ── V_bat
```

⚠️ **L'inverser provoque un court-circuit V_bat → GND dès la mise sous tension.** Vérifier 3 fois.

### Test à vide (sans pompe, sans charge sur le Drain)

Mettre sous tension, robot en l'air :

| Test | Mesure | Attendu |
|------|--------|---------|
| Drain_pump en l'absence de déclenchement | V mesurée | ~0 V (MOSFET OFF, mais flottant — peut être ~V_bat à cause de la diode) |
| Provoquer un déclenchement (carton blanc devant capteur) | V_555 pendant 0.5 s | ~4.5 V ✓ |
| Drain_pump pendant ce déclenchement | V mesurée | ~0 V (MOSFET ON tire à GND) ✓ |

→ Si Drain_pump descend bien à ~0 V quand le 555 est HIGH, l'étage de puissance fonctionne.

> 🔥 **Ne JAMAIS toucher la grille du MOSFET au doigt** entre les tests : la décharge statique peut le tuer.

---

## B8 — Sous-bloc 5 : Branchement de la pompe (À SEC)

### Câblage final

```
   V_bat ───┬── Pompe (+) ── Pompe (−) ── Drain du MOSFET
            │
            └── (déjà la diode 1N5819 antiparallèle sur la pompe)
```

→ Souder ou utiliser des dominos vissés pour les fils de pompe (jamais en breadboard pour 500 mA).

### Insertion du fusible PTC 1 A (FMEA #3 et #4)

```
   V_bat ──[PTC 1A]── Pompe (+)
```

Le PTC est polymère réarmable : en cas de surcourant prolongé, il se réchauffe et coupe ; il se réarme tout seul après refroidissement (1 min). Excellent garde-fou.

### Test à sec (pompe sans tube ni peinture)

Mettre sous tension, robot en l'air, faire passer la main devant le capteur central :

| Geste | Comportement attendu |
|-------|----------------------|
| Main devant capteur (1 s) | LED rouge s'allume + **pompe vrombit** pendant ~0.5 s |
| Main retirée | Pompe s'arrête |
| Maintenir la main devant 5 s | Pompe ne tourne **qu'une fois** (0.5 s), puis silence — c'est le monostable |
| Faire 10 cycles d'affilée | Toucher le MOSFET : doit rester froid ou tiède |

### ✅ Point de contrôle 4

Si tout est correct → **le module fonctionne entièrement en mode "sec"**. Il ne reste qu'à ajouter le fluide.

> 🔍 **Symptôme à surveiller :** si la pompe "rebroussre" (tourne dans le mauvais sens), inverser les deux fils de la pompe (et seulement ceux-là). Si elle pulse irrégulièrement → mauvais découplage : ajouter un 100 µF de plus sur V_bat zone E.

---

## B9 — Sous-bloc 6 : Intégration mécanique du circuit fluidique

### Montage du réservoir

1. Choisir un flacon souple (style flacon de colle 30–50 ml) avec bouchon vissable.
2. Percer 2 trous de Ø 4 mm dans le bouchon : un pour le tube d'aspiration, un pour la mise à l'air libre.
3. Fixer le flacon **sur le dessus du châssis**, vers l'arrière, avec un velcro double-face (pour pouvoir le retirer pour rincer).

### Routage du tube

```
   [RÉSERVOIR (haut)]
        │ tube silicone Ø 3mm
        ▼
   [POMPE PÉRISTALTIQUE]
        │ tube silicone Ø 3mm
        ▼
   [BUSE]  ← orientée vers le sol, juste derrière le capteur central
```

- Longueur tube côté aspiration : 10 cm (court pour réduire le temps d'amorçage).
- Longueur tube côté refoulement : 5 cm + buse.
- **Fixer la buse** par collier de serrage à 5 mm du sol, alignée verticalement avec le capteur central.

### Amorçage initial

1. Remplir le réservoir d'eau claire (test sans peinture d'abord !).
2. Provoquer 2-3 déclenchements manuels (carton devant capteur).
3. Vérifier qu'une goutte sort de la buse à chaque cycle.
4. Si non : tube plié, joint mal serré, ou pompe à l'envers.

### Remplissage avec peinture

- Vidanger l'eau, sécher le réservoir.
- Remplir avec **peinture acrylique diluée à 50 % avec de l'eau** (sinon trop visqueuse pour la pompe péristaltique).
- Effectuer 1 cycle d'amorçage (~2 secondes manuelles).
- Le système est prêt pour test sur piste.

---

## B10 — Test sur piste réelle

### Préparation de la piste de test

Sur une grande feuille blanche :
- Tracer une ligne noire de 18 mm de large, longue de 1 m.
- Réaliser **3 interruptions** :
  - Interruption A : 10 mm (devrait être ignorée)
  - Interruption B : 30 mm (devrait déclencher)
  - Interruption C : 80 mm (devrait déclencher 1 seule fois — pas 2)

### Procédure de test

1. Poser le robot au début de la ligne, contact ON.
2. Le robot doit avancer en suivant la ligne.
3. Observer à chaque interruption :

| Interruption | Comportement attendu | LED rouge | Goutte de peinture |
|--------------|----------------------|-----------|--------------------|
| A (10 mm) | Le robot continue droit (capt. G/D restent sur les bords noirs) | Pas allumée | Aucune |
| B (30 mm) | Robot continue, LED clignote ~0.5 s | OUI | Trace de ~5 cm |
| C (80 mm) | Robot continue (peut-être perte momentanée de suivi), LED clignote 1 fois (pas 2) | OUI ×1 | Trace de ~5 cm puis rien |

### En cas d'écart

| Symptôme | Diagnostic | Correction |
|----------|------------|------------|
| Interruption A déclenche aussi | Filtre RC trop court | Augmenter $C_f$ à 2.2 µF |
| Interruption B ne déclenche pas | Seuil trop bas | Augmenter $V_{seuil\_pompe}$ vers 3.0 V |
| Trace trop courte / pas de peinture | $T_{on}$ trop bref ou pompe pas amorcée | Re-amorcer ou augmenter $R_t$ à 680 kΩ |
| Trace énorme / flaque | $T_{on}$ trop long ou peinture trop liquide | Réduire $R_t$ à 220 kΩ ou peinture moins diluée |
| Robot s'arrête de suivre quand pompe tourne | Couplage parasite par GND | Tirer un fil GND séparé du module pompe vers la batterie |

---

## B11 — Mise en service finale et procédures d'utilisation

### Procédure de démarrage normale

1. Vérifier le réservoir (peinture > 1/4 plein).
2. Vérifier l'amorçage (1 goutte au bout de la buse).
3. Allumer le robot.
4. Vérifier la LED rouge éteinte au repos.
5. Lâcher sur la piste.

### Procédure de fin de session

⚠️ **Critique :** la peinture acrylique sèche en 30 min dans le tube → blocage permanent.

1. Vidanger la peinture restante du réservoir.
2. Remplir le réservoir d'eau claire.
3. Provoquer 5–10 cycles de pompe pour rincer le tube.
4. Vidanger l'eau.
5. Provoquer 2–3 cycles à sec pour vider le tube.
6. Stocker la pompe avec le tube sec.

### Maintenance hebdomadaire

- Vérifier l'état du tube silicone (doit rester souple à l'intérieur de la pompe — la péristaltique l'use à la longue).
- Remplacer le tube tous les ~50 h de pompe (5 € le mètre).

---

## B12 — Synthèse de l'intégration en une page

```
                                Inventaire (B1)
                                       │
                                       ▼
                       Test individuel des composants (B2)
                                       │
                                       ▼
                    ┌──────────────────────────────────────┐
                    │  Sous-bloc 1 : Capteur central (B4)  │── 🔍 ✅ Point contrôle 1
                    └──────────────┬───────────────────────┘
                                   ▼
                    ┌──────────────────────────────────────┐
                    │  Sous-bloc 2 : Comparateur (B5)      │── 🔍 ✅ Point contrôle 2
                    └──────────────┬───────────────────────┘
                                   ▼
                    ┌──────────────────────────────────────┐
                    │  Sous-bloc 3 : RC + 555 + LED (B6)   │── 🔍 ✅ Point contrôle 3
                    └──────────────┬───────────────────────┘
                                   ▼
                    ┌──────────────────────────────────────┐
                    │  Sous-bloc 4 : MOSFET à vide (B7)    │── 🔍 ✅ Point contrôle 4
                    └──────────────┬───────────────────────┘
                                   ▼
                    ┌──────────────────────────────────────┐
                    │  Sous-bloc 5 : Pompe à sec (B8)      │── 🔍 vrombit OK
                    └──────────────┬───────────────────────┘
                                   ▼
                    ┌──────────────────────────────────────┐
                    │  Sous-bloc 6 : Fluidique (B9)        │── 🔍 amorçage OK
                    └──────────────┬───────────────────────┘
                                   ▼
                          Test sur piste (B10)
                                   │
                                   ▼
                          Mise en service (B11)
```

### Les 5 commandements de l'intégration physique

1. **Tester chaque composant SEUL** avant de l'intégrer (B2).
2. **Construire DU PASSIF VERS L'ACTIF** (B3) — capteur d'abord, pompe en dernier.
3. **VALIDER À CHAQUE ÉTAPE** au multimètre avant de passer à la suivante.
4. **JAMAIS souder ou connecter sous tension.** Toujours couper l'alimentation, modifier, puis remettre sous tension.
5. **GARDER LA LED ROUGE TÉMOIN.** Elle reste à demeure : c'est le meilleur outil de diagnostic pour la maintenance future.

> **Mot de la fin (bonus) :** un schéma juste sur le papier ne garantit pas un robot qui marche. C'est la **discipline du montage** — tester avant, monter en couches, valider à chaque étape — qui transforme un design en machine. Le temps "perdu" à tester un LM393 seul est toujours rentabilisé au centuple lors du débogage final.
