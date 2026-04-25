# hassio-z2m-multi

Dépôt Home Assistant contenant plusieurs instances de l'addon Zigbee2MQTT, maintenues automatiquement à jour via GitHub Actions.

## Instances disponibles

| Addon | Réseau | Coordinateur | Canal |
|---|---|---|---|
| Zigbee2MQTT 3 | Annexe | 192.168.0.17 | 15 |
| Zigbee2MQTT 4 | Local Piscine | 192.168.0.16 | 11 |

## Fonctionnement

Ce repo utilise [zigbee2mqtt-multi-addon-action](https://github.com/andreypolyak/zigbee2mqtt-multi-addon-action) pour synchroniser automatiquement les versions des addons avec les releases officielles de [hassio-zigbee2mqtt](https://github.com/zigbee2mqtt/hassio-zigbee2mqtt).

Le workflow tourne toutes les 30 minutes et met à jour les versions si une nouvelle release est disponible.

## Ajout dans Home Assistant

Paramètres → Modules complémentaires → Dépôts → ajouter :

```
https://github.com/TON_USER/hassio-z2m-multi
```

## Mise à jour manuelle

Si besoin de forcer une synchronisation :

Actions → "Multiple Zigbee2mqtt Home Assistant add-ons" → Run workflow
