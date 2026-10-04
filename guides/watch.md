# Arborisis — MeshCore sur Apple Watch, sans iPhone

Une app Apple Watch **autonome** qui parle **directement** au **Heltec V3** en Bluetooth : lire et
écrire des messages LoRa, gérer contacts et canaux, voir les nœuds autour de soi sur un radar,
administrer les relais et piloter la radio. Sur le terrain il n'y a que **la montre et le Heltec** :
pas d'iPhone, pas de réseau, pas de serveur.

```
 [Apple Watch]  ── Bluetooth (Just Works, chiffré) ──  [Heltec V3]  ── LoRa ──  réseau MeshCore
 écran, GPS, boussole                                   relais + compagnon
 notifications, dictée                                  (+ USB en même temps)
```

La montre apporte ce qui manque au Heltec : un écran pour lire et écrire, **le GPS** (le Heltec
n'en a pas) et la boussole pour le radar.

> **Statut** : le firmware `ArborisisOS_HeltecV3_watch` compile ; l'app compile pour montre réelle
> (arm64 / arm64_32) et simulateur, sans avertissement. Elle a été vérifiée dans le simulateur contre
> une radio de démonstration intégrée qui produit les trames octet par octet comme
> `examples/companion_radio/MyMesh.cpp` (**84 vérifications de bout en bout, toutes réussies**, plus
> un parcours manuel des écrans et des complications sur le cadran). Les codes de région et le
> chiffrement des messages de canal sont vérifiés contre des valeurs calculées par le code du firmware
> et sa bibliothèque Crypto, compilés sur le Mac. **Elle n'a pas encore tourné sur une vraie montre avec un vrai
> Heltec** (le Bluetooth n'existe pas dans le simulateur) : voir « À vérifier sur l'appareil ».

---

## Pourquoi un firmware « watch »

Le firmware Heltec habituel exige un **code PIN à 6 chiffres** pour l'appairage Bluetooth. watchOS
ne sait pas saisir ce code pour un accessoire tiers : l'appairage échoue. Le firmware
`ArborisisOS_HeltecV3_watch` est identique au firmware principal (relais + compagnon Bluetooth +
USB, 350 contacts) mais appaire en **« Just Works »** : la liaison reste **chiffrée et mémorisée**
(bond), simplement sans code. C'est le même niveau de protection que le PIN par défaut `123456`.
L'appli MeshCore de l'iPhone ou d'Android fonctionne aussi avec ce firmware.

| Fichier (`release/`) | Usage |
|---|---|
| `ArborisisOS_HeltecV3_watch-v1.0.0-merged.bin` | Image complète, à flasher à l'adresse `0x0` |
| `ArborisisOS_HeltecV3_watch-v1.0.0.bin` | Même firmware, mise à jour seule (adresse `0x10000`) |

Compilation : `pio run -e ArborisisOS_HeltecV3_watch` (option `-D BLE_JUST_WORKS`, dans
`src/helpers/esp32/SerialBLEInterface.cpp`). Sur l'écran du Heltec, la page « Liaison » affiche
« Sans code PIN · Apple Watch prête » au lieu du PIN.

---

## Installation

### 1. Le Heltec V3

Comme pour les autres firmwares Heltec (voir [ARBORISIS-HELTEC-V3.md](ARBORISIS-HELTEC-V3.md)) :

```bash
esptool.py --chip esp32s3 write_flash 0x0 release/ArborisisOS_HeltecV3_watch-v1.0.0-merged.bin
```

Depuis un autre firmware Arborisis / MeshCore compagnon Heltec V3, l'identité, les contacts et les
canaux sont conservés ; `…watch-v1.0.0.bin` à `0x10000` suffit aussi.

### 2. L'app sur la montre

À faire **une seule fois** (l'iPhone ne sert qu'à cette étape, parce que la montre lui est jumelée) :

1. Sur le Mac, **Xcode** › Réglages › Comptes : ajoutez votre identifiant Apple (un compte gratuit
   suffit).
2. Branchez l'**iPhone** jumelé à la montre au Mac (ou même Wi‑Fi), déverrouillé.
3. Ouvrez Xcode › Window › **Devices and Simulators** : la montre apparaît sous l'iPhone ; laissez
   Xcode la préparer.
4. Sur la montre : Réglages › Confidentialité et sécurité › **Mode développeur** › activer
   (la montre redémarre).

Puis, à chaque installation :

```bash
watch/install_watch.sh
```

Le script compile l'app, l'enregistre sur votre compte et l'installe sur la montre. On peut aussi
ouvrir `watch/ArborisisWatch.xcodeproj` dans Xcode, choisir la montre comme destination et ▶︎.

> **Compte Apple gratuit** : Apple limite la validité des apps installées ainsi à **7 jours** ;
> relancez `install_watch.sh` pour la prolonger (les messages et réglages sont conservés). Avec un
> compte développeur payant, c'est un an (changez `TEAM` / `DEVELOPMENT_TEAM`).

### 3. Première connexion

Ouvrez **Arborisis** sur la montre. Elle cherche les radios MeshCore à portée : touchez votre Heltec.
Acceptez la demande Bluetooth et la localisation. Ensuite la montre se **reconnecte seule** dès que
le Heltec est à portée.

Le Heltec n'accepte **qu'une liaison Bluetooth à la fois** : si l'iPhone est connecté à l'appli
MeshCore, déconnectez-le. L'USB reste utilisable en même temps.

Pas de radio sous la main ? **Radio Bluetooth › Essayer sans radio (démo)** : une radio simulée,
avec contacts, canaux, messages et relais, pour découvrir l'app (ses données restent à part).

---

## Ce que fait l'app

**Accueil** : état de la liaison et de la batterie du Heltec, région, messages non lus, accès à tout,
bouton **SOS**.

**Messages**
- conversations privées, canaux, salons (room servers), triées par activité, non lus, masquer ;
- écrire par **dictée**, **griffonnage** ou clavier, **réponses rapides** (modifiables), émojis,
  **« Envoyer ma position »** (GPS de la montre) ; compteur d'octets (140 max, comme le firmware) ;
- **accusés de réception** : ✓ envoyé, ✓ vert + temps aller-retour quand le destinataire confirme ;
  « Échec · toucher » pour renvoyer à la main ;
- **renvoi automatique** (réglable) :
  - message privé : 3 essais, le dernier en inondation après avoir oublié la route ;
  - **hors de portée de la radio** : le message attend (« En attente de la radio ») et part tout seul
    à la reconnexion, dans l'ordre d'écriture (abandonné au bout de 24 h) ;
  - **contact injoignable** : dès qu'il réapparaît (advert, nouvelle route, message de sa part), les
    messages en échec des 12 dernières heures sont renvoyés (2 fois au plus) ;
  - **canaux** : un canal n'a pas d'accusé ; la montre reconnaît son propre message quand un relais le
    répète (« relayé ×2 »). Si aucun relais ne le répète, elle le renvoie (3 essais, à l'identique :
    ceux qui l'ont déjà reçu ignorent la copie), puis affiche « aucun relais · toucher » ;
- messages reçus avec le nombre de sauts et le SNR ; expéditeur affiché dans les canaux et salons ;
- **Double tap** (Series 9 / Ultra 2) : ouvre « Écrire » dans une conversation.

**Notifications** : message reçu → notification sur la montre, avec les 3 premières réponses rapides
en boutons (on répond sans ouvrir l'app). Vibration quand l'app est ouverte. Réglables (canaux
inclus ou non).

**Contacts** : compagnons, relais, salons, capteurs ; favoris ; route (direct / n sauts / inondation)
et chemin ; dernier advert ; **distance et direction** depuis la montre ; ajout des nœuds entendus
quand l'ajout automatique est coupé. Actions : message, **découverte de chemin**, **trace** (ping
d'un relais avec le SNR de chaque saut), **télémétrie** (batterie, température, humidité, pression…),
réinitialiser la route, partager le contact (0 saut), supprimer.

**Administration des relais / salons / capteurs** : connexion (mot de passe admin ou invité), **état**
(batterie, uptime, bruit, RSSI/SNR, paquets, temps d'antenne, file, doublons), **console** : commandes
CLI (`ver`, `neighbors`, `get radio`, `advert`, `reboot`…) et leurs réponses.

**Canaux** : public, **#hashtag** (la clé se déduit du nom : tous ceux qui tapent le même #nom se
retrouvent), privé avec clé (aléatoire ou saisie en hex) ; voir la clé pour la partager ; **région**
du canal ; quitter.

**Régions** (MeshCore 1.10 et plus) : un message en inondation peut porter une **région** ; chaque
relais choisit les régions qu'il relaie. Un message « grenoble » reste autour de Grenoble au lieu
d'inonder tout le réseau, ce qui soulage les relais et le temps d'antenne.
- **Région de la radio** : celle que le Heltec met sur tout ce qu'il inonde (messages, adverts),
  gardée par le Heltec même après redémarrage ; « Aucune » = tout le réseau ;
- **par conversation** : un canal ou un contact peut utiliser une autre région, ou aucune
  (« Comme la radio » par défaut) ; la région utilisée s'affiche sous « Écrire » ;
- **mes régions** : liste sur la montre ; même nom = même région pour tout le monde (la clé est
  SHA-256(« #nom »)) ; lettres, chiffres et tirets, sans espace, majuscules comprises ;
- **entendu par la radio** : les paquets que le Heltec reçoit portent le code de leur région ; la
  montre les range par région (barres), « sans région » et « région inconnue » ;
- **demander aux relais** : les relais qui vous entendent en direct disent quelles régions ils
  relaient (sans mot de passe) ; les régions inconnues deviennent des suggestions, et le trafic
  « inconnu » est reclassé. Aussi depuis la fiche d'un relais (il faut une route vers lui) ;
- **administrer les régions d'un relais** (connecté en admin) : arbre des régions, interrupteur
  « relaie cette région » par région (et `*` = paquets sans région), ajouter dans une région parente,
  supprimer, région par défaut, région « maison », puis **Enregistrer sur le relais**.

**Autour de moi**
- **Radar** : les contacts positionnés autour de vous, orienté par la **boussole** (ou nord en
  haut), zoom à la **Digital Crown** (250 m → 100 km), flèches au bord pour ceux hors champ ;
- **Carte** (fond de carte si la montre a du réseau) ;
- **Relais à portée** : demande de découverte, les relais qui vous entendent en direct répondent
  avec le SNR ; ajout en un geste ;
- **Paquets** : les 200 derniers paquets reçus par le Heltec, pour vous ou pour les autres : type
  (message, canal, advert, accusé…), qui l'envoie quand c'est lisible (nom d'advert, canal connu),
  région, sauts, dernier relais, SNR / RSSI ;
- **Entendus** : adverts reçus pendant la session.

**Radio (le Heltec)**
- advert aux voisins ou à tout le réseau ;
- **position de la montre → radio** (et option « position auto » toutes les 5 à 120 min, suivie d'un
  advert) ;
- nom, partage de la position dans l'advert, ajout manuel des contacts, télémétrie ;
- préréglages radio (EU/UK Narrow, EU/UK Long, USA/Canada) ou fréquence / bande / SF / CR à la main,
  puissance d'émission ;
- **relais** on/off ;
- **identifiant des sauts** (1, 2 ou 3 octets) ;
- statistiques (uptime, bruit de fond, RSSI/SNR, temps d'émission/réception, paquets), capteurs du
  Heltec, horloge (mise à l'heure depuis la montre à chaque connexion), firmware, redémarrage.

**Complications** (cadran et pile intelligente), **mises à jour en direct** :
- **Messages LoRa** : nombre de non lus, dernier expéditeur, texte et depuis quand (le temps défile
  sur le cadran) ; toucher ouvre directement la conversation non lue ; avec des non lus, elle remonte
  en tête de la pile intelligente ;
- **Radio Heltec** : liaison (point vert / rouge), batterie du Heltec en jauge, région ;
- **SOS LoRa** : ouvre la confirmation de SOS.

Rond, coin, rectangle et ligne. L'app les actualise à chaque message reçu ou lu, connexion ou
déconnexion, changement de région ou de batterie (et au moins toutes les 10 min) ; si l'app est
suspendue plus de 30 min, la complication cesse d'afficher « connecté ». Réglages › Complications :
aperçu, et option pour masquer le texte des messages sur le cadran.

**SOS** : message configurable + position GPS, vers le canal ou le contact de votre choix, après
confirmation.

**Hors connexion** : contacts, canaux et conversations sont gardés sur la montre (par radio) ; les
messages arrivés pendant l'absence sont récupérés à la reconnexion (file du Heltec).

---

## En arrière-plan, batterie

L'app déclare le mode Bluetooth en arrière-plan (`bluetooth-central`) : la liaison reste ouverte
quand l'écran s'éteint, et les messages arrivent en notification. watchOS peut malgré tout suspendre
l'app après un long moment ; en la rouvrant, elle se reconnecte et récupère les messages en attente
sur le Heltec (jusqu'à 256). Le radar et la carte n'utilisent le GPS que lorsqu'ils sont affichés.

---

## À vérifier sur l'appareil

Ce qui ne peut pas être testé dans le simulateur :

- l'appairage **Just Works** entre watchOS et le Heltec (demande de jumelage sur la montre, puis
  reconnexion automatique) ;
- la réception des notifications quand l'écran est éteint (dépend de la gestion d'énergie de
  watchOS) ;
- la taille de trame négociée (MTU) : affichée dans Radio › Firmware ;
- la fréquence de rafraîchissement des complications quand l'app tourne en arrière-plan (watchOS
  limite les rechargements) ;
- les régions face à de vrais relais (MeshCore 1.10+ ; ceux d'avant ignorent simplement les régions).

Si l'appairage échoue : sur le Heltec, vérifiez que la page « Liaison » dit bien « Sans code PIN » ;
sur la montre, Réglages › Bluetooth › oubliez l'appareil s'il y figure, puis reconnectez depuis
l'app. Radio › Firmware et Réglages › Avancé › **Journal** montrent les trames échangées.

---

## Pour les développeurs

```
watch/
  ArborisisWatch.xcodeproj
  ArborisisWatch/
    Protocol/   Frames.swift (codes, commandes, décodage), Bytes.swift, LPP.swift (télémétrie),
                GroupCipher.swift (reconstitue nos messages de canal pour reconnaître leur écho)
    BLE/        BLEManager.swift (Nordic UART), DemoRadio.swift (radio simulée, régions comprises)
    Model/      MeshService.swift (file de requêtes, synchro, accusés, renvois),
                MeshService+Regions.swift, Regions.swift (clés, codes de transport, paquets), Models, Store
    Services/   LocationService (GPS, boussole), Notifier (notifications, vibrations)
    Views/      écrans SwiftUI
    App/        point d'entrée, ComplicationSync.swift, SelfTest.swift (debug)
  ArborisisComplication/   complications WidgetKit
  Shared/                  compilé dans l'app et l'extension : état du cadran, vues des complications
  install_watch.sh
```

- Protocole compagnon MeshCore v3 (`docs/companion_protocol.md`) : une trame par écriture GATT
  (RX `6E400002…`, avec réponse) et par notification (TX `6E400003…`). Une requête à la fois ; les
  « push » (≥ `0x80`) arrivent à tout moment.
- watchOS 10 minimum, SwiftUI, aucune dépendance.
- Régions : clé = SHA-256(« #nom »)[0..16] ; code de transport = HMAC-SHA256(clé, type ‖ charge)[0..2]
  (`src/helpers/TransportKeyStore.cpp`). Commandes : `CMD_SET_FLOOD_SCOPE_KEY` (54) pour la région
  d'un envoi, `CMD_SET/GET_DEFAULT_FLOOD_SCOPE` (63/64) pour celle de la radio, `CMD_SEND_ANON_REQ`
  (57, type 0x01) pour demander ses régions à un relais ; `PUSH_CODE_LOG_RX_DATA` (0x88) donne les
  paquets entendus. Chaque envoi en inondation fixe la région de session sous un verrou, pour qu'une
  région ne déborde jamais sur un autre message.
- Complications : un compte développeur gratuit n'a pas d'App Group ; l'état passe par un élément du
  **trousseau partagé** (`keychain-access-groups` = `$(AppIdentifierPrefix)com.bastienjavaux.arborisiswatch.shared`,
  autorisé par tous les profils d'équipe), puis `WidgetCenter.reloadAllTimelines()`.
- Tests de bout en bout dans le simulateur :

```bash
cd watch
# signé « pour exécution locale » (sans compte) : il faut les droits du trousseau partagé
xcodebuild -project ArborisisWatch.xcodeproj -scheme ArborisisWatch \
  -destination 'platform=watchOS Simulator,name=Apple Watch Series 12 (46mm)' -derivedDataPath build/sim build
xcrun simctl install booted build/sim/Build/Products/Debug-watchsimulator/Arborisis.app
xcrun simctl location booted set 45.1885,5.7245                                       # le test partage une position
xcrun simctl launch --console-pty booted com.bastienjavaux.arborisiswatch -selftest   # → SELFTEST DONE pass=84 fail=0
```

`-demo` lance directement la radio de démonstration (relais qui ne relaient que certaines régions,
trafic dans plusieurs régions dont une inconnue, écho des messages de canal, console `region`).
En debug, `-open regions` (ou `packets`, `radio`, `messages`, `unread`, `complications`) ouvre un
écran au lancement, comme les liens `arborisis://` des complications.
