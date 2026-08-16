# Infrastructure - ArgoCD

![Bannière Loutik](https://raw.githubusercontent.com/loutik/design-assets/main/banniere_loutik.png)

## Contexte

Ce dépôt contient la configuration Kubernetes déployée sur le cluster via ArgoCD. L’infrastructure de base est toutefois gérée par un rôle Ansible nommé k3s-argocd, hébergé dans le dépôt suivant : https://github.com/loutik/infrastructure-ansible. Cette approche permet de séparer la gestion de l’infrastructure du déploiement des applications, tout en conservant un fonctionnement GitOps cohérent.

-----

## Structure du dépôt

L’organisation du dépôt suit la logique suivante :

```text
.
├── apps/
│   └── librespeed/
├── bootstrap/
│   ├── app-of-apps.yml
│   └── apps/
├── infra/
└── readme.md
```

- **apps/** : contient les manifests Kubernetes de l’application LibreSpeed.
- **bootstrap/app-of-apps.yml** : Application ArgoCD principale qui pointe vers le dossier bootstrap/apps.
- **bootstrap/apps/** : contient les Applications ArgoCD enfant, comme LibreSpeed.
- **infra/** : espace réservé pour les composants d’infrastructure partagés ou futurs.
- **readme.md** : documentation du dépôt.

-----

## Déploiement avec ArgoCD

### 1. Cloner le dépôt localement

```bash
git clone https://github.com/loutik/infrastructure-argocd.git
cd infrastructure-argocd
```

### 2. Appliquer l’App of Apps sur le cluster

Le fichier bootstrap/app-of-apps.yml permet d’enregistrer une Application ArgoCD maîtresse sur le cluster. En pratique, ce déploiement est normalement pris en charge automatiquement par le rôle Ansible k3s-argocd, mais la commande manuelle ci-dessous reste disponible si besoin de l’appliquer directement.

```bash
kubectl apply -f bootstrap/app-of-apps.yml
```

Une fois ce fichier appliqué sur le cluster, toutes les configurations définies dans le dépôt seront automatiquement déployées par ArgoCD, selon la logique GitOps.

### 3. Vérifier la synchronisation

```bash
kubectl get applications -n argocd
kubectl get pods -n toolbox
```

-----

## Test local avec k3d

Pour valider rapidement le bon fonctionnement des manifests sans impacter le cluster de production, il est possible de lancer un cluster Kubernetes local avec k3d.

### 1. Installer k3d

```bash
wget -q -O - https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash
```

### 2. Créer un cluster jetable pour le développement

```bash
k3d cluster create dev-cluster --servers 1 -p "80:80@loadbalancer" -p "443:443@loadbalancer"
```

Ensuite, vérifiez que le cluster est bien disponible :

```bash
kubectl get nodes
```

### 3. Test statique des manifests

Avant d’appliquer un manifest sur le cluster local, il est recommandé de valider sa syntaxe et sa structure avec un test sans execution réelle.

```bash
kubectl apply -k apps/homepage/ -n toolbox --dry-run=client
```

Cette commande vérifie que le dossier Kustomize est bien lisible et que les ressources générées sont conformes, sans créer d’objets dans le cluster.

### 4. Test réel sur le cluster local

```bash
kubectl apply -k apps/homepage/ -n toolbox
```

Puis vérifiez le déploiement :

```bash
kubectl get pods -n toolbox
kubectl get ingress -A
kubectl get svc -n toolbox
```

### 5. Nettoyage

Une fois les tests terminés, vous pouvez supprimer le cluster local pour repartir d’une base propre :

```bash
k3d cluster delete dev-cluster
```

> Cette méthode est idéale pour valider un manifeste avant un déploiement GitOps ou pour reproduire rapidement un environnement de développement local.

-----

## Bonnes pratiques et sécurité

1. **Versionner toutes les modifications** : chaque évolution doit passer par une revue de code avant déploiement.
2. **Limiter les permissions** : les ressources Kubernetes doivent être définies avec un périmètre minimal et un namespace explicite.
3. **Surveiller les déploiements** : vérifier l’état des Applications ArgoCD et des pods après chaque synchronisation.

```bash
kubectl get application librespeed -n argocd
```

-----

## 👨‍💻 Mainteneurs

- **Louis MEDO** | [LinkedIn](https://www.linkedin.com/in/louismedo/) | [Portfolio](https://louis.loutik.fr/) | [GitHub](https://github.com/FireToak) | [louis.medo@loutik.fr](mailto:louis.medo@loutik.fr)

-----

<div align="center">
<br>
<small><i>Dernière mise à jour : 10 août 2026</i></small>
</div>
