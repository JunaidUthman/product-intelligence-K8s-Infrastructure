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

### 2. Déployer l'application GitOps
Cette commande enregistre l'application dans ArgoCD. Dès cet instant, ArgoCD observe le dossier `base/` et applique automatiquement toutes les modifications du dépôt public :
```bash
kubectl apply -f argocd/application.yaml
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
