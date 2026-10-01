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
- **Clips** : capture d’événements ou d’une partie entière, avec lecture dans
  le dashboard.
- **Réglages** : affichage des blocs de l’overlay, clips, compte League et
  préférences générales.

## Démarrage

Trackr cible Windows : la détection du client utilise la ligne de commande de
`LeagueClientUx.exe`.

Prérequis : Node.js et npm installés.

```powershell
npm install
npm start
```

Pour les statistiques Riot enrichies en développement, ajoute `RIOT_API_KEY`
et `DEFAULT_REGION` dans un fichier `.env` à la racine du projet. Le fichier
`.env` est ignoré par Git et ne doit jamais être ajouté à une release. Sans clé,
le dashboard et les infos de base de partie restent utilisables, mais les
rangs et maîtrises détaillés ne sont pas disponibles.

## Construire l’application

Pour créer l’installeur Windows :

```powershell
npm run dist
```

Le résultat est placé dans `dist/`. La construction Windows doit être faite
sous Windows ou sur une machine CI Windows.

Pour publier une release, incrémente `version` dans `package.json`, puis lance :

```powershell
npm run release
```

Cette commande utilise `electron-builder` et demande un `GH_TOKEN` autorisé à
publier dans `KIZAFOX/trackr-release`.

## Benchmarks

Les moyennes par rôle sont calculées avec Match-V5 à partir de parties Ranked
Solo/Duo récentes de joueurs Master, Grandmaster et Challenger. Un workflow
GitHub Actions les régénère chaque lundi. Il vit dans le dépôt privé
`KIZAFOX/trackr-benchmark-automation` ; seul `benchmarks.json` est publié sur
la branche `gh-pages` de `KIZAFOX/trackr-release`.

Trackr télécharge le JSON au démarrage et en garde une copie locale pour
continuer à fonctionner hors ligne. Les benchmarks sont des moyennes de fin
de partie, pas des courbes détaillées par minute. Les clés Riot de développement
expirent après 24 heures ; le workflow utilise une clé durable.

## Structure

```text
dashboard/   Interface principale et styles
overlay/     Overlay transparent en jeu
recorder/    Capture des clips
splash/      Écran et état de démarrage
src/         Accès Riot, logique de partie et IPC Electron
test/        Tests automatisés
scripts/     Clone local du dépôt privé d’automatisation des benchmarks
main.js      Cycle de vie Electron et orchestration des fenêtres
preload.js   API sécurisée entre Electron et l’interface
```

Le dossier `scripts/` est un dépôt Git séparé et ignoré par le dépôt Trackr.
Il ne fait pas partie de l’application ni de son installeur.

## Vérifications

```powershell
npm test
```

## Notes techniques

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

- Ajouter la date et le nombre d’observations des benchmarks dans les réglages.
- Tester les transitions de gameflow, l’ouverture des onglets et les messages
  du splash pour éviter les régressions.
- Ajouter des références de benchmark par durée de partie.
- Compléter les événements de clips (multi-kills, Ace, First Blood) et les
  timers d’objectifs.
- Étudier le choix de l’écran de capture, les profils par mode et le partage
  de clips vers Discord.