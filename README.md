# K8s Alist Longhorn

[K8s](https://kubernetes.io/) + [Alist](https://alist.nn.ci/) + [Longhorn](https://longhorn.io/)

## Pre-requisites

- Install Longhorn, Ref: [K8s PostgreSQL Longhorn](https://github.com/jerryshell/k8s-postgres-longhorn)
- TLS Ingress, Ref: [K8s Traefik cert-manager DNS01 TLS](https://github.com/jerryshell/k8s-traefik-cert-manager-dns01-tls)

## Git clone

```bash
git clone https://github.com/jerryshell/k8s-alist-longhorn.git
cd k8s-alist-longhorn
```

## PVC + Deployment + Service

```bash
kubectl apply -f k8s/
```

## Ingress

```bash
HOST=pan.jerryshell.eu.org # Replace your domain
cat k8s/ingress/ingress.yaml | envsubst | kubectl apply -f -
```

## TLS Ingress

```bash
HOST=pan.jerryshell.eu.org # Replace your domain
cat k8s/ingress/tls-ingress.yaml | envsubst | kubectl apply -f -
```

## Delete

```bash
kubectl delete --ignore-not-found=true -f k8s/ -f k8s/ingress/
```

## Note

<details>

<summary>Change alist-pvc Access Mode to ReadWriteMany</summary>

```bash
kubectl get pvc alist-pvc -o jsonpath='{.spec.volumeName}'
# pvc-UUID
kubectl patch pv pvc-UUID -p '{"spec":{"persistentVolumeReclaimPolicy":"Retain"}}'
kubectl scale --replicas=0 deployment alist
kubectl delete pvc alist-pvc
kubectl patch pv pvc-UUID -p '{"spec":{"claimRef":{"uid":""}}}'
kubectl patch pv pvc-UUID -p '{"spec":{"accessModes":["ReadWriteMany"]}}'
# Change alist-pvc.yaml accessModes to ReadWriteMany
vim alist-pvc.yaml
kubectl apply -f alist-pvc.yaml
kubectl patch pv pvc-UUID -p '{"spec":{"persistentVolumeReclaimPolicy":"Delete"}}'
kubectl scale --replicas=1 deployment alist
```

</details>

## LICENSE

[GNU Affero General Public License v3.0](https://choosealicense.com/licenses/agpl-3.0/)
