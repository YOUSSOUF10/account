

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



kubectl run test-all-apigee -n apigee --rm -i --tty \
  --image=curlimages/curl \
  --overrides='{
    "spec": {
      "securityContext": {
        "runAsNonRoot": true,
        "runAsUser": 1000,
        "runAsGroup": 1000,
        "seccompProfile": {
          "type": "RuntimeDefault"
        }
      },
      "containers": [{
        "name": "test-all-apigee",
        "image": "curlimages/curl",
        "command": ["sh", "-c"],
        "args": ["for url in apigee.eu.rep.googleapis.com europe-west9-pubsub.googleapis.com apigee.googleapis.com oauth2.googleapis.com iamcredentials.googleapis.com storage.googleapis.com pkg.dev; do echo \"=== Test vers $url:443 ===\"; curl -m 5 -svI https://$url > /dev/null 2>&1 && echo \"SUCCESS: Flux Ouvert\" || echo \"FAILED: Bloque par pare-feu\"; echo \"\"; done"],
        "securityContext": {
          "allowPrivilegeEscalation": false,
          "capabilities": {
            "drop": ["ALL"]
          }
        }
      }]
    }
  }'



  kubectl run test-egress -n apigee --rm -i --tty \
  --image=curlimages/curl \
  --overrides='{
    "spec": {
      "securityContext": {
        "runAsNonRoot": true,
        "runAsUser": 1000,
        "runAsGroup": 1000,
        "fsGroup": 1000,
        "seccompProfile": {
          "type": "RuntimeDefault"
        }
      },
      "containers": [{
        "name": "test-egress",
        "image": "curlimages/curl",
        "args": ["curl", "-vI", "https://apigee.eu.rep.googleapis.com"],
        "securityContext": {
          "allowPrivilegeEscalation": false,
          "capabilities": {
            "drop": ["ALL"]
          },
          "readOnlyRootFilesystem": false
        }
      }]
    }
  }'
