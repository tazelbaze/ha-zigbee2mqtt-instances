# Gestion des versions Zigbee2MQTT sur les 4 instances custom

Ce document explique pourquoi le repo `ha-zigbee2mqtt-instances` est passé d'un bot tiers à un script maison pour la mise à jour de version, et ce que fait exactement ce script.

## Contexte

Le repo héberge 4 manifestes d'addon Home Assistant, un par instance Zigbee2MQTT :

| Dossier | Lieu | data_path | IP coordinateur | Adapter |
|---|---|---|---|---|
| `zigbee2mqtt-rdc` | RDC | `/config/zigbee2mqtt` | 192.168.0.18:6638 | zstack |
| `zigbee2mqtt-etage` | Étage | `/config/zigbee2mqtt_2` | 192.168.0.25:6639 | ember |
| `zigbee2mqtt-annexe` | Annexe | `/config/zigbee2mqtt-3` | 192.168.0.17:6640 | zstack |
| `zigbee2mqtt-piscine` | Local piscine | `/config/zigbee2mqtt-4` | 192.168.0.16:6641 | zstack |

Chaque `config.json` contient les vraies valeurs `mqtt` et `serial` de son instance (sauf Annexe, qui garde sa config MQTT/série dans son propre `configuration.yaml` interne plutôt que dans les options Supervisor).

## Le script d'origine (abandonné)

Le workflow initial utilisait l'action tierce `andreypolyak/zigbee2mqtt-multi-addon-action@v1.0`, censée créer et maintenir automatiquement les addons listés dans `addon_names`, en synchronisant leur version avec les releases officielles de Zigbee2MQTT toutes les 30 minutes.

### Pourquoi on ne l'a pas gardé

En lisant le code source (`main.py`) de cette action, deux problèmes bloquants :

**1. Elle écrase tout le `config.json`, pas seulement la version.**
À chaque exécution, pour un addon déjà connu, l'action ne relit jamais le `config.json` existant du repo. Elle récupère le `config.json` officiel à jour depuis `zigbee2mqtt/hassio-zigbee2mqtt`, en fait une copie, ne change que `slug`, `name` et `description`, puis remplace intégralement notre fichier par cette copie. Résultat : nos valeurs `mqtt`, `serial` et `data_path` personnalisées sont perdues et remplacées par les valeurs par défaut vides du template officiel, dès qu'une nouvelle version de Z2M sort.

C'est déjà arrivé une fois sur l'instance piscine avant qu'on ne le remarque : le `config.json` du repo affichait des options vides alors que l'addon réellement installé tournait avec les bonnes valeurs. Ça n'avait rien cassé uniquement parce que Supervisor garde les options réellement configurées dans son propre stockage interne, indépendant du repo. Mais ça veut dire que le repo GitHub ne reflète plus la réalité, et qu'une réinstallation depuis ce manifeste repartirait sur des valeurs vides.

**2. La correspondance nom → dossier est fragile sur les accents.**
Le nom de dossier est calculé en mettant le nom de l'addon en minuscule, sans retirer les accents. `"Étage"` donnerait `zigbee2mqtt-étage`, qui ne correspond pas à notre dossier `zigbee2mqtt-etage` (sans accent). L'action ne trouverait pas de correspondance, conclurait que l'addon n'existe pas encore, et créerait un nouveau dossier en doublon avec le template par défaut, à côté du bon.

### Conclusion

Cette action est faite pour créer des addons Z2M "vierges" et les maintenir à jour en version, pas pour cohabiter avec des manifestes déjà personnalisés à la main. On ne peut pas l'utiliser sans perdre le travail de configuration qu'on a fait.

## Le script maison

Remplace entièrement le job dans `.github/workflows/main.yml`. Principe : ne toucher QUE le champ `version` de chaque `config.json`, rien d'autre.

### Ce qu'il fait

1. Récupère la dernière version officielle de Zigbee2MQTT en lisant le `config.json` du repo officiel `zigbee2mqtt/hassio-zigbee2mqtt` (juste le champ `version`, via `curl` + `jq`).
2. Pour chacun des 4 dossiers (`zigbee2mqtt-rdc`, `zigbee2mqtt-etage`, `zigbee2mqtt-annexe`, `zigbee2mqtt-piscine`), remplace uniquement la valeur du champ `.version` dans son `config.json`, via `jq --arg v ... '.version = $v'`. Tout le reste du fichier (mqtt, serial, data_path, name, slug, schema...) reste identique bit à bit.
3. Commit uniquement s'il y a un changement réel (`git diff --cached --quiet ||`), pour éviter des commits vides tous les jours.
4. Tourne une fois par jour (`cron: '0 6 * * *'`), plus déclenchement manuel possible (`workflow_dispatch`).

### Ce qu'il ne fait pas

- Il ne crée jamais de nouveau dossier.
- Il ne touche jamais aux champs `mqtt`, `serial`, `data_path`, `name`, `slug`, `schema`.
- Il ne gère aucun addon en dehors de la liste explicite des 4 dossiers écrite en dur dans le script.

### Conséquence à connaître

Le champ `version` du `config.json` est directement le tag Docker de l'image tirée (`ghcr.io/zigbee2mqtt/zigbee2mqtt-{arch}:{version}`). Si ce script tombe en panne silencieusement (API GitHub down, changement de format du `config.json` officiel, etc.), les 4 instances resteront figées sur l'ancienne version sans notification. Vérifier de temps en temps que le workflow tourne bien (onglet Actions du repo) plutôt que de considérer que "pas de nouvelles, bonnes nouvelles".

### Vérification après une mise à jour de version

Le bump de version change l'image Docker tirée au prochain démarrage de l'addon. Avant de redémarrer un addon après un bump :
1. Vérifier le changelog de la nouvelle version Zigbee2MQTT pour d'éventuelles breaking changes.
2. Valider la config HA avec `ha_check_config` avant tout redémarrage.
3. Redémarrer les 4 instances une par une, pas toutes en même temps, pour isoler un éventuel problème.
