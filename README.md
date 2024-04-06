# K8s Alist Longhorn

[K8s](https://kubernetes.io/) + [Alist](https://alist.nn.ci/) + [Longhorn](https://longhorn.io/)

## Install open-iscsi

```bash
sudo apt install open-iscsi
```

## Install Longhorn

```bash
kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/v1.6.1/deploy/longhorn.yaml
```

## Alist PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: alist-pvc
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: longhorn
  resources:
    requests:
      storage: 10Gi
```

## Alist Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: alist
  labels:
    app: alist

spec:
  replicas: 1
  selector:
    matchLabels:
      app: alist
  template:
    metadata:
      labels:
        app: alist
    spec:
      containers:
        - env:
            - name: PGID
              value: "0"
            - name: PUID
              value: "0"
            - name: UMASK
              value: "022"
          image: xhofe/alist:latest
          name: alist
          ports:
            - name: http
              containerPort: 5244
          volumeMounts:
            - mountPath: /opt/alist/data
              name: alist-pvc
      restartPolicy: Always
      volumes:
        - name: alist-pvc
          persistentVolumeClaim:
            claimName: alist-pvc
```

## Alist Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: alist

spec:
  ports:
    - name: http
      port: 5244
      targetPort: http
  selector:
    app: alist
```

## Alist Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: alist-ingress

spec:
  rules:
    - host: pan.jerryshell.eu.org
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: alist
                port:
                  name: http
```

## Alist TLS Ingress

Ref: [K8s Traefik cert-manager DNS01 TLS](https://github.com/jerryshell/k8s-traefik-cert-manager-dns01-tls)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: alist-tls-ingress
  annotations:
    spec.ingressClassName: traefik
    cert-manager.io/cluster-issuer: letsencrypt-prod
    traefik.ingress.kubernetes.io/router.middlewares: default-redirect-https@kubernetescrd
spec:
  rules:
    - host: pan.jerryshell.eu.org
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: alist
                port:
                  name: http
  tls:
    - secretName: alist-tls
      hosts:
        - pan.jerryshell.eu.org
```

## Change alist-pvc Access Mode to ReadWriteMany

```bash
kubectl get pvc alist-pvc -o jsonpath='{.spec.volumeName}'
# pvc-c070174a-1ae0-40f5-81d0-3b83dd4ad38b
kubectl patch pv pvc-c070174a-1ae0-40f5-81d0-3b83dd4ad38b -p '{"spec":{"persistentVolumeReclaimPolicy":"Retain"}}'
kubectl scale --replicas=0 deployment alist
kubectl delete pvc alist-pvc
kubectl patch pv pvc-c070174a-1ae0-40f5-81d0-3b83dd4ad38b -p '{"spec":{"claimRef":{"uid":""}}}'
kubectl patch pv pvc-c070174a-1ae0-40f5-81d0-3b83dd4ad38b -p '{"spec":{"accessModes":["ReadWriteMany"]}}'
kubectl apply -f alist-pvc.yaml
kubectl patch pv pvc-c070174a-1ae0-40f5-81d0-3b83dd4ad38b -p '{"spec":{"persistentVolumeReclaimPolicy":"Delete"}}'
kubectl scale --replicas=2 deployment alist
```

## LICENSE

[GNU Affero General Public License v3.0](https://choosealicense.com/licenses/agpl-3.0/)
