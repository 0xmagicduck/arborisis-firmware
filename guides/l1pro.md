# Arborisis OS — MeshCore autonome pour Seeed Wio Tracker L1 Pro

Firmware MeshCore (base `v1.17.1`, commit `e941259`) pour le **Wio Tracker L1 Pro**
(nRF52840 + SX1262 + GNSS L76K + OLED SH1106 1,3" + joystick 5 directions).

Il fonctionne de deux façons, en même temps :

- **seul** : lire et écrire des messages directement sur l'appareil (clavier piloté au
  joystick avec suggestions de mots, conversations, contacts, canaux, radar, GPS, SOS, réglages) ;
- **avec l'appli compagnon MeshCore** (Bluetooth) : l'appli fonctionne comme d'habitude, et
  l'appareil garde son propre historique, y compris ce que vous envoyez depuis le téléphone.

L'appareil peut aussi **relayer** les messages des autres (relais automatique selon la batterie),
envoie ses **adverts tout seul** et gère les **régions** MeshCore. L'interface est en français.

> **Statut** : version **2.2.0 Ultimate**, compilée en BLE et USB (flash utilisée à 91 %),
> **375 contrôles du simulateur** avec AddressSanitizer/UndefinedBehaviorSanitizer, **90 captures**,
> T9 mesuré sur 7 000 phrases jamais vues, tests de gestion radio et de
> protection du stockage (coupure de courant à chaque écriture) réussis.
> **Flash réel du L1 Pro effectué le 4 octobre 2026** : démarrage 2.1.0 confirmé par USB,
> identité existante chargée, nom conservé, connexion et protocole compagnon BLE vérifiés.
> **2.1.1 flashée le 4 octobre 2026** par USB, sans effacement : `info` relit l'identité existante,
> 9 contacts, 5 canaux et 99 messages, avec `storage=mounted` (sonde : système de fichiers valide,
> 7 ms). **2.2.0 flashée le 4 octobre 2026** de la même façon : `info` relit la même identité,
> 9 contacts, 5 canaux et 133 messages, `storage=mounted`. Reste à confirmer un redémarrage sur
> batterie et la réactivité du T9 au doigt (voir
> [À vérifier sur l'appareil](#à-vérifier-sur-lappareil)).
> Les preuves et limites figurent dans le
> [rapport UI/UX et validation](docs/l1pro-ultimate-ux.md).

### Nouveautés 2.2

- **Vrai T9** : un pavé de téléphone au joystick (une pression par lettre, `2665687` →
  *Bonjour*), qui devient le clavier par défaut (**appui long sur ⇧** pour le clavier complet,
  et retour). Autres mots des mêmes touches avec ⇄, complétion avec ⇥, apostrophe des élisions
  et ponctuation sur la touche 1, modes **Abc** (appuis répétés) et **123**.
- **Dictionnaire français embarqué** de 20 000 mots avec leur fréquence, leur nature et 6 000
  enchaînements, pour le T9 comme pour le clavier complet : complétions et mots suivants
  beaucoup plus justes, mots choisis selon le mot d'avant… et corrigés selon le mot d'après.
- Aide embarquée : page **Clavier T9**.

### Nouveautés 2.1.1

- **Contacts, canaux et historique conservés au redémarrage** : la flash est réveillée et vérifiée
  avant d'être montée, et n'est plus jamais formatée d'office (détails sous *Installation*).
  Sauvegardes des contacts et canaux atomiques ; **Infos › Stockage** pour l'état et la réparation.
- **Heure par GPS** (Réglages › Heure › Via GPS, activé) : quand l'heure est inconnue, le GPS
  s'allume juste le temps d'un fix, même en mode GPS « off », puis une fois par jour.
- **Clavier prédictif adaptatif** : il apprend vos mots et vos tournures (messages envoyés depuis
  l'appareil ou l'appli), propose le **mot suivant** avant toute lettre, corrige les **accents**
  (« deja » → « déjà ») et offre plusieurs candidats (**appui long** sur la touche ⇥).

### Nouveautés Ultimate (2.1)

- Menu **avec noms** par défaut, position mémorisée ; **OK long** bascule vers la grille.
- Réglages en **8 catégories**, aide embarquée en **7 pages** (8 depuis la 2.2), commandes contextuelles.
- Recherche de contacts : **haut** depuis la première entrée, puis **OK** et saisir le nom.
- Chat : **droite = Renvoyer** sur un message en échec ; **OK long** conserve les détails.
  Le brouillon survit au refus d'un envoi ; l'arrivée d'un message préserve la lecture de l'historique.
- Réessais automatiques configurables : **Réglages › Réseau › Réessais**, 1–5 essais au total,
  **3** par défaut. Les refus temporaires de la radio sont réessayés avec temporisation croissante.
- **Synchronisation L1 Pro → appli** à la connexion : canaux sous votre nom et copies des DM
  marquées **« Moi L1 Pro »** dans la conversation correspondante. **Réglages › Réseau › Sync appli** :
  `tout*` (défaut), `canaux` ou `off`. Le protocole MeshCore représente ces copies comme des messages
  reçus ; il ne permet pas de créer une bulle native « envoyée » depuis le boîtier. Les DM longs
  sont copiés en deux parties. L'historique reste sur le L1 Pro et les messages provenant de
  l'appli ne lui sont pas renvoyés.
- **Région par canal**, avec assez de places pour les **40 canaux** : **Canaux › OK long › Région**.
  Les choix s'appliquent aux envois autonomes et aux envois de l'appli compagnon.
- Optimisation automatique **normal / éco / réserve**, luminosité réduite avant veille,
  acquisitions GPS infructueuses espacées ; navigation et SOS prioritaires.
- Sauvegardes de l'horloge, des préférences et de l'historique par remplacement atomique ;
  les envois interrompus par un redémarrage deviennent accessibles au renvoi manuel.

### Nouveautés de la 2.0

- **Interface redessinée** : menu en grille d'icônes, grosse horloge, logo, listes arrondies,
  bulles de discussion avec l'heure, pastilles de non-lus, rotation de l'écran à 180°.
- **Historique 5× plus grand** dans la même mémoire (jusqu'à 400 messages, selon leur longueur).
- **Suggestions de mots** au clavier (mots de vos conversations et liste de mots français).
- **Détails d'un message** : chemin, SNR, essais ; répondre, renvoyer, citer, supprimer, y aller.
- **Radar** des contacts localisés, **navigation** vers un contact, un repère ou un SOS reçu,
  **trajet GPS** (distance, vitesse, cap).
- **SOS** : alerte répétée avec votre position ; un SOS reçu sonne même en silencieux.
- **Relais automatique**, **adverts automatiques**, **régions** MeshCore.
- **Horloge** gardée après un redémarrage, réglable à la main ou prise sur le réseau.
- **CAD adaptatif** et **calibration du bruit adaptative**.
- **Lampe**, **contacts favoris**, tri par nom ou distance, **statistiques radio**, silence la nuit.

---

## Installation

Fichiers produits par `tools/release.sh` dans `release/` :

| Fichier | Usage |
|---|---|
| `ArborisisOS_L1Pro-v2.1.1.uf2` | Firmware principal (Bluetooth + autonome) |
| `ArborisisOS_L1Pro-v2.1.1-ota.zip` | Même firmware, mise à jour Bluetooth (nRF DFU) |
| `ArborisisOS_L1Pro_usb-v2.1.1.uf2` | Variante USB-série (clients web/PC), sans Bluetooth |

**Par câble (UF2)** : branchez l'USB, appuyez **deux fois rapidement sur RESET** : un lecteur
USB apparaît. Glissez-y le fichier `.uf2`. L'appareil redémarre tout seul.

**Par câble, depuis l'ordinateur** (sans toucher au boîtier) :

```bash
~/.platformio/penv/bin/pio run -e ArborisisOS_L1Pro -t upload
```

Le fichier a la même famille UF2 (`0xADA52840`) et la même adresse de départ (`0x27000`) que les
firmwares MeshCore que vous utilisez déjà sur ce boîtier, avec le bootloader
`wio_tracker_l1_bootloader-0.9.2-OTAFIX2.2`.

**Par Bluetooth (OTA)** : maintenir le bouton **Menu** pendant la mise sous tension démarre le
bootloader en mode OTA ; envoyez le `.zip` avec l'appli *nRF Connect* ou *nRF DFU*.

**Données existantes** : ces trois méthodes ne remplacent que le programme. L'identité, les
contacts, les canaux, les réglages et l'historique sont conservés ; l'historique et les réglages
de la 1.x sont convertis au premier démarrage. En 2.1, une erreur de montage du stockage interne
arrête le démarrage avec un diagnostic au lieu de formater les données. Une identité existante
illisible n'est jamais remplacée. Si un ancien firmware utilise un format de contacts différent,
privilégier une sauvegarde/import compatible avant toute réparation du stockage.

**Contacts, canaux et historique au redémarrage** (corrigé en 2.1.1) : ils vivent sur la flash
QSPI externe, que la veille d'énergie met en *deep power-down*. Un redémarrage ou un réveil de
veille profonde ne coupe pas son alimentation : jusqu'en 2.1.0, elle pouvait rester endormie au
démarrage, le pilote lisait du bruit et la **formatait**, d'où des contacts et canaux remis à zéro.
Désormais, au démarrage :

1. la puce est réveillée et on attend la fin d'une écriture coupée par le reset ;
2. son contenu est vérifié **en lecture seule** avant que le pilote n'y touche ;
3. le pilote ne la reçoit que si elle se monte, ou si elle est vierge (carte neuve) ;
4. sinon rien n'est effacé : l'appareil démarre sans contacts enregistrés et ouvre
   **Infos › Stockage**, qui propose **Redémarrer** (nouvel essai) ou **Formater** (confirmé).

Les listes de contacts et de canaux sont aussi écrites à côté de l'ancienne puis échangées
d'un bloc : une coupure pendant une sauvegarde laisse l'ancienne liste ou la nouvelle, jamais
une liste tronquée. Les contacts en attente d'écriture sont enregistrés avant chaque
redémarrage ou mise en veille depuis l'appareil. Sur USB, la flash reste éveillée, pour qu'un
flash par câble trouve toujours une puce qui répond.

**Diagnostic USB de la variante BLE** : ouvrir le port série à 115200 bauds et envoyer `info`
ou `ver` suivi d'un retour à la ligne. La réponse contient la version, le nom, la **clé publique**,
le nombre de contacts, de canaux et l'historique, puis l'état du stockage (`storage=`,
`storage_chip=`, `storage_boot=` — par exemple `woken from deep power-down` — et
`storage_probe=` avec la taille des fichiers de contacts et de canaux trouvés au démarrage) ;
aucune clé privée n'est exportée.

---

## Commandes

| Bouton | Action |
|---|---|
| Joystick haut / bas | naviguer (maintenir = défilement, qui accélère) |
| Joystick gauche | retour (dans les réglages : valeur précédente) |
| Joystick droite | ouvrir (dans les réglages : valeur suivante) |
| Appui joystick (OK) | valider · **appui long** = options |
| Bouton Menu (côté) | retour · **appui long** = accueil · sur l'accueil = éteindre l'écran |

Le premier appui quand l'écran est éteint ne fait que le rallumer. Les confirmations proposent
**Non** (sélectionné) et **Oui** : droite pour choisir Oui, puis OK.

**Accueil** : grosse horloge, date, nouveaux messages (nombre et expéditeur), PIN Bluetooth ou
état de l'appli. OK/bas = menu · droite = messages · gauche = canaux · haut = contacts ·
**appui long OK = SOS**.

**Menu** : Messages · Contacts · Canaux · Radar · GPS · Réseau · SOS · Lampe · Réglages · Infos ·
Énergie · **Aide**. La liste nommée affiche trois entrées : haut/bas pour choisir, gauche/droite
pour changer de page, OK pour ouvrir. **OK long** alterne liste et grille ; le choix est conservé.
Dans la grille, naviguez dans les quatre directions ; l'entrée choisie est nommée en bas.
Un point signale les messages non lus ou un état actif. Le menu conserve votre position au retour.

**Barre d'état** : batterie (pourcentage quand elle baisse), charge, relais actif, GPS, Bluetooth
(inversé quand l'appli est connectée), non-lus, son coupé ou nuit, SOS en cours.

### Écrire un message

Depuis une conversation : **OK** ouvre le clavier ; **droite** ouvre les réponses rapides.
Deux claviers, au choix dans **Réglages › Saisie › Clavier** ou d'un **appui long sur ⇧** :
le **pavé T9** (par défaut depuis la 2.2) et le **clavier complet** (AZERTY / QWERTY / ABC).

#### Pavé T9

```
  ' . , ?   abc    def    ←        ← effacer (long : le mot)
  ghi       jkl    mno    ⇥        ⇥ compléter / mot suivant (long : autre, puis réponses rapides)
  pqrs      tuv    wxyz   ↵        ↵ envoyer
  ⇄         ␣      ⇧      T9       ⇄ autre mot · ␣ espace · ⇧ majuscule · mode T9/Abc/123
```

- **Un appui par lettre** : `2665687` → **Bonjour**. Le mot en cours est souligné ; il change à
  chaque touche, puis se fige à l'espace (ou à la ponctuation). Le curseur reste sur la touche :
  au joystick, 4 touches de lettres sur 8 sont à un pas, les autres à deux.
- **⇄** (en bas à gauche) affiche les autres mots des mêmes touches : `63` → *ne*, *me*, *né* ;
  la touche indique `2/3`, l'**appui long** revient en arrière.
- **⇥ complète** : la pastille montre le mot le plus probable de ces touches (`2665` → *Bonjour*,
  le reste en pastille après le mot). **OK** sur ⇥ le prend avec son espace. Avant toute touche,
  la pastille propose le **mot suivant** (*Je* en début de message, *suis* après *je*).
- **Touche 1** : dans un mot, l'**apostrophe** des élisions (`51` → *j'* ou *l'*, `781` → *qu'*,
  `21` → *c'*) ; la lettre suivante commence le mot d'après (*j'arrive*). Après un mot, la
  **ponctuation** : `.` puis, en appuyant encore, `,` `?` `!` `'` `-` `:` `;` `@`.
- **Le contexte décide** : le mot d'avant (« il » + `2` → *a*, « à » sinon), sa nature (après
  *le* un nom, après *je* un verbe) et vos habitudes ordonnent les mots. Le mot d'**après**
  corrige aussi celui d'avant s'il a été choisi automatiquement : *Je* + *téléphone* devient
  *Le téléphone*, *l'* + *arrive* devient *j'arrive* (jamais un mot choisi avec ⇄ ou ⇥).
- **Mot inconnu** : le mot reste affiché avec les premières lettres des touches et l'aide indique
  « Inconnu : mode Abc ». La **touche de mode** (en bas à droite) passe en **Abc** : appuis
  répétés (`2` deux fois = *b*, puis *c*, *à*, *â*, *ç*, *2* ; `3` donne aussi *é è ê ë*…) ; une
  pause d'une seconde ou un déplacement du joystick valide la lettre. Le mot est appris à l'envoi
  et proposé ensuite en T9.
- **123** : chiffres (⇄ = *0*, ⇧ = *.*). En T9 ou Abc, l'**appui long** sur une touche tape
  son chiffre. **Appui long sur la touche de mode** : symboles, accents et émojis (la touche
  *T9* y ramène au pavé).
- **⇧** : majuscule au mot suivant, verrouillage au deuxième appui ; sur un mot en cours,
  *mot* → *Mot* → *MOT*. Majuscule automatique en début de phrase.
- **Revenir sur un mot** : effacer l'espace qui le suit le reprend comme mot en cours (⇄ pour
  le changer, d'autres touches pour l'allonger).

#### Clavier complet

- Pages : lettres (AZERTY / QWERTY / ABC au choix), `123` chiffres et symboles, `é` accents,
  `☺` emoji et émoticônes.
- **Appui long sur une lettre** = variante accentuée : `e→é`, `a→à`, `u→ù`, `c→ç`, `o→ô`,
  `i→î`, `n→ñ`… (sinon majuscule). Appui long sur `.` = `…`, sur `,` = `;`.
- **Suggestions** : la fin du mot proposé s'affiche dans une pastille après le curseur et la
  touche bulle (en bas à droite) devient une flèche ⇥ : **OK** l'accepte (avec l'espace),
  **appui long** montre le candidat suivant (l'aide affiche « long:2/3 »), puis les réponses
  rapides.
  - **Accents** : tapez sans accents, le mot corrigé s'affiche sur celui tapé (« deja » → « déjà »,
    « foret » → « forêt ») — sauf si ce que vous tapez est déjà un mot (« a », « ou », « la »).
    La casse suit votre saisie (« BONJ » → « BONJOUR »).
  - Une seule lettre ne suffit que pour un mot très probable.
- `⇧` : une fois = une majuscule, deux fois = verrouillage ; **appui long** = pavé T9.
- Bouton **Menu** = effacer le dernier caractère ; appui long sur `←` = effacer le mot.

#### Prédiction (les deux claviers)

- **Dictionnaire français de 20 000 mots** embarqué (formes conjuguées et accordées, élisions,
  mots composés comme *peut-être* ou *aujourd'hui*, quelques abréviations : *ok*, *stp*, *rdv*,
  *mdr*…), avec la fréquence de chaque mot, sa nature (nom, verbe, article…) et **6 000
  enchaînements** fréquents (*je* → *suis*, *bonne* → *nuit*, *il* → *a*). Les mots de randonnée
  et de radio (*refuge*, *sommet*, *relais*, *batterie*…) sont favorisés.
- **Apprentissage** : chaque message envoyé, depuis l'appareil ou depuis l'appli, enrichit un
  lexique personnel (160 mots, 200 enchaînements) classé par fréquence et récence, gardé en
  flash ; il passe avant le dictionnaire. Les noms de contacts et de canaux et les mots des
  conversations récentes complètent. À la première utilisation, il apprend de l'historique.
- **Réglages › Système › Effacer l'historique** efface aussi les mots appris.
- Mesuré sur 7 000 phrases françaises jamais vues (Tatoeba, voir *Simulateur*) : le bon mot
  sort **en premier dans 95 % des cas** (96,6 % une fois le mot suivant tapé), et la saisie
  coûte **1,04 appui par lettre**, espaces compris, grâce à la complétion.

Dans les deux cas : appui long sur **Menu** = quitter en gardant le brouillon ; la touche bulle
(sans suggestion) insère une réponse rapide ou **votre position GPS** ; le compteur en haut à
droite indique les octets restants (160 max, moins votre nom pour un canal), précédé du mode
(`T9`, `Abc`, `123`) sur le pavé.

### Conversations

Vos messages sont à droite, avec une barre verticale, suivis de : `○` en attente, `✓` reçu
(accusé de réception), `!` échec. L'heure s'intercale quand plus de 30 minutes séparent deux
messages (« hier 18:02 », « lun. 28 sept. 14:31 » plus loin). Sur un canal, l'auteur est en
pastille. Les SOS sont en inversé.

Un message direct sans accusé est **renvoyé automatiquement** (3 essais au total par défaut,
le dernier en inondation après réinitialisation du chemin). Le nombre d'essais est réglable de
1 à 5 dans **Réglages › Réseau › Réessais**. Sur le message en échec, le bandeau affiche
**› Renvoyer** : appuyez à droite. Le renvoi manuel crée un nouvel envoi ; les réessais automatiques
gardent le même horodatage pour éviter les doublons chez le destinataire. Si la radio refuse
provisoirement un envoi, les reprises sont espacées de 3, 6, 12 puis 24 secondes.

Sur un **canal**, il n'y a pas d'accusé de réception : les réessais couvrent un refus local de la
radio, et « envoyé » indique une mise en file réussie, sans preuve de réception distante.
Les six envois suivis sont préservés lorsque la file est pleine : le nouveau message reste en brouillon.

**Appui long OK** = détails du message affiché en bas (ou de celui marqué d'un trait à droite si
vous avez remonté la conversation) : expéditeur, heure, chemin (voisin direct, N sauts, routé),
SNR, état et essais ; actions **Répondre** (sur un canal : `@[nom] `), **Renvoyer**, **Citer** ou
**Modifier et renvoyer**, **Y aller** (si le message contient une position), **Supprimer**, puis
**Conversation…** : fiche du contact, y aller, réinitialiser le chemin, **région**, tout marquer
lu, cible des SOS, effacer l'historique, quitter le canal.

### Canaux

« Rejoindre #canal » crée un canal *hashtag* : la clé est dérivée du nom
(`sha256("#nom")`, 16 premiers octets), compatible avec l'appli. Appui long sur un canal
pour ses **options** (région, cible SOS, historique, quitter avec confirmation). Les non-lus apparaissent en pastille, sinon l'âge du dernier message.

### Contacts

Gauche/droite change le filtre : Tous, **Favoris**, Compagnons, Relais, Salons, Capteurs.
**Appui long = favori** (même marque que dans l'appli) : les favoris restent en tête.
Recherche : **haut** depuis le premier contact, puis OK. Le filtre ignore la casse, y compris les accents.
Tri (Réglages › Réseau › Contacts) : récents, A-Z ou **proches** (distance, si votre position est connue).
La fiche indique aussi la distance et la direction, avec **Y aller**.

---

## Radar, navigation et GPS

**Radar** (menu) : les contacts qui partagent leur position, votre repère et la destination en
cours, nord en haut, à une échelle qui s'adapte (Ø = rayon). Haut/bas = point suivant (du plus
proche au plus lointain), OK = **y aller**, appui long OK = **marquer ma position comme repère**.
Le radar allume le GPS le temps de l'écran.

**Navigation** : une flèche vers la destination, la distance, le cap (« NE 45° ») et le temps
restant. À l'arrêt, le haut de l'écran est le nord ; en marchant, la flèche est relative à votre
sens de marche (calculé par le GPS : le boîtier n'a pas de boussole). L'écran reste allumé ;
« Arrivé ! » à moins de 15 m. On y arrive depuis le radar, une fiche contact, un message ou un
SOS contenant une position, ou GPS › Retour au repère.

**GPS** : mode off / continu / éco (5 à 60 min), partage de la position dans les adverts, fix et
satellites, position, altitude, **vitesse et cap**, **trajet** (distance parcourue, OK pour
remettre à zéro), dernier fix, **Marquer ce point** et **Retour au repère**.

## SOS

Accueil › appui long OK, ou menu › SOS. Choisissez le destinataire (un canal ou un contact favori,
par défaut le premier canal) et la répétition (2, 5, 10 ou 15 min), puis **maintenez OK sur
DÉCLENCHER**. Le message part tout de suite et à chaque période :
`SOS ! Besoin d'aide 📍45.92365,6.86933 batt 65%` (« position inconnue » sans GPS, et l'âge de la
position si elle date). Pendant l'alerte, le GPS reste allumé, le relais se met en pause et
« SOS » clignote dans la barre d'état. Appui long OK dans l'écran SOS pour l'arrêter : le message
« SOS terminé : tout va bien. » est envoyé. Si la batterie lâche, une dernière alerte part avant
l'extinction.

Un **SOS reçu** (message qui commence par « SOS » ou contient 🆘) sonne même si le son est coupé ou
la nuit, s'affiche en grand, et **droite** lance la navigation vers sa position.

## Lampe

Écran blanc à pleine luminosité. Gauche/droite : fixe, clignotant, **SOS en morse** ; OK :
allumer/éteindre. L'écran ne se met pas en veille tant que la lampe est ouverte.

---

## Réseau

**Adverts automatiques** (Réglages › Advert auto : off, 1, 3, 6, **12 h** par défaut, 24 h) : un
advert de voisinage une minute après le démarrage, un advert réseau dix minutes après, puis à la
période choisie. **Si déplacé** : quand votre position est partagée, un advert réseau part dès
que vous vous êtes déplacé de plus de 2 km (au plus un tous les quarts d'heure). L'heure est
enregistrée à chaque advert, pour que les suivants restent plus récents même après un
redémarrage (sinon les autres nœuds les ignorent). Réseau › Dernier : il y a combien de temps.

**Relais** (Réseau › Relais) : l'appareil retransmet les messages des autres, avec les mêmes
garde-fous qu'un relais MeshCore (limite de sauts, détection de boucle, pause tant qu'il attend
l'accusé d'un de vos messages). Modes : **auto** (par défaut : actif sur USB, ou batterie au-dessus
de 60 %, jusqu'à ce qu'elle descende sous 35 %, en pause pendant un SOS), oui, non. L'écran
montre l'état et sa raison, les paquets relayés et refusés, et le nombre de sauts maximum.

**Régions** (Réseau › Régions) : les « portées » MeshCore. Un message en inondation porte un code
calculé avec la clé de sa région (`sha256("#nom")`) et les relais ne retransmettent que les
régions qu'ils acceptent. Ajoutez des régions par leur nom (la première devient celle de la
radio), choisissez celle de la radio (gauche/droite ou OK sur une région), oubliez-en une par
appui long. Chaque **canal** peut avoir **sa propre région** (**Canaux › OK long › Région** : défaut,
aucune, ou une région connue), et les messages privés peuvent également avoir un choix spécifique.
Le choix du canal est conservé avec sa clé, indépendamment de son nom ou de l'ordre des canaux,
et s'applique également aux textes et données envoyés par l'appli compagnon. Modifier un canal
ne change pas les autres canaux ni la région globale. L'écran compte aussi le **trafic entendu** par région, sans
région, ou dans une région inconnue. La région de la radio est la même que celle réglée par
l'appli.

**Statistiques** : paquets reçus/émis, erreurs, flood et direct, temps d'antenne et taux
d'occupation, dernier RSSI/SNR, bruit, prochaine calibration, **taux d'occupation du canal**
mesuré avant nos envois, file d'envoi.

---

## Horloge

Le boîtier n'a pas de puce horloge sauvegardée. Arborisis OS :

- reprend au démarrage la dernière heure connue (enregistrée toutes les heures, à chaque advert
  et à l'extinction) : elle ne peut que rester en retard, jamais repartir en 2024 ;
- prend l'heure de l'appli ou du GPS ;
- **Heure via GPS** (activée par défaut) : tant que l'heure est inconnue, le GPS s'allume, quel
  que soit son mode, jusqu'à un fix solide (3 minutes au plus, ciel dégagé conseillé) ; sans
  fix, il réessaie après 15, 30, 60 puis toutes les 120 minutes. Une fois l'heure connue, il la
  recale une fois par jour (sauf en profil réserve). Un GPS déjà allumé (mode continu, éco,
  radar, SOS) recale l'heure tout seul ;
- **Heure via réseau** (activée par défaut) : tant que l'heure est inconnue, deux nœuds
  différents d'accord à 10 minutes près la fixent (on garde la plus ancienne des deux) ;
- **Réglages › Heure › Régler** : date et heure à la main (en avant seulement, comme l'appli :
  les autres ignoreraient des adverts antérieurs).

Infos › Heure indique d'où elle vient (appli, GPS, réseau, manuelle).

---

## Mode compagnon (appli MeshCore)

- Appairage : le **PIN** s'affiche sur l'accueil (nouveau à chaque démarrage tant qu'aucun
  PIN fixe n'est défini dans l'appli).
- Quand l'appli est connectée, l'appareil ne sonne pas et n'affiche pas de popup (le téléphone
  s'en charge, sauf pour un SOS) ; les messages sont quand même enregistrés sur l'appareil.
- Les messages envoyés **depuis l'appli** apparaissent aussi dans l'historique de l'appareil.
- Limite du protocole MeshCore : les messages tapés **sur l'appareil** ne remontent pas dans
  l'historique de l'appli (le protocole ne prévoit pas ce cas).
- Le GPS réglé depuis l'appli est suivi par l'appareil, et inversement ; de même pour la région
  de la radio et les favoris.

---

## Optimisations d'autonomie

**Optimisation auto** (activée par défaut, désactivable dans **Réglages › Automatique** ou
**Énergie › Optim. auto**) adapte les réglages effectifs, sans modifier vos préférences :

| Profil | Entrée / sortie | Effet |
|---|---|---|
| Normal | USB ou batterie remontée à 35 % | Réglages choisis ; écran réduit à faible luminosité après deux tiers du délai de veille |
| Éco | Batterie ≤ 25 %, sortie à 35 % | Veille au maximum 15 s, luminosité au maximum 2/5, GPS éco au moins 30 min |
| Réserve | Batterie ≤ 10 %, sortie à 15 % | Veille au maximum 10 s, luminosité 1/5, GPS éco au moins 60 min ; GPS continu utilisé par cycles hors navigation/SOS |

La saisie conserve un délai d'au moins 60 s. Navigation, lampe et SOS gardent leurs priorités.
Le GPS désactivé reste désactivé, hormis les actions qui demandent explicitement une position.
Un cycle GPS éco est limité à deux minutes ; sans fix, l'intervalle double jusqu'à une heure.
Les adverts automatiques attendent les envois suivis et le SOS, et, avec l'optimisation activée,
un canal très occupé ou la fin du profil réserve. Les adverts manuels restent disponibles.
**Énergie** affiche le profil, le délai de veille effectif, la période GPS et la file suivie.

Gains **estimés** par rapport au firmware MeshCore compagnon d'origine sur ce boîtier (écran
éteint, appli non connectée). Ils sont déduits du schéma, de la documentation Nordic et des
mesures Seeed ; ils restent à mesurer (PPK2 ou multimètre en série sur la batterie).

| Optimisation | Détail | Gain estimé |
|---|---|---|
| Flash QSPI en veille | Errata nRF52840 n°122 : le contrôleur QSPI consomme ~630 µA tant qu'il reste activé ; il est maintenant coupé 0,4 s après chaque accès, et la puce flash passe en *deep power-down* (réveillée au démarrage, avant un redémarrage et en permanence sur USB) | ≈ 0,6 mA |
| UART GPS arrêté | Le GPS éteint laissait l'UART en réception (~900 µA d'après le cœur Adafruit) | ≈ 0,9 mA |
| Pont diviseur batterie | `BAT_ADC_CTR` restait à 1 : 10 kΩ + 10 kΩ en permanence sur la batterie ; il n'est alimenté que pendant la mesure (1 fois/min) | ≈ 0,19 mA |
| Veille CPU « tickless » | Écran éteint : le processeur dort 25 ms par cycle au lieu d'être réveillé 1 024 fois/s | ≈ 0,1–0,2 mA |
| Bluetooth | Annonces lentes à 1 022,5 ms (au lieu de 152,5 ms) après 30 s ; latence de liaison 8 quand l'appli est connectée | ≈ 0,05–0,1 mA |
| Écran | Contraste réglable (défaut 2/5 au lieu du maximum), veille 20 s, envoi I²C des seules zones modifiées, messages de canal sans réveil de l'écran | fort, écran allumé (Seeed : +8 mA écran allumé) |
| GPS « éco » | Un point toutes les N minutes puis veille (au lieu de 53 mA en continu) | ≈ 53 mA → ≈ 0,9 mA à 15 min |
| Gain RX | Option « éco » (sans *boosted gain*) : −0,7 mA, sensibilité légèrement réduite | optionnel |
| Veille CPU écran allumé | Écran allumé mais sans appui depuis 2 s : même veille de 25 ms par cycle (un appui reste pris en compte en moins de 25 ms) | ≈ 0,1–0,2 mA, écran allumé |
| Calibration adaptative | Les rondes périodiques ne mesurent que le bruit (la puce n'est réinitialisée que toutes les 6 h ou après un changement de réglage), et s'espacent jusqu'à 1 h quand le bruit est stable | moins d'interruptions de la réception |
| Historique compact | Messages stockés à leur longueur réelle : 5× plus d'historique pour 10 Ko de RAM de plus ; sauvegarde par fichier temporaire (une coupure pendant l'écriture garde l'ancien) | — |

Ordre de grandeur (batterie 2 000 mAh utilisable à 90 %, écran éteint, sans émission) :
≈ 7,5 mA → ≈ 5,5 mA, soit environ **10 → 13–14 jours** ; ≈ 15 jours avec le gain RX « éco ».
La réception LoRa continue (≈ 5 mA) reste le poste principal.

Le **relais automatique** consomme de l'émission pour les autres : c'est pourquoi, en mode auto,
il ne fonctionne que sur USB ou avec une batterie bien chargée.

**Veille profonde** (Énergie › Veille profonde) : MCU en *System OFF*, radio en sommeil,
GPS en veille. Réveil : **maintenir OK ~1 s** (un appui bref, par exemple dans une poche,
laisse l'appareil en veille). Pour un stockage long, préférez l'interrupteur matériel.

---

## Réglages

Par catégories (OK pour entrer, bouton Retour pour revenir) :

- **Appareil** : nom du nœud, Bluetooth.
- **Écran** : veille (10 s à 5 min), luminosité (1–5), **retourner** (écran et joystick à 180°).
- **Saisie** : clavier (T9 / AZERTY / QWERTY / ABC), majuscule automatique, réponses rapides (8, éditables).
- **Alertes** : son, son des canaux, réveil par les canaux, **nuit 22h-7h** (ni son ni lumière,
  sauf SOS), LED des non-lus.
- **Heure** : fuseau (Paris, Londres, Athènes avec heure d'été UE automatique, ou UTC±N), **régler**,
  **via réseau**, **via GPS**.
- **Réseau** : advert auto, si déplacé, tri des contacts, nombre de réessais.
- **Automatique** : optimisation auto, menu liste/grille, profil actif.
- **Système** : effacer l'historique, **réglages par défaut** (les réponses rapides sont gardées),
  redémarrer.

Réseau › Radio : fréquence, modulation, puissance TX, gain RX, bruit, **région**, préréglages
(EU/UK Narrow 869,618 MHz / 62,5 kHz / SF8 / CR8, EU/UK Long Range ancien, USA/Canada).

### Calibration automatique et CAD

Même code que le Heltec V3 (`src/helpers/radiolib/ArborisisCompanionRadio.h`,
`src/helpers/CompanionCalibration.h`) :

- calibration du SX1262 ~15 s après le démarrage puis **période adaptative** : 15 min, doublée
  jusqu'à 1 h tant que le bruit ne bouge pas (±2 dB), ramenée à 5 min quand il change (vous vous
  déplacez) ; bruit mesuré sur 64 échantillons en réception calme (quartile bas), mesure instable
  ou signal fort continu (> −80 dBm) rejetés, dernier bruit validé gardé ; la puce elle-même n'est
  recalibrée qu'au démarrage, après un changement de fréquence, BW/SF/CR ou gain RX, et toutes les 6 h ;
- avant chaque émission, **CAD matériel** avec un seuil RSSI de bruit + 12 dB : un canal occupé
  garde le message en file jusqu'à ce qu'il se libère, sans émission forcée ;
- **CAD adaptatif** : l'attente avant de réessayer passe de 200 ms à 400 ou 800 ms quand le canal
  a été souvent occupé dans la dernière minute (40 % / 75 %), avec un tirage aléatoire pour que
  les nœuds bloqués par la même émission ne repartent pas tous en même temps.

Réseau › Radio affiche `Bruit calib… CAD` puis `Bruit -1xx dBm CAD` ; Réseau › Statistiques donne
la prochaine calibration, la période en cours et le taux d'occupation du canal. Tests sur
ordinateur : `sh tools/test-heltec-companion.sh`.

L'historique (jusqu'à 400 messages, 24 Ko) est enregistré sur le flash externe avec un débit
d'écriture limité (au plus toutes les 10 min, ou 30 min si ça n'arrête pas), et à l'extinction.

Rescue : appui long sur **Menu** pendant l'écran de démarrage → CLI rescue MeshCore sur
l'USB série (115 200 bauds).

---

## Compiler

```bash
~/.platformio/penv/bin/pio run -e ArborisisOS_L1Pro
```

```bash
./tools/release.sh
```

Sur Mac Apple Silicon sans Rosetta, la cible utilise le toolchain `gcc-arm-none-eabi 12.3`
(le 7.2.1 par défaut de PlatformIO n'existe qu'en x86_64).

### Simulateur

`tools/sim/build.sh` compile la vraie interface (`examples/companion_radio/ui-arbo`) pour le
Mac avec des bouchons matériels, la pilote au joystick via le même code de boutons que
l'appareil, vérifie le comportement (texte, heure, géographie, historique,
clavier, envois et renvois, menus, GPS, radar, navigation, SOS, relais, régions, adverts,
horloge, veille…) et écrit une capture PNG de chaque écran dans `tools/sim/out/`. Il faut avoir
compilé le firmware une fois (pour la police de l'écran).

Le dictionnaire du clavier (`ui-arbo/DictData.cpp`) est généré par `tools/dict/gen_dict.py`
(téléchargement des sources au premier lancement, dans `~/.cache/arborisis-dict`). Il garde les
phrases Tatoeba numérotées en multiple de 10 hors de l'apprentissage, pour mesurer le T9 sur des
phrases jamais vues :

```bash
python3 tools/dict/gen_dict.py --words 20000 --pairs 6000 --eval /tmp/t9_eval.txt
T9_EVAL=/tmp/t9_eval.txt T9_EVAL_N=7000 ./tools/sim/arbo_sim
```

Sources des données : [Lexique 3.83](http://www.lexique.org) (B. New, C. Pallier — formes,
fréquences dans les sous-titres, nature des mots), FrequencyWords fr 2018 (H. Dave, d'après
OpenSubtitles 2018), toutes deux sous licence CC BY-SA 4.0, et les phrases françaises de
[Tatoeba](https://tatoeba.org) (CC BY 2.0 FR) pour les enchaînements. Les tables générées en
dérivent et sont distribuées sous CC BY-SA 4.0.

Les icônes, le logo et les gros chiffres sont générés par `tools/assets/gen_assets.py`
(icônes dessinées en code, chiffres de la police libre FreeSans Bold d'Adafruit GFX).

```bash
./tools/sim/build.sh
# Même parcours avec AddressSanitizer + UndefinedBehaviorSanitizer
SIM_SANITIZE=1 ./tools/sim/build.sh
```

`tools/test-fsprobe.sh` teste la protection du stockage sur le vrai code littlefs du cœur
nRF52 : verdicts de la sonde de démarrage (vierge, valide, puce endormie, superbloc abîmé,
contenu étranger), puis une coupure de courant injectée à chaque écriture flash d'une
sauvegarde de contacts. La sauvegarde atomique garde toujours une liste entière ; l'écriture
d'origine perd la liste sur presque chaque coupure.

```bash
./tools/test-fsprobe.sh
```

---

## Organisation du code

| Chemin | Rôle |
|---|---|
| `variants/arborisis-l1pro/` | Carte : brochage complet (dont charge `P1.15`/`P1.13`, reset GNSS `P1.06`), batterie, veille, pilote SH1106 différentiel, veille QSPI et vérification de la flash au démarrage (`ArboQspiGate`, `ArboFsProbe`), `platformio.ini` |
| `examples/companion_radio/ui-arbo/` | Interface : écrans (`Screens*.cpp`), claviers T9 et complet (`Keyboard.cpp`), prédiction et T9 (`Predict.cpp`), dictionnaire compressé (`Dict.cpp`, `DictData.cpp` généré), historique (`MsgStore`), préférences, géographie (`Geo`), dessin (`Gfx`, `Assets` générés), texte UTF-8 → CP437 + icônes emoji |
| `tools/dict/gen_dict.py` | Génère le dictionnaire : 20 000 mots en ~24 bits chacun (codage de Huffman, préfixes partagés, points d'entrée toutes les 64 entrées), natures de mots et enchaînements |
| `src/helpers/CompanionCalibration.h`, `src/helpers/radiolib/ArborisisCompanionRadio.h` | Calibration adaptative du bruit, CAD et recul adaptatif (partagés avec le Heltec V3) |
| `examples/companion_radio/MyMesh.*`, `AbstractUITask.h` | Points d'accroche ajoutés à MeshCore (historique, envoi, canaux, radio, favoris, régions, relais, paquets entendus) |
| `src/helpers/nrf52/SerialBLEInterface.cpp` | Délais BLE rendus configurables (valeurs d'origine inchangées pour les autres cibles) |
| `tools/sim/` | Simulateur hôte |

---

## À vérifier sur l'appareil

Le flash, le démarrage et le chargement de l’identité existante ont été vérifiés sur le L1 Pro.
Les scénarios suivants demandent encore une validation matérielle dédiée :

1. **Consommation réelle** de chaque optimisation (surtout la veille QSPI et la veille profonde).
2. **Flash QSPI** : contacts et canaux conservés après **Redémarrer**, après une veille
   profonde et sur batterie seule ; `info` sur le port série doit indiquer `storage=mounted`
   (Infos › Stockage affiche l'état, l'état au démarrage et le nombre de réveils).
3. **Indicateur de charge** : polarité des broches `CHG` du CN3165 sur batterie seule
   (un éclair affiché en permanence sans USB indiquerait l'inverse).
4. **GNSS L76K** : passage en veille via `WAKE_UP` (même méthode que les firmwares d'origine),
   et **heure via GPS** : en mode GPS « off », l'heure est réglée en extérieur dans les minutes
   qui suivent le démarrage (Infos › Heure : « GPS »).
5. **Orientation du joystick** : haut/bas/gauche/droite suivent les noms du schéma.
6. **Calibration et CAD** : le bruit validé s'affiche dans Radio, et les messages partent
   bien avec un second nœud à portée (les tests sur ordinateur ne couvrent pas la RF réelle).
7. **Relais** : consommation en mode actif, et bascule auto au branchement/débranchement.
8. **Régions** : avec un relais MeshCore configuré pour une région, un message scopé passe et
   le trafic entendu se répartit bien par région.
9. **Rotation, lampe et luminosité maximale**, **GPS** : vitesse et cap en marchant, navigation.
10. **Réactivité du T9** : chaque touche relit une partie du dictionnaire en flash (de 1 000
   à 5 000 mots selon la première touche, quelques dizaines de millisecondes estimées) ; la frappe
   doit rester fluide, y compris avec 150 messages d'historique et des centaines de contacts.
