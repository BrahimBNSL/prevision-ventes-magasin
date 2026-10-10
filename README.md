# Prévision des ventes d'un magasin (Rossmann)

Prédiction des ventes quotidiennes de 1 115 magasins Rossmann sur un horizon de **6 semaines**, à partir de l'historique des ventes, du calendrier, des promotions et de l'environnement concurrentiel.

Projet réalisé avec Python dans Jupyter, à partir des données de la compétition Kaggle [Rossmann Store Sales](https://www.kaggle.com/competitions/rossmann-store-sales).

## Objectif

Prévoir les ventes journalières sur les 6 dernières semaines d'historique, sans utiliser les ventes réelles de la période prédite. La métrique principale est le **RMSPE** (erreur relative quadratique moyenne), métrique officielle de la compétition.

## Méthode

Le point central est l'**horizon de 42 jours** : pour prédire un jour `d`, toutes les variables historiques n'utilisent que des ventes observées au plus tard le jour `d - 42`. Cela évite la fuite d'information qui gonflerait artificiellement les scores.

Les variables sont regroupées en quatre familles :

- **Historique décalé** : ventes du même jour de la semaine six semaines plus tôt, moyennes sur 7, 8, 13 et 26 semaines, moyenne des 4 dernières occurrences du même jour de la semaine.
- **Calendrier** : année, mois, jour, semaine, encodage sinus/cosinus de la semaine et du mois.
- **Événements** : compteurs de jours avant et après les jours fériés, plafonnés à 14 jours.
- **Magasin et concurrence** : type et assortiment du magasin, distance du concurrent, ancienneté de la concurrence et de la promotion longue durée, mois de promotion active.

Les modèles comparés sont :

1. **Baseline** : moyenne des ventes par magasin et jour de la semaine (référence).
2. **XGBoost** entraîné sur le logarithme des ventes.
3. **LightGBM**.
4. **Random Forest**.
5. **Moyenne XGBoost (log) + LightGBM**.

L'évaluation se fait sur la période de test des 6 dernières semaines, avec l'entraînement sur toutes les données antérieures.

## Résultats

Résultats sur la période de test :

| Modèle | RMSPE | RMSE | MAE | R² |
|---|---|---|---|---|
| Baseline | 0,2356 | 1 664 | 1 254 | 0,703 |
| XGBoost (log) | 0,1264 | 925 | 622 | 0,908 |
| LightGBM | 0,1294 | 914 | 622 | 0,910 |
| Random Forest | 0,1446 | 988 | 687 | 0,896 |
| XGBoost (log) + LightGBM | 0,1268 | 913 | 616 | 0,911 |

Les trois meilleurs modèles de boosting sont très proches (écart inférieur à 2 %). Ils battent nettement la baseline.

Les erreurs sont plus élevées le samedi et le dimanche. Les magasins 292 et 909 sont très mal prédits : leurs ventes sont souvent faibles, ce qui gonfle le RMSPE.

## Structure du projet

```
Prévision_magasin/
├── Prevision_magasin_final.ipynb   # notebook principal
├── train.csv                       # ventes par jour et par magasin (Kaggle)
├── store.csv                       # caractéristiques des magasins (Kaggle)
└── README.md
```

Les fichiers `train.csv` et `store.csv` ne sont pas inclus dans le dépôt : ils doivent être téléchargés depuis Kaggle.

## Limites

- Les scores sont obtenus sur une seule période de test de 6 semaines.
- Les variables météo, les événements locaux et les ruptures de stock ne sont pas disponibles, alors qu'ils expliquent une part des erreurs.
- Les magasins à ventes très faibles (292, 909) faussent le RMSPE moyen et doivent être analysés à part.
- Une prévision à 6 semaines reste moins précise qu'une prévision à 1 jour : le modèle est utilisable pour la planification, pas pour une décision au jour près.

## Pistes d'amélioration

- Validation sur plusieurs périodes de test pour mesurer la stabilité du classement.
- Traitement séparé des magasins à faibles ventes.
- Ajout de données externes (météo, calendrier scolaire régional).

## Références

- Compétition Kaggle : [Rossmann Store Sales](https://www.kaggle.com/competitions/rossmann-store-sales)
- Solution gagnante (Gert Jacobusse) : horizon de prévision, variables de tendance et d'historique récent, cible en logarithme.
