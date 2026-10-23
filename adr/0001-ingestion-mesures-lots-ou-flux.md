# ADR-0001 : Ingestion des mesures : traitement par lots ou en flux ?

- Date : 2026-10-23
- Statut : Proposé (amorcé, à terminer en séance 6)
- Groupe : GBAGUIDI Micke-Phil, SOUMAHORO Namory, NKOT Daniel · Spécialité : Data
- Diagramme C4 concerné : niveau 2 · conteneurs QueenBee (Passerelle de site, Broker MQTT, Ingestion, Base)

## Contexte
Les mesures des capteurs (UC-T02) passent par une passerelle LoRaWAN par site, puis arrivent au central souvent en différé : en mode dégradé, la passerelle garde les trames en cache et les renvoie d'un bloc au retour du réseau (UC-CY01). Il faut contrôler chaque trame (plages, rejeu, UC-CY02) et publier `MesureValidée.v1` pour Prévision, Santé des colonies, Production et Impact. Point chaud d'origine : « Ingestion : lots ou flux ? ».

## Options étudiées
| Option | Avantages | Inconvénients |
|---|---|---|
| A · Lots (toutes les heures ou toutes les nuits) | Simple à mettre en place et à rejouer ; adapté aux paquets envoyés après le mode dégradé ; adapté aux indicateurs d'Impact | Une alerte peut attendre jusqu'au prochain lot : trop lent pour un risque d'essaimage et pour la surveillance renforcée (1 mesure / 5 min) |
| B · Flux (chaque trame traitée à son arrivée, via MQTT) | Faible latence des alertes ; chaque mesure est publiée dès qu'elle est validée | Plus complexe (ordre d'arrivée, déduplication en continu) ; les paquets de resynchronisation arrivent d'un coup |
| C · Hybride (flux pour la validation et la publication, lots pour les indicateurs) | Alertes rapides et consolidation nocturne pour Impact | Deux chaînes de traitement à maintenir |

## Critères
- **Latence des alertes** : un risque doit être détecté vite (UC-A02), surtout en surveillance renforcée (1 mesure / 5 min).
- **Connectivité intermittente** : les mesures arrivent en retard et en paquets (mode dégradé, retard maximal de 72 h, UC-CY01).
- **Volumétrie** : 38 400 mesures par jour (400 ruches × 96), jusqu'à 3 fois plus pour les ruches en surveillance renforcée.

## Décision
À compléter en séance 6.

## Conséquences
À compléter en séance 6.
