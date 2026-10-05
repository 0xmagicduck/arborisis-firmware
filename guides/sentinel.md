# Arborisis Sentinelle — observateur MeshCore sans Internet (Heltec V3)

Une **sentinelle** est un observateur MeshCore qui n'a **pas besoin d'Internet** : une Heltec WiFi LoRa 32 V3
sur batterie, posée n'importe où dans la ville (poteau, toit, balcon). Elle écoute le réseau, résume ce
qu'elle entend et confie ce résumé, **par MeshCore**, aux observateurs qui ont Internet (les *passerelles*).
Le serveur suit en temps réel la batterie de chaque sentinelle et indique sur la carte **lesquelles changer,
quand et où**.

La cible PlatformIO est `ArborisisOS_HeltecV3_sentinel` (version **1.0.0**). Le protocole est décrit dans
[docs/sentinelle-protocole.md](docs/sentinelle-protocole.md).

> **Statut** : firmware compilé, logique de routage et d'ACK testée sur l'ordinateur (130 contrôles avec
> AddressSanitizer/UBSan, dont la couche MeshCore réelle avec une radio simulée), paquets du firmware décodés
> par le serveur, aller-retour complet rapport → serveur → pont observateur → ACK testé de bout en bout.
> **Aucune carte n'a encore été flashée** : la consommation réelle et le taux de réception en écoute
> cyclique restent à mesurer (voir [À vérifier sur le matériel](#à-vérifier-sur-le-matériel)).

## Comment ça marche

```
 Sentinelle (sans Internet)          Répéteurs MeshCore            Passerelle (observateur connecté)        Serveur
 ┌──────────────────────┐   rapport  ┌────────┐   ┌────────┐      ┌──────────────────────────┐   HTTPS   ┌──────────────┐
 │ écoute, compte,      │──────────▶ │ relais │──▶│ relais │────▶ │ compagnon + pont 1.3     │─────────▶ │ map.arborisis│
 │ résume (≈70 octets)  │  (direct)  └────────┘   └────────┘      │ (envoie tout ce qu'il    │           │  .com        │
 │                      │ ◀──────────────────────────────────────│  entend, comme avant)    │ ◀──────── │ choisit la   │
 └──────────────────────┘        ACK + meilleure route            └──────────────────────────┘ ACK à     │ passerelle   │
                                                                                               émettre   └──────────────┘
```

1. La sentinelle écoute en continu (radio en **écoute cyclique**, processeur en **veille légère**) et compte :
   paquets par type, relais entendus en direct avec leur SNR, bruit de fond, occupation du canal.
2. Toutes les 30 min (réglable), elle envoie un **rapport compact** (50 à 90 octets) signé par un code
   d'authentification. Les passerelles l'entendent et le transmettent au serveur **avec tous les autres
   paquets** : rien à changer pour les observateurs existants.
3. Le serveur vérifie le rapport, enregistre la batterie, puis répond par un **ACK** via la passerelle qui l'a
   entendu au plus près. L'ACK indique la route la plus courte à utiliser la fois suivante.
4. Sans route connue, la sentinelle cherche avec le moins d'émissions possible : chemin d'une passerelle connue
   (gratuit), portée radio directe, voisin, chaîne de deux voisins, et en dernier recours **une** inondation de
   découverte, **au plus toutes les 12 h**.

## Ne pas saturer le réseau

Un rapport n'est jamais inondé dans tout le réseau en régime normal : il suit une **route directe** (seuls les
relais de la route le répètent) ou n'est pas répété du tout (portée directe d'une passerelle).

| Paquet | Taille | Temps d'antenne (SF8, 62,5 kHz, CR 4/8) |
|---|---|---|
| Rapport sans voisins | ~50 octets | ~0,6 s |
| Rapport avec 8 voisins | ~85 octets | ~0,9 s |
| ACK | ~35 octets | ~0,5 s |
| (pour comparer) un advert | ~125 octets | ~1,2 s |

Avec un rapport toutes les 30 min sur une route de 2 relais et un ACK un rapport sur 4 : environ **2 minutes
d'antenne par jour et par sentinelle** sur tout le réseau (0,14 %). En portée directe d'une passerelle :
~40 s/jour. À titre de comparaison, le même rapport inondé par 20 répéteurs coûterait environ 12 minutes par jour.

Autres garde-fous : budget d'émission propre limité à **1 %** de l'heure, écoute avant émission (CAD matériel),
décalage aléatoire de ±10 % des rapports, intervalle doublé sous 40 % de batterie, quadruplé sous 20 %,
inondation de découverte limitée (réglable avec `flood <heures>`), ACK demandés une fois sur 4 quand la route
est confirmée. La sentinelle **ne relaie jamais** et **n'envoie pas d'advert**.

## Matériel

- **Heltec WiFi LoRa 32 V3** (ESP32-S3 + SX1262 + OLED), antenne 868 MHz.
- **Batterie Li-ion 1S** (18650 protégée ou LiPo) sur le connecteur batterie de la carte. Les cellules
  LiFePO4 sont prises en charge par le suivi (choisir « LiFePO4 » sur la fiche), à condition que la tension
  convienne à la carte.
- Option : panneau solaire avec régulateur 5 V sur l'USB. La carte charge la cellule ; la fiche affiche « En
  charge » et la prévision tient compte des journées ensoleillées.
- Boîtier étanche, antenne vers le ciel, carte à l'abri du soleil direct.

## Autonomie (estimations)

Wi-Fi et Bluetooth ne sont jamais démarrés, l'écran est coupé (alimentation Vext), le processeur dort entre
deux paquets. Valeurs **calculées**, pas encore mesurées :

| Profil | Écoute | Courant moyen estimé | 18650 3000 mAh | 2 × 18650 |
|---|---|---|---|---|
| `full` | continue | ~5,6 mA | ~3 semaines | ~6 semaines |
| `eco` (défaut) | cyclique : la radio se réveille seule assez souvent pour attraper le préambule de 32 symboles de MeshCore | ~1,8 mA | ~2 mois | ~4 mois |
| `ultra` | fenêtres d'écoute (5 min sur 15 par défaut, `ultra <écoute> <cycle>`) | ~0,8 mA | ~5 mois | ~10 mois |

La sentinelle estime aussi sa propre consommation (mAh depuis le démarrage, modèle du temps passé dans chaque
état) et l'envoie dans chaque rapport. Le serveur utilise d'abord ce **modèle**, puis la **pente mesurée** de
la batterie dès qu'il a 12 h de mesures (régression robuste de Theil–Sen sur 7 jours, depuis le dernier
changement de batterie).

Protection de la cellule : sous 3,35 V (trois mesures de suite), la sentinelle envoie un dernier rapport
**« batterie vide »** puis passe en veille profonde. Elle se réveille toutes les 6 h et ne redémarre que si la
tension est remontée (panneau solaire) ; une batterie neuve la redémarre aussitôt.

## Installation

1. **Flasher** la carte (USB, mode téléchargement : maintenir PRG, appuyer sur RST, relâcher PRG) :
   ```sh
   esptool.py --chip esp32s3 write_flash 0x0 release/ArborisisOS_HeltecV3_sentinel-v1.0.0-merged.bin
   ```
   ou depuis le code : `pio run -e ArborisisOS_HeltecV3_sentinel -t upload`.
   Fabriquer les images et rejouer les tests : `sh tools/release-sentinel.sh`.
2. Sur **map.arborisis.com**, onglet **Sentinelles** (jeton administrateur), bouton **Installer**, puis
   **Connecter en USB** (Chrome ou Edge sur ordinateur). La page lit l'identité et la **clé de rapport** de la
   carte, puis enregistre la sentinelle et la configure : nom, position (ma position ou clic sur la carte),
   profil, intervalle et les **4 passerelles les plus proches** (leurs adverts donnent une route sans aucune
   émission). Sans USB : saisir l'identifiant et la clé lus avec les commandes `info` et `key`.
3. **Sur place**, un appui long sur **PRG** lance le **test de passerelle** : la sentinelle envoie un rapport et
   attend la réponse du serveur (20 s par essai, 5 essais en changeant de chemin). L'écran affiche le résultat :
   « OK : rapport reçu, route 2 sauts, GW 1a2b3c4d » ou « ÉCHEC : aucune passerelle ne répond ». Le test peut
   aussi être lancé depuis la page d'installation.

Au moins un **pont observateur 1.3** (commande d'installation habituelle, mise à jour en relançant la même
commande) doit entendre la sentinelle pour lui répondre. Les rapports entendus par d'autres observateurs
(MQTT, anciens ponts) sont enregistrés aussi, mais sans ACK la sentinelle ne connaît pas sa meilleure route
et continue d'explorer.

## Bouton et écran

| Appui PRG | Action |
|---|---|
| Court | Allume l'écran 30 s : nom, batterie, prochain rapport, route, dernière passerelle, voisins |
| Long (1,2 s) | Test de passerelle (écran allumé pendant le test) |

L'écran s'allume aussi 20 s au démarrage (changement de batterie). Il reste coupé le reste du temps.

## Console USB

115200 bauds, une commande par ligne, une réponse JSON par ligne. L'ouverture du port redémarre la carte ;
elle reste ensuite éveillée tant que des commandes arrivent (2 min après la dernière), et en permanence quand
elle est alimentée par l'USB sans batterie.

| Commande | Effet |
|---|---|
| `info` | État complet (identité, batterie, radio, route, passerelles) |
| `key` | Clé de rapport à donner au serveur (jamais envoyée par radio) |
| `name <texte>`, `pos <lat> <lon>` | Nom et position (informatifs) |
| `interval <min>` | Période des rapports (5 à 1440 min) |
| `ackevery <n>` | Demander un ACK un rapport sur n quand la route est confirmée |
| `profile full\|eco\|ultra`, `ultra <écoute> <cycle>` | Profil d'écoute |
| `hash 1\|2\|3` | Taille des identifiants de saut du réseau (Belgique : 2) |
| `radio <MHz> <kHz> <sf> <cr>`, `tx <dBm>` | Radio (défaut : 869,618 / 62,5 / SF8 / CR8, 22 dBm) |
| `gw add <8 hex>`, `gw clear` | Passerelles connues (préfixe de leur clé publique) |
| `report`, `test` | Rapport immédiat, test de passerelle |
| `route`, `route reset` | Route actuelle, oublier la route |
| `adc <multiplicateur>` | Calibrer la mesure de tension |
| `rxwake <symboles>`, `rxpre 16\|32` | Réglage fin de l'écoute cyclique |
| `flood <heures>` | Intervalle minimal entre deux inondations de découverte |
| `flip`, `reboot` | Écran retourné, redémarrage |

Le serveur peut aussi changer l'intervalle, le rythme des ACK et le profil à distance : la modification faite
sur la fiche part avec le prochain ACK.

## Carte et administration

Onglet **Sentinelles** de map.arborisis.com (lien direct `#sentinel/<id>`) — réservé à l'administration, car
les positions désignent du matériel posé dans la rue :

- **Carte** : chaque sentinelle colorée selon sa batterie (correcte, en charge, bientôt, à changer, vide,
  silencieuse), avec le pourcentage ; un anneau pour celles qui demandent une visite.
- **Liste par urgence** : charge, date de remplacement prévue, dernier rapport, route et passerelle.
- **Fiche** : charge et tension, courbe avec la projection jusqu'au seuil de remplacement, consommation
  mesurée ou modélisée, dernier rapport (écoute, bruit, température, chemin, ACK), relais entendus en direct
  (reliés aux nœuds de la carte), historique (installations, redémarrages, changements de batterie détectés
  automatiquement par le saut de tension, alertes). Boutons **Itinéraire**, **Batterie changée**, **Modifier**.
- **Tournée de remplacement** : sentinelles vides, silencieuses ou à changer dans 7 à 60 jours, dans l'ordre du
  trajet le plus court depuis votre position, avec lien **Google Maps** et export **GPX**.
- **Temps réel** : flux d'événements admin ; chaque rapport met la fiche à jour.
- **Alertes** : `SENTINEL_WEBHOOK_URL` reçoit un POST JSON (`text` et `content`, compatibles Slack, Discord,
  Mattermost, ntfy…) à chaque passage à « bientôt », « à changer », « vide » ou « silencieuse » (trois périodes
  sans rapport), et au retour à la normale.
- **Publique (option par sentinelle)** : la sentinelle apparaît alors comme observateur sur la carte publique,
  avec ses liens radio mesurés vers les relais qu'elle entend.

API (jeton administrateur `ADMIN_TOKEN`) : `GET/POST /api/mesh/sentinels`, `GET/PATCH/DELETE
/api/mesh/sentinels/{id}`, `POST /api/mesh/sentinels/{id}/battery`, `GET /api/mesh/sentinels/maintenance`,
`GET /api/mesh/sentinels/gateways`, `GET /api/mesh/sentinels/events`, `GET /api/mesh/sentinels/stream?token=`.
Passerelles (jeton observateur) : `GET /api/mesh/downlink?wait=25`, `POST /api/mesh/downlink/{id}/result`.

## Sécurité

- Chaque rapport et chaque ACK portent un code HMAC-SHA256 tronqué à 6 octets, calculé avec une clé de 16 octets
  dérivée de la clé privée de la carte. Cette clé ne circule que par USB, à l'installation. Un faux ACK ne
  peut donc pas détourner la route d'une sentinelle, ni un faux rapport fausser sa batterie.
- Rejeu : le serveur n'accepte que des numéros (compteur de démarrages, séquence) plus récents.
- Les rapports ne contiennent **pas** la position. Elle n'existe que côté serveur, en accès administrateur.
- Le contenu d'un rapport (batterie, compteurs) n'est pas chiffré : il n'a rien de confidentiel.

## À vérifier sur le matériel

- Courant réel dans chaque profil (multimètre ou PPK sur la batterie) et autonomie.
- Taux de paquets entendus en écoute cyclique (`eco`) par rapport à l'écoute continue (`full`), selon
  `rxwake` (6 symboles par défaut). Les nœuds dont le firmware utilise un préambule de 16 symboles sont
  moins bien entendus en `eco` (`rxpre 16` corrige au prix d'une écoute plus longue).
- Réveil par le bouton et par la radio depuis la veille légère, extinction et rallumage de l'écran.
- Mesure de tension (`adc`) sur la batterie réellement utilisée, seuil de 3,35 V.
- Réception de l'ACK par la sentinelle via `CMD_SEND_RAW_PACKET` sur les compagnons Arborisis ; les compagnons
  qui ne le connaissent pas utilisent `CMD_SEND_RAW_DATA`, limité aux chemins de 1 octet.

## Tests

```sh
sh tools/sim-sentinel/build.sh            # logique (112 contrôles) + couche MeshCore simulée (18 contrôles)
cd web && node --test tests/sentinel.test.mjs tests/sentinel-bridge.test.mjs
```

Démonstration locale sans matériel (12 sentinelles autour de Namur, 10 jours d'historique, rapports en direct) :

```sh
ADMIN_TOKEN=dev MESH_OPEN_INGEST=1 MESH_DB=/tmp/sentinel.sqlite PORT=3100 npm --prefix web start
node web/scripts/sentinel-simulate.mjs --server http://127.0.0.1:3100 --admin dev --db /tmp/sentinel.sqlite
```

puis http://127.0.0.1:3100/carte#sentinels avec le jeton `dev`.
