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
