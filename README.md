

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

Mise en place de la couche d'exposition des services pour le cluster Hybrid, reposant aujourd'hui sur F5 (VIP) + Ingress Traefik, avec une cible moyen terme vers Gateway API (API Kubernetes standardisée, en remplacement progressif du modèle Ingress).

Architecture actuelle (cible court terme)
Client HTTPS → F5 LoadBalancer (VIP) → Traefik (Ingress Controller) → Service → Pod

1. Exposition externe – VIP F5

loadBalancerClass: datalab/f5 obligatoire sur le Service de type LoadBalancer.
Annotations obligatoires : bnp/entity, bnp/tier-app, bnp/vipclass: f5, + bnp/tenant ou bnp/smarttenant (DATALAB/DATAHUB).
Option bnp/proxy-protocol: enableProxyProtocolInitiatorv2 pour préserver l'IP client (nécessite support côté backend).

2. Routage interne – Ingress Traefik

Traefik assure la terminaison TLS externe et le routage vers les Services.
Deux modes de bout en bout :
HTTPS → HTTP backend : interdit en production.
HTTPS → HTTPS backend (certificat pod via cert-manager + ServersTransport) : obligatoire en production.

3. Résolution DNS – External DNS

Génération automatique des entrées DNS pour Ingress, Gateway API et Services LoadBalancer annotés.
Zone unique supportée :  (la zone  n'est plus disponible, process manuel non compatible avec l'automatisation).
Convention :  (≤ 63 caractères).
Trajectoire cible – Gateway API (moyen terme)
Objectif : remplacer à terme le modèle Ingress Traefik par Gateway API, déjà pris en charge par External DNS pour la génération automatique des entrées DNS.
Le socle F5 (VIP, annotations obligatoires) reste inchangé : seule la couche de routage/exposition applicative (aujourd'hui Traefik/Ingress) migre vers les ressources Gateway API (Gateway, HTTPRoute).
Point à cadrer : compatibilité du modèle de certificats (cert-manager) et du proxy-protocol avec les ressources Gateway API avant bascul





