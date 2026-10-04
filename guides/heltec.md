# Arborisis OS — Heltec V3 : relais + compagnon

Pour un appareil fixe reconnu comme répéteur MeshCore et administrable à distance, voir la
[version répéteur dédiée Heltec V3](ARBORISIS-REPEATER-HELTEC-V3.md).

Firmware MeshCore (base `v1.17.1`) pour la **Heltec WiFi LoRa 32 V3** (ESP32-S3 + SX1262 +
OLED 0,96" 128×64 + bouton PRG). Le boîtier fait **deux choses en même temps** :

- **relais (répéteur)** : il retransmet les paquets du réseau MeshCore, avec les mêmes garde-fous
  qu'un répéteur dédié (limite de sauts, détection de boucles, budget d'émission) ;
- **compagnon** : l'appli MeshCore s'y connecte par **Bluetooth et par USB**, simultanément.

L'écran est pensé pour le **boîtier à l'horizontale** (128×64 en paysage, rotation 180° possible),
avec une interface complète pilotée par le seul bouton PRG. Pas de GPS sur cette carte.

> **Statut** : compilé et testé dans un simulateur (32 vérifications automatiques + 34 captures de
> tous les écrans), **pas encore testé sur le matériel**. Voir « À vérifier sur l'appareil ».

---

## Firmwares

Produits par `tools/release.sh` dans `release/` :

| Fichier | Usage |
|---|---|
| `ArborisisOS_HeltecV3-v1.0.0-merged.bin` | **Principal** : relais + appli en Bluetooth **et** USB. Image complète (adresse `0x0`) |
| `ArborisisOS_HeltecV3-v1.0.0.bin` | Même firmware, mise à jour seule (adresse `0x10000`) |
| `ArborisisOS_HeltecV3_usb-v1.0.0-merged.bin` | Relais + appli en **USB seulement** (Bluetooth jamais démarré) |
| `ArborisisOS_HeltecV3_usb-v1.0.0.bin` | Même firmware, mise à jour seule |
| `ArborisisOS_HeltecV3_watch-v1.0.0-merged.bin` | Pour l'**Apple Watch** : Bluetooth sans PIN (Just Works), voir [ARBORISIS-WATCH.md](ARBORISIS-WATCH.md) |
| `ArborisisOS_HeltecV3_watch-v1.0.0.bin` | Même firmware, mise à jour seule |

Compilation : `pio run -e ArborisisOS_HeltecV3` (ou `ArborisisOS_HeltecV3_usb`).

## Auto-calibration et CAD en mode normal

Les versions compagnons **Bluetooth + USB, USB, Watch et Mio** activent automatiquement
la calibration du SX1262 et le CAD. Cet ajout concerne le mode normal, indépendamment
de l'option de retransmission des paquets ; la version répéteur dédiée utilise son propre pilote.

- Calibration du récepteur après environ **15 secondes au démarrage**, puis **toutes les
  15 minutes**, et après un changement de fréquence, BW/SF/CR ou gain RX.
- Mesure du bruit sur **64 échantillons espacés de 50 ms**, uniquement en réception calme,
  sans préambule ni paquet/interrupt en attente. Une mesure instable est rejetée ; une
  tentative échouée reprend après une minute et conserve le dernier bruit validé.
- Restauration de la calibration d'image pour la bande configurée et du gain RX après
  chaque rafraîchissement. La calibration ne change ni la fréquence choisie ni la puissance TX.
- Avant émission, détection d'activité LoRa par **CAD matériel**, avec un seuil RSSI de
  **bruit + 12 dB** une fois le bruit validé. Un canal occupé ou une erreur CAD garde le
  message en file ; l'émission reprend quand le canal est libre, sans émission forcée
  après le délai de quatre secondes du firmware de base.

Le CAD cherche l'activité LoRa avec les paramètres radio en cours ; il ne garantit pas
l'absence de collisions. Voir la [documentation SX1262 de Semtech](https://www.semtech.com/products/wireless-rf/lora-connect/sx1262).
La page **Radio** affiche `Calib... CAD actif`, puis `Bruit … Auto CAD`.

Vérification de la calibration et de la file CAD sur le code de production :
`sh tools/test-heltec-companion.sh`. Ces tests sur ordinateur et la compilation ne prouvent
pas la réception/émission RF réelle : celle-ci reste à vérifier sur l'Heltec et un second nœud.

## Installation

Branchez la carte en USB. Si elle n'est pas reconnue, maintenez **PRG**, appuyez sur **RST**, puis
relâchez PRG (mode téléchargement).

**Depuis un navigateur** (Chrome/Edge) : <https://espressif.github.io/esptool-js/>, *Connect*, puis
fichier `…-merged.bin` à l'adresse `0x0`, *Program*.

**En ligne de commande** :

```bash
esptool.py --chip esp32s3 write_flash 0x0 ArborisisOS_HeltecV3-v1.0.0-merged.bin
```

- Depuis un autre firmware MeshCore compagnon Heltec V3 : l'identité, les contacts et les canaux
  sont conservés.
- Depuis Meshtastic ou un firmware inconnu : effacez d'abord la mémoire
  (`esptool.py --chip esp32s3 erase_flash`), puis flashez l'image complète.
- Mise à jour d'Arborisis : le `.bin` simple à `0x10000` suffit.

---

## Le bouton PRG

Tout se fait avec un seul bouton ; **c'est la durée de l'appui qui compte**. Pendant l'appui, une
jauge en bas de l'écran montre ce qui va se passer au relâchement :

| Appui | Action |
|---|---|
| **court** | suivant : page suivante, ligne suivante, défilement |
| **long** (≈ 0,5 s, jauge « » … ») | ouvrir / valider / changer la valeur |
| **très long** (≈ 1,5 s, jauge « « … ») | retour ; sur une page → Accueil ; sur l'Accueil → éteindre l'écran |

Quand l'écran est éteint, le premier appui ne fait que le rallumer. Chaque liste se termine par
une ligne « « Retour » pour qui ne connaît pas encore l'appui très long.

## Les pages

Un appui court fait défiler les pages en boucle ; la barre épaisse sous le titre indique la page.
En haut à droite : messages non lus, mât du **relais** (clignote en inversé à chaque paquet
relayé), **Bluetooth** (plein = appli connectée), **USB** (appli active en USB), batterie ou prise.

| Page | Contenu | Appui long |
|---|---|---|
| **Accueil** | grande horloge, date, activité du relais sur la dernière heure, appli connectée | actions rapides |
| **Messages** | les deux dernières discussions, non lus | liste des discussions |
| **Relais** | état, paquets relayés/heure, **histogramme de la dernière heure** (barres de 2 min), relayés flood/direct, temps d'émission, paquets refusés | activer / arrêter le relais |
| **Voisins** | nœuds reçus **en direct** (portée radio) avec barres de signal et âge, + nombre de nœuds lointains | liste complète (SNR, sauts, détails) |
| **Radio** | fréquence, BW/SF/CR, puissance, gain RX, bruit, dernier paquet (RSSI/SNR) | réglages |
| **Connexion** | Bluetooth (visible / connecté / coupé) et **PIN en grand**, état USB | couper / activer le Bluetooth |
| **Système** | batterie (% et tension) ou alimentation USB, temps de fonctionnement, contacts, canaux, versions | actions rapides |
| **Réglages** | aperçu des réglages principaux | ouvrir les réglages |

**Actions rapides** : advert (voisins), advert (réseau), arrêter/activer le relais,
couper/activer le Bluetooth, éteindre l'écran, redémarrer, veille profonde.

### Messages

L'appareil garde son propre historique (100 messages, sauvegardé en flash), y compris ce que
l'appli envoie. Un message privé reçu **quand aucune appli n'est connectée** allume l'écran et
s'affiche en entier (appui long = ouvrir, appui court = fermer) ; la LED blanche clignote tant que
des messages restent non lus.

Dans une discussion : appui court = remonter dans l'historique (puis retour au plus récent), appui
long = **répondre** avec une réponse rapide (OK, Oui, Non, Bien reçu, J'arrive, Rappelle-moi, Tout
va bien, Besoin d'aide !), marquer comme lu, effacer le fil. Vos messages sont précédés de `»` et
suivis de `○` (en attente), `✓` (reçu) ou `!` (échec, après 3 essais dont le dernier en inondation).

---

## Le relais

Le relais est **activé par défaut**. Il retransmet :

- les paquets **inondés** (flood), sauf s'ils dépassent la limite de sauts ou si le boîtier figure
  déjà trop de fois dans leur chemin (boucle) ;
- les paquets **routés** (direct) dont il est le prochain saut : les chemins appris à travers lui
  fonctionnent comme avec un répéteur.

Réglages › Relais :

| Réglage | Valeurs | Défaut |
|---|---|---|
| Relais | actif / arrêté | actif |
| Sauts max | 2 … 16, illimité | illimité (64) |
| Sauts adverts | jamais, 1 … 16, illimité | 8 (comme un répéteur) |
| Anti-boucle | non, légère, modérée, stricte | modérée |
| Émission max | 50 %, 33 %, 20 %, 10 % du temps | 50 % |
| Remettre à zéro | compteurs du relais | |

L'interrupteur « répéter » de l'appli MeshCore pilote le même réglage, **sur toutes les
fréquences** (le compagnon d'origine le limite à trois canaux hors réseau). Une appli ancienne qui
ne connaît pas cette option ne désactive plus le relais en changeant la radio.

À savoir :

- Le boîtier s'annonce comme un **compagnon** (son identité de chat), pas comme un répéteur : il
  n'apparaît pas dans la liste des répéteurs de l'appli et ne propose pas d'administration à
  distance (connexion admin, statistiques distantes).
- En Europe, la bande 869,4–869,65 MHz autorise 10 % d'émission : réglez « Émission max » sur
  **10 %** pour un relais fixe très sollicité.
- Un relais doit rester allumé : pensez à l'alimentation USB. Sur secteur, l'extinction pour
  batterie faible est désactivée.

## Réglages

- **Relais** : voir ci-dessus.
- **Radio** : préréglage (EU Narrow 869,618 MHz / 62,5 kHz / SF8 / CR8, EU Long 869,525 / 250 /
  SF11, USA/CA 910,525 / 62,5 / SF7), puissance 2–22 dBm, gain RX (boost / éco), advert
  automatique (non, 3, 6, 12, 24 h, en inondation), envoyer un advert.
- **Connexion** : Bluetooth oui/non, PIN (affiché ; un PIN fixe se règle depuis l'appli).
- **Écran** : veille (10 s à 5 min, ou jamais ; l'horloge se décale d'un pixel chaque minute
  pour ménager l'OLED), luminosité 1–5, **orientation normale / 180°**
  (carte montée à l'envers dans le boîtier), réveil sur message (privés / tous / jamais), LED non lus.
- **Système** : fuseau (Paris, Londres, Athènes avec heure d'été automatique, ou UTC±N), effacer
  l'historique, redémarrer, veille profonde.

L'heure s'affiche dès qu'une appli l'a réglée (la carte n'a pas d'horloge sauvegardée).

**Veille profonde** : tout est coupé (radio, écran, Vext), le relais et l'appli aussi. Un appui sur
**PRG** redémarre la carte.

---

## À vérifier sur l'appareil

- Orientation réelle de l'écran dans votre boîtier (réglage *Orientation*).
- Bluetooth et USB en même temps avec l'appli MeshCore (l'USB ne signale pas la connexion : il est
  considéré « actif » tant que l'appli a parlé dans les 5 dernières minutes).
- Détection de l'alimentation USB (déduite de la tension > 4,27 V, la carte n'a pas de détection VBUS).
- Consommation : l'ESP32-S3 reste éveillé (relais + Bluetooth), compter plusieurs dizaines de mA.

## Simulateur

`tools/sim-heltec/build.sh` compile la vraie interface contre des bouchons, pilote le bouton PRG,
vérifie le comportement et écrit une capture de chaque écran dans `tools/sim-heltec/out/`
(nécessite d'avoir compilé le firmware une fois, pour les polices Adafruit GFX).
