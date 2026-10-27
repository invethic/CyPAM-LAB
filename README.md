# PhotoCoque Lab — mini-usine de coques personnalisées (BTS ATI 2)

Inspiré du kiosque « Coque personnalisée » de Photomaton (choisir sa photo, personnaliser sa coque
parmi 60+ modèles, la récupérer en 3 minutes), ce labo transpose le service dans une **mini-usine de 6 postes**
sur laquelle **5 équipes** relèvent **5 défis** d'organisation industrielle :

| Équipe | Défi | Onglet de la supervision |
|---|---|---|
| 1 | Équilibrage de la ligne (goulot P4 75 s > takt 60 s) | Défi 1 · Équilibrage |
| 2 | Réduction des en-cours (Kanban, CONWIP, loi de Little) | Défi 2 · Kanban |
| 3 | Mesure et amélioration du TRS | Défi 3 · TRS |
| 4 | Plan de maintenance préventive (P2 imprimante, P4 robot) | Défi 4 · Maintenance |
| 5 | Maîtrise de la qualité visuelle (Pareto, carte p) | Défi 5 · Qualité |

Le 6e défi (stocks de coques brutes) est reporté à un TP spécifique.

## La ligne

| Poste | Désignation | Stations de l'infographie | TC départ |
|---|---|---|---|
| P1 | Alimentation & identification | 1 + 2 | 35 s |
| P2 | Impression du visuel | 3 | 50 s |
| P3 | Découpe du sticker | 4 | 40 s |
| P4 | Pose robot & marouflage | 5 + 6 | 75 s (goulot) |
| P5 | Contrôle qualité caméra | 7 | 30 s |
| P6 | Emballage & sortie | 8 | 25 s |

Demande 420 coques/jour, ouverture 7 h → takt 60 s.

## Contenu

```
mosquitto/   mosquitto.conf (1883 + WebSockets 9001), acl.conf, README_mosquitto.md
nodered/     flows_photocoque.json (à importer), src/*.js (code des fonctions en clair), build_flows.py
web/         index.html (supervision), kiosque.html (commande client), pupitre.html (postes pilotés à la main en mode RÉEL), lib/mqtt.min.js
firmware/    PC_Kiosque, PC_Poste, PC_Qualite, PC_Maintenance (Tab5, Arduino)
```

## Démarrage rapide (un PC par équipe)

1. `mosquitto -c mosquitto/mosquitto.conf -v`
2. `node-red` → http://localhost:1880 → ☰ Importer → `nodered/flows_photocoque.json` → Déployer
3. Ouvrir `web/index.html` → Se connecter (localhost) → onglet *Simulateur & réglages* → **Démarrer**
4. Commandes depuis un téléphone : `python -m http.server 8080` dans `web/`, puis `http://<IP-PC>:8080/kiosque.html`

Le journal de la session se télécharge depuis la supervision (`http://<PC>:1880/photocoque/journal.csv`) ;
chaque « Nouveau départ » crée un nouveau fichier dans `journaux/` → comparaison avant / après dans Excel.

## Tester un ordre de production

Voir `Procedure_test_ordre_PhotoCoque.docx` : test A en mode SIM (commandes automatiques désactivées, vitesse ×5, ≈ 1 min), test B en mode RÉEL avec les Tab5 et/ou `web/pupitre.html` (actionner P1 → P6 dans l'ordre).

## Modes

- **SIM** : le simulateur Node-RED fabrique les coques (vitesse ×1 à ×60, ×60 ≈ une journée en 7 min).
- **RÉEL** : les Tab5 de la ligne publient les mêmes messages ; Node-RED calcule les mêmes indicateurs.

## Topics MQTT

| Topic | Contenu |
|---|---|
| `photocoque/commande/new` | `{cmd, modele, visuel, src}` |
| `photocoque/poste/Px/piece` | `{cmd, ct}` une coque sort du poste |
| `photocoque/poste/Px/etat` | `{etat: RUN/ATTENTE/BLOQUE/PANNE/MAINT, cause}` |
| `photocoque/qualite/resultat` | `{cmd, verdict: OK/NOK, defaut}` |
| `photocoque/maintenance/Px/vibration` | `{rms}` en mm/s |
| `photocoque/kpi` (retenu) | indicateurs calculés, toutes les 2 s |
| `photocoque/config` (retenu), `/config/set`, `/config/defaut` | configuration commune |
| `photocoque/sim/cmd` | `start`, `pause`, `reset`, `maintenance`, `action` |

## Parcours cybersécurité (GI-CNAM)

Mêmes flux : capture Wireshark en clair, puis `acl.conf` + mots de passe, puis MQTTS (section commentée
dans `mosquitto.conf`). Les anciennes captures `capture_wireshark_CPLab400` utilisent les anciens topics du CP Lab 400 :
à refaire avec `photocoque/…`.

Documents associés : `Guide_PhotoCoque_Lab_BTS_ATI2.docx`, `Dossier_defis_PhotoCoque_BTS_ATI2.xlsx`.
