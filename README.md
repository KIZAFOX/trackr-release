# Trackr

Un overlay et un dashboard pour League of Legends, développés avec Electron.
J’ai rassemblé au même endroit les infos du client, les données en partie et
quelques outils autour des sessions et des clips.

## Ce que fait l’application

- **Dashboard** : profil, rang et dernières parties. Si le client League n’est
  pas ouvert, Trackr affiche un état d’attente et réessaie automatiquement.
- **Partie en cours** : sélection des champions, puis informations sur les
  deux équipes pendant la partie (rangs, maîtrises, runes et score d’équipe).
- **Overlay en jeu** : CS/min, gold/min, kill participation, vision/min,
  timers d’objectifs et estimation de différence de gold.
- **Fin de partie** : résultat, statistiques et historique de session.
- **Historique** : filtres par session, champion, file et résultat, avec comparaison des performances entre sessions.
- **Clips** : capture d’événements ou d’une partie entière, avec lecture dans
  le dashboard.
- **Réglages** : affichage des blocs de l’overlay, clips, compte League et
  préférences générales.

## Installation

Le guide utilisateur et mainteneur est dans [docs/INSTALLATION.md](docs/INSTALLATION.md).
Il couvre l'installateur Windows, les mises à jour, le dépannage, `.env` local
et le flux de publication.

## Démarrage en développement

Trackr cible Windows : la détection du client utilise la ligne de commande de
`LeagueClientUx.exe`.

Prérequis : Node.js 20+ et npm installés.

```powershell
npm install
npm start
```

Pour les statistiques Riot enrichies, copie `.env.example` vers `.env` à la racine
et renseigne `RIOT_API_KEY` et `DEFAULT_REGION`. Le fichier `.env` est ignoré par
Git et n'est jamais embarqué dans l'installeur. Sans clé, le dashboard et les
infos de base de partie restent utilisables, mais les rangs et maîtrises
détaillés ne sont pas disponibles.

## Construire l'application

```powershell
npm run dist      # installeur Windows dans dist/, sans publication
npm run release   # build + publication GitHub (GH_TOKEN, CSC_LINK, CSC_KEY_PASSWORD requis)
```

Incrémente `version` dans `package.json` avant une release. La construction doit
se faire sous Windows (ou une CI Windows). L'installeur est un assistant NSIS :
choix du dossier d'installation, puis écran de progression avec la liste des
fichiers copiés (voir `build/installer.nsh` et la section `build.nsis` de `package.json`).

## Benchmarks

Les moyennes par rôle sont calculées avec Match-V5 à partir de parties Ranked
Solo/Duo récentes de joueurs Master, Grandmaster et Challenger. Un workflow
GitHub Actions les régénère chaque lundi. Il vit dans le dépôt privé
`KIZAFOX/trackr-benchmark-automation` ; seul `benchmarks.json` est publié sur
la branche `gh-pages` de `KIZAFOX/trackr-release`.

Trackr télécharge le JSON au démarrage puis toutes les 12 h, et en garde une copie
locale pour continuer à fonctionner hors ligne. Les benchmarks sont des moyennes de
fin de partie, pas des courbes détaillées par minute.

Le dépôt d'automatisation se clone dans `benchmark-automation/` (dossier ignoré par
Git, hors de l'application et de l'installeur) :

```powershell
git clone https://github.com/KIZAFOX/trackr-benchmark-automation.git benchmark-automation
```

## Structure

```text
src/
  main/            Process principal Electron
    index.js         Point d'entrée : cycle de vie et assemblage
    settings.js      Réglages (fusion profonde, écriture atomique)
    liveGame.js      Boucle « partie en cours » : overlay + déclencheurs de clips
    benchmarkSync.js Benchmarks distants + cache local
    updater.js       Mises à jour automatiques
    windows/         Une fabrique par fenêtre (splash, dashboard, overlay, recorder, tray)
    ipc/             Routes IPC par domaine (app, préférences, clips, client League, joueurs live)
  preload/         Ponts sécurisés renderer <-> main (un par type de fenêtre)
  core/            Logique métier sans Electron (testable seule)
    riot/            API locale du client (lcu), API publique Riot + Data Dragon (external)
    gameflowWatcher, champSelect, endOfGame, liveStats, livePlayers, benchmarks...
  shared/          Modules UMD utilisés à la fois par Node et par les pages (positions, historique, phases)
  renderer/        Interfaces : dashboard/, overlay/, splash/, recorder/
assets/            Icône de l'application
build/             Icône Windows (.ico) et script NSIS de l'installeur
tools/             Signature Windows, vérification de release, mesure de performance
test/              Tests automatisés (node --test)
docs/              Installation, checklist de release, TODO
```

## Vérifications

```powershell
npm test
```

## Notes techniques

- Une seule instance de Trackr tourne à la fois : relancer l'app ramène le dashboard au premier plan.

- Les données live viennent de l’API locale Riot `https://127.0.0.1:2999`.
- La différence de gold des autres joueurs est une estimation : Riot ne fournit
  leur gold exact via l’API Live Client Data.
- Les clips sont sauvegardés en `.webm` dans `Vidéos/Trackr Clips` par défaut.
  La capture de fenêtre peut revenir à l’écran principal si la fenêtre League
  n’est pas détectée.
- Le projet reste en lecture seule vis-à-vis du jeu : pas d’injection ni de
  modification de mémoire. Avant toute distribution publique, je vérifie la
  conformité avec la politique développeur Riot.

## Pistes pour la suite

- Ajouter des références de benchmark par durée de partie.
- Compléter les événements de clips (multi-kills, Ace, First Blood) et les
  timers d’objectifs.
- Étudier le choix de l'écran de capture, les profils par mode et le partage de
  clips vers Discord.