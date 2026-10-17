# Déploiement GitOps ArgoCD

Ce dossier contient les manifests déclaratifs permettant d'installer et de configurer **ArgoCD** de manière 100 % reproductible pour la plateforme Product Intelligence.

Comme le dépôt est public, **aucun secret ni token d'authentification Git n'est nécessaire** : ArgoCD clone et synchronise directement les manifests publiquement.

---

## Structure du dossier

```text
argocd/
├── install/
│   ├── namespace.yaml           # Namespace argocd
│   └── kustomization.yaml       # Installation officielle d'ArgoCD (v2.10.4)
├── application.yaml             # Application ArgoCD (observe base/ et auto-synchronise)
└── README.md                    # Guide pas-à-pas
```

---

## Procédure de déploiement (2 étapes simples)

### 1. Installer ArgoCD dans le cluster
```bash
kubectl apply -k argocd/install/
```
*(Optionnel) Attendre que tous les pods soient prêts :*
```bash
kubectl wait --for=condition=available deployment -l app.kubernetes.io/part-of=argocd -n argocd --timeout=300s
```

---

### 2. Déployer les applications GitOps

* **Platform Product Intelligence (observe `base/`) :**
  ```bash
  kubectl apply -f argocd/application.yaml
  ```

* **Kubeflow Pipelines (observe `kubeflow/` avec Sync Waves ordonnées) :**
  ```bash
  kubectl apply -f argocd/kubeflow-app.yaml
  ```
  *L'application orchestre automatiquement le déploiement en vagues séquentielles :*
  1. **Wave -2** : Namespace & CRDs (`01-crds`)
  2. **Wave -1** : PersistentVolume & PersistentVolumeClaim (`02-storage`)
  3. **Wave 0** : Composants core Kubeflow Pipelines (`03-complete`)

* **AWS Load Balancer Controller (observe `aws-load-balancer-controller/`) :**
  ```bash
  kubectl apply -f argocd/aws-lbc-app.yaml
  ```


---

## Accéder à l'interface web ArgoCD

1. **Récupérer le mot de passe initial du compte `admin` :**
   ```bash
   kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
   ```

2. **Ouvrir le tunnel vers l'interface :**
   ```bash
   kubectl port-forward svc/argocd-server -n argocd 8080:443
   ```

3. Ouvrir votre navigateur sur **`https://localhost:8080`** :
   * **Identifiant** : `admin`
   * **Mot de passe** : *(la valeur obtenue à l'étape 1)*
