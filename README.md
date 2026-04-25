# hassio-z2m-multi

Dépôt Home Assistant contenant une instance additionnelle de l'addon Zigbee2MQTT, maintenue automatiquement à jour via GitHub Actions.

## Instance disponible

| Addon | Réseau | Canal |
|---|---|---|
| Zigbee2MQTT 4 | Local Piscine | 11 |

## Fonctionnement

Ce repo utilise [zigbee2mqtt-multi-addon-action](https://github.com/andreypolyak/zigbee2mqtt-multi-addon-action) pour synchroniser automatiquement la version de l'addon avec les releases officielles de [hassio-zigbee2mqtt](https://github.com/zigbee2mqtt/hassio-zigbee2mqtt).

Le workflow tourne toutes les 30 minutes et met à jour la version si une nouvelle release est disponible.

## Ajout dans Home Assistant

Paramètres → Modules complémentaires → Dépôts → ajouter :

```
https://github.com/tazelbaze/hassio-z2m-multi
```

## Mise à jour manuelle

Si besoin de forcer une synchronisation :

Actions → "Multiple Zigbee2mqtt Home Assistant add-ons" → Run workflow
