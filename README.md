# Sample command to work with chart
- helm repo add shmaks-vshk-certification https://shmaks0.github.io/vshk-certification-helm 
- helm template vshk-local shmaks-vshk-certification/vshk-certification --set image.repository=vshk-local --set image.tag=latest-dev --set image.pullSecret=local-gitlab-registry --set db.host=postgresql --set db.port=5432 --set db.name=postgresql --set db.user=postgres --set db.password=postgres --set ingress.host=vshk-certification.minikube.slurm.io --set pv.enabled=true --set pv.server=10.0.0.1 --set pv.path=/local/local-nfs
- output:
```
---
# Source: vshk-certification/templates/secret.yaml
apiVersion: v1
kind: Secret
type: "Opaque"
metadata:
  name: vshk-local-vshk-certification
  labels:
    helm.sh/chart: vshk-certification-0.0.4
    app.kubernetes.io/name: vshk-certification
    app.kubernetes.io/instance: vshk-local
    app.kubernetes.io/version: "0.0.1"
    app.kubernetes.io/managed-by: Helm
stringData:
  db_user: "postgres"
  db_password: "postgres"
---
# Source: vshk-certification/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: vshk-local-vshk-certification
  labels:
    helm.sh/chart: vshk-certification-0.0.4
    app.kubernetes.io/name: vshk-certification
    app.kubernetes.io/instance: vshk-local
    app.kubernetes.io/version: "0.0.1"
    app.kubernetes.io/managed-by: Helm
data:
  db_host: "postgresql"
  db_port: "5432"
  db_name: "postgresql"
---
# Source: vshk-certification/templates/pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: vshk-local-vshk-certification-pv
spec:
  accessModes:
    - ReadWriteMany
  mountOptions:
    - hard
    - nfsvers=4.0
    - timeo=60
    - retrans=10
  capacity:
    storage: 10Gi
  nfs:
    server: "10.0.0.1"
    path: "/local/local-nfs"
  persistentVolumeReclaimPolicy: "Recycle"
---
# Source: vshk-certification/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: vshk-local-vshk-certification
  labels:
    helm.sh/chart: vshk-certification-0.0.4
    app.kubernetes.io/name: vshk-certification
    app.kubernetes.io/instance: vshk-local
    app.kubernetes.io/version: "0.0.1"
    app.kubernetes.io/managed-by: Helm
spec:
  type: ClusterIP
  ports:
    - port: 80
      targetPort: http
      protocol: TCP
      name: http
  selector:
    app.kubernetes.io/name: vshk-certification
    app.kubernetes.io/instance: vshk-local
---
# Source: vshk-certification/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vshk-local-vshk-certification
  labels:
    helm.sh/chart: vshk-certification-0.0.4
    app.kubernetes.io/name: vshk-certification
    app.kubernetes.io/instance: vshk-local
    app.kubernetes.io/version: "0.0.1"
    app.kubernetes.io/managed-by: Helm
spec:
  progressDeadlineSeconds: 180
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: vshk-certification
      app.kubernetes.io/instance: vshk-local
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  revisionHistoryLimit: 5
  template:
    metadata:
      labels:
        app.kubernetes.io/name: vshk-certification
        app.kubernetes.io/instance: vshk-local
    spec:
      imagePullSecrets:
        - name: local-gitlab-registry
      securityContext:
        {}
      containers:
        - name: vshk-certification
          securityContext:
            {}
          image: "vshk-local:latest-dev"
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
          livenessProbe:
            initialDelaySeconds: 90
            failureThreshold: 3
            httpGet:
              path: /users
              port: http
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 2
          readinessProbe:
            failureThreshold: 30
            httpGet:
              path: /users
              port: http
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 1
          resources:
            limits:
              cpu: 200m
              memory: 256Mi
            requests:
              cpu: 200m
              memory: 256Mi
          env:
            - name: PORT
              value: "80"  

            - name: DB_HOST
              valueFrom:
                configMapKeyRef:
                  name: vshk-local-vshk-certification
                  key: db_host
            - name: DB_PORT
              valueFrom:
                configMapKeyRef:
                  name: vshk-local-vshk-certification
                  key: db_port
            - name: DB_NAME
              valueFrom:
                configMapKeyRef:
                  name: vshk-local-vshk-certification
                  key: db_name
            
            - name: DB_USER
              valueFrom:
                secretKeyRef:
                  name: vshk-local-vshk-certification
                  key: db_user
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: vshk-local-vshk-certification
                  key: db_password
      restartPolicy: Always
      terminationGracePeriodSeconds: 20
---
# Source: vshk-certification/templates/cronjob.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello-world-shmaks
spec:
  schedule: "*/5 * * * *"
  concurrencyPolicy: Replace
  startingDeadlineSeconds: 20
  successfulJobsHistoryLimit: 2
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      backoffLimit: 5
      activeDeadlineSeconds: 60
      template:
        spec:
          containers:
          - name: helloer
            resources: 
              limits:
                cpu: 100m
                memory: 128Mi
              requests:
                cpu: 100m
                memory: 128Mi
            image: busybox
            args:
            - /bin/sh
            - -c
            - echo Hello world!
          restartPolicy: Never
---
# Source: vshk-certification/templates/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: vshk-local-vshk-certification
  labels:
    helm.sh/chart: vshk-certification-0.0.4
    app.kubernetes.io/name: vshk-certification
    app.kubernetes.io/instance: vshk-local
    app.kubernetes.io/version: "0.0.1"
    app.kubernetes.io/managed-by: Helm
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  ingressClassName: nginx
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name:  vshk-local-vshk-certification
                port:
                  number: 80
---
# Source: vshk-certification/templates/tests/test-connection.yaml
apiVersion: v1
kind: Pod
metadata:
  name: "vshk-local-vshk-certification-test-connection"
  labels:
    helm.sh/chart: vshk-certification-0.0.4
    app.kubernetes.io/name: vshk-certification
    app.kubernetes.io/instance: vshk-local
    app.kubernetes.io/version: "0.0.1"
    app.kubernetes.io/managed-by: Helm
  annotations:
    "helm.sh/hook": test
spec:
  containers:
    - name: wget
      image: busybox
      command: ['wget']
      args: ['vshk-local-vshk-certification:80']
  restartPolicy: Never

```
