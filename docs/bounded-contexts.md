# Bounded contexts QueenBee

## Référentiel apicole
- Responsabilité (une phrase) : tenir la liste officielle des sites et des ruches, et leur position.
- Type : support · spécialité la plus concernée : Data
- Agrégats : Site (invariants principaux : code PP-SITE unique ; rattaché à un seul pays), Ruche (invariants principaux : identifiant PP-SITE-NNN unique ; un seul site ; la position ne change que par transhumance)
- Événements publiés : SiteEnregistré, RucheInstallée, TranshumanceDéclarée
- Événements consommés : aucun
- Cas d'utilisation sources : UC-AD01, UC-CY04, §7.1
- Mots propres à ce contexte : Site, Ruche, Transhumance, position de référence

## Collecte des mesures
- Responsabilité (une phrase) : recevoir les trames des passerelles, les contrôler et publier les mesures validées.
- Type : support · spécialité la plus concernée : Data
- Agrégats : LotDeTrames (invariants principaux : pas de doublon hive_id + timestamp ; valeurs dans les plages physiques), Passerelle (invariants principaux : une seule active par site ; hors ligne après 15 min sans battement de cœur)
- Événements publiés : TrameReçue, MesureValidée, TrameRejetée, TrameRejouéeDétectée, SurveillanceRenforcéeActivée, PasserelleHorsLigneDétectée, ModeDégradéActivé, LiaisonRétablie, MesuresEnDifféréResynchronisées, PositionHorsToléranceDétectée
- Événements consommés : RucheInstallée, TranshumanceDéclarée (venant de Référentiel apicole) ; trames (venant des capteurs + passerelle LoRaWAN)
- Cas d'utilisation sources : UC-T02, UC-CY01, UC-CY02
- Mots propres à ce contexte : Trame, Lot de trames, Mesure validée, Rejeu, Mode dégradé, Liaison rétablie, Surveillance renforcée, Synchronisation (des mesures), Validation (d'une trame)

## Supervision et sécurité
- Responsabilité (une phrase) : détecter et suivre les incidents d'infrastructure et de sécurité des sites.
- Type : support · spécialité la plus concernée : Cyber
- Agrégats : Incident (invariants principaux : rattaché à un site ; un incident clos est figé ; toute action est journalisée)
- Événements publiés : IncidentOuvert (site muet), IncidentIntégritéOuvert, BrouillageRadioDétecté, DéplacementNonDéclaréDétecté
- Événements consommés : PasserelleHorsLigneDétectée, TrameRejouéeDétectée, PositionHorsToléranceDétectée (venant de Collecte des mesures) ; TranshumanceDéclarée (venant de Référentiel apicole) ; journaux (venant de FortiGate / SIEM)
- Cas d'utilisation sources : UC-CY01, UC-CY02, UC-CY03, UC-CY04
- Mots propres à ce contexte : Incident, site muet, intégrité, brouillage, canal

## Prévision
- Responsabilité (une phrase) : évaluer le risque de chaque colonie et prédire le rendement à partir des mesures et de la météo.
- Type : cœur · spécialité la plus concernée : Data
- Agrégats : ÉvaluationRisque (invariants principaux : confiance entre 0 et 1 ; version du modèle obligatoire), PrédictionRendement (invariants principaux : une par ruche et par saison ; volume ≥ 0)
- Événements publiés : DonnéesMétéoIntégrées, ScoreSantéRecalculé, RisqueEssaimageDétecté, PrédictionRendementCalculée
- Événements consommés : MesureValidée (venant de Collecte des mesures) ; météo (venant d'OpenWeatherMap) ; résultats (venant du Modèle de prédiction)
- Cas d'utilisation sources : UC-A02, UC-A03
- Mots propres à ce contexte : Évaluation de risque, Score de santé, Niveau de confiance, Prédiction de rendement

## Santé des colonies
- Responsabilité (une phrase) : lever les alertes sur les colonies et les suivre jusqu'à leur résolution ou leur annulation.
- Type : cœur · spécialité la plus concernée : Dev
- Agrégats : Alerte (invariants principaux : type de risque connu ; confiance entre 0 et 1 ; pas de résolution sans prise en charge ; escalade ≥ 48 h ; une alerte résolue ou annulée est figée)
- Événements publiés : AlerteLevée, ApicultriceNotifiée, AlertePriseEnCharge, AlerteEscaladée, AlerteRésolue, AlerteAnnulée
- Événements consommés : RisqueEssaimageDétecté, ScoreSantéRecalculé (venant de Prévision) ; MesureValidée (venant de Collecte des mesures) ; InspectionSynchronisée (venant d'Opérations terrain)
- Cas d'utilisation sources : UC-A02, UC-A01
- Mots propres à ce contexte : Alerte, Prise en charge, Escalade, Fausse alerte, type de risque

## Opérations terrain
- Responsabilité (une phrase) : planifier les visites et enregistrer les inspections, même hors ligne.
- Type : support · spécialité la plus concernée : Dev
- Agrégats : Visite (invariants principaux : assignée à une apicultrice du site ; une visite réalisée est figée), Inspection (invariants principaux : ruche existante ; conflit résolu avant publication)
- Événements publiés : VisitePlanifiée, VisiteAssignée, InspectionEnregistréeHorsLigne, InspectionSynchronisée, ConflitSynchronisationDétecté, AnomalieConstatée
- Événements consommés : AlerteLevée, AlerteEscaladée, AlerteAnnulée (venant de Santé des colonies) ; RucheInstallée (venant de Référentiel apicole)
- Cas d'utilisation sources : UC-C01, UC-A01
- Mots propres à ce contexte : Visite, Inspection, Anomalie, Synchronisation (d'une inspection), conflit

## Production
- Responsabilité (une phrase) : enregistrer les récoltes, les comparer à la prédiction et les faire valider.
- Type : cœur · spécialité la plus concernée : Data
- Agrégats : Récolte (invariants principaux : volume > 0 ; une récolte validée est figée ; un écart signalé exige un commentaire)
- Événements publiés : RécolteDéclarée, ÉcartRendementCalculé, CoordinateurNotifié, RécolteValidée, ÉcartMisEnInvestigation
- Événements consommés : PrédictionRendementCalculée (venant de Prévision) ; MesureValidée (venant de Collecte des mesures, pour les poids) ; RucheInstallée (venant de Référentiel apicole)
- Cas d'utilisation sources : UC-A03, UC-C02
- Mots propres à ce contexte : Récolte, Écart de rendement, Validation (d'une récolte)

## Impact et reporting
- Responsabilité (une phrase) : consolider les indicateurs multi-sites et produire les rapports d'impact pour les bailleurs.
- Type : support · spécialité la plus concernée : Data
- Agrégats : Indicateur (invariants principaux : calculé pour un site et une période ; jamais saisi à la main), Rapport (invariants principaux : début < fin ; un rapport envoyé est figé et horodaté)
- Événements publiés : IndicateursConsolidés, PériodeRapportChoisie, RapportImpactGénéré, RapportEnvoyéAuBailleur
- Événements consommés : MesureValidée (venant de Collecte des mesures) ; RécolteValidée (venant de Production) ; AlerteLevée, AlerteRésolue, AlerteAnnulée (venant de Santé des colonies) ; IncidentOuvert (venant de Supervision et sécurité)
- Cas d'utilisation sources : UC-G01, UC-G02
- Mots propres à ce contexte : Indicateur, Rapport d'impact, Période, Bailleur
