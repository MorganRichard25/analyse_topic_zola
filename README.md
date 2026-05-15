# Topic Modeling sur un corpus de Zola

Ce dépôt contient des expérimentations de **topic modeling** appliquées à un corpus de romans d’Émile Zola.

L’objectif est d’explorer différentes méthodes de détection automatique de thèmes afin d’identifier des regroupements lexicaux pertinents dans les textes.

## Objectif du projet

Le but est de tester plusieurs approches de topic modeling pour faire émerger des thèmes récurrents dans le corpus de Zola, puis de comparer les résultats obtenus.

Pour le moment, deux méthodes principales ont été utilisées :

- **NMF** (*Non-negative Matrix Factorization*)
- **LDA** (*Latent Dirichlet Allocation*)

## Prétraitement des textes

Avant l’application des modèles, les textes ont été prétraités afin de réduire le bruit lexical :

- nettoyage général du texte ;
- segmentation du corpus ;
- lemmatisation ;
- suppression des mots vides ;
- conservation de certaines catégories grammaticales pertinentes ;
- vectorisation des textes.

## Méthodes testées

### NMF

La méthode NMF est utilisée pour extraire des thèmes à partir d’une représentation vectorielle du corpus.

Elle semble pour le moment produire les résultats les plus intéressants, même si les paramètres doivent encore être ajustés pour obtenir des topics plus stables et plus interprétables.

### KMeans

La méthode KMeans a également été testée afin de comparer les résultats avec ceux obtenus par NMF.

Elle permet d’obtenir une autre représentation thématique du corpus, mais les premiers résultats nécessitent encore des améliorations et des réglages.

## État actuel du projet

Le projet est encore en phase d’expérimentation.

Les premiers notebooks permettent surtout de comparer les méthodes et d’observer les différences entre les topics générés automatiquement.

Les prochains objectifs sont :

- améliorer la stabilité des topics ;
- tester différents paramètres ;
- comparer les résultats entre NMF et LDA ;
- annoter manuellement les topics obtenus ;
- relier les topics à des passages précis du corpus.

## Technologies utilisées

- Python
- pandas
- scikit-learn => pour NMF et KMeans
- spaCy


## Remarque

Ce projet est exploratoire. Les résultats actuels ne sont pas définitifs et servent principalement à identifier les méthodes les plus adaptées à l’analyse thématique du corpus de Zola.