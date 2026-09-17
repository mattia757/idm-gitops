# idm-gitops

Configurazione GitOps locale per il PoC IDM su Rancher Desktop Kubernetes.

## Struttura

```text
apps/
  idmccnobackend/
    base/
    overlays/local/
  keycloak/
    base/
    overlays/local/
  keycloak-db/
    base/
    overlays/local/
argocd/
```

Il setup container di riferimento usa:

- `idmccnobackend:local`
- Keycloak `quay.io/keycloak/keycloak:26.6.4`
- MySQL `mysql:8.4.5`
- backend su porta `8080`
- Keycloak su porta container `8080`
- realm operativo rilevato da Compose: `IdmCCNO`
- property backend rilevate: `KEYCLOAK_BASE_URL`, `KEYCLOAK_REALM`, `keycloak.base-url`, `keycloak.realm`

## Secret richiesti

I valori reali devono essere creati solo nel cluster locale e non vanno committati.

```powershell
kubectl create namespace idm-local

kubectl create secret generic idm-local-db `
  -n idm-local `
  --from-literal=username=<db-user> `
  --from-literal=password=<db-password> `
  --from-literal=root-password=<db-root-password>

kubectl create secret generic keycloak-admin `
  -n idm-local `
  --from-literal=username=<keycloak-admin-user> `
  --from-literal=password=<keycloak-admin-password>
```

Il file realm JSON presente nel progetto sorgente contiene password e client secret in chiaro,
quindi non e' versionato qui. Per il PoC va importato/configurato localmente nel cluster con
valori locali appropriati.

## Validazione Kustomize

```powershell
kubectl kustomize apps/keycloak-db/overlays/local
kubectl kustomize apps/keycloak/overlays/local
kubectl kustomize apps/idmccnobackend/overlays/local
```

## Deploy manuale locale

Da usare prima di Argo CD:

```powershell
kubectl apply -k apps/keycloak-db/overlays/local
kubectl apply -k apps/keycloak/overlays/local
kubectl apply -k apps/idmccnobackend/overlays/local
```

## Argo CD

Le Application in `argocd/` puntano a:

- repository: `https://github.com/mattia757/idm-gitops`
- branch: `main`
- namespace destinazione: `idm-local`

Non applicare le Application finche' questa repository non e' stata pubblicata manualmente su GitHub.
Se la repository GitHub e' privata, configurare le credenziali Git in Argo CD tramite Secret o UI,
senza inserirle nei manifest.
