

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



To clarify the exact data residency scope and confirm the setup for :

Data Residency vs. Advanced Data Residency: From a product standpoint, official documentation states that full Advanced Data Residency (ADR) is not supported for Apigee Hybrid because Google does not manage the Runtime plane (in-transit/in-use data on local infrastructure). However, for the Google-managed Control Plane, enabling CMEK encryption enforces an advanced data residency setup (Control Plane in EUROPE, Analytics in europe-central2 Warsaw).

Assured Workloads: An Assured Workloads folder is not required, as this advanced Control Plane setup with CMEK was provisioned directly at the Apigee organization level.

Endpoint Recommendation (apigee.eu.rep.googleapis.com): Because the Control Plane operates under these advanced residency requirements, Apigee Support previously confirmed that using [https://apigee.eu.rep.googleapis.com](https://apigee.eu.rep.googleapis.com) is mandatory for administrative calls to ensure compliance and avoid authorization errors (403).

Please ensure your CI/CD pipelines and deployment tools target [https://apigee.eu.rep.googleapis.com](https://apigee.eu.rep.googleapis.com).





