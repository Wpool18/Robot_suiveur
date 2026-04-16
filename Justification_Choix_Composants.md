# Justification des Choix Techniques — BOM

## Robot Suiveur de Ligne "Pur Hard"

> Ce document reprend chaque composant de la nomenclature (BOM) et justifie son choix technique par des critères fonctionnels, des calculs de dimensionnement et/ou des comparaisons avec les alternatives.

---

## Table des matières

1. [Capteurs IR — TCRT5000](#1-capteurs-ir--tcrt5000)
2. [Résistance 220Ω — Limitation LED IR](#2-résistance-220ω--limitation-led-ir)
3. [Résistance 10kΩ — Pull-up & diviseur de seuil](#3-résistance-10kω--pull-up--diviseur-de-seuil)
4. [Résistance 4.7kΩ — Pull-up sortie LM393](#4-résistance-47kω--pull-up-sortie-lm393)
5. [Potentiomètre 10kΩ — Réglage seuil](#5-potentiomètre-10kω--réglage-seuil)
6. [Comparateur — LM393](#6-comparateur--lm393)
7. [Pont en H — L293D](#7-pont-en-h--l293d)
8. [Condensateurs de découplage — 100nF céramique](#8-condensateurs-de-découplage--100nf-céramique)
9. [Condensateurs de filtrage — 100µF électrolytique](#9-condensateurs-de-filtrage--100µf-électrolytique)
10. [Régulateur 5V — LM7805](#10-régulateur-5v--lm7805)
11. [Diodes de protection — 1N4007](#11-diodes-de-protection--1n4007)
12. [LED témoin + Résistance 1kΩ](#12-led-témoin--résistance-1kω)
13. [Interrupteur à glissière](#13-interrupteur-à-glissière)
14. [Moteurs DC avec réducteur](#14-moteurs-dc-avec-réducteur)
15. [Roues motrices Ø 42–65 mm](#15-roues-motrices-ø-4265-mm)
16. [Roue folle (ball caster)](#16-roue-folle-ball-caster)
17. [Batterie LiPo 2S 7.4V](#17-batterie-lipo-2s-74v)

---

## 1. Capteurs IR — TCRT5000

| Critère | Valeur |
|---------|--------|
| **Référence** | TCRT5000 |
| **Quantité** | 2 |
| **Rôle** | Détection de la ligne noire sur fond blanc |

### Justification

Le TCRT5000 intègre dans un seul boîtier une **LED infrarouge** (émetteur, λ = 950 nm) et un **phototransistor** (récepteur), ce qui simplifie le câblage et garantit un alignement optique précis.

**Pourquoi le TCRT5000 et pas une alternative ?**

| Critère | TCRT5000 | QRD1114 | IR333 + PT333 (discret) |
|---------|----------|---------|--------------------------|
| Intégration | Émetteur + récepteur intégrés | Intégré | 2 composants séparés |
| Distance optimale | 2–15 mm | 0.5–5 mm | Variable, alignement manuel |
| Disponibilité | Très courant, bon marché (~0.50€) | Moins courant | Courant |
| Facilité de montage | Simple (4 pins) | Simple (4 pins) | Plus complexe |
| Robustesse optique | Bonne isolation émetteur/récepteur | Moyenne | Dépend du montage |

**Le TCRT5000 est retenu car :**
- Sa distance optimale de **5 mm** correspond parfaitement à un montage sous châssis à faible hauteur du sol
- Son format compact facilite le positionnement symétrique des deux capteurs
- Son prix unitaire (~0.50€) est le plus compétitif
- Sa longueur d'onde de 950 nm est peu perturbée par la lumière ambiante visible

---

## 2. Résistance 220Ω — Limitation LED IR

| Critère | Valeur |
|---------|--------|
| **Référence** | Résistance 220Ω, 1/4W |
| **Quantité** | 2 (une par LED IR) |
| **Rôle** | Limiter le courant traversant la LED IR à une valeur sûre |

### Calcul de dimensionnement

La LED IR du TCRT5000 a les caractéristiques suivantes :
- Tension directe : $V_F = 1.2\,V$ (typique)
- Courant direct nominal : $I_F = 20\,mA$
- Courant direct max : $I_{F,max} = 60\,mA$

$$R_{LED} = \frac{V_{CC} - V_F}{I_F} = \frac{5\,V - 1.2\,V}{20\,mA} = \frac{3.8}{0.020} = 190\,\Omega$$

La valeur normalisée immédiatement supérieure est **220Ω** (série E24).

**Vérification du courant réel :**

$$I_F = \frac{V_{CC} - V_F}{R} = \frac{5 - 1.2}{220} = 17.3\,mA$$

Ce courant est :
- **Suffisant** pour une émission IR efficace (datasheet : sensibilité optimale entre 10–20 mA)
- **Inférieur au maximum** de 60 mA → marge de sécurité ×3.5
- **Légèrement en dessous** de 20 mA, ce qui prolonge la durée de vie de la LED

**Vérification de la puissance dissipée :**

$$P_R = R \times I^2 = 220 \times (0.0173)^2 = 66\,mW$$

Une résistance 1/4W (250 mW) est largement suffisante (ratio de sécurité ×3.8).

---

## 3. Résistance 10kΩ — Pull-up & diviseur de seuil

| Critère | Valeur |
|---------|--------|
| **Référence** | Résistance 10kΩ, 1/4W |
| **Quantité** | 4 (2 pull-up phototransistor + 2 diviseur seuil) |
| **Rôle** | Convertir le courant du phototransistor en tension + fixer le seuil |

### Justification — Pull-up phototransistor (×2)

Le phototransistor du TCRT5000 fonctionne en collecteur ouvert. La résistance de pull-up détermine la **plage de tension de sortie** et la **sensibilité**.

$$V_{out} = V_{CC} - R_{pull-up} \times I_{photo}$$

**Pourquoi 10kΩ ?**

| Valeur | Signal sur blanc | Signal sur noir | Plage utile | Verdict |
|--------|-----------------|-----------------|-------------|---------|
| 1kΩ | ~4.5V | ~4.0V | **0.5V** — trop faible | Sensibilité insuffisante |
| **10kΩ** | ~4.2V | ~0.5V | **3.7V** — excellent | **Bon compromis** |
| 100kΩ | ~4.8V | ~0.01V | Saturation, réponse lente | Trop lent |

Avec 10kΩ :
- **Surface blanche** : phototransistor conduit fort → $V_{out} \approx 3.5–4.5\,V$
- **Surface noire** : phototransistor conduit peu → $V_{out} \approx 0.3–1.0\,V$
- **Écart** de ~3V → facile à discriminer par le comparateur

De plus, le courant consommé reste faible :

$$I_{max} = \frac{V_{CC}}{R} = \frac{5}{10\,000} = 0.5\,mA$$

### Justification — Diviseur de seuil (×2)

Les 2 autres résistances de 10kΩ servent de valeur fixe dans le diviseur de tension avec le potentiomètre pour définir la plage de réglage du seuil. Utiliser la même valeur que le potentiomètre (10kΩ) permet un réglage centré autour de $V_{CC}/2 = 2.5\,V$.

---

## 4. Résistance 4.7kΩ — Pull-up sortie LM393

| Critère | Valeur |
|---------|--------|
| **Référence** | Résistance 4.7kΩ, 1/4W |
| **Quantité** | 2 (une par sortie du LM393) |
| **Rôle** | Tirer la sortie à $V_{CC}$ quand le transistor interne est OFF |

### Justification

Le LM393 a une **sortie collecteur ouvert** : le transistor interne ne peut que tirer la sortie vers GND. Sans résistance de pull-up, la sortie serait flottante à l'état HIGH.

**Pourquoi 4.7kΩ ?**

La sortie du LM393 doit piloter l'entrée IN du L293D. Le courant nécessaire à l'entrée du L293D est faible (~10 µA), donc la résistance de pull-up détermine principalement :

1. **Le courant de sink** ($V_{OL}$ du LM393) :

$$I_{sink} = \frac{V_{CC}}{R_{pull-up}} = \frac{5}{4700} = 1.06\,mA$$

Le LM393 peut absorber jusqu'à **16 mA** en sortie → marge de sécurité ×15.

2. **La vitesse de commutation** : avec 4.7kΩ, la constante de temps $RC$ avec les capacités parasites (~10 pF) reste très faible :

$$\tau = R \times C = 4700 \times 10 \times 10^{-12} = 47\,ns$$

C'est négligeable devant le temps de réponse du LM393 (~1.3 µs).

**Pourquoi pas une autre valeur ?**
- **1kΩ** : courant de 5 mA inutilement élevé → consommation excessive
- **10kΩ** : fonctionnel mais commutation plus lente, tension $V_{OL}$ plus élevée
- **4.7kΩ** : valeur standard, bon compromis vitesse/consommation, recommandée dans le datasheet du LM393

---

## 5. Potentiomètre 10kΩ — Réglage seuil

| Critère | Valeur |
|---------|--------|
| **Référence** | Potentiomètre 10kΩ, multitour |
| **Quantité** | 2 (un par canal capteur) |
| **Rôle** | Régler le seuil de comparaison $V_{seuil}$ |

### Justification

Le potentiomètre sert de diviseur de tension ajustable pour fixer $V_{seuil}$ entre 0V et $V_{CC}$.

$$V_{seuil} = V_{CC} \times \frac{R_{bas}}{R_{total}} \quad \text{ajustable de 0 à 5V}$$

**Pourquoi multitour ?**

| Type | Résolution | Précision de réglage | Stabilité mécanique |
|------|-----------|---------------------|---------------------|
| Monotour | ~300° | ~15 mV/degré | Sensible aux vibrations |
| **Multitour** (10 ou 25 tours) | ~3600° | **~1.4 mV/degré** | Excellent |

Le seuil optimal se situe autour de **2.0–2.5V** et doit être réglé avec précision (±50 mV). Un potentiomètre multitour offre une résolution **10× supérieure**, ce qui est crucial pour calibrer finement le robot sur une piste donnée.

**Pourquoi 10kΩ ?**
- Même ordre de grandeur que la résistance de pull-up (10kΩ) → interaction minimale
- Courant consommé : $I = V_{CC}/R = 5V/10kΩ = 0.5\,mA$ → négligeable dans le bilan de puissance
- Valeur standard très disponible

**Pourquoi un potentiomètre par canal ?**
Les deux capteurs IR ont des tolérances de fabrication différentes. Deux potentiomètres indépendants permettent de compenser ces écarts et d'obtenir un comportement symétrique gauche/droite.

---

## 6. Comparateur — LM393

| Critère | Valeur |
|---------|--------|
| **Référence** | LM393 (dual comparator) |
| **Quantité** | 1 (contient 2 comparateurs) |
| **Rôle** | Convertir le signal analogique des capteurs en signal logique tout-ou-rien |

### Justification

Le robot nécessite **2 comparateurs** (un par capteur). Le LM393 contient exactement 2 comparateurs dans un seul boîtier DIP-8.

**Comparaison avec les alternatives :**

| Critère | LM393 | LM358 (AOP) | LM324 (AOP) |
|---------|-------|-------------|-------------|
| Type | **Comparateur dédié** | AOP généraliste | AOP généraliste |
| Temps de réponse | **1.3 µs** | ~20 µs | ~20 µs |
| Sortie | Collecteur ouvert | Push-pull | Push-pull |
| Nb canaux | 2 | 2 | 4 (surdimensionné) |
| $V_{CC}$ | 2–36V | 3–32V | 3–32V |
| $I_{CC}$ | **0.8 mA** | 1.5 mA | 3 mA |
| Compatible 5V direct | Oui | Oui | Oui |
| Prix | ~0.30€ | ~0.30€ | ~0.40€ |

**Le LM393 est retenu car :**

1. **Temps de réponse 15× plus rapide** qu'un AOP utilisé en comparateur (1.3 µs vs 20 µs). Bien que les deux soient suffisants pour un robot lent, un comparateur dédié garantit des fronts propres sans oscillation parasite.

2. **Sortie collecteur ouvert** : permet de choisir librement la tension de pull-up. Si on alimente les moteurs en 6V et la logique en 5V, la sortie peut être adaptée à n'importe quel niveau.

3. **Pas d'oscillation parasite** : un AOP en boucle ouverte (mode comparateur) peut osciller quand les entrées sont proches l'une de l'autre. Le LM393, conçu comme comparateur, est naturellement stable.

4. **Consommation minimale** : 0.8 mA pour les deux canaux.

5. **2 canaux = exactement le besoin** : pas de gaspillage de ressources.

---

## 7. Pont en H — L293D

| Critère | Valeur |
|---------|--------|
| **Référence** | L293D |
| **Quantité** | 1 (commande 2 moteurs) |
| **Rôle** | Fournir le courant aux moteurs et contrôler leur sens de rotation |

### Justification

Les sorties du LM393 ne peuvent fournir que ~1 mA. Les moteurs consomment **250–700 mA en charge**. Le pont en H est l'interface de puissance indispensable.

**Comparaison avec les alternatives :**

| Critère | L293D | L298N | TB6612FNG | Pont H discret (MOSFET) |
|---------|-------|-------|-----------|------------------------|
| Courant continu/canal | 600 mA | **2A** | 1.2A | >10A |
| Courant crête | 1.2A | 3A | 3.2A | >20A |
| Diodes intégrées | **Oui** | Non | Oui | Non |
| $V_{drop}$ | ~1.4V | ~2V | ~0.5V | ~0.1V |
| Nb moteurs | 2 | 2 | 2 | 1 par pont |
| Complexité | DIP-16, simple | Module, moyen | CMS, moyen | 4 MOSFET + diodes + driver |
| Protection thermique | **Oui** | Non | Oui | Non |
| Prix | ~1.50€ | ~3€ | ~2€ | ~5€+ |

**Le L293D est retenu car :**

1. **Diodes de roue libre intégrées** : pas besoin de composants supplémentaires pour protéger le circuit des surtensions induites par les moteurs. Cela réduit le nombre de composants et les risques d'erreur de câblage.

2. **Courant suffisant** : les moteurs TT consomment ~250 mA à vide et ~400 mA en charge. Le L293D supporte 600 mA continu, soit une **marge de 50%**.

**Vérification :**

$$I_{moteur,charge} = 400\,mA < I_{max,L293D} = 600\,mA \quad ✓$$

$$I_{crête,démarrage} \approx 800\,mA < I_{crête,L293D} = 1200\,mA \quad ✓$$

3. **Protection thermique intégrée** : en cas de blocage moteur, le L293D se coupe au lieu de griller.

4. **Commande directe depuis le LM393** : les entrées IN1-IN4 du L293D sont compatibles TTL/CMOS 5V. La sortie du LM393 (avec pull-up 4.7kΩ) attaque directement ces entrées sans circuit intermédiaire.

5. **Boîtier DIP-16** : montable directement sur breadboard ou PCB traversant — adapté au prototypage.

**Inconvénient accepté :**
La chute de tension de ~1.4V (2 × 0.7V par les transistors Darlington internes) réduit la tension aux bornes du moteur :

$$V_{moteur} = V_{bat} - V_{drop} = 7.4 - 1.4 = 6.0\,V$$

Cela reste suffisant pour les moteurs TT (tension nominale 3–6V).

---

## 8. Condensateurs de découplage — 100nF céramique

| Critère | Valeur |
|---------|--------|
| **Référence** | Condensateur céramique 100nF (0.1µF) |
| **Quantité** | 4 |
| **Rôle** | Filtrer les parasites haute fréquence sur les alimentations des CI |

### Justification

Chaque circuit intégré (LM393, L293D, LM7805) nécessite un condensateur de découplage **au plus près de ses broches d'alimentation** pour filtrer les transitoires haute fréquence.

**Pourquoi 100nF ?**

C'est la valeur standard recommandée par la quasi-totalité des datasheets pour le découplage local. L'impédance d'un condensateur céramique de 100nF est minimale autour de **1–10 MHz**, ce qui correspond à la bande de fréquence des parasites de commutation :

$$Z_C = \frac{1}{2\pi f C} = \frac{1}{2\pi \times 1\,MHz \times 100\,nF} = 1.6\,\Omega$$

**Pourquoi céramique ?**
- ESR (résistance série équivalente) très faible → filtrage efficace en HF
- Pas de polarité → pas de risque d'inversion
- Compact, bon marché (~0.05€)
- Stable en température (type X7R ou C0G)

**Placement :** 1 sur $V_{CC}$ du LM393, 1 sur $V_{CC1}$ du L293D, 1 sur $V_S$ du L293D, 1 sur la sortie du LM7805.

---

## 9. Condensateurs de filtrage — 100µF électrolytique

| Critère | Valeur |
|---------|--------|
| **Référence** | Condensateur électrolytique 100µF, 16V |
| **Quantité** | 2 |
| **Rôle** | Filtrer les ondulations basse fréquence sur les rails d'alimentation |

### Justification

Les condensateurs électrolytiques complètent les céramiques en filtrant les **basses fréquences** (ondulations de courant dues aux moteurs, appels de courant lors des démarrages).

**Pourquoi 100µF ?**

Lors d'un à-coup de courant moteur ($\Delta I \approx 400\,mA$ sur ~1 ms), la chute de tension sur le rail est :

$$\Delta V = \frac{\Delta I \times \Delta t}{C} = \frac{0.4 \times 0.001}{100 \times 10^{-6}} = 4\,V$$

Sans le condensateur, un appel de courant provoquerait un effondrement de l'alimentation. Avec 100µF, cette chute est absorbée par le condensateur qui joue le rôle de **réservoir de charge**.

**Pourquoi 16V ?**
- Tension maximale du circuit : $V_{bat} = 7.4\,V$ (LiPo 2S, jusqu'à 8.4V en pleine charge)
- La tension nominale du condensateur doit être **≥ 1.5× la tension de service** : $8.4 \times 1.5 = 12.6\,V < 16\,V$ ✓
- 16V est la valeur standard immédiatement supérieure

**Placement :** 1 en entrée du LM7805 (côté batterie), 1 en sortie du LM7805 (côté 5V).

---

## 10. Régulateur 5V — LM7805

| Critère | Valeur |
|---------|--------|
| **Référence** | LM7805 |
| **Quantité** | 1 |
| **Rôle** | Fournir une tension stable de 5V pour la logique (capteurs, LM393) |

### Justification

La batterie LiPo 2S fournit entre **6.4V** (déchargée) et **8.4V** (pleine charge). Les capteurs IR et le LM393 nécessitent une tension **stable et régulée à 5V**.

**Pourquoi le LM7805 ?**

| Critère | LM7805 | AMS1117-5.0 (LDO) | Buck converter |
|---------|--------|-------------------|----------------|
| Type | Linéaire | LDO linéaire | Commuté |
| $V_{dropout}$ | ~2V | 1.1V | ~0.3V |
| $V_{in,min}$ pour 5V | **7V** | 6.1V | ~5.3V |
| $I_{out,max}$ | 1A | 800 mA | >1A |
| Bruit de sortie | Très faible | Très faible | Moyen (switching) |
| Complexité | 3 composants | 3 composants | >6 composants |
| Prix | ~0.30€ | ~0.30€ | ~2€ |

**Le LM7805 est retenu car :**

1. **Simplicité maximale** : 3 composants (régulateur + 2 condensateurs). Pas d'inductance, pas de diode Schottky, pas de calcul de compensation.

2. **Bruit de sortie très faible** : crucial pour ne pas perturber les capteurs IR et le comparateur. Un régulateur à découpage génère des parasites de commutation qui pourraient provoquer de faux déclenchements.

3. **$V_{in}$ compatible** : avec une LiPo 2S ($V_{min} = 6.4\,V$, typique en service $\approx 7.2\,V$), le dropout de 2V est respecté : $7.2 - 5 = 2.2\,V > 2\,V$ ✓

**Vérification de la puissance dissipée :**

Le LM7805 dissipe l'excès de tension en chaleur :

$$P_{diss} = (V_{in} - V_{out}) \times I_{out} = (7.4 - 5) \times 0.05 = 0.12\,W$$

Le courant logique total est faible (~50 mA : capteurs + LM393). La dissipation de 120 mW est largement dans les capacités du boîtier TO-220 (sans dissipateur, $R_{th,JA} \approx 65°C/W$ → élévation de ~8°C).

> **Note :** Les moteurs sont alimentés **directement** par la batterie (via le L293D, broche $V_S$), pas par le régulateur 5V. Cela évite de faire transiter 800 mA à travers le LM7805.

---

## 11. Diodes de protection — 1N4007

| Critère | Valeur |
|---------|--------|
| **Référence** | 1N4007 |
| **Quantité** | 4 (si pont en H discret) |
| **Rôle** | Diodes de roue libre — protéger les MOSFET contre les surtensions inductives |

### Justification

> **Note :** Ces diodes ne sont nécessaires que si le pont en H est réalisé en **composants discrets** (MOSFET). Le **L293D intègre déjà ses propres diodes de protection** — dans cette configuration, les 1N4007 sont optionnelles (protection supplémentaire en amont).

Un moteur DC est une charge inductive. À la coupure du courant, l'inductance du bobinage génère une surtension :

$$V_{pic} = -L \times \frac{dI}{dt}$$

Pour un moteur TT ($L \approx 2\,mH$, $I = 400\,mA$, $dt \approx 1\,\mu s$) :

$$V_{pic} = 2 \times 10^{-3} \times \frac{0.4}{1 \times 10^{-6}} = 800\,V$$

Sans diode de roue libre, cette surtension **détruirait** instantanément les transistors.

**Pourquoi la 1N4007 ?**
- $V_{RRM} = 1000\,V$ → supporte largement les pics inductifs
- $I_F = 1\,A$ → supporte le courant moteur
- $t_{rr} \approx 30\,\mu s$ → suffisant pour les fréquences de commutation de ce circuit (pas de PWM rapide)
- Prix : ~0.05€, ultra-courant

**Alternative envisagée :** Schottky 1N5819 ($V_F = 0.4\,V$ vs 0.7V pour la 1N4007, $t_{rr} \approx 0$). Plus rapide mais limitée à $V_{RRM} = 40\,V$. Suffisante ici mais moins de marge.

---

## 12. LED témoin + Résistance 1kΩ

| Critère | Valeur |
|---------|--------|
| **Référence** | LED rouge 3mm + Résistance 1kΩ |
| **Quantité** | 2 de chaque (optionnel) |
| **Rôle** | Visualiser l'état de sortie de chaque comparateur |

### Justification

Composants de **debug et diagnostic** — purement optionnels mais très utiles lors de la phase de calibration et de test.

Quand la sortie du LM393 est HIGH (surface blanche), la LED s'allume → permet de vérifier visuellement le fonctionnement de chaque canal sans multimètre.

**Calcul de la résistance :**

$$R_{LED} = \frac{V_{CC} - V_F}{I_F} = \frac{5 - 2.0}{5\,mA} = 600\,\Omega$$

On choisit **1kΩ** pour limiter davantage le courant :

$$I_F = \frac{5 - 2.0}{1000} = 3\,mA$$

À 3 mA, une LED rouge 3mm est **clairement visible** tout en consommant un minimum de courant. Pas besoin de plus de luminosité pour un indicateur de diagnostic.

---

## 13. Interrupteur à glissière

| Critère | Valeur |
|---------|--------|
| **Référence** | Interrupteur à glissière (slide switch) |
| **Quantité** | 1 |
| **Rôle** | Coupure générale de l'alimentation |

### Justification

Un interrupteur mécanique est la solution la plus simple et la plus fiable pour couper l'alimentation. Le choix d'un modèle **à glissière** (plutôt qu'à bascule ou poussoir) se justifie par :

- **État visuel clair** : on voit immédiatement si le robot est ON ou OFF
- **Pas d'appui continu nécessaire** (contrairement à un bouton poussoir)
- **Compact** et montable en bordure de châssis
- Courant nominal typique : 0.5–3A → suffisant pour le courant total du robot (~850 mA)

---

## 14. Moteurs DC avec réducteur

| Critère | Valeur |
|---------|--------|
| **Référence** | Moteur TT jaune (hobby) ou GA12-N20 |
| **Quantité** | 2 |
| **Rôle** | Propulsion du robot en commande différentielle |

### Justification

Le robot nécessite deux moteurs indépendants pour la **commande différentielle** (tourner en coupant un moteur).

**Pourquoi un moteur avec réducteur intégré ?**

Un moteur DC nu tourne à **~10 000 RPM** sous 6V — beaucoup trop rapide, pas assez de couple. Le réducteur (1:48 typique) réduit la vitesse et multiplie le couple :

$$\omega_{sortie} = \frac{\omega_{moteur}}{N} = \frac{10\,000}{48} \approx 200\,RPM$$

$$\tau_{sortie} = \tau_{moteur} \times N \times \eta \approx 0.03 \times 48 \times 0.7 \approx 1.0\,kg \cdot cm$$

**Pourquoi le moteur TT jaune ?**

| Critère | Moteur TT jaune | GA12-N20 | Moteur N20 sans réducteur |
|---------|-----------------|----------|---------------------------|
| Tension | 3–6V | 3–12V | 3–6V |
| Réducteur intégré | Oui (1:48) | Oui (variable) | Non |
| Vitesse (6V) | ~200 RPM | ~100–300 RPM | ~10 000 RPM |
| Couple | ~0.8 kg·cm | ~1.5 kg·cm | ~0.03 kg·cm |
| Fixation | Bracket standard inclus | Nécessite support | Nécessite support |
| Prix | ~1€ | ~3€ | ~1€ |
| Disponibilité | Extrêmement courant (kits Arduino) | Courant | Courant |

Le moteur TT jaune est retenu pour son **prix imbattable**, sa **compatibilité directe** avec le L293D (courant < 600 mA), et la disponibilité universelle de **supports et roues** adaptés.

---

## 15. Roues motrices Ø 42–65 mm

| Critère | Valeur |
|---------|--------|
| **Référence** | Roues caoutchouc compatibles moteur TT |
| **Quantité** | 2 |
| **Rôle** | Transmission du mouvement au sol |

### Justification

Le diamètre de la roue détermine la **vitesse linéaire** du robot :

$$v = \frac{\pi \times D \times \omega}{60}$$

Pour $D = 65\,mm$ et $\omega = 200\,RPM$ :

$$v = \frac{\pi \times 0.065 \times 200}{60} = 0.68\,m/s \approx 2.4\,km/h$$

Pour $D = 42\,mm$ :

$$v = \frac{\pi \times 0.042 \times 200}{60} = 0.44\,m/s \approx 1.6\,km/h$$

Ces vitesses sont adaptées au suivi de ligne (suffisamment lent pour que le système analogique corrige la trajectoire, suffisamment rapide pour un fonctionnement fluide).

**Matériau caoutchouc** : bonne adhérence sur les surfaces lisses typiques des pistes de test (PVC, papier cartonné).

---

## 16. Roue folle (ball caster)

| Critère | Valeur |
|---------|--------|
| **Référence** | Ball caster (bille castor) type Pololu |
| **Quantité** | 1 |
| **Rôle** | Point d'appui avant, libre en rotation |

### Justification

Le robot a **2 roues motrices à l'arrière**. Il faut un 3ᵉ point d'appui à l'avant pour la stabilité (configuration tricycle).

**Pourquoi un ball caster plutôt qu'une roulette ?**

| Critère | Ball caster | Roulette pivotante |
|---------|-------------|-------------------|
| Mouvement | **Omnidirectionnel** (360°) | 360° mais avec inertie de pivot |
| Frottement | Très faible | Moyen |
| Encombrement | Faible hauteur | Plus haut |
| Effet sur la trajectoire | **Aucune résistance directionnelle** | Peut résister au changement de cap |

Le ball caster est préféré car il **n'oppose aucune résistance directionnelle**, ce qui est critique pour un robot à commande différentielle qui tourne en coupant un moteur. Une roulette pivotante aurait tendance à résister aux changements de direction brusques.

---

## 17. Batterie LiPo 2S 7.4V

| Critère | Valeur |
|---------|--------|
| **Référence** | LiPo 2S, 7.4V nominal, 1000–2200 mAh |
| **Quantité** | 1 |
| **Rôle** | Source d'énergie principale |

### Justification

**Pourquoi LiPo 2S ?**

| Critère | LiPo 2S (7.4V) | 4×AA (6V) | 6×AA (9V) | 9V bloc |
|---------|----------------|-----------|-----------|---------|
| Tension | 7.4V (6.4–8.4V) | 6V (4.8–6.4V) | 9V (7.2–9.6V) | 9V (6–9V) |
| Capacité | 1000–2200 mAh | ~2500 mAh | ~2500 mAh | ~500 mAh |
| Courant max | **>5A** | ~1A | ~1A | **~200 mA** |
| Poids | ~50g (1500mAh) | ~100g | ~150g | ~45g |
| Rechargeable | Oui | Si NiMH | Si NiMH | Faible capacité |
| Taille | Compact | Encombrant | Très encombrant | Compact |

**La LiPo 2S est retenue car :**

1. **Tension compatible** avec le LM7805 : $V_{min} = 6.4\,V$ en fin de décharge, le dropout du LM7805 (2V) impose $V_{in} \geq 7\,V$. En pratique, on coupe l'utilisation à ~6.8V (seuil de sécurité LiPo), donc $V_{in} \approx 6.8–8.4\,V$ → ✓ dans la plage.

2. **Courant de décharge suffisant** : une LiPo 1500 mAh 25C peut fournir $1.5 \times 25 = 37.5\,A$ — largement au-dessus des ~850 mA nécessaires.

3. **Légèreté** : ~50g pour 1500 mAh, vs 100–150g pour des piles AA équivalentes. Le poids réduit améliore la réactivité du robot et diminue la consommation des moteurs.

4. **Autonomie estimée** :

$$t = \frac{C}{I_{total}} = \frac{1500\,mAh}{850\,mA} \approx 1.8\,h$$

Soit environ **1h30 à 2h** de fonctionnement continu — suffisant pour une séance de test et démonstration.

---

## Récapitulatif — Tableau de synthèse

| # | Composant | Référence | Qté | Critère déterminant |
|---|-----------|-----------|-----|---------------------|
| 1 | Capteur IR | TCRT5000 | 2 | Intégré, distance optimale 5mm, 0.50€ |
| 2 | R limitation LED | 220Ω | 2 | Calcul : $I_F = 17.3\,mA$ < 60 mA max |
| 3 | R pull-up photo | 10kΩ | 4 | Plage de sortie 3.7V, compromis sensibilité/vitesse |
| 4 | R pull-up LM393 | 4.7kΩ | 2 | Recommandation datasheet, $I_{sink} = 1\,mA$ |
| 5 | Potentiomètre | 10kΩ multitour | 2 | Résolution 1.4 mV/°, calibration fine |
| 6 | Comparateur | LM393 | 1 | Temps de réponse 1.3 µs, 2 canaux, collecteur ouvert |
| 7 | Pont en H | L293D | 1 | Diodes intégrées, 600 mA, protection thermique |
| 8 | Capa découplage | 100nF céramique | 4 | Standard HF, ESR faible |
| 9 | Capa filtrage | 100µF/16V | 2 | Réservoir de charge, appels de courant moteurs |
| 10 | Régulateur | LM7805 | 1 | Simple, 3 composants, bruit faible |
| 11 | Diodes | 1N4007 | 4 | $V_{RRM} = 1000\,V$, protection inductive (si pont discret) |
| 12 | LED témoin | Rouge 3mm | 2 | Debug visuel, 3 mA |
| 13 | R LED témoin | 1kΩ | 2 | $I_F = 3\,mA$, visibilité suffisante |
| 14 | Interrupteur | Glissière | 1 | État visuel ON/OFF, simple |
| 15 | Moteurs | TT jaune 1:48 | 2 | 200 RPM, 400 mA, bracket inclus, 1€ |
| 16 | Roues | Ø 65mm caoutchouc | 2 | $v \approx 0.68\,m/s$, bonne adhérence |
| 17 | Roue folle | Ball caster | 1 | Omnidirectionnel, sans résistance |
| 18 | Batterie | LiPo 2S 1500mAh | 1 | 7.4V, légère, autonomie ~1h45 |

---

*Document de justification technique — Projet "Le Suiveur de Ligne Pur Hard"*
