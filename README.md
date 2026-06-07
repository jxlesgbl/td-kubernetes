# creation du cluster k3d

k3d cluster create td-k8s \
  --servers 1 --agents 2 \
  --port "8080:80@loadbalancer" \
  --k3s-arg "--disable=traefik@server:0"

# creation du secret

kubectl create secret generic backend-secret --from-literal=admin-token="s3cr3t-token-td"

# application des manifests YAML

kubectl apply -f manifests/02-configmap.yaml
kubectl apply -f manifests/03-deployment-back.yaml
kubectl apply -f manifests/04-deployment-front.yaml
kubectl apply -f manifests/05-service-back.yaml
kubectl apply -f manifests/05-service-front.yaml

# verifications

## etape 1
kubectl get secrets

## etape 2
kubectl get configmaps

## etape 3
kubectl rollout status deployment/backend
kubectl get pods -l app=backend
kubectl logs backend-7649946489-v4mq8

## etape4

kubectl apply -f manifests/04-deployment-front.yaml
kubectl get pods -l app=frontend

## etape 5 apply all

d'abord les services puis ensuite deployments, sinon:

kubectl apply -f manifests/.

# etat global des ressources
kubectl get all,configmaps,secrets

# etape 6 rolling update
## split terminal
## terminal 1 : surveiller les pods en temps réel
kubectl get pods -l app=frontend -w

## terminal 2 : modifier le manifest (changer la version d'image), puis :
kubectl apply -f manifests/04-deployment-front.yaml

## qsuivre le rollout
kubectl rollout status deployment/frontend

# etape 7 tests sur le secret

# sans token → 401
curl -X POST http://localhost:8080/api/admin/clear -v

# avec le token → 200
curl -X POST http://localhost:8080/api/admin/clear -H "X-Admin-Token: s3cr3t-token-td" -v

## réponse à Q1!

Quand je supprime un pod front-end, le pod disparait, mais le Deployment agit en contrôleur et détecte qu'il n'y a plus que 1 replica, le ReplicaSet crée donc un nouveau pod pour retablir le nombre desiré de replicas. on attend 2-3 secondes puis on vérifie avec kubectl get pods pour constater que les pods frontends sont revenus au nombre de 2. Ce comportement est appelé self/auto-healing.