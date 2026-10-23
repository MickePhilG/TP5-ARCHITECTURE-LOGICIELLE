# Points chauds

| # | Question | Impact si on se trompe | Qui peut répondre | Devient un ADR ? |
|---|---|---|---|---|
| 1 | Tolérance GPS : combien de mètres ? | Fausses alertes de déplacement (dérive GPS) ou vols non détectés | Coordinateurs BeeAfrica + équipe Cyber | non (paramètre) |
| 2 | Fréquence des mesures : 15 ou 30 min ? (UC-T02 dit 30 min, la volumétrie de 38 400 mesures/jour correspond à 15 min) | Volumétrie et autonomie des batteries mal dimensionnées, risques détectés trop tard | Client BeeAfrica | non (paramètre du contrat `MesureValidée`) |
| 3 | Retard maximal accepté en mode dégradé ? | Mesures trop anciennes qui lèvent des alertes déjà dépassées, ou mesures perdues | BeeAfrica + équipe Data | non (règle du contrat `MesureValidée`) |
| 4 | Ingestion des mesures : lots ou flux ? | Latence des alertes trop forte, ou système trop complexe pour une connectivité intermittente | Équipe Data | oui (ADR de spécialité Data) |
| 5 | Authentification des passerelles : clé partagée, mTLS ou VPN ? | Trames injectées par une fausse passerelle | Équipe Cyber | oui |
| 6 | Confiance minimale pour lever une alerte ? | Trop d'alertes inutiles (les apicultrices ne les lisent plus) ou risques manqués | Coordinateurs BeeAfrica + équipe Data | non (paramètre) |
| 7 | Délai d'escalade de 48 h : fixé par qui ? | Escalades trop tôt ou trop tard | Coordinateurs BeeAfrica | non (paramètre) |
| 8 | Où vit la politique d'escalade (tâche planifiée, file différée, base) ? | Alertes jamais escaladées si le mécanisme tombe | Équipe Dev | oui |
| 9 | Conflit de synchronisation : qui gagne ? | Inspection écrasée, donc perte d'information terrain | BeeAfrica (apicultrices, coordinateurs) + équipe Dev | oui |
| 10 | Seuil d'écart de rendement : 20 % ? par coopérative ? | Alertes inutiles ou manquées | Coordinateurs BeeAfrica | non (paramètre) |
| 11 | Qui corrige une récolte validée ? | Données de rendement fausses dans les rapports aux bailleurs | BeeAfrica (coordinateurs, gestionnaire) | non (règle métier) |
| 12 | Données nominatives dans le rapport d'impact ? | Fuite de données personnelles des apicultrices vers les bailleurs | BeeAfrica + bailleurs | oui |
| 13 | « Alerte » : colonie ou infrastructure ? Même mot, deux sens ? | Un seul modèle avec des règles mélangées | Équipe | non : tranché, « Alerte » dans Santé des colonies et « Incident » dans Supervision et sécurité |
