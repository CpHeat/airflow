# Projet Airflow - Pipeline ML d'Analyse Publicitaire

## Description

Projet académique d'orchestration d'un pipeline Machine Learning avec Apache Airflow pour la prédiction de clics publicitaires. Le système automatise l'entraînement d'un modèle de régression logistique et expose une API Flask pour consulter l'état des exécutions.

## Architecture

### Composants principaux

- **Airflow** (v3.1.0) : Orchestration du pipeline ML
- **PostgreSQL** (v16) : Base de données métier
- **Redis** (v7.2) : Message broker pour Celery
- **Flask** (API) : Interface de consultation des résultats
- **scikit-learn** : Framework ML

### DAGs implémentés

#### 1. `Airflow_Lab2` (Pipeline ML)
Pipeline séquentiel d'entraînement de modèle :
```
load_data → data_preprocessing → separate_data_outputs
    → build_save_model → load_model → trigger_dag_task
                               └→ send_email
```

**Tâches :**
- Chargement du dataset `advertising.csv`
- Prétraitement (MinMaxScaler + StandardScaler)
- Entraînement d'un modèle LogisticRegression
- Sauvegarde et chargement du modèle
- Envoi d'email de notification
- Déclenchement du DAG Flask

#### 2. `Airflow_Lab2_Flask` (API de monitoring)
Expose une API Flask (port 5555) avec routes :
- `/` : Redirection selon statut
- `/success` : Page de succès d'entraînement
- `/failure` : Page d'échec
- `/health` : Health check

## Dataset

**advertising.csv** : 1000 lignes de données publicitaires
- Features : Temps passé sur site, Âge, Revenu, Usage Internet, Genre
- Target : Clicked on Ad (0/1)

## Structure du projet

```
airflow/
├── dags/
│   ├── main.py                    # DAG principal ML
│   ├── Flask_API.py               # DAG Flask API
│   ├── src/
│   │   └── model_development.py  # Fonctions ML
│   ├── data/
│   │   └── advertising.csv       # Dataset
│   └── templates/
│       ├── success.html          # Template succès
│       └── failure.html          # Template échec
├── model/                         # Modèles sauvegardés
├── logs/                          # Logs Airflow
├── config/                        # Configuration Airflow
├── plugins/                       # Plugins personnalisés
├── docker-compose.yaml            # Configuration Docker
└── requirements.txt               # Dépendances Python
```

## Installation et lancement

### Prérequis
- Docker et Docker Compose
- 4GB RAM minimum
- 10GB espace disque

### Démarrage

```bash
# Lancer l'environnement Airflow
docker compose --profile flower up -d

# Vérifier que tous les services sont lancés
docker compose ps
```

### Accès aux interfaces

- **Airflow UI** : http://localhost:8080
  - Username : `airflow`
  - Password : `airflow`

- **Flask API** : http://localhost:5555 (après déclenchement du DAG Flask)

## Utilisation

1. Accéder à l'interface Airflow (http://localhost:8080)
2. Activer le DAG `Airflow_Lab2`
3. Déclencher manuellement l'exécution via le bouton "Play"
4. Suivre l'exécution dans la vue Graph/Grid
5. Le DAG `Airflow_Lab2_Flask` se déclenche automatiquement
6. Consulter l'API Flask (http://localhost:5555) pour voir le statut

## Workflow ML détaillé

1. **load_data_task** : Charge `advertising.csv` → sauvegarde en pickle
2. **data_preprocessing_task** :
   - Split train/test (70/30)
   - Feature scaling (MinMaxScaler + StandardScaler)
   - Features : Daily Time Spent, Age, Area Income, Daily Internet Usage, Male
3. **build_save_model_task** : Entraîne LogisticRegression → sauvegarde dans `/model/`
4. **load_model_task** : Charge le modèle et évalue sur test set
5. **send_email** : Notification email (vers higha59@gmail.com)
6. **trigger_dag_task** : Déclenche le DAG Flask API

## Technologies utilisées

- **Apache Airflow 3.1.0** : Orchestration
- **CeleryExecutor** : Exécution distribuée
- **Flask** : API REST
- **scikit-learn** : ML (LogisticRegression, preprocessing)
- **pandas & numpy** : Manipulation de données
- **PostgreSQL** : Database backend
- **Redis** : Message broker

## Configuration avancée

### Variables d'environnement (docker-compose.yaml)

```yaml
AIRFLOW__CORE__EXECUTOR: CeleryExecutor
AIRFLOW__CORE__LOAD_EXAMPLES: 'true'
AIRFLOW_CONFIG: '/opt/airflow/config/airflow.cfg'
```

### Email notifications

Configuration SMTP requise dans `config/airflow.cfg` pour les alertes email.

## Logs et monitoring

- Logs Airflow : `./logs/dag_id=<dag_name>/`
- Logs task-specific : `./logs/dag_id=<dag_name>/run_id=<run_id>/task_id=<task_id>/`

## Notes importantes

- Schedulé en `@daily` (désactivé avec `catchup=False`)
- Un seul run actif simultanément (`max_active_runs=1`)
- Flask API bloque intentionnellement pour rester active
- Modèles sauvegardés dans `/opt/airflow/model/` (mappé sur `./model/`)

## Auteur

Projet académique - Formation Simplon AI Developer (W26)

## Licence

Apache License 2.0
