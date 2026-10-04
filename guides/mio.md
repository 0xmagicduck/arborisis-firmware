# Arborisis — MeshCore sur Mio Cyclo Discover Connect

Transformer un GPS vélo **Mio Cyclo Discover Connect** en écran de contrôle MeshCore : lire et
répondre aux messages LoRa, voir les relais autour de soi avec leur distance et leur direction,
sur un radar, et piloter la radio.

Le Mio n'a pas de radio LoRa : il parle en **Wi‑Fi** à un **Heltec V3** qui fait la radio (relais +
compagnon, voir [ARBORISIS-HELTEC-V3.md](ARBORISIS-HELTEC-V3.md)). Le Heltec crée son propre réseau
Wi‑Fi, donc aucune box ni téléphone n'est nécessaire sur le vélo.

```
 [Mio Cyclo]  ── Wi‑Fi « Arborisis-XXXX » ──  [Heltec V3]  ── LoRa ──  réseau MeshCore
 écran tactile, GPS                            relais + compagnon
                                               (+ appli téléphone en Bluetooth, en même temps)
```

> **Statut** : le firmware Heltec tourne sur la carte (réseau Wi‑Fi visible, identité conservée).
> L'appli tourne dans un simulateur sur Mac (43 vérifications de bout en bout contre
> une fausse radio + 58 tests unitaires, 20 captures d'écran). **Elle n'a pas encore tourné sur le
> vrai Mio** : voir « À vérifier sur l'appareil ».

---

## Ce qu'il y a dans le Mio

Analyse faite sur l'appareil branché en USB :

| | |
|---|---|
| Système | Windows CE 5.0, processeur ARM, environ 112 Mo de RAM |
| Écran | 320×480 tactile |
| Appli d'origine | `\SystemDrive\Dodge\Program\dodge.exe` (navigation Mio) |
| GPS | port série `COM8:` (NMEA) |
| Wi‑Fi | géré par `WIFI\MioWiFi.exe` (Qt4 + Windows Zero Config) |
| Partitions USB | `NO NAME` (200 Mo, traces) et `NO NAME 1` (7,8 Go, système `\SystemDrive`) |

On ne remplace pas le système du Mio : on ajoute un programme Windows CE (`MeshCore.exe`) et un
petit **lanceur** (`ArboLaunch.exe`) qui prend la place d'un programme du Mio et propose
« MeshCore » ou le programme d'origine.

## Les deux façons de l'installer

| Mode | Comment on ouvre MeshCore | Risque |
|---|---|---|
| **Menu Wi‑Fi** (par défaut) | menu du Mio › Wi‑Fi › « MeshCore » | très faible : seul le menu Wi‑Fi passe par le lanceur |
| **Au démarrage** (`--boot`) | écran de choix à l'allumage | faible : sans toucher l'écran, la navigation Mio démarre seule après 6 s |

Dans les deux cas l'appli d'origine est **renommée, jamais effacée** (`MioWiFi_orig.exe`,
`dodge_mio.exe`), une copie est gardée sur l'ordinateur, et `--uninstall` remet tout comme avant.

---

## Installation

### 1. Le Heltec V3

Firmware `release/ArborisisOS_HeltecV3_mio-v1.0.0-merged.bin` (image complète à l'adresse `0x0`),
ou `pio run -e ArborisisOS_HeltecV3_mio`. C'est le firmware Heltec habituel (relais + compagnon
Bluetooth + USB) avec en plus :

- un **point d'accès Wi‑Fi** `Arborisis-XXXX` (fin de l'adresse MAC, par exemple `Arborisis-FF98`),
  mot de passe `arborisis`, puissance réduite (le Mio est à un mètre) ;
- pour laisser de la mémoire au Wi‑Fi et au Bluetooth ensemble : **160 contacts** au plus (350 sur
  les autres firmwares) et 48 messages en attente pour l'appli ;
- le protocole compagnon MeshCore en TCP sur `192.168.4.1:5000`.

La page « Liaison » de l'écran du Heltec affiche le nom du réseau, puis « Wi‑Fi : Mio connecté ».

### 2. Le Mio

Branchez le Mio en USB, puis :

```bash
make -C mio ce          # compile MeshCore.exe et ArboLaunch.exe (Docker, voir plus bas)
mio/tools/install_mio.sh            # mode menu Wi‑Fi
mio/tools/install_mio.sh --boot     # ou : aussi au démarrage
```

Depuis `release/ArborisisOS_Mio-v1.0.0/`, `./install_mio.sh` fonctionne sans rien compiler.

Le script trouve la partition du Mio, sauvegarde les programmes d'origine dans
`~/MioBackup/programmes-origine`, crée `\SystemDrive\MeshCore\` et installe le lanceur.
Éjectez le Mio avant de le débrancher.

### 3. Relier le Mio au Heltec (une seule fois)

Menu du Mio › Wi‑Fi › **Wi‑Fi du Mio** (l'appli Wi‑Fi d'origine) : choisissez le réseau
`Arborisis-XXXX`, mot de passe `arborisis`. Ensuite, menu Wi‑Fi › **MeshCore**.

---

## L'appli

Quatre onglets en bas de l'écran, gros boutons pour les doigts :

| Onglet | Contenu |
|---|---|
| **Messages** | canaux et conversations, non lus, aperçu. « Écrire à un contact » pour en démarrer une |
| **Relais** | relais, contacts ou tous : distance et flèche de direction, sauts, dernier advert. Toucher = fiche détaillée avec boussole |
| **Radar** | relais (ou tous les nœuds) autour de vous, zoom 500 m à 200 km, nord en haut ou sens de la marche |
| **État** | liaison radio, batterie du Heltec, radio (fréquence, SF), GPS, batterie du Mio, actions et réglages |

**Répondre** : clavier AZERTY tactile avec majuscules automatiques, chiffres et une page d'accents
(é è ê à ç œ « »…), compteur de caractères. **Rapide** : réponses toutes faites en un appui, plus
« Envoyer ma position » (coordonnées GPS du Mio). Les réponses se changent dans
`MeshCore\reponses.txt` (une par ligne).

Suivi des messages privés : « envoi… », « envoyé, en attente », « reçu » (accusé de réception du
destinataire). Sans accusé, l'appli renvoie jusqu'à 3 fois comme l'appli téléphone ; un message en
échec se renvoie d'un appui.

Le GPS du Mio sert à tout ce qui est distance et direction. Le Heltec n'ayant pas de GPS ni
d'horloge, le Mio lui donne l'heure à la connexion, et peut lui donner sa position :

- **Partager ma position** (onglet État) : met la position du Mio dans le Heltec et envoie un advert
  sur tout le réseau ;
- **Position auto** (réglage, désactivé par défaut) : garde la position du Heltec à jour sans rien
  émettre ; elle partira avec le prochain advert.

Autres réglages : mode nuit (écran sombre), son des notifications. Les messages et les contacts
connus sont enregistrés sur le Mio, l'historique reste lisible sans la radio.

Le bouton **Ouvrir la navigation Mio** bascule vers l'appli GPS d'origine ; on revient à MeshCore
par le menu Wi‑Fi (ou tout seul quand la navigation se ferme). Pendant ce temps le GPS est laissé à
la navigation. **Mode USB (PC)** lance l'écran USB du Mio pour le brancher à l'ordinateur.

### Fichiers sur le Mio (`\SystemDrive\MeshCore\`)

| Fichier | Rôle |
|---|---|
| `meshcore.ini` | adresse du Heltec (`host`, `port`), port GPS (`gps_port=COM8:`), réglages |
| `reponses.txt` | réponses rapides |
| `messages.txt`, `contacts.txt` | historique et derniers nœuds connus |
| `launcher.ini` | mode démarrage : `boot_meshcore=1` pour lancer MeshCore directement, `boot_delay` |
| `meshcore.log`, `launcher.log` | journaux, à regarder en cas de souci |

---

## À vérifier sur l'appareil

Tout ce qui suit n'a pas pu être testé sans le matériel :

1. **Le lanceur démarre** depuis le menu Wi‑Fi (`launcher.log` note la version de Windows CE, la
   taille d'écran et les arguments reçus de l'appli Mio).
2. **Le Wi‑Fi reste connecté** une fois `MioWiFi` fermé. Si l'appli Wi‑Fi d'origine coupe le Wi‑Fi
   en sortant, l'onglet État le montrera (« Wi‑Fi sans adresse ») : il faudra que MeshCore allume
   lui-même le Wi‑Fi (Windows Zero Config).
3. **Le GPS sur `COM8:`** quand la navigation Mio ne tourne pas, et s'il peut être partagé quand
   elle tourne (sinon « port indisponible », on garde la dernière position).
4. ~~Mémoire du Heltec~~ : vérifié sur la carte. Avec 350 contacts il ne restait que 8 Ko et le
   Heltec plantait ; avec 160 contacts, 80 Ko restent libres, Wi‑Fi et Bluetooth démarrent tous les
   deux. Diagnostic au démarrage sur l'USB : compiler avec `-DWIFI_AP_DIAG`.
5. **Consommation** : le Wi‑Fi allumé des deux côtés réduit l'autonomie du Mio (10 h annoncées) et
   du Heltec ; l'appli empêche la mise en veille du Mio pendant qu'elle est affichée.
6. Le son des notifications (bip généré par l'appli) et le mode démarrage (`--boot`).

---

## Pour les développeurs

Code dans [`mio/`](mio/) :

| Dossier | Contenu |
|---|---|
| `src/core/` | protocole compagnon (trames TCP `<`/`>` + longueur), client (file de commandes, synchro, accusés, renvois, reconnexion), modèle et sauvegarde, NMEA, géo, UTF‑8 |
| `src/gfx/` | rendu logiciel RGB565 anticrénelé, polices Atkinson Hyperlegible (bitmap 4 bits) |
| `src/ui/` | l'appli : écrans, clavier, radar (mode immédiat, zones tactiles) |
| `src/platform/wince/` | Windows CE : fenêtre plein écran, DIB 16 bits, Winsock, port série, sons |
| `src/platform/host/` | Mac/Linux : simulateur qui pilote l'appli et fait des captures |
| `src/launcher/` | `ArboLaunch.exe` |
| `tools/` | fausse radio (`mock_companion.py`), police, installation |

```bash
make -C mio test     # tests unitaires + simulateur contre la fausse radio, captures dans mio/build/sim_out/
make -C mio ce       # exécutables Windows CE
```

La chaîne de compilation Windows CE (GCC 9.3 `arm-mingw32ce`, projet CeGCC) se construit une fois
dans Docker (`mio/toolchain/Dockerfile`, une vingtaine de minutes). Les exécutables n'importent que
`COREDLL.dll` et `WS2.dll`, présents sur le Mio.

Côté firmware : `WIFI_AP_SSID` / `WIFI_AP_PWD` / `WIFI_AP_TX_POWER` dans
`examples/companion_radio/main.cpp` (point d'accès au lieu de rejoindre un réseau), et
`SerialWifiInterface` signale maintenant quand sa file d'envoi est pleine, pour ne pas perdre de
contacts pendant la synchronisation.
