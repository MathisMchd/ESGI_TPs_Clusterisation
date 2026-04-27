

Il faut penser a appliquer les modifications de fichiers `kubectl apply -f .\k8s\...`

# Etape 1

## (Démarrer minkube)
```bash
minikube start
```

## Appliquer les labels

Dans `tp-5\k8s\base\namespaces` se trouvent les labels

```bash
kubectl apply -f k8s/base/namespaces/
```

## Voir les labels

```bash
kubectl get ns --show-labels
```


# Etape 2 : RBAC, définir qui fait quoi, sur quels environnements, avec quels droits

## Tableau RBAC 

| Rôle           | dev | staging | prod | actions principales  |
| -------------- | --- | ------- | ---- | -------------------- |
| platform-admin | ✔   | ✔       | ✔    | tout (cluster admin) |
| dev-user       | ✔   | ✔       | ❌    | deploy, debug        |
| qa-user        | ❌   | ✔       | ❌    | read, logs, restart  |
| prod-deployer  | ❌   | ❌       | ✔    | deploy production    |


# Etape 3 - Simuler des utilisateurs (contexts kubectl)

```bash
kubectl config set-context dev-user@app-cluster --cluster=app-cluster --user=dev-user
kubectl config set-context qa-user@app-cluster  --cluster=app-cluster --user=qa-user
```

Pour tester :
```bash 
kubectl --context dev-user@app-cluster auth can-i get pods -n app-dev
```

## Définition des rôles 

Dans `tp-5\k8s\base\rbac` sont définis les rôles.
Le dev et staging sont tous deux liés au `developer-role`.

```bash
kubectl --context dev-user@app-cluster auth can-i create deployment -n app-dev
kubectl --context dev-user@app-cluster auth can-i get pods/log -n app-staging
```


# Étape 4 — RBAC pour dev/staging

```bash
kubectl --context dev-user@app-cluster auth can-i create deployment -n app-dev
kubectl --context dev-user@app-cluster auth can-i get pods/log -n app-staging
```

# Étape 5 — RBAC “prod verrouillée” + identité de déploiement

```bash
- `kubectl --context dev-user@app-cluster auth can-i get pods -n app-prod` => `no`
- `kubectl --context prod-deployer@app-cluster auth can-i patch deployment -n app-prod` => `yes`
```


# Étape 6 — Déployer l’app dans les 3 environnements

Tester l'app lancé 
`kubectl port-forward svc/demo-app-service 7070:80 -n app-dev` puis `localhost:7070`.

Les `overlays\***\kustomization.yaml` appliquent les patchs à la config de base sous `base\app\configmap.yaml`.


# Étape 7 — Garde-fous ressources (ResourceQuota + LimitRange)

Tester la config, que les ressources sont bien appliquées à chaque environnements.
```bash

kubectl get resourcequota -n app-dev
kubectl describe resourcequota dev-quota -n app-dev

kubectl get limitrange -n app-dev
kubectl describe limitrange dev-limits -n app-dev
```

# Étape 8 — Sécurité des pods (Pod Security Standards = PSS)

Appliquer les namespaces 

```bash
kubectl label namespace app-dev \
  pod-security.kubernetes.io/enforce=baseline \
  pod-security.kubernetes.io/audit=baseline \
  pod-security.kubernetes.io/warn=baseline
```

Crash volontaire avec les restrictions 
```bash
kubectl get pods -n app-prod
NAME                        READY   STATUS             RESTARTS     AGE
demo-app-574656884c-549gl   1/1     Running            0            62m
demo-app-574656884c-whvwq   1/1     Running            0            62m
demo-app-796d7fc7d6-m6lgw   0/1     CrashLoopBackOff   1 (9s ago)   12s
```


# "Mémo bonnes pratiques" 


```bash
tp-5/
├── README.md
└── k8s/
    ├── base/
    │   ├── kustomization.yaml
    │   ├── app/
    │   │   ├── configmap.yaml
    │   │   ├── deployment.yaml
    │   │   └── service.yaml
    │   ├── namespaces/
    │   │   ├── app-dev.yaml
    │   │   ├── app-prod.yaml
    │   │   └── app-staging.yaml
    │   └── rbac/
    │       ├── developer-role.yaml
    │       ├── prod-deployer-role.yaml
    │       ├── rolebinding-dev.yaml
    │       ├── rolebinding-prod.yaml
    │       └── rolebinding-staging.yaml
    └── overlays/
        ├── dev/
        │   ├── kustomization.yaml
        │   ├── limitrange.yaml
        │   └── quota.yaml
        ├── prod/
        │   ├── kustomization.yaml
        │   ├── limitrange.yaml
        │   ├── networkpolicy-deny-all.yaml
        │   ├── networkpolicy-egress.yaml
        │   ├── networkpolicy-ingress.yaml
        │   └── quota.yaml
        └── staging/
            ├── kustomization.yaml
            ├── limitrange.yaml
            └── quota.yaml
```
## 1. Séparation des environnements

- Utilisation de **namespaces** :
  - `app-dev`
  - `app-staging`
  - `app-prod`
- Isolation logique des workloads
- Différences gérées via **Kustomize (overlays)**

---

## 2. RBAC (contrôle d’accès)

- Principe du **moindre privilège**
- Devs :
  - accès lecture + debug en dev/staging
  - aucun accès prod
- CI/CD :
  - service account dédié pour déploiement prod
- Interdiction d’accès direct aux ressources critiques (ex : secrets en prod)

---

## 3. Gestion des secrets

- Ne jamais stocker de secrets en clair dans Git
- Utilisation de :
  - Kubernetes Secrets (baseline)
  - ou solution externe (Vault, SealedSecrets)
- Accès limité via RBAC

---

## 4. ResourceQuota & LimitRange

- Limitation des ressources par namespace :
  - CPU / RAM total
  - nombre de pods max
- Définition de requests/limits par défaut
- Prévention du “noisy neighbor”

---

## 5. Pod Security Standards (PSS)

- `dev` / `staging` : baseline
- `prod` : restricted
- Contraintes :
  - `runAsNonRoot: true`
  - `allowPrivilegeEscalation: false`
  - `capabilities: drop ALL`
  - `seccompProfile: RuntimeDefault`

---

## 6. NetworkPolicies

- Politique par défaut : **deny all**
- Autorisations explicites uniquement :
  - ingress (ingress controller)
  - egress (DNS, services nécessaires)
- Réduction de la surface réseau

---

## 7. Stratégie de release & rollback

- RollingUpdate par défaut
- Déploiement progressif via staging
- Rollback rapide :
  - `kubectl rollout undo deployment`
- Validation avant passage en prod

---

## 8. Gestion des images

- Utilisation d’un registry contrôlé (ex : private registry)
- Scan de vulnérabilités (Trivy, etc.)
- Versionnage explicite (pas de `latest`)
- Optionnel : signature d’images (cosign)

---

## 9. Observabilité (bonus recommandé)

- Logs centralisés
- Readiness / Liveness probes
- Monitoring (Prometheus / Grafana si dispo)

---

## 10. Bonnes pratiques générales

- Infrastructure as Code (YAML versionné)
- Kustomize ou Helm pour la modularité
- Séparation stricte dev/staging/prod
- Principe du moindre privilège partout