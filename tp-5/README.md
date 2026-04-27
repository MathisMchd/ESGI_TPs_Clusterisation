

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



