# Arborisis — firmwares MeshCore

Firmwares Arborisis OS publiés pour le Wio Tracker L1 Pro, la Heltec V3 et le Mio Cyclo. Flash depuis le navigateur, guides et outils sur **[arborisis.com](https://arborisis.com)**.

<p><img src="screens/l1pro/t9-sentence.png" width="384" alt="Clavier T9 du Wio L1 Pro"> <img src="screens/l1pro/menu-grid.png" width="384" alt="Menu en grille du Wio L1 Pro"></p>

## Dernières versions

| Appareil | Version | Validation | Description |
|---|---|---|---|
| **Wio Tracker L1 Pro** | [2.2.0](https://github.com/0xmagicduck/arborisis-firmware/releases/tag/l1pro-v2.2.0) | Testée sur l’appareil | Messagerie autonome au joystick, clavier T9 et dictionnaire français, GPS, radar, SOS. Fonctionne seul ou avec l’appli MeshCore. |
| **Heltec V3 · relais + compagnon** | [1.0.0](https://github.com/0xmagicduck/arborisis-firmware/releases/tag/heltec-v1.0.0) | Testée en simulateur | Relais MeshCore et compagnon de l’appli en même temps, en Bluetooth et en USB. Variantes Apple Watch et Mio Cyclo. |
| **Heltec V3 · répéteur** | [1.1.0](https://github.com/0xmagicduck/arborisis-firmware/releases/tag/repeater-v1.1.0) | Testée en simulateur | Répéteur MeshCore dédié avec calibration automatique, CAD adaptatif et administration par le mesh. |
| **Mio Cyclo · appli MeshCore** | [1.0.0](https://github.com/0xmagicduck/arborisis-firmware/releases/tag/mio-v1.0.0) | Testée en simulateur | L’appli MeshCore sur le compteur vélo Mio Cyclo Discover Connect, reliée au Heltec V3 par Wi-Fi. |

L’application Apple Watch s’installe depuis Xcode ([guide](guides/watch.md)) avec la variante **Apple Watch** du firmware Heltec V3.

## Installer

- **Depuis le navigateur** : [arborisis.com/#flash](https://arborisis.com/#flash) (Chrome ou Edge sur ordinateur, USB). Le site télécharge les fichiers de ces releases et vérifie leur SHA-256 avant d’écrire.
- **Wio L1 Pro** : double appui sur RESET, puis copier le `.uf2` sur le disque de démarrage. Le bootloader, l’identité MeshCore et les données sont conservés.
- **Heltec V3** : `esptool.py --chip esp32s3 write_flash 0x0 <fichier>-merged.bin`.

Chaque release contient `SHA256SUMS` et `manifest.json` (adresses de flash, tailles, sommes, statut de validation). Les guides détaillés sont dans [`guides/`](guides).

## Toutes les releases

- [`mio-v1.0.0`](https://github.com/0xmagicduck/arborisis-firmware/releases/tag/mio-v1.0.0) — Mio Cyclo · appli MeshCore 1.0.0, Appli MeshCore pour Mio Cyclo (2026-09-29)
- [`repeater-v1.1.0`](https://github.com/0xmagicduck/arborisis-firmware/releases/tag/repeater-v1.1.0) — Heltec V3 · répéteur 1.1.0, Calibration automatique et CAD adaptatif (2026-10-01)
- [`repeater-v1.0.0`](https://github.com/0xmagicduck/arborisis-firmware/releases/tag/repeater-v1.0.0) — Heltec V3 · répéteur 1.0.0, Répéteur dédié (2026-10-01) · remplacée
- [`heltec-v1.0.0`](https://github.com/0xmagicduck/arborisis-firmware/releases/tag/heltec-v1.0.0) — Heltec V3 · relais + compagnon 1.0.0, Relais + compagnon, calibration automatique et CAD (2026-10-01)
- [`l1pro-v2.2.0`](https://github.com/0xmagicduck/arborisis-firmware/releases/tag/l1pro-v2.2.0) — Wio Tracker L1 Pro 2.2.0, T9 et dictionnaire français (2026-10-04)
- [`l1pro-v2.1.0`](https://github.com/0xmagicduck/arborisis-firmware/releases/tag/l1pro-v2.1.0) — Wio Tracker L1 Pro 2.1.0, Ultimate (2026-10-04) · remplacée
- [`l1pro-v1.0.0`](https://github.com/0xmagicduck/arborisis-firmware/releases/tag/l1pro-v1.0.0) — Wio Tracker L1 Pro 1.0.0, Première version (2026-09-29) · remplacée

## Licences

Basé sur [MeshCore](https://github.com/meshcore-dev/MeshCore) v1.17.1 de Scott Powell et ses contributeurs, licence MIT ([LICENSE](LICENSE)).
Le dictionnaire du Wio L1 Pro 2.2 dérive de Lexique 3.83 (B. New, C. Pallier, CC BY-SA 4.0), FrequencyWords 2018 (H. Dave, OpenSubtitles, CC BY-SA 4.0) et Tatoeba (CC BY 2.0 FR) ; les tables générées sont distribuées sous CC BY-SA 4.0.
