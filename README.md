

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



# =============================================
# Telemetry — values-common.yaml
# Un seul fichier, un deploy par cluster
# =============================================

gcp:
  region: europe-west9-a
  projectID: "<TON-GCP-PROJECT-ID>"
  workloadIdentity:
    enabled: false

k8sCluster:
  name: "<NOM-CLUSTER-ENREGISTRE>"
  region: europe-west9-a

org: "<TON-ORG-APIGEE>"
instanceID: "<NOM-CLUSTER-ENREGISTRE>"
validateOrg: false
contractProvider: https://apigee.googleapis.com

ao:
  certManagerCAIssuerEnabled: true

# --- Placement nodes ---
nodeSelector:
  requiredForScheduling: true
  apigeeRuntime:
    key: "workload"
    value: "apigee-runtime"
  apigeeData:
    key: "workload"
    value: "apigee-data"

# --- Images ---
imagePullSecrets:
  - name: "image-pull-secret"

# --- Proxy BNP ---


# --- Métriques / Logger — disabled ---
metrics:
  enabled: false
  serviceAccountPath: "<ton-sa>.json"

logger:
  enabled: false
  serviceAccountPath: "<ton-sa>.json"

customAutoscaling:
  enabled: true

# --- Guardrails ---
guardrails:
  image:
    url: "private.fr2.icr.io/ap88257-hprd/apigee-release/hybrid/apigee-watcher"
    tag: "1.17.0"
    pullPolicy: Always
  resources:
    requests:
      cpu: 500m
      memory: 64Mi
  envVars:
    SSL_CERT_FILE: "/etc/ssl/certs/ca-certificates.crt"
    no_proxy: "198.18.0.1,kubernetes.default.svc,.svc,.cluster.local,localhost,127.0.0.1"
    NO_PROXY: "198.18.0.1,kubernetes.default.svc,.svc,.cluster.local,localhost,127.0.0.1"

# --- CA bundle  ---
customCA:

