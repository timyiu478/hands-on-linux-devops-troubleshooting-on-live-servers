## "Ruaka": Kubernetes pod in distress

### Problem

A developer wants to deploy an open-source tool on Kubernetes. The tool unfortunately has limited documentation.

They built a helm chart and a container image. When the application is deployed, for some reason the server in Kubernetes doesn't seem to work but when the binary is started on their laptop/machine it works perfectly.

The application server is deployed by Helm. The command they used is: helm upgrade --install ruaka charts/ruaka.

Debug and help the developer find the issue. NOTE: Do not delete any current Helm field in the chart.

Remember to give enough time to k8S after you apply a change before checking the solution.

src: https://sadservers.com/scenario/ruaka

### Solution

#### 1. check the Kubernetes events

We can see the liveness and readiness probes are failing.

```
admin@i-0fa65992103fea071:~/charts/ruaka/templates$ kubectl events
59s                  Normal    Killing                          Pod/ruaka-c64f8549c-phjmk    Container ruaka failed liveness probe, will be restarted
59s                  Warning   Unhealthy                        Pod/ruaka-c64f8549c-phjmk    Readiness probe failed: Get "http://10.42.0.6:3333/": dial tcp 10.42.0.6:3333: connect: connection refused
59s (x2 over 73s)    Normal    Started                          Pod/ruaka-c64f8549c-phjmk    Started container ruaka
59s (x2 over 73s)    Normal    Pulled                           Pod/ruaka-c64f8549c-phjmk    Container image "localhost:5000/ruaka:v0.0.3" already present on machine
48s (x10 over 72s)   Warning   Unhealthy                        Pod/ruaka-c64f8549c-phjmk    Readiness probe failed: HTTP probe failed with statuscode: 503
44s (x6 over 69s)    Warning   Unhealthy                        Pod/ruaka-c64f8549c-phjmk    Liveness probe failed: HTTP probe failed with statuscode: 503
```

#### 2. check whether the helm values are set

No values are set.

```
admin@i-0fa65992103fea071:~$ helm get values ruaka 
USER-SUPPLIED VALUES:
null
```

#### 3. helm upgrade with values

```
admin@i-0fa65992103fea071:~$ helm upgrade ruaka ~/charts/ruaka/ --values ~/charts/ruaka/values.yaml 
Release "ruaka" has been upgraded. Happy Helming!
NAME: ruaka
LAST DEPLOYED: Sun Sep 14 01:23:29 2025
NAMESPACE: default
STATUS: deployed
REVISION: 2
TEST SUITE: None
NOTES:
ruaka application version v0.0.3 deployed 🚀!
admin@i-0fa65992103fea071:~$ helm get values ruaka 
USER-SUPPLIED VALUES:
affinity: {}
fullnameOverride: ""
image:
  pullPolicy: IfNotPresent
  repository: localhost:5000/ruaka
  tag: v0.0.3
imagePullSecrets: []
livenessProbe:
  httpGet:
    path: /healthz
    port: http
  periodSeconds: 5
  terminationGracePeriodSeconds: 3
nameOverride: ""
nodeSelector: {}
podAnnotations: {}
podLabels: {}
podSecurityContext: {}
readinessProbe:
  httpGet:
    path: /
    port: http
  periodSeconds: 5
replicaCount: 1
resources:
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 100m
    memory: 128Mi
securityContext: {}
tolerations: []
volumeMounts: []
volumes: []
```

#### 4. check the pod status

still CrashLoopBackOff.

```
admin@i-0fa65992103fea071:~$ kubectl get pods
NAME                    READY   STATUS             RESTARTS         AGE
ruaka-c64f8549c-phjmk   0/1     CrashLoopBackOff   11 (4m43s ago)   10h
```

#### 5. check the pod logs

The app depends on a database and it takes around 1 second to connect to the database. The liveness and readiness probes can be too aggressive.

```
admin@i-0fa65992103fea071:~$ kubectl logs ruaka-c64f8549c-phjmk
2025-09-14 01:32:27 connecting to the database...
2025-09-14 01:32:28 connected to database.
2025-09-14 01:32:28 listening on port :3333.
```

tried to add the `5` initialDelaySeconds of the probes and found the pod also needs more time to load its configurations.

```
admin@i-0fa65992103fea071:~$ kubectl logs ruaka-c64f8549c-phjmk
2025-09-14 01:37:58 connecting to the database...
2025-09-14 01:37:59 connected to database.
2025-09-14 01:37:59 listening on port :3333.
2025-09-14 01:38:05 load server configurations... // <--- takes time
```

#### 6. change initialDelaySeconds to 20 seconds

Then the pod is running fine.

```
admin@i-0fa65992103fea071:~$ kubectl get deployments.apps 
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
ruaka   1/1     1            1           10h
```
