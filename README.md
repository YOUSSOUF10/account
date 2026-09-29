

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


Catégorie	Champs (Console API Hub)	Présent MP / Hybrid ?	Couverture CMAT	Source de la donnée	Empreinte sécurité & commentaires	Mitigation / contrôle
Identité de l'API	API Name, API ID, Gateway Type, Description	Oui	Déjà couvert	Auto (plugin Apigee)	Identifiant et descriptif de proxy standard. Même exposition que les proxies Apigee.	—
Données personnelles	Owner name / email	Partiellement	Delta à évaluer	Saisie manuelle	Donnée personnelle (RGPD), même si faiblement sensible.	Préférer des adresses fonctionnelles ou de BAL d'équipe. Informer le DPO.
Arborescence & contrats	Versions, Operations (paths, méthodes HTTP), fichiers de spec (OpenAPI/Swagger)	Partiellement (le MP stocke des révisions et basepaths, pas les specs au niveau opération)	Delta à évaluer	Upload manuel / CI	Structure d'exposition. Risque résiduel : les examples/default peuvent contenir de vrais payloads ou des tokens, et les servers des URLs internes.	Règles Spectral bloquantes (pas d'exemples réels, pas d'URLs internes), revue avant publication.
Routage & runtime	Deployments, Environment (prod-co-mtts), Source revision, Resource URI, Endpoints (FQDN)	Oui	Déjà couvert	Auto (plugin Apigee)	Cartographie d'infra déjà existante, mais centralisée dans un même outil (effet d'agrégation).	IAM restrictif sur la lecture.
Taxonomies & gouvernance	Business unit, Team, Maturity level, Target users, API style, Service type	Partiellement (custom attributes sur les API products seulement)	Delta à évaluer	Saisie manuelle	Nouveau référentiel de métadonnées centralisé. Les champs libres peuvent recevoir n'importe quoi.	Valeurs contrôlées (listes fermées) plutôt que texte libre.
Conformité & contrôle	Compliance, Accreditation, Lifecycle, Lint results (Spectral)	Non	Delta API Hub	Auto (lint) + manuel	Statuts d'accréditation et rapports de conformité OpenAPI.	Ruleset Spectral maîtrisé et versionné.
Documentation & liens	Requirement / Functional / Technical specifications (URLs)	Partiellement	Delta à évaluer	Saisie manuelle	Liens vers la doc interne/externe. Ils révèlent des noms d'outils ou d'espaces internes.	Liens vers des espaces à accès contrôlé uniquement.
Supervision & supply chain	Insights (TPS, latence), Security (scores), Supply chain (dépendances)	Non	Delta API Hub	Auto (analytics / Advanced API Security)	Données dérivées du trafic. Scores de sécurité et graphe de dépendances : cartographie d'attaque exploitable par agrégation.	Accès restreint aux profils sécu/archi.
Fonctions IA	Recherche sémantique (Gemini / embeddings)	Non	Delta API Hub	Auto	Traitement IA du contenu des specs et métadonnées, point sensible en contexte bancaire.	Vérifier si c'est désactivable. Validation sécu/conformité.
Localisation / résidence	Région de provisioning API Hub	Oui (région du control plane Hybrid)	Delta à évaluer	Config provisioning	La région doit être alignée avec vos contraintes EU. Choix fait au provisioning.	Région EU imposée, validée avant provisioning.
Chiffrement	Google-managed vs CMEK	Selon config org	Delta à évaluer	Config provisioning	Choix à l'onboarding, en principe irréversible (à confirmer dans la doc).	CMEK via KMS si exigé par la politique entreprise.
Flux d'ingestion	Plugin Apigee (auto-enregistrement), sens des flux	Oui (control plane)	Delta à évaluer	Auto	Seul le control plane est concerné, pas le runtime ROKS. Support Hybrid à confirmer.	Schéma de flux validé par la sécu réseau.
IAM & accès	Rôles apihub.*, service accounts	Partiellement	Delta API Hub	Config	Vue plus large que les rôles Apigee actuels.	Moindre privilège, groupes dédiés.
Traçabilité	Cloud Audit Logs (Admin Activity / Data Access)	Oui (audit Apigee)	Delta à évaluer	Auto	Preuve d'audit des consultations et modifications.	Activer Data Access logs, export vers le SIEM.
Périmètre réseau	VPC Service Controls	Selon projet GCP	Delta à évaluer	Config	Protection contre l'exfiltration.	À valider selon votre périmètre GCP.
Cycle de vie des données	Rétention, suppression, export, réversibilité	Non	Delta API Hub	—	Question classique de conformité et de sortie.	Procédure documentée de purge et d'export.
Payloads métier & secrets	Corps req/resp, clés API, tokens OAuth, mots de passe backend	Non	Hors périmètre	—	Pas de payload transactionnel ni de secret par conception. Risque résiduel via les specs et les saisies libres.	Lint bloquant sur les specs, sensibilisation des équipes.


