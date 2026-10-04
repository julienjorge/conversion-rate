# Conversion Rate Challenge — prédire l'abonnement à une newsletter

## Le projet

Le site datascienceweekly.org publie une newsletter gratuite. Ses éditeurs veulent savoir
s'il est possible de prédire, à partir de quelques informations sur une visite, si le
visiteur va s'abonner, et quelles variables expliquent ce choix.

Nous avons construit un modèle de classification qui prédit la conversion (la visite se
termine par un abonnement) à partir de cinq variables : le pays, l'âge, le statut de
nouveau visiteur, la source de trafic et le nombre de pages visitées. Le travail couvre
l'exploration des données, leur préparation, l'entraînement et la comparaison de
plusieurs modèles, la prédiction sur un fichier sans réponses, et la lecture du modèle
pour en tirer des leviers d'action.

La métrique est le f1-score, imposée par l'énoncé : elle ne regarde que la classe rare,
les visites converties, que l'exactitude ignorerait presque.

## Les données

Deux fichiers fournis par Jedha, rangés dans data/raw.

- **conversion_data_train.csv** : 284 580 visites, chacune décrite par les cinq variables
  et par la cible, converti ou non. Seules 3,2 % des visites sont converties. Aucune valeur
  manquante ; deux âges aberrants (123 et 111 ans) ont été retirés. Nous en tirons
  l'exploration, l'entraînement et l'évaluation des modèles.
- **conversion_data_test.csv** : 31 620 visites, mêmes variables, sans la cible. Nous en
  tirons le fichier de prédictions final.

## Comment ça marche

Tout le travail est dans le notebook 02-Conversion_rate_challenge_template, en six
chapitres :

1. **Chargement et exploration.** Vue d'ensemble, retrait des âges aberrants, chaque
   variable seule, puis chaque variable face à la conversion : matrice de corrélation,
   taux de conversion par pays, par source et par statut, pages visitées selon l'issue de
   la visite.
2. **Modèle de référence.** Une régression logistique sur le seul nombre de pages visitées,
   après un découpage train / test (90 / 10). C'est le score à battre.
3. **Modèle sur toutes les variables.** Standardisation des variables numériques, encodage
   des variables catégorielles en indicatrices, puis régression logistique sur les huit
   colonnes obtenues.
4. **Amélioration du f1-score.** Ajustement du seuil de décision, puis comparaison avec une
   forêt aléatoire dont les réglages sont choisis par validation croisée.
5. **Prédictions.** Le modèle retenu est réentraîné sur toutes les visites étiquetées,
   prétraitement compris, puis appliqué au fichier sans réponses.
6. **Lecture du modèle et recommandations.** Coefficients du modèle final et leviers
   d'action.

Trois choix ont orienté le travail :

- **Un seul découpage, stratifié, pour tous les modèles.** Le découpage conserve la part de
  3,2 % de visites converties dans le train et dans le test, et tous les modèles sont
  notés sur les mêmes visites, ce qui rend leurs scores comparables.
- **Un seuil de décision choisi sur le train.** Le modèle prédit une conversion quand la
  probabilité dépasse un seuil. Nous l'avons fixé à 0,40, meilleure valeur sur le train,
  sans regarder le test, qui reste réservé à la vérification finale.
- **La régression logistique plutôt que la forêt aléatoire.** La forêt ne fait pas mieux et
  sur-apprend légèrement. La régression logistique est plus simple, et ses coefficients se
  lisent directement, ce qui est indispensable pour les recommandations.

## Les résultats

Scores sur le test, soit 28 458 visites dont 918 converties :

| Modèle | f1-score |
|---|---|
| Régression logistique, pages visitées seules | 0,696 |
| Régression logistique, cinq variables, seuil 0,50 | 0,771 |
| Régression logistique, cinq variables, seuil 0,40 (retenu) | 0,767 |
| Forêt aléatoire | 0,749 |

Le modèle retenu retrouve 674 des 918 visites converties (rappel de 73 %) et 80 % de ses
alertes sont justes (précision). Sur le fichier sans réponses, il prédit 920 conversions
sur 31 620 visites, soit 2,9 %, un taux proche de celui observé.

Ce que le modèle nous apprend, toutes choses égales par ailleurs :

- **Le pays domine.** L'Allemagne, le Royaume-Uni et les États-Unis convertissent bien plus
  que la Chine, qui apporte un quart des visites et presque aucune conversion (0,13 %).
- **Les visites longues convertissent.** Le nombre de pages visitées est le signal continu
  le plus fort : la moitié des visites converties comptent 14 pages ou plus, contre 4 pages
  ou moins pour la moitié des autres.
- **Les visiteurs déjà venus convertissent cinq fois plus** que les nouveaux (7,2 % contre
  1,4 %).
- **L'âge joue peu**, légèrement en faveur des plus jeunes. **La source de trafic n'a
  presque aucun effet.**

Nos recommandations : comprendre pourquoi la Chine ne convertit pas avant tout
investissement sur ce trafic ; faire revenir les visiteurs et leur proposer l'abonnement
en priorité ; proposer l'abonnement en cours de visite, vers 7 à 8 pages, juste avant la
zone où les visites basculent, plutôt que dès la première page.

## Exécuter le projet

```
conversion-rate-challenge/
├── 02-Conversion_rate_challenge.ipynb            énoncé du projet
├── 02-Conversion_rate_challenge_template.ipynb   notebook de travail
├── data/
│   ├── raw/                                      les deux fichiers d'origine
│   └── processed/                                le fichier de prédictions
└── README.md
```

- **Prérequis** : Python 3.13 avec pandas, numpy, scikit-learn et plotly. Aucune clé ni
  fichier de configuration.
- **Lancement** : ouvrir le notebook de travail et exécuter toutes les cellules depuis la
  racine du projet. L'exécution complète prend environ 1 min 30 et régénère le fichier
  data/processed/conversion_data_test_predictions_JULIEN-model1.csv, une colonne converted,
  une ligne par visite du fichier sans réponses.

## Limites et suites possibles

- **Les visiteurs déjà abonnés sont invisibles.** Rien n'indique si un visiteur déjà venu
  était déjà abonné. Sa visite compte alors comme non convertie alors qu'elle ne pouvait
  pas l'être. L'effet, s'il existe, ne domine pas, mais il sous-estime le taux des
  visiteurs déjà venus.
- **Le nombre de pages est mesuré pendant la visite.** Il accompagne la conversion plus
  qu'il ne la précède : ce n'est pas un levier direct, d'où la recommandation sur le moment
  de la proposition d'abonnement.
- **Le modèle plafonne vers 0,77.** Ni le seuil ni la forêt aléatoire ne font mieux : les
  cinq variables ne suffisent pas à expliquer les conversions restantes. Une variable sur
  la durée de la visite ou le temps passé par page aiderait sans doute.
- **Le score des prédictions finales est inconnu**, faute de réponses pour le fichier de
  31 620 visites. Le seul score mesuré est celui du test.
- **L'écart de la Chine n'est pas expliqué** par les données : langue, parcours
  d'abonnement ou public différent, il faudrait une enquête côté site pour trancher.
