# Déploiement GitOps ArgoCD (Pattern "App of Apps")

Ce dossier contient les manifests déclaratifs permettant d'installer et d'orchestrer **ArgoCD** de manière 100 % reproductible pour l'ensemble du cluster via le pattern **App of Apps**.

Comme le dépôt est public, **aucun secret ni token d'authentification Git n'est nécessaire** : ArgoCD clone et synchronise directement les manifests publiquement.

---

## Structure du dossier

```text
argocd/
├── install/
│   ├── namespace.yaml             # Namespace argocd
│   └── kustomization.yaml         # Installation officielle d'ArgoCD (v2.10.4)
├── root-app.yaml                  # Application racine (App of Apps)
├── apps/                          # Applications enfants observées par root-app
│   ├── kustomization.yaml         # Agrégateur Kustomize des applications
│   ├── aws-lbc-app.yaml           # Wave 1 : AWS Load Balancer Controller (kube-system)
│   ├── kubeflow-app.yaml          # Wave 2 : Kubeflow Pipelines (kubeflow)
│   └── platform-app.yaml          # Wave 3 : Platform Product Intelligence (product-intelligence)
└── README.md
```

---

## Procédure de déploiement (2 étapes uniques)

### 1. Installer ArgoCD dans le cluster
```bash
kubectl apply -k argocd/install/
```
*(Optionnel) Attendre que tous les pods soient prêts :*
```bash
kubectl wait --for=condition=available deployment -l app.kubernetes.io/part-of=argocd -n argocd --timeout=300s
```

---

### 2. Déployer l'Application Racine (App of Apps)
Une seule commande suffit pour orchestrer l'intégralité du cluster :
```bash
kubectl apply -f argocd/root-app.yaml
```

**Séquencement automatique via Sync Waves :**
1. **Wave 1 (`aws-lbc-app`)** : Installe le contrôleur AWS Load Balancer et ses Webhooks/IngressClasses dans `kube-system`.
2. **Wave 2 (`kubeflow-app`)** : Installe la stack Kubeflow Pipelines dans le namespace `kubeflow` (avec ses sous-vagues internes : CRDs -> Storage -> Core).
3. **Wave 3 (`platform-app`)** : Installe les microservices Product Intelligence (`base/`) dans `product-intelligence`.

> [!NOTE]
> L'application racine `root-app.yaml` n'a **volontairement pas de finalizer** de suppression en cascade, ce qui protège votre cluster contre toute destruction accidentelle de vos applications si la root application venait à être supprimée.

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
