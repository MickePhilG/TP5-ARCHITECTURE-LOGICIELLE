# Contrat de données · événement « MesureValidée »

## 1. Propriétaire et consommateurs

- **Propriétaire** : Collecte des mesures (seul contexte qui publie et fait évoluer le contrat).
- **Consommateurs** :
  - Prévision : score de santé, risque, prédiction de rendement
  - Santé des colonies : suivi des colonies
  - Production : comparaison des récoltes aux poids mesurés (UC-C02)
  - Impact et reporting : indicateurs multi-sites
- **Sur la context map** : `Collecte des mesures -- U/D · PL : MesureValidée.v1 --> Prévision, Santé des colonies, Production, Impact et reporting`

## 2. Tableau des champs (payload)

| Champ | Type | Unité | Plage valide | Oblig. | Règle / remarque |
|---|---|---|---|---|---|
| `hive_id` | string | – | format `PP-SITE-NNN` | oui | ruche connue et active dans le référentiel, sinon rejet |
| `site_id` | string | – | format `PP-SITE` | oui | doit être le site de la ruche dans le référentiel, sinon rejet |
| `timestamp` | string ISO 8601 UTC | – | ≤ maintenant + 5 min et ≥ maintenant − 72 h | oui | avec `hive_id` : clé de déduplication |
| `weight_kg` | number | kg | 0 – 120 | oui | une variation brutale n'est pas un rejet : la valeur est dans la plage, c'est à Prévision / Santé de l'interpréter (essaimage ou capteur défaillant) |
| `temp_internal_c` | number | °C | −10 – 50 | oui | colonie saine autour de 34 – 36 °C ; hors plage = rejet |
| `temp_external_c` | number | °C | −10 – 60 | non | `null` si la ruche n'a pas de sonde externe |
| `humidity_pct` | number | % | 0 – 100 | non | `null` si non mesurée |
| `acoustic_activity_score` | number | sans unité | 0 – 1 | non | `null` si non mesuré |
| `battery_pct` | integer | % | 0 – 100 | oui | < 20 % : signal de maintenance, pas un rejet |
| `position_lat` | number | degrés décimaux (WGS 84) | −90 – 90 | non | comparée à la position du référentiel : écart > tolérance GPS → `PositionHorsToléranceDétectée` (UC-CY04), la mesure reste publiée |
| `position_lon` | number | degrés décimaux (WGS 84) | −180 – 180 | non | idem `position_lat` (les deux sont présents ou absents ensemble) |

## 3. Enveloppe de l'événement

| Champ | Type | Rôle |
|---|---|---|
| `eventId` | string UUID v4 | identifiant unique de l'événement |
| `type` | string | toujours `"MesureValidée"` |
| `version` | integer | version majeure du contrat (1) |
| `occurredAt` | string ISO 8601 UTC | date de la mesure (= `payload.timestamp`) |
| `receivedAt` | string ISO 8601 UTC | date de réception au central |
| `producer` | string | toujours `"collecte-des-mesures"` |
| `payload` | object | les champs du tableau 2 |

## 4. Exemple JSON valide

```json
{
  "eventId": "6f1c2a9e-5b7d-4c1e-9a3f-2d8e0b4c7a11",
  "type": "MesureValidée",
  "version": 1,
  "occurredAt": "2027-03-14T08:15:00Z",
  "receivedAt": "2027-03-14T08:41:12Z",
  "producer": "collecte-des-mesures",
  "payload": {
    "hive_id": "SN-CASA-014",
    "site_id": "SN-CASA",
    "timestamp": "2027-03-14T08:15:00Z",
    "weight_kg": 34.2,
    "temp_internal_c": 34.8,
    "temp_external_c": 27.1,
    "humidity_pct": 58,
    "acoustic_activity_score": 0.72,
    "battery_pct": 91,
    "position_lat": 12.5833,
    "position_lon": -16.2833
  }
}
```

**Pourquoi deux dates ? Quel est l'impact sur les alertes ?**
- Les calculs (score de santé, chute de poids) se font sur `occurredAt` : c'est la vraie chronologie de la ruche, même si les mesures arrivent en désordre.
- En mode dégradé, une mesure peut arriver plusieurs heures après : un risque détecté sur une mesure ancienne est peut-être déjà dépassé. L'alerte garde donc les deux dates, et le délai d'escalade de 48 h part de la levée de l'alerte, pas de `occurredAt`.
- L'écart `receivedAt − occurredAt` mesure aussi la qualité de la liaison d'un site (indicateur « couverture réseau »).

## 5. Règles

### Déduplication
- Clé : `hive_id` + `timestamp`.
- Même clé et mêmes valeurs (renvoi après une resynchronisation) : ignorée, sans nouvel événement (idempotence).
- Même clé et valeurs différentes : trame rejetée, `TrameRejouéeDétectée` publié et journalisé (UC-CY02).
- Les consommateurs doivent aussi être idempotents sur `eventId`.

### Rejet
- Champ obligatoire absent ou hors plage, ruche inconnue, `site_id` incohérent, `timestamp` dans le futur (> maintenant + 5 min) : trame rejetée, `TrameRejetée` publié avec le motif, trame brute mise en **quarantaine** (motif + date de réception).
- Champ facultatif hors plage : le champ passe à `null`, la mesure est publiée et l'anomalie est journalisée.
- Rejeux répétés sur une même passerelle : Supervision et sécurité ouvre un incident d'intégrité (`IncidentIntégritéOuvert`, seuil N à fixer, point chaud).

### Fréquence
- Nominale : 1 mesure / 15 min par ruche, soit 96 par jour et 38 400 par jour pour 400 ruches.
- Surveillance renforcée : 1 mesure / 5 min pendant 6 h après un seuil dépassé (`SurveillanceRenforcéeActivée`).
- Les consommateurs ne doivent pas supposer un intervalle fixe ni un ordre d'arrivée : ils trient sur `occurredAt`.

### Retard maximal accepté (mode dégradé, UC-CY01)
- Au retour de la liaison (`LiaisonRétablie`), la passerelle renvoie son cache (`MesuresEnDifféréResynchronisées`) : chaque trame suit les mêmes règles de validation et de déduplication.
- `receivedAt − occurredAt` ≤ 72 h : la mesure est publiée normalement.
- Au-delà de 72 h : quarantaine, rejouable à la main pour l'historique (Impact), mais pas publiée en temps réel pour ne pas lever d'alertes dépassées.

### Versionnement
- `version` = version majeure du contrat ; l'événement est nommé `MesureValidée.v1` sur la context map.
- Changement compatible (ajout d'un champ **facultatif**, ex. un nouveau capteur) : on reste en v1 (noté v1.1 dans l'historique du contrat) ; les consommateurs ignorent les champs qu'ils ne connaissent pas.
- Changement incompatible (renommage, changement d'unité, nouveau champ obligatoire, suppression) : v2 publiée **en parallèle** de la v1 jusqu'à ce que les 4 consommateurs aient migré, puis arrêt de la v1.
