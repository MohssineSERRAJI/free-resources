# Construction multi-étapes : Environnement de développement/build
FROM continuumio/miniconda3:latest AS build

# Définir le répertoire de travail
WORKDIR /app

# Copier le fichier d'environnement
COPY environment.yml .

# Créer l'environnement conda
RUN conda env create -f environment.yml

# Activer l'environnement et installer les dépendances supplémentaires
SHELL ["conda", "run", "-n", "myenv", "/bin/bash", "-c"]
RUN pip install --no-cache-dir "tensorflow>=2.8.0" "torch>=1.11.0"

# Étape d'exécution : Image plus petite et optimisée
FROM continuumio/miniconda3:slim AS runtime

# Définir le répertoire de travail
WORKDIR /app

# Copier l'environnement conda depuis l'étape de build
COPY --from=build /opt/conda/envs/myenv /opt/conda/envs/myenv

# Ajouter l'environnement conda au PATH
ENV PATH /opt/conda/envs/myenv/bin:$PATH

# Dépendances système
RUN apt-get update && apt-get install -y --no-install-recommends \
    libgomp1 \
    libxgboost-dev \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Copier le code de l'application (pas les modèles)
COPY ./app /app

# Définir un volume pour les modèles (les garde séparés du code)
VOLUME /app/models

# Vérification de santé
HEALTHCHECK --interval=30s --timeout=30s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

# Définir le point d'entrée pour exécuter avec l'environnement conda
ENTRYPOINT ["conda", "run", "--no-capture-output", "-n", "myenv", "python", "app.py"]