

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





## 2. Première Action de Passation : Structure Preprod / Prod

Dans le cadre du transfert vers Run, la première action consiste en la création de deux folders GCP distincts :

* **Folder `preprod`**
* **Folder `prod`**

Chaque folder doit intégrer les **groupes d'utilisateurs (user groups)** correspondants. Pour le moment, le périmètre est limité aux **utilisateurs de la Pologne** uniquement (extension à d'autres pays à prévoir ultérieurement, hors périmètre actuel).

| Folder | Périmètre utilisateurs actuel | Statut |
| :--- | :--- | :---: |
| `preprod` | Users Pologne | À créer |
| `prod` | Users Pologne | À créer |




