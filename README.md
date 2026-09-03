

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













Component / Function	Destination FQDN / URL	Port	Direction	Required / Optional
Control Plane (EU Data Residency)	apigee.eu.rep.googleapis.com	443	Outbound	Required
Analytics Ingestion (France - Paris)	europe-west9-pubsub.googleapis.com	443	Outbound	Required
Global Control Plane Fallback / Management	apigee.googleapis.com	443	Outbound	Required
Apigee Connect / MART Server	apigeeconnect.googleapis.com	443	Outbound	Required
Management API Operation (MART)	www.googleapis.com	443	Outbound	Required
OAuth 2.0 / Token Authentication	oauth2.googleapis.com	443	Outbound	Required
IAM Service Account Credentials	iamcredentials.googleapis.com	443	Outbound	Required
Security Token Service (STS)	sts.googleapis.com	443	Outbound	Required
Service Usage & Quotas	serviceusage.googleapis.com	443	Outbound	Required
Debug / Trace Subscriptions	pubsub.googleapis.com	443	Outbound	Required
Proxy Bundle Download (GCS Storage)	storage.googleapis.com	443	Outbound	Required
Artifact Registry (Container Images)	pkg.dev, *.pkg.dev	443	Outbound	Required
Google Container Registry (Container Images)	gcr.io, *.gcr.io	443	Outbound	Required


Component / Function	Destination FQDN / URL	Port	Direction	Required / Optional
Control Plane (EU Data Residency)	apigee.eu.rep.googleapis.com	443	Outbound	Required
Analytics Ingestion (France - Paris)	europe-west9-pubsub.googleapis.com	443	Outbound	Required
Global Control Plane Fallback / Management	apigee.googleapis.com	443	Outbound	Required
Apigee Connect / MART Server	apigeeconnect.googleapis.com	443	Outbound	Required
Management API Operation (MART)	www.googleapis.com	443	Outbound	Required
OAuth 2.0 / Token Authentication	oauth2.googleapis.com	443	Outbound	Required
IAM Service Account Credentials	iamcredentials.googleapis.com	443	Outbound	Required
Security Token Service (STS)	sts.googleapis.com	443	Outbound	Required
Service Usage & Quotas	serviceusage.googleapis.com	443	Outbound	Required
Debug / Trace Subscriptions	pubsub.googleapis.com	443	Outbound	Required
Proxy Bundle Download (GCS Storage)	storage.googleapis.com	443	Outbound	Required
Artifact Registry (Container Images)	pkg.dev, *.pkg.dev	443	Outbound	Required
Google Container Registry (Container Images)	gcr.io, *.gcr.io	443	Outbound	Required
Cert-Manager Registry	quay.io	443	Outbound	Required
Cloud Logging Agent (apigee-telemetry)	logging.googleapis.com	443	Outbound	Optional (if Logging enabled)
Cloud Monitoring Agent (apigee-telemetry)	monitoring.googleapis.com	443	Outbound	Optional (if Metrics enabled)
Cert-Manager Registry	quay.io	443	Outbound	Required
Cloud Logging Agent (apigee-telemetry)	logging.googleapis.com	443	Outbound	Optional (if Logging enabled)
Cloud Monitoring Agent (apigee-telemetry)	monitoring.googleapis.com	443	Outbound	Optional (if Metrics enabled)
