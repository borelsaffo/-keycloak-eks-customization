# -keycloak-eks-customization
# 🚀 Customisation de Keycloak dans EKS (Amazon EKS)

## 🧩 Objectifs

- Déployer des **extensions `.jar` personnalisées** (SPI, mappers, authentificateurs, etc.) dans Keycloak sur EKS.
- Modifier la **page d'accueil** de Keycloak (branding).
- Utiliser une **image Docker custom** pour un déploiement propre et maintenable.

---

## 🧱 Prérequis

- Keycloak **v17+ (Quarkus)** déployé dans EKS (via Helm ou manifest).
- Accès Docker (Docker Hub ou ECR).
- Repertoire avec :
  - vos fichiers `.jar` (`/providers`)
  - vos fichiers de thème personnalisé (`/themes/<nom-du-theme>`)

---

## 📦 Structure du projet

```bash
keycloak-custom/
├── Dockerfile
├── providers/
│   └── my-extension.jar
└── themes/
    └── my-theme/
        ├── login/
        │   ├── theme.properties
        │   └── login.ftl
        └── resources/

# Dockerfile pour Keycloak custom

FROM quay.io/keycloak/keycloak:24.0.3

# Ajout des extensions .jar
COPY providers /opt/keycloak/providers

# Ajout du thème personnalisé
COPY themes /opt/keycloak/themes

# Build pour activer extensions et thème
RUN /opt/keycloak/bin/kc.sh build

'''
#  Construction et publication de l'image
# Build de l'image
docker build -t <your-repo>/keycloak-custom:latest .

# Push vers Docker Hub ou ECR
docker push <your-repo>/keycloak-custom:latest
'''

# 🚀 Déploiement sur EKS
A. Via manifest Deployment.yaml
yaml
Copier
Modifier
spec:
  containers:
    - name: keycloak
      image: <your-repo>/keycloak-custom:latest
bash
Copier
Modifier
kubectl apply -f keycloak-deployment.yaml

# g
B. Via Helm
Dans values.yaml (Bitnami ou chart officiel) :

yaml
Copier
Modifier
image:
  repository: <your-repo>/keycloak-custom
  tag: latest
Puis :

bash
Copier
Modifier
helm upgrade keycloak oci://registry-1.docker.io/bitnamicharts/keycloak -f values.yaml

## Personnalisation de la page d’accueil

Crée un dossier themes/my-theme/

Place tes fichiers dans :

login/theme.properties

login/login.ftl

resources/ (CSS/images)

Exemple de theme.properties :

properties
Copier
Modifier
parent=keycloak
import=common/keycloak
Active le thème dans l’interface admin :

Realm Settings > Themes > Login Theme > my-theme

Ou via CLI/API :

bash
Copier
Modifier
kc.sh set-theme --login-theme=my-theme
🔍 Vérifications
Voir les logs :
bash
Copier
Modifier
kubectl logs -n keycloak <pod-name>
Tu dois voir :

lua
Copier
Modifier
Provider 'my-extension' loaded
Theme 'my-theme' loaded
Tester le thème :
Accéder à l'URL de login : https://keycloak.example.com/auth/realms/myrealm/account

Vérifier que l’interface a bien changé.

✅ Résumé
Étape	Action
Ajouter des .jar	Mettre dans /providers + kc.sh build
Customiser la page de login	Mettre dans /themes/my-theme/ + activer le thème
EKS	Utiliser une image Docker custom déployée via Helm ou k8s
