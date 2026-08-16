# Analyse thématique du corpus de Zola

Ce projet explore plusieurs méthodes de topic modeling sur un corpus de romans d’Émile Zola. Il compare une factorisation NMF et un clustering KMeans à partir d’une même représentation TF-IDF des textes.

Le dépôt correspond à une phase expérimentale : les données sont préparées et les deux méthodes ont été testées, mais le nombre de thèmes et leur interprétation ne constituent pas encore un modèle définitif.

## État du projet

- chargement et segmentation du corpus : terminés ;
- exploration statistique des segments : réalisée ;
- nettoyage et lemmatisation : terminés ;
- expérimentation avec NMF : réalisée avec 29 topics ;
- expérimentation avec KMeans : réalisée avec 29 clusters ;


Les noms donnés aux topics, aux clusters et aux familles de clusters sont des interprétations manuelles fondées sur leurs mots les plus représentatifs. Ils doivent donc être considérés comme provisoires.

## Structure du projet

```text
analyse_topic_zola/
├── notebooks/
│   ├── 0_segmentation_paquet.ipynb
│   ├── 00_statistique_sur_segment.ipynb
│   ├── 1_lemmatisation_paquet.ipynb
│   ├── 2_nmf_topics_phrases.ipynb
│   └── 3_kmeans_clustering.ipynb
├── data/
│   ├── 1_raw/
│   │   └── corpus_zola/
│   ├── 2_interim/
│   │   └── paquets_phrases.csv
│   └── 3_processed/
│       └── paquet_phrases_lemm.csv
├── requirements.txt
└── README.md
```

## Installation

Le projet utilise Python 3.12.

```bash
python3.12 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

Sous Windows :

```powershell
py -3.12 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

Les notebooks peuvent ensuite être ouverts dans VS Code ou Jupyter. Si JupyterLab n’est pas déjà installé :

```bash
python -m pip install jupyterlab
jupyter lab
```

Il faut sélectionner le noyau Python associé à l’environnement `.venv`.

## Ordre d’exécution

Les notebooks sont prévus pour être exécutés dans l’ordre suivant :

| Ordre | Notebook | Rôle |
|---:|---|---|
| 1 | [`0_segmentation_paquet.ipynb`](notebooks/0_segmentation_paquet.ipynb) | Charger les romans et les regrouper en paquets de lignes |
| 2 | [`00_statistique_sur_segment.ipynb`](notebooks/00_statistique_sur_segment.ipynb) | Contrôler la segmentation et étudier la longueur des paquets |
| 3 | [`1_lemmatisation_paquet.ipynb`](notebooks/1_lemmatisation_paquet.ipynb) | Nettoyer et lemmatiser les textes avec spaCy |
| 4 | [`2_nmf_topics_phrases.ipynb`](notebooks/2_nmf_topics_phrases.ipynb) | Extraire et explorer 29 topics avec NMF |
| 5 | [`3_kmeans_clustering.ipynb`](notebooks/3_kmeans_clustering.ipynb) | Construire et explorer 29 clusters avec KMeans |

Les deux derniers notebooks sont indépendants une fois le corpus lemmatisé. Ils peuvent donc être exécutés dans l’ordre souhaité.

Les chemins relatifs présents dans le code supposent que le répertoire de travail correspond au dossier `notebooks/`.

## 1. Segmentation du corpus

Le notebook `0_segmentation_paquet.ipynb` charge les fichiers `.txt` placés dans `data/1_raw/corpus_zola/`. Le corpus actuel contient **31 romans**.

La segmentation repose sur la mise en forme des fichiers source :

- le texte est découpé à chaque retour à la ligne ;
- les lignes vides sont supprimées ;
- les lignes restantes sont regroupées par blocs de 40 ;
- le dernier bloc de chaque roman peut contenir moins de 40 lignes.

Il ne s’agit donc pas d’une segmentation linguistique en phrases : chaque ligne non vide du fichier source est traitée comme une unité.

Le résultat est enregistré dans :

```text
data/2_interim/paquets_phrases.csv
```

| Colonne | Contenu |
|---|---|
| `nom_fichier` | Nom du fichier source |
| `id_paquet` | Position du paquet dans le roman, à partir de 0 |
| `phrases_paquet` | Texte brut des lignes regroupées |

L’exécution actuelle produit **5 327 paquets**.

## 2. Exploration statistique

Le notebook `00_statistique_sur_segment.ipynb` vérifie la qualité du fichier segmenté, puis décrit la taille des paquets et leur répartition entre les romans.

### Contrôles du jeu de données

| Contrôle | Résultat actuel |
|---|---:|
| Valeurs manquantes | 0 |
| Doublons sur `nom_fichier` et `id_paquet` | 0 |
| Doublons sur le texte des paquets | 0 |

### Répartition des paquets

| Indicateur | Valeur actuelle |
|---|---:|
| Nombre de romans | 31 |
| Moyenne de paquets par roman | 171,84 |
| Médiane de paquets par roman | 171 |
| Minimum | 60 |
| Maximum | 259 |

### Longueur des paquets

Le nombre de mots est estimé avec un découpage du texte sur les espaces.

| Indicateur | Valeur actuelle |
|---|---:|
| Nombre de paquets | 5 327 |
| Moyenne | 785,30 mots |
| Médiane | 753 mots |
| Premier quartile | 636 mots |
| Troisième quartile | 902,5 mots |
| Minimum | 42 mots |
| Maximum | 2 944 mots |

Le notebook relève également 12 paquets de moins de 300 mots et 245 paquets de plus de 1 200 mots. Les extrêmes sont affichés pour inspection, mais ils ne sont pas retirés du corpus.

Cette étape est descriptive et ne crée pas de nouveau fichier.

## 3. Nettoyage et lemmatisation

Le notebook `1_lemmatisation_paquet.ipynb` utilise le modèle français `fr_core_news_lg` de spaCy. Le parser et la reconnaissance d’entités nommées sont désactivés afin d’alléger le traitement.

Pour chaque paquet, le code :

- transforme les tokens en lemmes minuscules ;
- retire les stop words définis par spaCy ;
- retire la ponctuation, les nombres et les espaces ;
- conserve uniquement les noms (`NOUN`) et les adjectifs (`ADJ`) ;
- conserve uniquement les lemmes de plus de deux caractères ;
- applique une liste complémentaire de mots à exclure.

La liste complémentaire contient notamment des verbes très fréquents, des formes peu informatives et plusieurs noms de personnages. Elle a été enrichie manuellement au fil des expérimentations.

Le résultat est enregistré dans :

```text
data/3_processed/paquet_phrases_lemm.csv
```

Le fichier conserve les **5 327 paquets** et ne contient actuellement aucun texte lemmatisé vide.

| Colonne | Contenu |
|---|---|
| `nom_fichier` | Nom du fichier source |
| `id_paquet` | Position du paquet dans le roman |
| `phrases_paquet` | Texte brut du paquet |
| `phrases_lemm` | Suite de lemmes utilisée pour la modélisation |

## 4. Représentation TF-IDF

Les notebooks NMF et KMeans utilisent la même vectorisation :

```python
TfidfVectorizer(
    min_df=3,
    max_df=0.8
)
```

Un terme doit donc apparaître dans au moins trois paquets et dans au plus 80 % du corpus pour être conservé.

Avec les données actuelles, la matrice TF-IDF a une dimension de **5 327 × 11 640** : une ligne par paquet et une colonne par terme retenu.

## 5. Modélisation avec NMF

Le notebook `2_nmf_topics_phrases.ipynb` applique une factorisation en matrices non négatives à la représentation TF-IDF.

### Recherche du nombre de topics

L’erreur de reconstruction est calculée une première fois pour des valeurs comprises entre 2 et 39, puis de nouveau entre 20 et 39. Le notebook ne fait pas apparaître de coude suffisamment net pour déterminer automatiquement un nombre optimal de topics.

L’exploration actuelle utilise donc **29 topics** avec les paramètres suivants :

```python
NMF(
    n_components=29,
    random_state=42,
    init="nndsvda",
    solver="cd",
    max_iter=1000
)
```

L’erreur de reconstruction enregistrée pour cette exécution est de **67,6815**.

### Interprétation des topics

Le notebook affiche les dix termes les plus importants de chaque topic, puis leur attribue un nom manuel. Les thèmes obtenus couvrent notamment :

- la famille, l’enfance et les relations amoureuses ;
- le travail, la mine et le monde ouvrier ;
- la religion et les pèlerinages ;
- la finance, la bourse et l’argent ;
- le chemin de fer ;
- la guerre et l’insurrection ;
- la peinture, le commerce et la vie urbaine.

La matrice document-topic est normalisée ligne par ligne. Chaque paquet reçoit ensuite :

- son topic dominant ;
- le poids normalisé de ce topic ;
- le nom manuel correspondant.

Le notebook permet enfin :

- d’afficher les cinq passages les plus représentatifs de chaque topic ;
- de compter les paquets associés à chaque topic dominant ;
- de calculer le poids moyen des topics dans le corpus ;
- de comparer leurs poids moyens entre les œuvres ;
- de visualiser les nombres et proportions de paquets par topic et par œuvre.

Ces enrichissements et graphiques restent dans le notebook : aucun fichier de résultats NMF n’est exporté actuellement.

## 6. Clustering avec KMeans

Le notebook `3_kmeans_clustering.ipynb` applique KMeans à la même matrice TF-IDF.

### Recherche du nombre de clusters

Le score de silhouette est calculé pour chaque valeur de `k` comprise entre 2 et 40, avec `random_state=42` et `n_init=10`.

Le meilleur score de la série est obtenu avec 37 clusters, mais il reste faible :

| Configuration | Score de silhouette |
|---|---:|
| `k=37` | 0,010506 |
| `k=29` | 0,010298 |

Le notebook poursuit l’exploration avec **29 clusters** :

```python
KMeans(
    n_clusters=29,
    random_state=42,
    n_init=10
)
```

Ce choix doit être compris comme un réglage exploratoire et non comme un optimum statistiquement établi. Les scores proches de zéro indiquent que les groupes restent peu séparés dans l’espace TF-IDF.

### Interprétation des clusters

Comme pour NMF, les dix termes les plus importants de chaque centroïde sont affichés et les clusters reçoivent un nom manuel. Ils couvrent notamment la religion, la politique, la famille, le commerce, le monde ouvrier, la mine, la guerre, la nature et les espaces domestiques.

Une classification ascendante hiérarchique est ensuite appliquée aux 29 centroïdes KMeans :

- méthode de liaison de Ward ;
- seuil de coupure fixé à `0.40` ;
- regroupement des clusters en cinq familles exploratoires.

Le notebook génère :

- un dendrogramme des clusters ;
- le nombre de paquets par cluster ;
- le nombre de paquets par famille hiérarchique ;
- la distribution proportionnelle des clusters dans chaque œuvre ;
- la distribution des grandes familles dans chaque œuvre.

Les noms des cinq familles sont également attribués manuellement et devront être revérifiés lors de la validation du modèle.

Comme pour NMF, les affectations et les graphiques KMeans ne sont pas exportés dans un fichier séparé.

## Fichiers produits

| Fichier | Contenu |
|---|---|
| `data/2_interim/paquets_phrases.csv` | 5 327 paquets issus de la segmentation des 31 romans |
| `data/3_processed/paquet_phrases_lemm.csv` | Corpus segmenté enrichi avec les textes lemmatisés |

Les modèles entraînés, les affectations thématiques et les figures ne sont pas sauvegardés automatiquement. Les notebooks doivent être réexécutés pour les reconstruire.

## Reproductibilité et limites

- Les modèles NMF et KMeans utilisent la graine aléatoire `42`.
- La segmentation dépend directement des retours à la ligne des fichiers source.
- La liste complémentaire de stop words et les noms des thèmes relèvent de choix manuels.
- Le choix de 29 topics ou clusters est exploratoire : NMF ne présente pas de coude net et KMeans obtient des scores de silhouette faibles.
- Les paquets ont des tailles variables et les valeurs extrêmes sont conservées.
- Les chemins relatifs imposent d’exécuter les notebooks depuis le dossier `notebooks/`.
- Les résultats de modélisation ne sont pas encore exportés ni versionnés séparément.

## Possibilité de suite du projet

Les prochaines étapes consistent notamment à :

- comparer plus formellement NMF et KMeans ;
- évaluer la cohérence et la stabilité des thèmes ;
- réexaminer le nombre de topics et de clusters ;
- consolider les noms attribués manuellement ;
- exporter les affectations et les visualisations utiles ;
- relier plus systématiquement les thèmes aux œuvres et aux passages du corpus.
