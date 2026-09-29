

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

MATRICE D'ANALYSE D'ÉCART ET DE CONFORMITÉ : APIGEE API HUB vs CMAT / MANAGEMENT PLANE						
<img width="3529" height="45" alt="image" src="https://github.com/user-attachments/assets/4add6e15-8ed3-4161-a990-1445e451b5fe" />


Identité de l'API
Données personnelles
Arborescence & contrats
Routage & runtime
Taxonomies & gouvernance
Conformité & contrôle
Documentation & liens
Supervision & supply chain
Fonctions IA
Localisation / résidence
Chiffrement
Flux d'ingestion
IAM & accès
Traçabilité
Périmètre réseau
Cycle de vie des données
Payloads métier & secrets

Champs (Console API Hub)
API Name, API ID, Gateway Type, Description
Owner name / email
Versions, Operations (paths, méthodes HTTP), fichiers de spec (OpenAPI/Swagger)
Deployments, Environment (prod-co-mtts), Source revision, Resource URI, Endpoints (FQDN)
Business unit, Team, Maturity level, Target users, API style, Service type
Compliance, Accreditation, Lifecycle, Lint results (Spectral)
Requirement / Functional / Technical specifications (URLs)
Insights (TPS, latence), Security (scores), Supply chain (dépendances)
Recherche sémantique (Gemini / embeddings)
Région de provisioning API Hub
Google-managed vs CMEK
Plugin Apigee (auto-enregistrement), sens des flux
Rôles apihub.*, service accounts
Cloud Audit Logs (Admin Activity / Data Access)
VPC Service Controls
Rétention, suppression, export, réversibilité
Corps req/resp, clés API, tokens OAuth, mots de passe backend


Présent MP / Hybrid ?
Oui
Partiellement
Partiellement (le MP stocke des révisions et basepaths, pas les specs au niveau opération)
Oui
Partiellement (custom attributes sur les API products seulement)
Non
Partiellement
Non
Non
Oui (région du control plane Hybrid)
Selon config org
Oui (control plane)
Partiellement
Oui (audit Apigee)
Selon projet GCP
Non
Non



Couverture CMAT
Déjà couvert
Delta à évaluer
Delta à évaluer
Déjà couvert
Delta à évaluer
Delta API Hub
Delta à évaluer
Delta API Hub
Delta API Hub
Delta à évaluer
Delta à évaluer
Delta à évaluer
Delta API Hub
Delta à évaluer
Delta à évaluer
Delta API Hub
Hors périmètre

Source de la donnée
Auto (plugin Apigee)
Saisie manuelle
Upload manuel / CI
Auto (plugin Apigee)
Saisie manuelle
Auto (lint) + manuel
Saisie manuelle
Auto (analytics / Advanced API Security)
Auto
Config provisioning
Config provisioning
Auto
Config
Auto
Config
—
—
Empreinte sécurité & commentaires
Identifiant et descriptif de proxy standard. Même exposition que les proxies Apigee.
Donnée personnelle (RGPD), même si faiblement sensible.
Structure d'exposition. Risque résiduel : les examples/default peuvent contenir de vrais payloads ou des tokens, et les servers des URLs internes.
Cartographie d'infra déjà existante, mais centralisée dans un même outil (effet d'agrégation).
Nouveau référentiel de métadonnées centralisé. Les champs libres peuvent recevoir n'importe quoi.
Statuts d'accréditation et rapports de conformité OpenAPI.
Liens vers la doc interne/externe. Ils révèlent des noms d'outils ou d'espaces internes.
Données dérivées du trafic. Scores de sécurité et graphe de dépendances : cartographie d'attaque exploitable par agrégation.
Traitement IA du contenu des specs et métadonnées, point sensible en contexte bancaire.
La région doit être alignée avec vos contraintes EU. Choix fait au provisioning.
Choix à l'onboarding, en principe irréversible (à confirmer dans la doc).
Seul le control plane est concerné, pas le runtime ROKS. Support Hybrid à confirmer.
Vue plus large que les rôles Apigee actuels.
Preuve d'audit des consultations et modifications.
Protection contre l'exfiltration.
Question classique de conformité et de sortie.
Pas de payload transactionnel ni de secret par conception. Risque résiduel via les specs et les saisies libres.


Mitigation / contrôle
—
Préférer des adresses fonctionnelles ou de BAL d'équipe. Informer le DPO.
Règles Spectral bloquantes (pas d'exemples réels, pas d'URLs internes), revue avant publication.
IAM restrictif sur la lecture.
Valeurs contrôlées (listes fermées) plutôt que texte libre.
Ruleset Spectral maîtrisé et versionné.
Liens vers des espaces à accès contrôlé uniquement.
Accès restreint aux profils sécu/archi.
Vérifier si c'est désactivable. Validation sécu/conformité.
Région EU imposée, validée avant provisioning.
CMEK via KMS si exigé par la politique entreprise.
Schéma de flux validé par la sécu réseau.
Moindre privilège, groupes dédiés.
Activer Data Access logs, export vers le SIEM.
À valider selon votre périmètre GCP.
Procédure documentée de purge et d'export.
Lint bloquant sur les specs, sensibilisation des équipes.


<img width="363" height="1139" alt="image" src="https://github.com/user-attachments/assets/104080f0-6853-4077-a856-e5db7ae3aed5" />


# Commande de démarrage

