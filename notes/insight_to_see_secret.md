## "Amygdala": Do you have enough insight to see the secrets?

### Problem

Troubleshoot and fix a Kubernetes web application running in the app namespace. Make the deployment run successfully so that it returns Hello handsome! when you curl it.

Fix first your admin user access to the local Kubernetes cluster; the KUBECONFIG environment variable **must be set** to `$HOME/.kube/config`.

You have full admin access to a Vault server (containing the secrets you need) from the admin user. All the used manifests for the application are placed on the `/home/admin/manifests` directory.

ref: https://sadservers.com/scenario/amygdala

### Solution

#### 1. get kubeconfig from vault kv

```bash
admin@i-020771155a48871f9:~/manifests$ vault kv list secrets/
Keys
----
app/
kubeconfig
admin@i-020771155a48871f9:~/manifests$ vault kv get secrets/kubeconfig
===== Secret Path =====
secrets/data/kubeconfig

======= Metadata =======
Key                Value
---                -----
created_time       2025-10-06T00:06:47.741228503Z
custom_metadata    <nil>
deletion_time      n/a
destroyed          false
version            1

======= Data =======
Key           Value
---           -----
kubeconfig    apiVersion: v1
clusters:
- cluster:
    server: https://127.0.0.1:6443
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJkekNDQVIyZ0F3SUJBZ0lCQURBS0JnZ3Foa2pPUFFRREFqQWpNU0V3SHdZRFZRUUREQmhyTTNNdGMyVnkKZG1WeUxXTmhRREUzTlRjM056SXdPRFV3SGhjTk1qVXdPVEV6TVRRd01USTFXaGNOTXpVd09URXhNVFF3TVRJMQpXakFqTVNFd0h3WURWUVFEREJock0zTXRjMlZ5ZG1WeUxXTmhRREUzTlRjM056SXdPRFV3V1RBVEJnY3Foa2pPClBRSUJCZ2dxaGtqT1BRTUJCd05DQUFUai9uODJhTDQ5aFRlc3orcVBBR0l3S3lLQzd6eThUSlQ0MHBza1B5RVkKeGl6enM0V3JXVzBRL0ZxMFJpSkZaS0l5RHdqMFh6REtXQXlaSWVhTExwRDVvMEl3UURBT0JnTlZIUThCQWY4RQpCQU1DQXFRd0R3WURWUjBUQVFIL0JBVXdBd0VCL3pBZEJnTlZIUTRFRmdRVWs5dnNhTjNOMUltdlNjK1N6SER1CmwzaUtYc0V3Q2dZSUtvWkl6ajBFQXdJRFNBQXdSUUloQUlIc2dFcHVoMmgwRlIzVUtYSXJaU2RVdGtua1NmYVcKRVBuSCszeUJYUXc5QWlCTzcvTFVFNXB0d2szRzYvNjRCaDU3NnlFWDVBY1ZLczF5QXNLdzFjZlhFQT09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
  name: default
contexts:
- context:
    cluster: default
    namespace: app
    user: admin
  name: default
current-context: default
users:
- name: admin
  user:
    token: eyJhbGciOiJSUzI1NiIsImtpZCI6InRkcFVjUzVxbHpESVdncWkxYWdrUGV3QzlRVFl6dEZIZGhoZGlCMjVqeUUifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJ1c2VycyIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi1zYXQiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiYWRtaW4iLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIxMmQ1NDBkOS04YjcxLTRjYzctOWI4OC0zZDk3OTU2MmEzZGYiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6dXNlcnM6YWRtaW4ifQ.CD8ljVlJW1fGxtBWn3kegyvZ5MPHD2LfP9OZdyHZ0AM5lK8nCwI-UWTMSSHbXiYFx3He5J6hOiWW7MbZ9ppK7iTJQWiQ_s3MXayPu2WJGChoHBsa8r0oFw-8NqXkok_RiVI8yA6p5UFgnmlMoOtdLdLWI8zegULKdpzUs3WzrrTeD6s_SCo2SYYIS8EKpniu0MW0wQr7PXdyT8XcFbAiBj74r6PfFCNoC6A1Ohrgq6QlRKX_wiaYzLPWEBpLhzdTQodZvEYfRSYQfdmiMtTJdqHW4hsg33sRIi51dkvahS-8aqS68zemMLfLY9WAUtBwX_-vUBndKQn4XoIDEHdBRA
```

#### 2. copy the kubeconfig from vault to ~/.kube/config and set KUBECONFIG environment variable 

```
admin@i-020771155a48871f9:~$ set KUBECONFIG=~/kube/config
```

#### 3. try to access the k8s cluster

```
admin@i-020771155a48871f9:~$ kubectl get all
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/app   0/1     0            0           16m

NAME                            DESIRED   CURRENT   READY   AGE
replicaset.apps/app-c65b696bf   1         0         0       16m
```

#### 4. check the app events

Three problems:

1. ReplicaSet/app-c65b696bf: `forbidden: violates PodSecurity "baseline:latest": privileged (container "nginx" must not set securityContext.privileged=true)`
2. ExternalSecret/env: `error processing spec.data[0] (key: app/override/.env), err: could not get ClusterSecretStore "vault-backend", ClusterSecretStore.external-secrets.io "vault-backend" not found`
3. ExternalSecret/env-hidden: `error processing spec.data[0] (key: app/override/.env), err: could not get ClusterSecretStore "vault-backend", ClusterSecretStore.external-secrets.io "vault-backend" not found`

```
admin@i-020771155a48871f9:~$ kubectl events deployment.apps/app
LAST SEEN            TYPE      REASON              OBJECT                      MESSAGE
17m                  Warning   FailedCreate        ReplicaSet/app-c65b696bf    Error creating: pods "app-c65b696bf-4g5ks" is forbidden: violates PodSecurity "baseline:latest": privileged (container "nginx" must not set securityContext.privileged=true)
17m                  Warning   FailedCreate        ReplicaSet/app-c65b696bf    Error creating: pods "app-c65b696bf-46c2m" is forbidden: violates PodSecurity "baseline:latest": privileged (container "nginx" must not set securityContext.privileged=true)
17m                  Warning   FailedCreate        ReplicaSet/app-c65b696bf    Error creating: pods "app-c65b696bf-mtxpg" is forbidden: violates PodSecurity "baseline:latest": privileged (container "nginx" must not set securityContext.privileged=true)
17m                  Warning   FailedCreate        ReplicaSet/app-c65b696bf    Error creating: pods "app-c65b696bf-thcr5" is forbidden: violates PodSecurity "baseline:latest": privileged (container "nginx" must not set securityContext.privileged=true)
17m                  Normal    ScalingReplicaSet   Deployment/app              Scaled up replica set app-c65b696bf to 1
17m                  Warning   FailedCreate        ReplicaSet/app-c65b696bf    Error creating: pods "app-c65b696bf-hcsbc" is forbidden: violates PodSecurity "baseline:latest": privileged (container "nginx" must not set securityContext.privileged=true)
17m                  Warning   FailedCreate        ReplicaSet/app-c65b696bf    Error creating: pods "app-c65b696bf-btbz9" is forbidden: violates PodSecurity "baseline:latest": privileged (container "nginx" must not set securityContext.privileged=true)
17m                  Warning   FailedCreate        ReplicaSet/app-c65b696bf    Error creating: pods "app-c65b696bf-669v2" is forbidden: violates PodSecurity "baseline:latest": privileged (container "nginx" must not set securityContext.privileged=true)
17m                  Warning   FailedCreate        ReplicaSet/app-c65b696bf    Error creating: pods "app-c65b696bf-pfn8g" is forbidden: violates PodSecurity "baseline:latest": privileged (container "nginx" must not set securityContext.privileged=true)
17m                  Warning   FailedCreate        ReplicaSet/app-c65b696bf    Error creating: pods "app-c65b696bf-w5krm" is forbidden: violates PodSecurity "baseline:latest": privileged (container "nginx" must not set securityContext.privileged=true)
7m (x9 over 17m)     Warning   FailedCreate        ReplicaSet/app-c65b696bf    (combined from similar events): Error creating: pods "app-c65b696bf-fbv9m" is forbidden: violates PodSecurity "baseline:latest": privileged (container "nginx" must not set securityContext.privileged=true)
61s (x11 over 16m)   Warning   UpdateFailed        ExternalSecret/env-hidden   error processing spec.data[0] (key: app/override/.env), err: could not get ClusterSecretStore "vault-backend", ClusterSecretStore.external-secrets.io "vault-backend" not found
61s (x11 over 16m)   Warning   UpdateFailed        ExternalSecret/env          error processing spec.data[0] (key: app/env), err: could not get ClusterSecretStore "vault-backend", ClusterSecretStore.external-secrets.io "vault-backend" not found
```

#### 5. fix the privileged security context in deployment

Edit the deployment to remove the privileged security context and add the capability to bind to port 80.

```
securityContext:
    capabilities:
    add: ["NET_BIND_SERVICE"]
```

#### 5. try to restart the deployment and check the events

```
37s                    Warning   FailedCreate        ReplicaSet/app-66b5f57c7f   Error creating: pods "app-66b5f57c7f-668sh" is forbidden: failed quota: compute: must specify limits.cpu for: init-html,nginx; limits.memory for: init-html,nginx; requests.cpu for: init-html,nginx; requests.memory for: init-html,nginx
18s (x4 over 36s)      Warning   FailedCreate        ReplicaSet/app-66b5f57c7f   (combined from similar events): Error creating: pods "app-66b5f57c7f-dv82c" is forbidden: failed quota: compute: must specify limits.cpu for: init-html,nginx; limits.memory for: init-html,nginx; requests.cpu for: init-html,nginx; requests.memory for: init-html,nginx
```

#### 6. check resource quota and add resources request and limit in the deployment

> A resource quota, defined by a ResourceQuota object, provides constraints that limit aggregate resource consumption per **namespace**.

resource quota:

```
admin@i-020771155a48871f9:~/manifests$ kubectl get resourcequotas 
NAME      AGE     REQUEST                                        LIMIT
compute   3d16h   requests.cpu: 0/20m, requests.memory: 0/64Mi   limits.cpu: 0/1, limits.memory: 0/512Mi
```

Updated deployment with resources request and limit:

updated the wrong secret name from `env-override` to `env-hidden` as well.

```
admin@i-020771155a48871f9:~/manifests$ cat app.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: app
  name: app
  namespace: app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app
  template:
    metadata:
      labels:
        app: app
    spec:
      initContainers:
      - name: init-html
        image: busybox
        resources:
          requests:  # Minimum resources for scheduling
            cpu: "10m"
            memory: "32Mi"
          limits:       # Maximum resources (hard caps)
            cpu: "50m"    # 0.5 cores
            memory: "256Mi"
        volumeMounts:
          - name: html
            mountPath: /usr/share/nginx/html
        envFrom:
          - secretRef:
              name: env
          - secretRef:
              name: env-hidden
        args:
          - sh
          - -c
          - 'echo "$first $second" > /usr/share/nginx/html/index.html'
      containers:
      - name: nginx
        image: nginx
        securityContext:
          capabilities:
            add: ["NET_BIND_SERVICE"]
        resources:
          requests:  # Minimum resources for scheduling
            cpu: "10m"
            memory: "32Mi"
          limits:       # Maximum resources (hard caps)
            cpu: "50m"
            memory: "256Mi"
        ports:
        - containerPort: 80
        volumeMounts:
          - name: html
            mountPath: /usr/share/nginx/html
      volumes:
        - name: html
          emptyDir: {}
```

#### 7. make sure the secrets are in vault kv

Yes, they are.

- `secrets/app/env` with `first=Hello`
- `secrets/app/.hidden/env` with `second=handsome!`

```
admin@i-04df897e6526ac033:~/manifests$ vault kv list /secrets/app
Keys
----
.hidden/
env
admin@i-04df897e6526ac033:~/manifests$ vault kv list /secrets/app/.hidden
Keys
----
env
admin@i-04df897e6526ac033:~/manifests$ vault kv get /secrets/app/.hidden/env
======== Secret Path ========
secrets/data/app/.hidden/env

======= Metadata =======
Key                Value
---                -----
created_time       2025-10-06T00:06:44.537909129Z
custom_metadata    <nil>
deletion_time      n/a
destroyed          false
version            1

===== Data =====
Key       Value
---       -----
second    handsome!
admin@i-04df897e6526ac033:~/manifests$ vault kv get /secrets/app/env
==== Secret Path ====
secrets/data/app/env

======= Metadata =======
Key                Value
---                -----
created_time       2025-10-06T00:06:44.539286887Z
custom_metadata    <nil>
deletion_time      n/a
destroyed          false
version            1

==== Data ====
Key      Value
---      -----
first    Hello
```


#### 8. fix external secret manifest

- change the secret store name from `vault-backend` to `vault`
- change secret key of the remote reference from `app/override/.env` to `app/.hidden/env` for the hidden secret
- change `spec.data.property` and `spec.data.secretKey` from `second(first)` to `first(second)`

```
admin@i-04df897e6526ac033:~/manifests$ cat external-secret.yaml 
apiVersion: external-secrets.io/v1
kind: ExternalSecret
metadata:
  name: env
  namespace: app
spec:
  refreshInterval: "15s"
  secretStoreRef:
    name: vault
    kind: ClusterSecretStore
  target:
    name: env
  data:
  - secretKey: first
    remoteRef:
      key: app/env
      property: first
---
apiVersion: external-secrets.io/v1
kind: ExternalSecret
metadata:
  name: env-hidden
  namespace: app
spec:
  refreshInterval: "15s"
  secretStoreRef:
    name: vault
    kind: ClusterSecretStore
  target:
    name: env-hidden
  data:
  - secretKey: second
    remoteRef:
      key: app/.hidden/env
      property: second
```

#### 9. test

Note: apply the external secret and restart the deployment after the external secret is created

```
admin@i-04df897e6526ac033:~/manifests$ POD_IP=$(kubectl get po -n app -l app=app -o jsonpath='{.items[0].status.podIP}') 
admin@i-04df897e6526ac033:~/manifests$ curl http://$POD_IP
Hello handsome
```
