

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

kubectl run test-egress -n apigee --rm -i --tty --image=fr2.icr.io/ap88257-hprd/apigee-release/hybrid/apigee-runtime:latest --overrides='{"spec":{"securityContext":{"runAsNonRoot":true,"runAsUser":1000,"runAsGroup":1000,"seccompProfile":{"type":"RuntimeDefault"}},"containers":[{"name":"test-egress","image":"fr2.icr.io/ap88257-hprd/apigee-release/hybrid/apigee-runtime:latest","command":["/bin/sh","-c","nc -zv apigee.eu.rep.googleapis.com 443 || curl -vI https://apigee.eu.rep.googleapis.com"],"securityContext":{"allowPrivilegeEscalation":false,"capabilities":{"drop":["ALL"]}}}]}}'

kubectl run test-egress -n apigee --rm -i --tty --image=$IMAGE --overrides='{"spec":{"securityContext":{"runAsNonRoot":true,"runAsUser":1000,"runAsGroup":1000,"seccompProfile":{"type":"RuntimeDefault"}},"containers":[{"name":"test-egress","image":"'$IMAGE'","command":["/bin/sh","-c","nc -zv apigee.eu.rep.googleapis.com 443"],"securityContext":{"allowPrivilegeEscalation":false,"capabilities":{"drop":["ALL"]}}}]}}'


IMAGE=$(kubectl get pods -n apigee -o jsonpath='{.items[0].spec.containers[0].image}' 2>/dev/null || echo "fr2.icr.io/ap88257-hprd/apigee-release/hybrid/apigee-runtime:latest")
