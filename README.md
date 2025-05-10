# Data Pipeline Project

**Un pipeline complet** pour ingérer, transformer et visualiser les données Open Data de StatsBomb.

---

## 🚀 Overview

Ce projet met en place une architecture de **Data Engineering** end-to-end :

1. **Ingestion** des fichiers JSON de StatsBomb (compétitions, matches, events, lineups) depuis MinIO.
2. **Stockage raw** dans une base **PostgreSQL** (schéma `pipeline_db.public`).
3. **Orchestration** avec **Apache Airflow** (DAGs pour ingestion et chargement dans PostgreSQL).
4. **Transformation & modélisation** via **dbt** (layers staging, intermediate, fact/dimension).
5. **Exploration & reporting** avec **Apache Superset**.

> Cette stack 100% conteneurisée utilise Docker & Docker Compose pour faciliter le déploiement local.

---

## 📦 Architecture & Services

```text
+---------------+        +---------+        +-------------+
| StatsBomb API |  --->  |   MinIO |  --->  | Airflow DAGs |
+---------------+        +---------+        +-------------+
                                              |           
                                              v           
                                         +-----------+    
                                         | PostgreSQL|    
                                         +-----------+    
                                              |           
                                              v           
                                           +-----+        
                                           | dbt |        
                                           +-----+        
                                              |           
                                              v           
                                        +-------------+   
                                        |  Superset   |   
                                        +-------------+   
```

* **MinIO** : bucket S3 local pour stocker les JSON StatsBomb.
* **PostgreSQL** : persistance des données raw + métadonnées Airflow/Superset.
* **Airflow** : exécution planifiée (DAGs `statsbomb_full_ingestion` et `statsbomb_load_postgres`).
* **dbt** : modélisation SQL, documentation et tests.
* **Superset** : UI pour dashboards & exploration.

---

## 📂 Structure du dépôt

```text
├── dags/                    # Airflow DAGs
│   ├── statsbomb_ingestion.py   # Ingestion depuis MinIO vers raw tables
│   ├── statsbomb_load_to_postgres.py  # Upsert raw -> tables de schéma public_intermediate
│   └── dbt_pipeline.py         # (optionnel) wrapper DockerOperator
├── statsbomb_dbt/           # Projet dbt
│   ├── dbt_project.yml      
│   ├── models/              # staging/, intermediate/, fact_dim/
│   ├── analyses/, macros/, tests/, snapshots/  
│   └── profiles.yml (via volume dbt_profiles)
├── dbt_profiles/            # profiles.yml (connexion Postgres)
├── plugins/                 # éventuels plugins Airflow
├── logs/                    # logs Airflow (ignorés par Git)
├── .env                     # variables d’environnement (ignoré par Git)
├── .gitignore               # règles Git ignore
├── create_tables.sql        # DDL pour raw tables (compétitions, matches, events, lineups)
├── docker-compose.yml       # Définition des 5 services Docker
├── Dockerfile-airflow       # Image custom Airflow (providers S3, Postgres, Docker)
├── requirements.txt         # Dépendances Python pour scripts d’ingestion
├── superset_config.py       # Config Superset (connexion metadata DB)
└── README.md                # ← Vous êtes ici !
```

---

## ⚙️ Prérequis

* **Docker** (≥ 20.10)
* **Docker Compose** (≥ 1.27)
* (Optionnel) Proxy HTTP/HTTPS configuré via `HTTP_PROXY`, `HTTPS_PROXY`, `NO_PROXY`

---

## 🔧 Configuration

1. Dupliquez le fichier `.env.template` en `.env` à la racine.
2. Remplissez les variables :

   ```dotenv
   # MinIO (S3)
   MINIO_ROOT_USER=minioadmin
   MINIO_ROOT_PASSWORD=minio123

   # PostgreSQL
   POSTGRES_USER=postgres
   POSTGRES_PASSWORD=postgres
   POSTGRES_DB=pipeline_db

   # Airflow
   FERNET_KEY=<une clé Fernet générée>
   AIRFLOW_SECRET_KEY=<mot de passe webserver>

   # (optionnel) proxy
   HTTP_PROXY=
   HTTPS_PROXY=
   NO_PROXY=
   ```

> **Ne commitez jamais** votre vrai `.env` sur GitHub.

---

## ▶️ Lancer la stack

```bash
   git clone https://github.com/FaresDG/data-pipeline-project.git
   cd data-pipeline-project

   # Construire et démarrer tous les services
   docker-compose up -d --build

   # (1) Initialiser la base Airflow / Superset
   # Airflow va créer un utilisateur `admin` par défaut

   # (2) Vérifiez que tous les conteneurs sont "Up"
   docker-compose ps
```

**Ports exposés** :

* 9000 → MinIO Console
* 5432 → PostgreSQL
* 8080 → Airflow UI
* 8085 → dbt Docs
* 8088 → Superset UI

---

## 📝 Utilisation

### 1. Airflow

* **URL**: `http://localhost:8080`
* **DAGs disponibles**:

  * `statsbomb_full_ingestion` : ingère JSON → raw PostgreSQL
  * `statsbomb_load_postgres` : charge raw → tables finalisées

### 2. dbt

```bash
   # dans un shell dbt (container dbt)
   docker-compose exec dbt bash
   dbt debug            # vérifie la connexion
   dbt run              # exécute tous les modèles
   dbt test             # lance les tests définis
   dbt docs generate   
   dbt docs serve       # UI sur http://localhost:8085
```

### 3. Superset

* **URL**: `http://localhost:8088`
* **Connexion**:

  * Metadata DB = PostgreSQL `pipeline_db` (via `superset_config.py`)
* **Créer un dataset** pointant sur les tables modélisées par dbt (`public_intermediate` schema).

---

## 🧹 Arrêter & Nettoyer

```bash
   docker-compose down --volumes --remove-orphans
```

---

## 📖 Documentation & Références

* **StatsBomb Open Data** – [https://github.com/statsbomb/open-data](https://github.com/statsbomb/open-data)
* **Apache Airflow** – [https://airflow.apache.org](https://airflow.apache.org)
* **dbt** – [https://docs.getdbt.com](https://docs.getdbt.com)
* **Apache Superset** – [https://superset.apache.org](https://superset.apache.org)

---

> *Bon pipeline !* 🚀
> **Fares D.** – Mai 2025
