# Compte rendu — TP4 : MaintAI devient un package Python

**Groupe :** À compléter  
**Membres :** TAHIR Charifa et MOMEN Ghita  
**Date :** 09/11/2026

---

## 1. Mise en place

### Résultat de `python -m pytest` après l'installation des dépendances

Après avoir créé un environnement virtuel et installé les dépendances avec `pip install -r requirements.txt`, nous avons exécuté les tests automatisés.

Lors de la vérification initiale du projet, deux tests liés au packaging échouaient :

```text
2 failed, 23 passed
```

Une vérification ultérieure sur Windows a donné le résultat suivant :

```text
1 failed, 26 passed, 1 warning in 5.25s
```

### Quels sont les deux tests qui échouent, et pourquoi ?

**Premier test : `test_installed_version_matches_the_code`**

Ce test vérifie que la version du package installé correspond à celle définie dans le code source.

L'échec initial s'explique par le fait que MaintAI n'était pas encore installé comme distribution Python. Les métadonnées nécessaires à la vérification de sa version n'étaient donc pas disponibles.

**Deuxième test : `test_command_is_declared`**

Ce test vérifie que la commande `maintai-predict` est déclarée dans la configuration du package.

Il échouait parce que la section `[project.scripts]` n'était pas correctement configurée.

Ces deux problèmes devaient être corrigés dans le cadre du premier sprint.

---

## 2. Sprint 1 : `pyproject.toml` et `pip install -e .`

### Configuration des dépendances et du point d'entrée

Nous avons modifié le fichier `pyproject.toml` afin de déclarer les dépendances nécessaires au fonctionnement de MaintAI.

```toml
[project]
name = "maintai"
dynamic = ["version"]
description = "Prédiction de pannes industrielles à partir de données de capteurs"
readme = "README.md"
requires-python = ">=3.11"

dependencies = [
    "numpy>=2.2",
    "pandas>=2.2",
    "scikit-learn>=1.6",
    "joblib>=1.4",
    "httpx>=0.28"
]

[project.optional-dependencies]
api = ["fastapi>=0.115", "uvicorn[standard]>=0.30"]
ui = ["streamlit>=1.40"]
dev = ["pytest>=8", "ruff"]

[project.scripts]
maintai-predict = "maintai.cli:main"
```

### Pourquoi `pytest` et `fastapi` ne sont-ils pas dans `dependencies` ?

`pytest` est un outil de test utilisé pendant le développement. Il n'est pas nécessaire pour exécuter les prédictions du package.

`fastapi` permet de développer une interface API, mais n'est pas indispensable à l'utilisation du programme en ligne de commande.

Ces bibliothèques sont donc déclarées comme dépendances optionnelles :

- `dev` pour les outils de développement et de test ;
- `api` pour les fonctionnalités de l'API ;
- `ui` pour l'interface Streamlit.

Cette organisation permet de limiter les dépendances installées par défaut.

### Installation du package en mode éditable

Nous avons exécuté :

```powershell
python -m pip install -e .
```

Cette commande installe MaintAI en mode éditable.

Les modifications apportées au code source peuvent ainsi être prises en compte sans reconstruire le package.

### Sortie de `pip show maintai`

```text
Name: maintai
Version: 0.2.0
Summary: Prédiction de pannes industrielles à partir de données de capteurs
Home-page:
```

*La sortie exacte des quatre premières lignes est à confirmer dans le terminal.*

### Résultat des tests à la fin du sprint 1

Après la correction de `pyproject.toml`, les tests doivent être relancés :

```powershell
python -m pytest -v
```

**Résultat final :** à compléter avec la dernière ligne affichée dans le terminal.

L'objectif est de valider les 27 tests.

### Exécution de `maintai-predict` depuis un autre dossier

Nous avons utilisé la commande suivante :

```powershell
maintai-predict 72 6 85 15000
```

Dans la correction initiale, le modèle entraîné a produit :

```text
Probabilité de panne : 95 % (ALERTE)
```

Ce résultat dépend des données et du modèle utilisés.

Nous avons également utilisé :

```powershell
python -c "import maintai; print(maintai.__file__)"
```

**Où Python trouve-t-il le code de MaintAI ?**

Grâce à l'installation en mode éditable, Python charge les fichiers sources du projet depuis le répertoire de développement.

Sous Windows, le chemin doit correspondre au fichier `maintai/__init__.py` du projet.

### Modification du message dans `maintai/cli.py`

Après une modification du texte affiché dans `maintai/cli.py`, une nouvelle exécution de `maintai-predict` doit prendre en compte le changement sans nouvelle installation.

Cela s'explique par le fonctionnement du mode éditable : l'installation fait référence au code source.

**Vérification pratique :** à confirmer après modification et exécution sur Windows.

---

## 3. Sprint 2 : construire et installer la roue

### Construction du package

Nous avons installé l'outil de construction :

```powershell
python -m pip install build
```

Puis exécuté :

```powershell
python -m build
```

Cette commande permet de produire une distribution source et une distribution wheel.

### Contenu du dossier `dist`

La construction doit produire les fichiers suivants :

```text
maintai-0.2.0-py3-none-any.whl
maintai-0.2.0.tar.gz
```

Le fichier `.whl` contient la distribution installable du package Python.

Le fichier `.tar.gz` contient la distribution source.

*La sortie exacte de `dir dist` reste à coller depuis le terminal.*

### Vérification du contenu du wheel

Commande utilisée :

```powershell
tar -tf dist\maintai-0.2.0-py3-none-any.whl
```

Le wheel construit dans la correction contient notamment :

```text
maintai/__init__.py
maintai/cli.py
maintai/client.py
maintai/config.py
maintai/data.py
maintai/dataset.py
maintai/predict.py
maintai/train.py
maintai/api/__init__.py
maintai/api/main.py
maintai/api/schemas.py
maintai/api/security.py
maintai-0.2.0.dist-info/METADATA
maintai-0.2.0.dist-info/WHEEL
maintai-0.2.0.dist-info/entry_points.txt
maintai-0.2.0.dist-info/top_level.txt
maintai-0.2.0.dist-info/RECORD
```

### Le modèle, les tests et le dossier `ui/` sont-ils inclus ?

Non, dans la distribution corrigée.

Le fichier `pyproject.toml` utilise :

```toml
[tool.setuptools.packages.find]
include = ["maintai*"]
```

Cette configuration limite la découverte automatique des packages aux modules Python de MaintAI.

Le modèle entraîné constitue un artefact distinct, tandis que les tests et l'interface sont destinés à d'autres usages.

Ils ne sont donc pas nécessairement inclus dans le wheel.

### Installation dans un environnement virtuel neuf

Nous avons créé un environnement de test :

```powershell
python -m venv .venv-test
```

Puis installé le wheel généré.

Lors des premières tentatives, les erreurs suivantes sont apparues :

```text
ModuleNotFoundError: No module named 'joblib'
```

Puis :

```text
ModuleNotFoundError: No module named 'pandas'
```

Ces erreurs étaient liées à la déclaration incorrecte des dépendances dans `pyproject.toml`.

Après correction et réinstallation du package, nous avons exécuté :

```powershell
.\.venv-test\Scripts\python.exe -m pip check
```

Résultat obtenu :

```text
No broken requirements found.
```

Cela signifie qu'aucune dépendance manquante ou incompatible n'a été détectée dans cet environnement.

### Résultats des essais dans le nouvel environnement

| Étape | Résultat |
|---|---|
| Sans `MAINTAI_MODELS_DIR` | Résultat exact à compléter : le modèle peut être introuvable si son chemin par défaut ne permet pas de le localiser. |
| Avec `MAINTAI_MODELS_DIR` vers `models` | Résultat de la prédiction à confirmer après exécution. |
| Après `pip install scikit-learn==1.8.0` | Résultat et éventuel avertissement à confirmer. |

### D'où vient `InconsistentVersionWarning` ?

Cet avertissement peut apparaître lorsqu'un modèle sauvegardé avec une version de scikit-learn est chargé avec une autre version.

La sérialisation du modèle dépend de la compatibilité entre les versions de la bibliothèque.

Une différence de version peut provoquer des avertissements ou des comportements incompatibles.

### Quelle ligne de `models/model_info.json` permet de le comprendre ?

Dans le modèle fourni dans la correction :

```json
"sklearn_version": "1.8.0"
```

Cette information correspond à la version de scikit-learn utilisée pendant l'entraînement.

Il faut comparer cette valeur avec la version installée dans l'environnement de test.

Si les versions sont identiques, un avertissement de différence de version ne devrait normalement pas apparaître pour ce modèle.

---

## 4. Questions de compréhension

### 1. Quelle différence entre `requirements.txt` et `dependencies` de `pyproject.toml` ?

Le fichier `requirements.txt` sert à installer les bibliothèques nécessaires à un environnement Python donné.

Il peut notamment être utilisé pour préparer un environnement de développement, de test ou de production.

La section `dependencies` de `pyproject.toml` décrit les dépendances obligatoires du package.

Lorsque le package est installé avec pip, les dépendances déclarées sont automatiquement prises en compte.

La principale différence est donc que `requirements.txt` concerne la préparation d'un environnement, tandis que `dependencies` décrit les besoins du package distribué.

### 2. Quelle différence entre `pip install -e .` et l'installation d'un wheel ?

La commande :

```powershell
pip install -e .
```

installe le package en mode éditable.

Les modifications du code source sont directement prises en compte lors des prochaines exécutions.

La commande :

```powershell
pip install dist\maintai-0.2.0-py3-none-any.whl
```

installe une distribution déjà construite.

Les changements ultérieurs dans le dossier source ne modifient pas automatiquement le package installé.

Il faut reconstruire puis réinstaller le wheel pour intégrer ces modifications.

Le mode éditable est donc adapté au développement, tandis que le wheel est adapté à la distribution.

### 3. Qu'est-ce qu'un point d'entrée `[project.scripts]` ?

Un point d'entrée permet d'associer une commande du terminal à une fonction Python.

Dans le projet MaintAI :

```toml
[project.scripts]
maintai-predict = "maintai.cli:main"
```

Cette configuration permet d'utiliser directement la commande :

```powershell
maintai-predict 72 6 85 15000
```

Elle appelle la fonction `main` située dans le module `maintai.cli`.

Sous Windows, pip crée l'exécutable dans le dossier `Scripts` de l'environnement virtuel, par exemple :

```text
.venv\Scripts\maintai-predict.exe
```

---

## 5. Le rendu Git

### Initialisation du dépôt

Lors de la première tentative, Git a retourné :

```text
fatal: not a git repository (or any of the parent directories): .git
```

Cela signifiait que le projet n'était pas encore initialisé comme dépôt Git.

Nous avons donc préparé l'initialisation et la configuration du dépôt.

### Dépôt GitHub

Adresse prévue :

https://github.com/ghita701/maintai-tp4

Une erreur de configuration avait été identifiée : le dépôt distant utilisait initialement l'adresse fictive `TON-USERNAME`.

Il fallait la remplacer par le véritable identifiant GitHub.

### Lien de la pull request

**À compléter après création de la pull request.**

La publication d'une branche avec `git push` ne crée pas automatiquement une pull request.

### Le tag `tp4` existe-t-il sur GitHub ?

**À vérifier avec :**

```powershell
git tag
git ls-remote --tags origin
```

### Les dossiers `dist/`, `build/` et `maintai.egg-info/` apparaissaient-ils avant le commit ?

**Réponse :** à confirmer avec le résultat de `git status`.

Ces dossiers doivent normalement être exclus du suivi Git grâce à `.gitignore`, sauf consigne contraire.

Nous avons prévu d'ignorer notamment :

```gitignore
.venv/
.venv-test/
__pycache__/
.pytest_cache/
.ruff_cache/
*.pyc
*.egg-info/
build/
dist/
.env
```

Cette configuration permet d'éviter la publication de fichiers temporaires et d'environnements virtuels.

---

## 6. Ce qui reste flou

Ce TP nous a permis de mieux comprendre le fonctionnement du packaging Python et les différentes étapes nécessaires pour rendre une application installable.

Nous avons notamment appris à configurer les dépendances dans `pyproject.toml`, à déclarer une commande exécutable et à construire une distribution wheel.

Nous avons rencontré plusieurs difficultés, notamment les doublons dans le fichier TOML, les dépendances manquantes lors de l'installation dans un nouvel environnement et la configuration du dépôt Git.

La distinction entre le mode éditable et une installation classique est maintenant plus claire.

Nous souhaitons encore approfondir la gestion des versions de bibliothèques lors du chargement de modèles entraînés, ainsi que les bonnes pratiques de distribution et de publication des packages Python.

---

**Réalisé par :**

**TAHIR Charifa**  
**MOMEN Ghita**

**TP4 — MaintAI : Packaging et distribution Python**
