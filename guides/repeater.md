# Arborisis OS — Répéteur Heltec V3

Version **Arborisis Repeater 1.2.0** dédiée à la **Heltec WiFi LoRa 32 V3**, ESP32-S3,
SX1262, OLED 128×64 et bouton PRG. Calibration automatique du récepteur et du bruit,
CAD matériel avant TX, délai adaptatif, diagnostics et historique de trafic sont intégrés.
Nouveau en 1.2.0 : le répéteur **observe le réseau pour la carte** map.arborisis.com, par USB
(pont observateur, HTTPS ou MQTT) ou, sans Internet, en **mode sentinelle** avec ACK
([Observateur pour la carte](#observateur-pour-la-carte)).
Elle utilise le moteur répéteur MeshCore `v1.17.1` : annonces de type répéteur, routage flood/direct,
anti-boucle, régions, découverte des voisins, connexion administrateur, console et statistiques
à distance. Le téléphone administre ce répéteur **par le mesh**, depuis un autre appareil compagnon.
L'USB de ce firmware fournit la console texte à 115200 bauds. Pour la messagerie et une connexion
Bluetooth directe, utiliser la [version relais + compagnon](ARBORISIS-HELTEC-V3.md).

La cible PlatformIO est `ArborisisOS_HeltecV3_repeater`.

## Installation

Images dans `release/` :

| Fichier | Adresse | Utilisation |
|---|---|---|
| `ArborisisOS_HeltecV3_repeater-v1.2.0-merged.bin` | `0x0` | Installation complète |
| `ArborisisOS_HeltecV3_repeater-v1.2.0.bin` | `0x10000` | Mise à jour du firmware |

Le dossier `release/ArborisisOS_HeltecV3_repeater-v1.2.0/` contient ce guide, les sommes
SHA-256, les journaux de compilation/tests et les captures OLED. Les anciennes images
1.0.0 et 1.1.0 peuvent coexister ; choisir explicitement 1.2.0. Depuis une 1.1.0, la mise à jour
garde l'identité et tous les réglages ; les deux modes d'observation démarrent arrêtés.

Brancher en USB. Si nécessaire, maintenir **PRG**, appuyer sur **RST**, puis relâcher PRG
pour entrer en mode téléchargement. Avec esptool installé :

```sh
esptool.py --chip esp32s3 write_flash 0x0 release/ArborisisOS_HeltecV3_repeater-v1.2.0-merged.bin
```

Sauvegarder l'identité et les réglages avant de changer de rôle. Les images n'effacent pas
volontairement le système de fichiers : les préférences répéteur déjà présentes sont reprises.
Les contacts, canaux et réglages d'un compagnon ne constituent pas une configuration répéteur ;
la compatibilité de cette transition reste à vérifier sur la carte. Ne pas effacer la flash
sans avoir sauvegardé les données nécessaires. Aucun appareil n'a été flashé pendant la création
de cette version.

## Premier démarrage

Sur une installation sans préférences répéteur existantes :

- Nom unique `Arborisis-XXXX`, suffixe issu de la clé publique.
- Répétition active, anti-boucle modérée, limite flood 64 sauts et annonces 8 sauts.
- Radio **EU Narrow : 869,618 MHz / BW 62,5 kHz / SF8 / CR 4/8**, TX 22 dBm, RX boost.
- Budget d'émission initial **10 %** ; ce budget interne n'est pas une mesure de conformité radio.
- CAD matériel actif ; calibration automatique toutes les **15 min**, marge de bruit **12 dB**,
  délai CAD adaptatif et récupération sur erreurs CAD actifs.
- Mot de passe admin de **12 caractères hexadécimaux**, généré sur l'appareil et sauvegardé.
- Écran éteint après 30 s, luminosité faible, relais toujours actif.

Les préférences existantes prennent priorité sur ces valeurs. En particulier, vérifier la
radio et le budget d'émission après une mise à jour.

## Bouton et écran

| Appui PRG | Action |
|---|---|
| Court | Page ou ligne suivante |
| Long, environ 0,5 s | Ouvrir le menu / valider |
| Très long, environ 1,5 s | Annuler / revenir à l'accueil |
| Très long sur l'accueil | Éteindre uniquement l'écran |
| Premier appui quand l'écran est éteint | Réveiller l'écran |

Onze pages : **Accueil**, **Trafic**, **Radio**, **Voisins**, **Administration**, **Réglages**,
**Calibration**, **CAD**, **Santé**, **Régions**, **Observateur**.
Le statut ON/OFF en haut correspond à la répétition. La page Trafic affiche RX/TX flood/direct,
file TX, erreurs RX, temps et budget TX. Les compteurs TX incluent les réponses et annonces du
nœud ; ils ne représentent pas uniquement les paquets relayés.
Un appui long sur Trafic alterne compteurs et histogramme RX : 30 colonnes regroupant
deux minutes chacune, avec les totaux RX/TX de l'heure. Les minutes terminées sont comptées ;
l'historique reste en RAM et repart de zéro après redémarrage.

Calibration affiche l'état, le bruit mesuré, la marge, l'indice de stabilité, le nombre
d'échantillons et la dernière erreur ; appui long pour demander un calibrage. CAD affiche
les scans, détections d'activité et erreurs ; appui long pour ouvrir son réglage. Le pourcentage
CAD représente les scans occupés, pas le temps d'occupation de la bande.
Santé affiche batterie, température du **MCU**, mémoire libre, doublons et récupérations.
Régions affiche les régions par défaut/locale et l'acceptation du flood sans région.

La page Voisins lance une découverte des répéteurs en portée directe ; une recherche au maximum
par minute. Les réponses peuvent prendre plusieurs secondes. Les voisins sont affichés par
préfixe de clé publique, SNR et âge depuis la dernière réception. **Appui long : voisin suivant**.
La liste est reconstruite après redémarrage.

Sur Administration, un **appui long révèle le code admin**. Il disparaît après 15 s, au changement
de page ou à la veille de l'écran. Ne pas partager une photo de cet écran avec le code visible.

Le menu de 28 entrées permet d'activer/arrêter le relais, envoyer un advert local ou réseau, rechercher les
voisins, régler les sauts, l'anti-boucle, le budget d'émission, l'orientation 180°, la luminosité,
la veille de l'écran, effacer les statistiques et redémarrer. Il permet aussi de gérer calibration,
intervalle, marge, CAD, délai adaptatif, récupération, gain RX, puissance TX, périodicité des
annonces et blocage du flood sans région, ainsi que l'observateur USB, le mode sentinelle et son
test de passerelle. Arrêter le relais, effacer les
statistiques ou redémarrer demande un second appui long ; un appui court annule.

Les réglages radio/relais du menu passent par la même console que l'administration distante.
L'écran a ses propres préférences persistantes (`/arbo_repeater_ui`).

## Calibration automatique et CAD

Le premier calibrage est demandé 15 secondes après démarrage. Le superviseur attend que
la radio soit en RX sans réception ni interruption en attente avant de rafraîchir le récepteur.
Il appelle le `resetAGC()` de RadioLib : sommeil chaud, calibration des blocs SX1262,
calibration image à la fréquence active et restauration des réglages RF. Les codes d'erreur
matériels remontent aux diagnostics ; la boucle reprend la réception après cet appel.

La mesure utilise 96 RSSI espacés de 50 ms, en excluant les réceptions en cours et les valeurs
hors −140 à −30 dBm. Le quartile inférieur fournit le plancher de bruit. Une fenêtre expire
après 20 s et demande au moins 48 mesures valides. Un plancher supérieur à −80 dBm ou une
dispersion interquartile supérieure à 16 dB est rejeté. L'indice affiché mesure la quantité et
la stabilité des échantillons ; il n'est pas une probabilité de réception.

Le seuil RSSI avant émission vaut **bruit validé + marge**. Les rondes périodiques limitent
le changement du plancher à ±6 dB et rafraîchissent le matériel au maximum toutes les six
heures ; un calibrage manuel ou un changement de radio/gain RX demande un rafraîchissement.
Une tentative échouée garde le dernier plancher validé et réessaie après une minute.
Une attente RX occupée expire après une minute sans forcer un reset en pleine réception.
Au changement de radio/gain, l'ancien plancher est invalidé. Les paramètres temporaires et leur
retour à la radio sauvegardée déclenchent aussi une nouvelle calibration.

| Erreur | Sens |
|---|---|
| `-1001` | Récepteur occupé, maintenance reportée |
| `-1002` | Trop peu de RSSI valides |
| `-1003` | Bruit trop variable |
| `-1004` | Signal continu trop fort pour être accepté comme bruit calme |
| Autre code | Erreur RadioLib remontée pendant le rafraîchissement ou le CAD |

CAD conserve les paramètres de détection recommandés par RadioLib. Chaque scan attend
l'interruption pendant un délai borné calculé à partir de SF/BW, limité entre 60 ms et 5 s.
Une erreur CAD est traitée comme un canal occupé. La réception est réarmée après le scan.
Le moteur garde le paquet en file tant que le canal est occupé : cette version supprime
l'émission forcée après quelques secondes du répéteur standard.

Le délai de nouvel essai est de 200–300 ms, 400–600 ms ou 800–1200 ms selon les détections CAD
des dix dernières secondes, avec au moins huit scans pour utiliser l'adaptation. Une part
aléatoire évite que plusieurs répéteurs réessaient simultanément. Désactiver l'adaptation
conserve le délai de base avec sa part aléatoire.
Au moins trois erreurs CAD en dix secondes demandent une récupération, limitée à une par
minute. Un réseau silencieux ou des détections d'activité légitimes ne déclenchent pas ce mécanisme.

La calibration ne règle pas automatiquement fréquence, largeur de bande, SF, CR, puissance
TX ou régions. Une correction absolue de fréquence ou de tension exige une référence externe.
Les résultats de bruit ne sont pas sauvegardés : ils sont remesurés au démarrage. Les options
sont persistantes dans `/arbo_repeater.json`, écrit via un fichier temporaire et une copie
de secours `/arbo_repeater.bak`. Un fichier principal absent/malformé permet de reprendre la
copie précédente ; en l'absence de copie valide, les valeurs par défaut restent opérationnelles.
Cette procédure tient compte du [renommage SPIFFS](https://github.com/pellepl/spiffs/blob/master/src/spiffs_hydrogen.c),
qui refuse de remplacer une destination existante.

## Observateur pour la carte

Un répéteur bien placé entend une grande partie du réseau. La 1.2.0 lui permet de le montrer sur
[map.arborisis.com](https://map.arborisis.com), de deux façons. Les deux peuvent rester actives :
quand le pont USB est branché, les rapports de sentinelle se mettent en pause.

### Par USB : pont observateur, HTTPS ou MQTT

Là où il y a Internet, brancher le répéteur en USB à un ordinateur ou un Raspberry Pi et installer
le pont observateur (1.4.0 ou plus récent) :

```sh
curl -fsSL https://arborisis.com/observer/install.sh | bash
```

Au port USB proposé, le pont reconnaît le répéteur (il répond à `obs info` ; un compagnon ignore
ce texte), active la sortie des paquets (`obs usb on`, gardé après redémarrage) puis publie chaque
paquet reçu sur la carte, avec SNR et RSSI. Par défaut l'envoi se fait en HTTPS ; `--mqtt arborisis`
passe par le broker `wss://mqtt.arborisis.com`, et `--mqtt letsmesh-eu --iata BRU` publie aussi
chez LetsMesh, comme meshcoretomqtt. Les jetons de connexion sont **signés par la carte**
(`obs sign`) : la clé privée ne quitte jamais le répéteur. Le pont transmet aussi par le répéteur les
ACK des sentinelles voisines (`obs tx`) ; le répéteur n'accepte par cette commande que des ACK de
sentinelle. `--device repeater` force la détection, `--device companion` la saute.

Avec le profil belge (défaut de l'installateur), le pont vérifie 869,618 MHz / BW 62,5 / SF8 / CR8
et les chemins de 2 octets ; s'il doit les corriger, il les enregistre (`set radio`,
`set path.hash.mode 1`) et redémarre le répéteur, puis se reconnecte. `--radio-profile keep`
laisse la radio telle quelle.

La sortie USB reprend exactement les deux lignes `RAW:` et `RX, …` de `MESH_PACKET_LOGGING` de
MeshCore : [meshcoretomqtt](https://github.com/Cisien/meshcoretomqtt) peut aussi la lire. Elle est
tamponnée (4 ko) pour ne pas ralentir le relais. `obs usb off` la coupe.

### Sans Internet : mode sentinelle avec ACK

Là où aucun observateur n'a Internet, le répéteur rapporte comme une
[sentinelle](ARBORISIS-SENTINELLE.md) : toutes les 30 minutes (± 10 %), un rapport compact et
authentifié (~70 octets : trafic entendu par type, voisins directs et leur SNR, bruit de fond,
batterie, température) part vers les observateurs connectés, et le serveur répond par un **ACK**
qui donne la route la plus courte pour le prochain, l'heure (le répéteur avance son horloge si elle
retarde) et éventuellement un nouvel intervalle. Un ACK est demandé au démarrage, tant que la route
n'est pas confirmée, puis un rapport sur quatre. Sans route : chemin d'une passerelle connue, zéro
saut, meilleur voisin, chaîne de deux voisins, puis au plus une inondation du canal
`#arbo-sentinelle` toutes les 12 h. Le protocole est celui des sentinelles
([docs/sentinelle-protocole.md](docs/sentinelle-protocole.md)), avec le drapeau `0x80`
« répéteur » : la carte affiche « Répéteur », et prévoit l'autonomie d'une batterie solaire avec la
consommation d'un répéteur (≈ 45 mA, modèle) jusqu'à ce qu'une tendance soit mesurée.

Installation : sur la carte, onglet **Sentinelles** (administrateur) › **Installer**, brancher le
répéteur en USB dans Chrome/Edge et **Connecter** : la page lit sa clé de rapport, l'enregistre,
règle l'intervalle, ajoute les passerelles proches et active le mode. Sans navigateur compatible,
saisir la clé publique (`get public.key`) et la clé de rapport (`sentinel key`, console USB
uniquement) dans « Sans USB », puis `sentinel on`.

| Commande | Effet |
|---|---|
| `obs` | État : sortie USB, pont connecté, mode sentinelle |
| `obs usb on` / `obs usb off` | Sortie des paquets sur l'USB |
| `sentinel` | Route, dernier ACK et passerelle, rapports et ACK, prochain rapport |
| `sentinel on` / `sentinel off` | Mode sentinelle |
| `sentinel report` / `sentinel test` | Rapport immédiat / test de passerelle (jusqu'à 5 essais) |
| `sentinel interval 30` / `sentinel ackevery 4` | Intervalle (5–1440 min) / un ACK sur n (1–50) |
| `sentinel gw add a1b2c3d4` / `sentinel gw clear` / `sentinel gw` | Passerelles connues (8 au plus) |
| `sentinel route reset` | Oublier la route et explorer à nouveau |
| `sentinel key` | Clé de rapport, sur la console USB seulement |

Ces commandes passent aussi par l'administration à distance, sauf `sentinel key`. Sur l'écran,
la page **Observateur** montre l'état USB (arrêté, en attente du pont, pont connecté), le prochain
rapport, la route, l'âge du dernier ACK et sa passerelle ; un appui long y lance le test de
passerelle (ou ouvre les réglages si le mode est arrêté). Les réglages sont gardés dans
`/arbo_observer` avec le compteur de démarrages et la dernière route confirmée, revérifiée par le
premier rapport après un redémarrage.

### Gestion à distance et mises à jour par le réseau (1.3)

Depuis la 1.3.0, le répéteur partage le moteur des sentinelles 1.1
([Gestion à distance et mises à jour](ARBORISIS-SENTINELLE.md#gestion-à-distance-et-mises-à-jour)) :

- il envoie son **état** (version, build, radio, position, passerelles) au serveur et exécute les **commandes**
  de la fiche (nom, position, puissance, taille des sauts, passerelles, route, test, mode sentinelle,
  redémarrage) — les réglages radio restent dans l'administration MeshCore habituelle ;
- il se **met à jour par le réseau** : paquet signé téléchargé morceau par morceau, installé dans la partition
  inactive, confirmé au premier ACK ou après 2 min sans plantage en entendant le réseau, sinon retour automatique à
  l'ancienne version (2 h sans rien entendre) ;
- il **sert les sentinelles autour de lui** (`ota seed on`, par défaut) : après une installation, ou quand le
  serveur lui a envoyé un paquet de sentinelle « pour les voisins », il l'annonce et le transmet en un saut ;
- **branché à un pont observateur**, ses rapports, états et demandes passent par l'USB au lieu de l'antenne, et
  les réponses du serveur reviennent par `obs tx` : il se gère et se met à jour à travers son propre pont.

| Commande | Effet |
|---|---|
| `ota` | Version, build, paquet en cours et reçus, paquet gardé |
| `ota auto on\|off` / `ota seed on\|off` | Mises à jour automatiques depuis les voisins / servir les voisins |
| `ota cancel` / `ota info` | Abandonner le paquet / envoyer l'état au serveur |
| `ota fetch <id>`, `ota offer <hex>`, `ota data <hex>` | Installer un paquet par USB (`tools/ota/usb-feed.mjs`) |

Les réglages du mode observateur écrits par la 1.2 sont repris tels quels. Un répéteur en 1.2 doit être flashé
une fois par USB pour passer en 1.3.

## Administration

Dans l'appli MeshCore, connectée à **un autre compagnon** réglé sur la même radio, recevoir
l'annonce du répéteur, l'ajouter puis ouvrir son administration avec le code affiché sur sa carte.
Le menu peut envoyer immédiatement une annonce locale ou en flood.

Exemples pour la console USB ou la console distante après connexion admin :

```text
get name
set name Arborisis-Toit
password MonCodePersonnel
set lat 50.8503
set lon 4.3517
set radio 869.618,62.5,8,8
set tx 22
set dutycycle 10
set repeat on
get repeat
get flood.max
get loop.detect
get arbo.version
get active.radio
get calibration
get cad
get diagnostics
get traffic.hour
calibrate now
set calibration on
set calibration.interval 15
set calibration.margin 12
set cad on
set cad.backoff on
set recovery on
```

Après `set radio`, redémarrer le nœud pour appliquer les paramètres sauvegardés.
`get active.radio` affiche la configuration réellement active, y compris un réglage temporaire.
L'intervalle de calibration accepte 5–1440 minutes ; la marge accepte 6–30 dB.
`set calibration off` rétablit le seuil manuel `int.thresh` et son échantillonnage stock.
`calibrate now` demande d'abord que la calibration automatique soit activée.

Pour corriger la lecture batterie, connecter une batterie et mesurer sa tension au voltmètre,
puis entrer la référence en millivolts, par exemple `calibrate battery 4010`. La commande
moyenne huit lectures et sauvegarde le multiplicateur ADC corrigé. Elle accepte 2500–4500 mV
et refuse une lecture absente/hors plage ; l'alimentation USB seule ne fournit pas une
référence de tension batterie.

En USB, configurer le terminal à **115200 bauds** et la fin de ligne **CR**. Le mot de passe est
limité à 15 caractères. Les coordonnées sont un exemple à remplacer par celles de l'installation.
Les régions et les autres commandes restent celles du répéteur MeshCore de ce dépôt.

Alimenter un répéteur fixe en USB. La veille OLED laisse la radio active ; le redémarrage et la
commande d'extinction du nœud interrompent le service. L'économie d'énergie MCU reste désactivée
par défaut.

## Compilation et vérification

```sh
sh tools/release-repeater.sh
```

Le script construit l'image complète, exécute les vérifications et rassemble les preuves dans
`release/ArborisisOS_HeltecV3_repeater-v1.2.0/`. Ses dossiers de build sont isolés pour éviter les
collisions avec une compilation compagnon. Il est possible de réutiliser un dossier via
`REPEATER_BUILD_ROOT=/chemin/absolu`. `tools/release.sh` appelle ce script et conserve la version
propre au répéteur. La matrice CI compile aussi la cible.
Le simulateur compile **les vrais fichiers d'interface** contre des doublures radio/matériel :
60 vérifications et 20 captures PNG dans `tools/sim-repeater/out/`. Il vérifie les appuis,
confirmations, commandes émises, découverte limitée, confidentialité de l'affichage et
persistance des réglages écran, ainsi que la page et le menu Observateur. Il ne vérifie pas les
transmissions LoRa ni une connexion admin.

L'observateur réutilise le code des sentinelles (`examples/arbo_sentinel/SentinelProto`,
`SentinelCore`, `SentinelHeard.h`), testé sur ordinateur par `sh tools/sim-sentinel/build.sh`
(131 contrôles, dont le vecteur d'un rapport de répéteur relu par le serveur). Côté pont et
serveur, `node --test web/tests/observer-repeater.test.mjs` simule la console USB d'un répéteur :
lignes de paquets coupées par l'écho d'une commande, détection, signature des jetons, envoi des
paquets à la carte, rapport d'un répéteur-sentinelle, ACK planifié par le serveur et transmis par
`obs tx`, répéteur relié par USB jamais déclaré silencieux.

Les **74 tests natifs** passent : 42 tests existants et 32 nouveaux tests sur la machine de
calibration réellement utilisée par le firmware, l'historique, les délais et le sérialiseur des
options. Les tests couvrent attente RX, délais, erreurs, rejet des mesures instables/fortes,
conservation du dernier plancher, replanification, débordement de `millis()` et configuration,
y compris sauvegardes successives, écriture courte, échec de renommage et reprise après une
coupure simulée entre les renommages.
Deux vérifications supplémentaires exécutent le dispatcher et le pool MeshCore de production
avec une radio contrôlée : maintien en file durant deux minutes d'occupation, puis émission
unique et libération du paquet lorsque le canal devient libre.
Les tests natifs utilisent l'inclusion explicite de `stdlib.h`, requise
par les appels `atoi/atof/atol` du sérialiseur dans la configuration native actuelle :

```sh
PLATFORMIO_BUILD_FLAGS='-std=c++17 -I src -I test/mocks -include stdlib.h' pio test -e native
sh tools/test-repeater-cad.sh
sh tools/sim-repeater/build.sh
```

À vérifier sur matériel : flash et redémarrage, radio SX1262 et orientation OLED, conservation
des données lors d'un changement de rôle, connexion administrateur depuis un compagnon, routage
flood/direct, propagation des régions, précision RSSI et batterie, efficacité CAD sur des signaux
réels, rafraîchissement/calibration SX1262 et consommation. La calibration automatique n'a
pas encore été validée sur une carte physique. La compilation, les tests contrôlés et les
captures OLED ne remplacent pas ces essais.
