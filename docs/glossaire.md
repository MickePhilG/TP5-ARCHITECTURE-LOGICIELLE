# Glossaire

| Terme | Définition validée | Contexte | Dans le code | Source |
|---|---|---|---|---|
| Site | Rucher identifié par un code PP-SITE, rattaché à un pays | Référentiel apicole | `Site` | UC-AD01 |
| Ruche (objet complet) | Colonie installée sur un site, identifiée par PP-SITE-NNN, avec une position de référence | Référentiel apicole | `Ruche` | UC-AD01, UC-CY04 |
| Ruche (référence) | Simple identifiant `hive_id` porté par une mesure, une alerte ou une récolte, sans les autres informations de la ruche | Collecte, Santé, Production, Opérations | `hiveId` | UC-T02 |
| Transhumance | Déplacement déclaré d'une ruche vers une nouvelle position (seul moyen de changer sa position) | Référentiel apicole | `ruche.declarerTranshumance()` | UC-CY04 |
| Trame | Message brut envoyé par un capteur via la passerelle LoRaWAN, pas encore contrôlé | Collecte des mesures | `Trame` | UC-T02, UC-CY02 |
| Lot de trames | Ensemble de trames transmises en une fois par une passerelle (souvent en différé) | Collecte des mesures | `LotDeTrames` | UC-T02, UC-CY01 |
| Mesure validée | Trame qui a passé les contrôles (ruche connue, plages physiques, unicité) et qui est publiée aux autres contextes | Collecte des mesures | `MesureValidee` | UC-T02 |
| Rejeu | Trame déjà reçue (même `hive_id` + `timestamp`) qui arrive une nouvelle fois | Collecte des mesures | `trameRejouee` | UC-CY02 |
| Passerelle | Équipement de site qui reçoit les trames LoRaWAN et les transmet au central ; une seule active par site | Collecte des mesures | `Passerelle` | UC-CY01 |
| Mode dégradé | Passerelle sans liaison avec le central : elle garde les trames en cache local | Collecte des mesures | `modeDegrade` | UC-CY01 |
| Liaison rétablie | Retour de la connexion entre une passerelle en mode dégradé et le central | Collecte des mesures | `passerelle.retablirLiaison()` | UC-CY01 |
| Surveillance renforcée | Passage à 1 mesure / 5 min pendant 6 h après un seuil dépassé | Collecte des mesures | `SurveillanceRenforcee` | UC-T02 |
| Incident | Problème d'infrastructure ou de sécurité sur un site (site muet, intégrité, brouillage, déplacement non déclaré) | Supervision et sécurité | `Incident` | UC-CY01 → UC-CY04 |
| Évaluation de risque | Résultat du modèle pour une ruche : type de risque, niveau de confiance, version du modèle | Prévision | `EvaluationRisque` | UC-A02 |
| Niveau de confiance | Probabilité (entre 0 et 1) donnée par le modèle pour un risque | Prévision, Santé des colonies | `confiance` | UC-A02 |
| Prédiction de rendement | Volume de miel attendu calculé par le modèle pour une ruche et une saison | Prévision | `PredictionRendement` | UC-A03 |
| Alerte (d'une colonie) | Risque sanitaire sur une colonie, levé par la prévision et suivi jusqu'à sa résolution par une apicultrice | Santé des colonies | `Alerte` | UC-A02 |
| Alerte (d'infrastructure) | Alerte de site, d'intégrité ou de brouillage sur l'infrastructure, suivie par l'administrateur ; nommée « Incident » dans le code | Supervision et sécurité | `Incident` | UC-CY01 → UC-CY04 |
| Escalade | Remontée d'une alerte non prise en charge sous 48 h | Santé des colonies | `alerte.escalader()` | UC-A02 |
| Fausse alerte | Alerte déclarée sans fondement par le coordinateur : elle est annulée et n'accepte plus de transition | Santé des colonies | `alerte.annuler()` | UC-A02 |
| Visite | Passage planifié d'une apicultrice sur une ruche, assigné par le coordinateur | Opérations terrain | `Visite` | UC-C01 |
| Inspection | Constat saisi par l'apicultrice sur la ruche (état visuel, reine, comportement, photo), même hors ligne | Opérations terrain | `Inspection` | UC-A01 |
| Synchronisation (d'une inspection) | Envoi au central d'une inspection saisie hors ligne, au retour du réseau | Opérations terrain | `inspection.synchroniser()` | UC-A01 |
| Synchronisation (des mesures) | Envoi au central des trames gardées en cache par la passerelle pendant le mode dégradé | Collecte des mesures | `passerelle.resynchroniser()` | UC-CY01 |
| Validation (d'une trame) | Contrôle automatique d'une trame : ruche connue, plages physiques, unicité, authenticité | Collecte des mesures | `lot.valider()` | UC-T02, UC-CY02 |
| Validation (d'une récolte) | Acceptation de la déclaration par le coordinateur, après comparaison aux poids mesurés | Production | `recolte.valider()` | UC-C02 |
| Récolte | Volume de miel déclaré pour une ruche, à une date | Production | `Recolte` | UC-A03 |
| Écart de rendement | Différence, en %, entre la récolte déclarée et la prédiction | Production | `ecartRendement` | UC-A03, UC-C02 |
| Indicateur | Valeur consolidée par site, pays et période (rendement, colonies à risque, couverture réseau) | Impact et reporting | `Indicateur` | UC-G01 |
| Rapport d'impact | Document PDF sur une période, des sites et des indicateurs, envoyé au bailleur | Impact et reporting | `Rapport` | UC-G02 |

## Mots à plusieurs sens
- « Alerte » comporte deux lignes, parce que le cycle de vie, les personnes et les règles sont différents. 