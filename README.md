# Analyse et allocation d'un portefeuille d'actions américaines

Ce projet construit et compare plusieurs méthodes d'allocation de portefeuille sur **39 grandes actions américaines** de 11 secteurs. La période couvre **2015 à 2025**, et l'ETF **SPY** (S&P 500) sert de référence de marché.

Le projet va de la collecte des données au **backtest glissant avec frais de transaction**. Toutes les méthodes sont évaluées avec les mêmes mesures de performance et de risque.

---

## Sommaire

- [Objectifs](#objectifs)
- [Principaux résultats](#principaux-résultats)
- [Méthodologie](#méthodologie)
- [Structure du projet](#structure-du-projet)
- [Installation et exécution](#installation-et-exécution)
- [Limites](#limites)
- [Références](#références)

---

## Objectifs

1. Constituer une base de données propre de prix ajustés et de rendements journaliers.
2. Décrire les propriétés statistiques des rendements : queues épaisses, corrélations et volatilité qui varient dans le temps.
3. Mettre en place des mesures de performance et de risque communes : rendement, volatilité, Sharpe, max drawdown, VaR et CVaR historiques.
4. Construire des portefeuilles selon quatre approches : Markowitz, risk parity, HRP et changement de régime (HMM).
5. Les comparer **hors échantillon** : chaque décision n'utilise que les données passées, et chaque transaction est payée.

---

## Principaux résultats

Backtest de février 2016 à décembre 2025. Les poids sont recalculés chaque fin de mois sur les 252 jours précédents, et les frais sont de 0,10 % du montant échangé.

| Stratégie | Rendement annualisé | Volatilité | Sharpe | Max drawdown | Rotation annuelle |
|---|---:|---:|---:|---:|---:|
| Sharpe maximal (Ledoit-Wolf) | 24,6 % | 21,5 % | 1,03 | −32,0 % | 687 % |
| Sharpe maximal plafonné à 10 % | 18,5 % | 17,3 % | 0,95 | −28,2 % | 529 % |
| **Équipondéré** | 17,8 % | 16,6 % | 0,94 | −31,8 % | 52 % |
| Risk parity (Ledoit-Wolf) | 16,0 % | 15,4 % | 0,90 | −30,9 % | 65 % |
| Régimes HMM | 14,6 % | 14,3 % | 0,87 | −29,2 % | 531 % |
| HRP | 13,6 % | 14,7 % | 0,80 | −29,3 % | 275 % |
| SPY (sans transaction) | 15,5 % | 18,0 % | 0,77 | −33,7 % | — |
| Variance minimale (Ledoit-Wolf) | 10,5 % | 13,8 % | 0,64 | −29,2 % | 301 % |

**Conclusions**

- **L'équipondéré est très difficile à battre.** Il n'estime aucun paramètre et coûte peu en frais.
- **Le Sharpe maximal obtient le meilleur ratio, mais ce résultat est fragile.**
  - Son avance sur l'équipondéré n'est pas statistiquement significative (t = 1,44).
  - Il est très concentré (environ 5 actions effectives) et sa rotation est très élevée.
  - Avec des frais de 0,50 %, il passe derrière l'équipondéré.
- **Les méthodes fondées sur le risque réduisent bien la volatilité et la CVaR, mais au prix du rendement.** Le risk parity offre le meilleur compromis entre stabilité et performance.
- **Plus une méthode dépend des rendements espérés, plus elle se dégrade hors échantillon.** Le Sharpe du portefeuille de Sharpe maximal baisse de 0,45 entre l'estimation sur toute la période et le backtest.
- **Le modèle de régimes (HMM)** détecte bien les phases de forte volatilité, mais trop tard pour une décision mensuelle : il réduit le risque sans améliorer le Sharpe.

---

## Méthodologie

### Étape 1 — Données et analyse exploratoire
- Prix de clôture ajustés (dividendes et divisions d'actions) téléchargés depuis Yahoo Finance, puis mis en cache pour que les résultats soient reproductibles.
- Valeurs manquantes traitées sans biais d'anticipation : uniquement un report du dernier prix connu, limité à 2 jours.
- Taux sans risque : bons du Trésor américain à 3 mois (`^IRX`), convertis en taux journalier.
- Analyse des rendements :
  - normalité rejetée pour toutes les séries (test de Jarque-Bera) ;
  - corrélations organisées en blocs sectoriels ;
  - corrélation moyenne qui monte jusqu'à 0,81 en mars 2020 ;
  - volatilité regroupée par périodes.

### Étape 2 — Mesures de performance et de risque
- Rendement annualisé, volatilité, ratio de Sharpe (avec intervalle de confiance), max drawdown, VaR et CVaR historiques à 95 % et 99 %.
- Comparaison entre VaR historique et VaR gaussienne : la VaR gaussienne sous-estime le risque à 99 %.
- Références : portefeuille équipondéré et SPY.

### Étape 3 — Optimisation de Markowitz
- Frontière efficiente sans vente à découvert, portefeuilles de variance minimale et de Sharpe maximal (reformulé en problème quadratique convexe).
- Estimateur de covariance de **Ledoit-Wolf** pour réduire l'erreur d'estimation.
- Test de sensibilité du Sharpe maximal aux rendements espérés : les poids changent complètement d'une sous-période à l'autre.

### Étape 4 — Risk parity
- Portefeuille à contributions au risque égales (*Equal Risk Contribution*), calculé par descente par coordonnées sur la formulation de Spinu.
- Comparaison avec la pondération par l'inverse de la volatilité.

### Étape 5 — Hierarchical Risk Parity
- Classification hiérarchique des actions à partir de leurs corrélations, puis réordonnancement de la matrice selon l'arbre (quasi-diagonalisation) et répartition du budget par bissection récursive (López de Prado, 2016).
- Comparaison des groupes obtenus avec les secteurs réels.

### Étape 6 — Backtest glissant
- Rééquilibrage mensuel, fenêtre d'estimation de 252 jours, performance calculée jour par jour.
- Entre deux rééquilibrages, les poids dérivent avec les prix. Les frais sont proportionnels à la rotation.
- Analyses complémentaires :
  - sensibilité aux frais (0 à 0,50 %) ;
  - comparaison entre les résultats estimés sur toute la période et ceux du backtest ;
  - performances année par année.
- **Extension HMM** : modèle de Markov caché gaussien à deux états sur SPY, estimé par l'algorithme de Baum-Welch (implémenté avec NumPy). Il utilise uniquement les probabilités filtrées, et ses paramètres sont réestimés chaque année.

Chaque méthode est suivie d'un tableau récapitulatif, de graphiques et d'une interprétation des résultats. Les fonctions principales sont vérifiées par des tests sur des cas dont la solution se calcule à la main.

---

## Structure du projet

```
portfolio-performance-analysis/
├── analyse_portefeuille.ipynb     # notebook unique : étapes 1 à 6
├── README.md
└── data/                          # données fournies avec le dépôt
    ├── prix_bruts.csv             # cache des prix téléchargés
    ├── prix_nettoyes.csv          # prix ajustés après nettoyage
    ├── rendements.csv             # rendements journaliers
    ├── taux_sans_risque.csv       # taux sans risque journalier
    ├── secteurs.csv               # secteur de chaque action
    ├── rendements_references.csv  # équipondéré et SPY
    ├── mesures_references.csv     # mesures des références
    ├── rendements_backtest.csv    # rendements de toutes les stratégies
    └── mesures_backtest.csv       # mesures, rotation et frais du backtest
```

---

## Installation et exécution

**Prérequis** : Python 3.12 (environnement testé : Anaconda).

```bash
git clone https://github.com/Asma2201/portfolio-performance-analysis.git
cd portfolio-performance-analysis
pip install yfinance pandas numpy scipy scikit-learn matplotlib seaborn jupyter
jupyter notebook analyse_portefeuille.ipynb
```

Exécuter ensuite toutes les cellules dans l'ordre. Une exécution complète prend environ 2 minutes.

- Les données sont fournies dans `data/` : le notebook lit `data/prix_bruts.csv` et n'a pas besoin d'accès à Internet. Les résultats obtenus sont exactement ceux cités dans le notebook.
- Pour retélécharger les prix depuis Yahoo Finance, supprimer `data/prix_bruts.csv` avant l'exécution. Yahoo Finance révise parfois les prix ajustés : les chiffres peuvent alors changer légèrement par rapport à ceux du notebook, qui proviennent du téléchargement du 10/10/2026.

---

## Limites

- **Une seule période d'environ 10 ans** : l'erreur-type d'un ratio de Sharpe est d'environ 0,35, donc la plupart des écarts entre stratégies ne sont pas significatifs.
- **Biais de survie** : l'univers est composé d'entreprises qui existent encore en 2025. Cela avantage les stratégies par rapport à SPY, en particulier celles qui suivent les gagnants récents.
- **Paramètres fixés à l'avance, sans optimisation** (fenêtre, fréquence, plafond, seuil du HMM), pour éviter le sur-ajustement.
- **Modèle de frais simplifié** : pas d'impact de marché ni de fiscalité.
- **Positions acheteuses uniquement** : pas de vente à découvert.

---

## Références

- Markowitz, H. (1952). *Portfolio Selection*. Journal of Finance, 7(1), 77–91.
- Michaud, R. (1989). *The Markowitz Optimization Enigma: Is "Optimized" Optimal?* Financial Analysts Journal, 45(1), 31–42.
- Ledoit, O. et Wolf, M. (2004). *A Well-Conditioned Estimator for Large-Dimensional Covariance Matrices*. Journal of Multivariate Analysis, 88(2), 365–411.
- Maillard, S., Roncalli, T. et Teïletche, J. (2010). *The Properties of Equally Weighted Risk Contribution Portfolios*. Journal of Portfolio Management, 36(4), 60–70.
- Spinu, F. (2013). *An Algorithm for Computing Risk Parity Weights*. SSRN.
- Griveau-Billion, T., Richard, J.-C. et Roncalli, T. (2013). *A Fast Algorithm for Computing High-Dimensional Risk Parity Portfolios*. SSRN.
- López de Prado, M. (2016). *Building Diversified Portfolios that Outperform Out of Sample*. Journal of Portfolio Management, 42(4), 59–69.
- DeMiguel, V., Garlappi, L. et Uppal, R. (2009). *Optimal Versus Naive Diversification: How Inefficient is the 1/N Portfolio Strategy?* Review of Financial Studies, 22(5), 1915–1953.
- Lo, A. (2002). *The Statistics of Sharpe Ratios*. Financial Analysts Journal, 58(4), 36–52.

---

## Auteur

**Asma Allaigui**

**Outils** : Python, pandas, NumPy, SciPy, scikit-learn, matplotlib, seaborn, yfinance, Jupyter.
