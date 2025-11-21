# 🧩 Customer Segmentation & Recommendation System

## 🎯 Objectif

Construire, à partir d’un historique de transactions e-commerce, un exemple complet de :

- **segmentation clients** avec les indicateurs RFM (Recency, Frequency, Monetary) ;
- **clustering non supervisé** avec K-means ;
- **système simple de recommandation de produits** basé sur les segments / clusters.

L’idée est de montrer tout le pipeline : des données brutes → aux segments clients → aux recommandations activables.

---

## 📊 Données

- **Source** : dataset *« Customer Segmentation & Recommendation System »* disponible sur Kaggle.  
- **Fichier brut** : `data/raw_data/data.csv`
- **Variables principales** :
  - `InvoiceNo` : identifiant de facture  
  - `StockCode` : identifiant produit  
  - `Description` : description du produit  
  - `Quantity` : quantité achetée  
  - `InvoiceDate` : date / heure de la transaction  
  - `UnitPrice` : prix unitaire  
  - `CustomerID` : identifiant client  
  - `Country` : pays  

Aucune donnée sensible ou nominative n’est utilisée.

---

## 🔁 Pipeline du projet

Tout le workflow est implémenté dans le notebook :

`customer_segmentation_recommendation_online_retail.ipynb`

### 1. Chargement des données

- Lecture du fichier brut `data/raw_data/data.csv`
- Vérification / correction des types (dates, numériques, etc.)

### 2. Nettoyage & préparation

- Suppression :
  - des lignes sans `CustomerID` ;
  - des quantités / prix unitaires négatifs ou nuls ;
  - des annulations et doublons.
- Création de :
  - `TotalPrice = Quantity × UnitPrice` ;
  - variables de temps (par ex. `YearMonth`).

👉 Résultat : un tableau de **transactions propres** (base pour l’EDA et les KPIs).

### 3. Analyse exploratoire (EDA)

- **KPIs de base** : nombre de clients, de produits, de pays, volume de transactions, revenu total.  
- **Top produits** :
  - par nombre de transactions,
  - par quantité vendue,
  - par revenu.
- **Répartition par pays** : volume de clients / transactions / revenu par pays.
- **KPIs temporels** :
  - revenu par mois,
  - nombre de clients actifs,
  - ARPU (Average Revenue Per User) mensuel.

### 4. Construction des indicateurs RFM

Pour chaque client (`CustomerID`) :

- **Recency** : nombre de jours depuis le dernier achat ;
- **Frequency** : nombre de factures (transactions distinctes) ;
- **Monetary** : revenu total généré par le client.

👉 Résultat : un tableau **RFM** avec une ligne par client.

### 5. Scoring RFM (R, F, M entre 1 et 5) & mini-segmentation

- Utilisation de `pd.qcut` pour découper Recency, Frequency et Monetary en **5 classes**.
- Attribution de scores :
  - **R_score** : 5 = très récent, 1 = très ancien ;
  - **F_score** : 5 = très fréquent, 1 = peu fréquent ;
  - **M_score** : 5 = gros contributeur, 1 = faible revenu.
- Score global :

  \[
  \text{RFM\_score} = R\_score + F\_score + M\_score
  \]

- Mini-segmentation à partir du score global :

  - `VIP / High value` (score élevé, récents, fréquents, gros revenus)
  - `Loyal`
  - `Occasional`
  - `At risk / Low value` (anciens, peu fréquents, faible revenu)

### 6. Clustering K-means & profilage

- Préparation des variables :
  - transformation logarithmique sur Frequency et Monetary pour limiter l’effet des très gros clients ;
  - standardisation (moyenne 0, écart-type 1).
- Choix du nombre de clusters **k** :
  - **méthode du coude (Elbow)** sur l’inertie ;
  - **score de silhouette** pour évaluer la qualité des clusters.
- Entraînement de K-means (k ≈ 4 dans ce projet).
- Profilage des clusters :
  - taille du cluster,
  - Recency moyenne,
  - Frequency moyenne,
  - Monetary moyen,
  - RFM_score moyen.

Chaque client reçoit un `cluster_kmeans` + un label lisible (`cluster_label`).

### 7. Top produits par cluster

À partir des transactions enrichies (avec le cluster de chaque client) :

- Agrégation par `cluster_kmeans` + `StockCode` + `Description`.
- Calcul, pour chaque produit et chaque cluster :
  - nombre de transactions (`transactions_count`) ;
  - quantité totale vendue (`quantity_sold`) ;
  - revenu généré (`revenue`).

👉 Résultat : les **produits stars** par segment de clients.

### 8. Système de recommandation

Logique simple :

1. Récupérer le **cluster** du client cible ;  
2. Prendre les **top produits** de ce cluster (par revenu ou volume) ;  
3. Retirer les produits déjà achetés par ce client ;  
4. Retourner les *N* meilleurs produits restants comme recommandations.

Objectif : montrer un exemple transparent de recommandation par similarité de segment (sans modèle complexe).

---

## 📂 Fichiers de sortie (processed data)

Le notebook génère plusieurs vues prêtes pour un dashboard :

- `data/processed_data/transactions_clean.csv`  
  → transactions nettoyées avec `TotalPrice`, `YearMonth`, etc.
- `data/processed_data/transactions_with_clusters.csv`  
  → transactions + cluster K-means et label de segment client.
- `data/processed_data/customers_rfm_clusters.csv`  
  → une ligne par client avec RFM, scores, cluster_kmeans, labels.
- `data/processed_data/top_products_by_cluster.csv`  
  → top produits (transactions, quantités, revenus) par cluster.

Ces fichiers sont pensés pour être utilisés dans **Power BI / Tableau / Google Sheets / Excel** ou tout autre outil de visualisation.

---

## 📊 Dashboard Power BI

Le fichier **`customer_segmentation_dashboard.pbix`** fournit un tableau de bord interactif construit à partir des données traitées (principalement `transactions_with_clusters.csv` et `customers_rfm_clusters.csv`).

Le dashboard est organisé en plusieurs vues :

1. **Page 1 – Vue globale (Customer Segmentation Dashboard)**  
   - Chiffre d’affaires total (2010–2011)  
   - Nombre total de clients uniques  
   - Nombre de transactions  
   - Évolution mensuelle du chiffre d’affaires  
   - Répartition du revenu par segment de clients (VIP / High value, Occasionnel, At risk / Low value)  
   - Top produits par revenu

2. **Page 2 – Vue par segment / cluster**  
   - Tableau détaillé des clients (RFM, scores, segment, cluster)  
   - Filtres par segment / cluster pour analyser le profil des clients et leur valeur.

3. **Page 3 – Top produits par segment**  
   - Slicer de segment client (`cluster_label`)  
   - Top 10 produits par chiffre d’affaires pour chaque segment  
   - Indicateurs : nombre de transactions, quantités vendues, chiffre d’affaires.

Le donut / slicer de segment est cliquable : lorsqu’on sélectionne un segment (par exemple les clients VIP), les autres visuels se filtrent automatiquement, ce qui permet d’analyser rapidement les produits et les périodes qui contribuent le plus à ce segment.

---

## 🛠 Environnement & installation

**Langage** : Python 3.x  

**Packages principaux** :

- `pandas`  
- `numpy`  
- `scikit-learn`  
- `matplotlib`  
- `seaborn` (optionnel)  
- `jupyter`  

Toutes les dépendances sont listées dans :

```text
requirements.txt
