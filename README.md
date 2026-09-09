

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







# Provisioning Apigee Organization – Poland (mgmt-plane-org)

Ce dépôt contient le code Terraform permettant de provisionner l'organisation **Apigee Hybrid** pour la région Pologne.

Ce document constitue le guide de passation entre l'équipe **Build** et l'équipe **Run**. Il détaille le workflow de déploiement, les prérequis techniques, les paramètres irréversibles et les étapes d'exécution.

---

## ⚠️ ALERTES ET POINTS D'ATTENTION CRITIQUES

### 1. Sécurité & Fichiers de Credentials JSON
> 🛑 **ATTENTION : NE JAMAIS COMMITTER DE FICHIERS JSON DE SERVICE ACCOUNT DANS GIT**
>
> Des fichiers de clés JSON (ex: `apigee-plg-dev-*.json`, `apigee-org-hors-prod-*.json`) sont actuellement présents dans l'arborescence locale de `mgmt-plane-org/`.
>
> **Actions requises immédiatement :**
> - Supprimer les clés JSON du suivi Git (`git rm --cached *.json`).
> - Vérifier que le fichier `.gitignore` contient au minimum :
>   ```gitignore
>   *.json
>   *.tfstate
>   *.tfstate.*
>   .terraform/
>   .tfvars
>   ```
> - Ne pas utiliser de chemins absolus locaux (ex: `credentials_path = "/Users/h33926/..."`). Privilégier l'utilisation des variables d'environnement (`GOOGLE_APPLICATION_CREDENTIALS`) ou passer le chemin de manière dynamique via la CI/CD.

### 2. Rattachement au Contrat Groupe GCP (Étape Préalable Obligatoire)
> 🚨 **NOTIFICATION OBLIGATOIRE À L'ÉQUIPE GCP GROUPE**
>
> Avant la création effective de l'organisation Apigee, vous **DEVEZ** communiquer à l'équipe GCP central/Groupe :
> 1. Le **GCP Project ID** (`project_id`)
> 2. Le **Nom de l'Organisation Apigee** (`display_name`)
>
> **Pourquoi ?** L'organisation Apigee doit être obligatoirement rattachée au contrat cadre groupe GCP avant d'être provisionnée pour activer la facturation et l'éligibilité au support.

---

## 🛠️ Statut de Configuration : Validé vs En Attente (KMS)

Pour une visibilité claire de l'état du provisioning :

| Domaine | Paramètres | Statut | Description / Action |
| :--- | :--- | :---: | :--- |
| **Data Residency** | `api_consumer_data_location = "europe-central2"`<br>`analytics_region = "europe-central2"` | ✅ **VALIDÉ** | Données localisées à **Varsovie (Pologne)**. <br>⚠️ **Choix définitif & irréversible** à la création. |
| **Apigee Endpoint** | `apigee_custom_endpoint = "https://apigee.eu.rep.googleapis.com/v1/"` | ✅ **VALIDÉ** | Endpoint Control Plane pour la zone UE. |
| **Chiffrement / KMS** | `control_plane_key_*`<br>`consumer_data_key_*` | ⏳ **EN ATTENTE** | En cours de validation avec l'équipe **ProdSec**. <br>Modèle KMS & KeyRings non définitifs pour la Prod. |

---

## 1. Prérequis Techniques

Avant d'exécuter le provisioning Terraform, les éléments suivants doivent être disponibles et validés :

### 1.1 GCP Project & Billing
- **Project ID** du projet GCP cible (ex: `apigee-plg-dev`).
- Billing activé et projet rattaché au billing account approprié.
- Service Account d'exécution avec les permissions requises :
  - `roles/apigee.admin`
  - `roles/resourcemanager.projectIamAdmin`
  - `roles/serviceusage.serviceUsageAdmin`
  - `roles/cloudkms.admin` (si gestion des clés KMS via Terraform)

### 1.2 Data Residency (Configuration Irréversible)
L'organisation Apigee Poland est configurée avec la Data Residency activée :
```hcl
api_consumer_data_location = "europe-central2"
analytics_region           = "europe-central2"
apigee_custom_endpoint     = "https://apigee.eu.rep.googleapis.com/v1/"
```

* **API Consumer Data :** `europe-central2` – Warsaw, Poland
* **Analytics Data :** `europe-central2` – Warsaw, Poland

> ⚠️ **IMPORTANT :** Le choix de la Data Residency est effectué lors de la création de l'organisation. Cette configuration est **irréversible** après la création. La région doit donc être impérativement validée avant tout `terraform apply`.

### 1.3 Communication Contrat Groupe GCP

Transmettre les éléments suivants à l'équipe GCP Groupe :

* `Project ID` : ID du projet cible
* `Display Name` : Nom affiché de l'organisation
* `Billing Type` : `SUBSCRIPTION`

### 1.4 Encryption / KMS (Chiffrement CMEK)

Le provisioning prévoit l'utilisation de clés de chiffrement KMS pour :

* **Control Plane :** `control_plane_key_location`, `control_plane_keyring_name`, `control_plane_key_name`
* **Consumer Data :** `consumer_data_key_location`, `consumer_data_keyring_name`, `consumer_data_key_name`

Statut actuel : **⏳ EN ATTENTE DE VALIDATION PRODSEC**

La configuration définitive des clés n'est pas considérée comme finale tant que :

* La validation SecOps/ProdSec n'est pas obtenue ;
* La définition des KeyRings et Key Names n'est pas figée ;
* Les droits IAM nécessaires pour leur utilisation par le Service Agent Apigee ne sont pas accordés.

### 1.5 APIs GCP Nécessaires

Les APIs requises sont déclarées dans la variable `apigee_services` et activées automatiquement :

* `apigee.googleapis.com`
* `apigeeconnect.googleapis.com`
* `monitoring.googleapis.com`
* `cloudresourcemanager.googleapis.com`
* `pubsub.googleapis.com`

---

## 2. Structure du Répertoire Terraform (`mgmt-plane-org`)

```text
mgmt-plane-org/
├── environments/
│   ├── dev.tfvars
│   └── hprod.tfvars          # Fichiers de variables par environnement
├── modules/
│   ├── identity/             # Service Accounts et rôles IAM
│   ├── kms/                  # Ressources et configurations KMS
│   ├── organization/         # Création de l'organisation Apigee Hybrid
│   └── services/             # Activation des APIs GCP requises
├── backend.tf                # State Terraform à distance (GCS)
├── main.tf                   # Orchestration des modules
├── outputs.tf                # Déclaration des sorties
├── providers.tf              # Provider Google & Google-Beta
├── variables.tf              # Déclaration des variables globales
└── terraform.tf              # Contraintes de version Terraform et providers
```

**Rôle des principaux modules :**

* `services` : Activation des APIs GCP nécessaires au projet.
* `identity` : Configuration des comptes de service (SA) et des droits IAM associatifs.
* `kms` : Création et/ou raccordement des clés KMS.
* `organization` : Instanciation de la ressource d'organisation Apigee.

---

## 3. Checklist Avant Provisioning

Avant tout `terraform apply`, cocher obligatoirement la liste suivante :

* [ ] **Sécurité Git :** Aucun fichier de clé JSON (`.json`) n'est commité dans le dépôt Git.
* [ ] **GCP Project :** Projet GCP créé et Billing correctement attaché.
* [ ] **Contrat Groupe :** `Project ID` et `Display Name` transmis à l'équipe GCP Groupe pour rattachement au contrat.
* [ ] **Service Account :** Service Account d'exécution Terraform prêt et permissions IAM validées.
* [ ] **Data Residency :** Validation explicite sur `europe-central2` (Warsaw, Poland).
* [ ] **Endpoint Apigee :** `apigee_custom_endpoint` validé sur `https://apigee.eu.rep.googleapis.com/v1/`.
* [ ] **KMS / Security :** Validation ProdSec obtenue pour le schéma KMS et les clés (particulièrement en Production).
* [ ] **Fichier tfvars :** Configuration dans `environments/<environment>.tfvars` revue et validée.

---

## 4. Exécution Terraform (Procédure Run)

Une fois tous les prérequis validés :

### Step 1 : Authentification GCP

```bash
export GOOGLE_APPLICATION_CREDENTIALS="/chemin/securise/vers/service_account.json"
```

### Step 2 : Initialisation

```bash
cd mgmt-plane-org
terraform init
```

### Step 3 : Validation de la syntaxe

```bash
terraform validate
```

### Step 4 : Simulation (Plan)

```bash
terraform plan -var-file="environments/<environment>.tfvars"
```

> 🔍 **Point de contrôle :** Vérifier avec attention dans la sortie du plan les valeurs d'analytics et de données consommateurs (`europe-central2`).

### Step 5 : Déploiement (Apply)

```bash
terraform apply -var-file="environments/<environment>.tfvars"
```

---

## 5. Procédures de Maintenance et Passation

* **Mise à jour des clés KMS :** Si la ProdSec fournit de nouveaux KeyRings/Keys après l'étape initiale, mettre à jour le fichier `environments/<environment>.tfvars` concerné et exécuter `terraform plan` pour vérifier l'impact avant `apply`.
* **Ajout d'APIs GCP :** Pour ajouter des dépendances GCP à l'organisation, enrichir la liste `apigee_services` dans le fichier `.tfvars` correspondant.






