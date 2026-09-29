

# Building
- Java 8
- plugin lombok 
- $ docker-compose -f docker/docker-compose.yml up -d  (for mongoDB)

## Launch tests

$ mvn clean install

FROM python:3.11-slim

# Evite les prompts interactifs apt
ENV DEBIAN_FRONTEND=noninteractive

WORKDIR /app

# Installer dépendances système
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
    libreoffice \
 && rm -rf /var/lib/apt/lists/*

# Copier requirements
COPY requirements.txt .

# Installer dépendances Python depuis Internet (PyPI)
RUN pip install --no-cache-dir --upgrade pip setuptools wheel && \
    pip install --no-cache-dir -r requirements.txt

# Copier le code
COPY . .

# Rendre le script exécutable (safe)
RUN chmod +x docker/entrypoint.sh || true

# Volume logs
VOLUME ["/var/log"]

# Exposer port
EXPOSE 8080

# Commande de démarrage
CMD ["./docker/entrypoint.sh"]


s données stockées par Apigee API Hub en lien avec nos environnements Apigee Hybrid :


la résidence des données et le chiffrement pour Apigee API Hub :

Architecture & Flux : Aucun accès entrant vers notre cluster Kubernetes. Les flux sont uniquement sortants et sécurisés vers le Control Plane GCP.

Métadonnées uniquement : API Hub ne stocke que la structure de nos API (noms des proxys, révisions, URLs d'endpoints, environnements et contrats OpenAPI). Zéro donnée métier (payloads) ni secrets.

Résidence des données (Data Residency) : L'intégralité des métadonnées au repos (data-at-rest) est hébergée strictement dans la région GCP choisie (ex. europe-west9 à Paris).

Sécurité & Chiffrement (CMEK) : Nos données au repos sont chiffrées avec nos propres clés gérées via Cloud KMS (BYOK), stockées dans la même région.   
