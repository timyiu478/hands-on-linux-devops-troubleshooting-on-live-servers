## "Kilifi": Speculative Misallocation!

### Problem

Description: A developer is having trouble deploying an application on a preconfigured cluster.

The application kilifi is to be deployed on kubernetes in the default namespace.
The application server is deployed by helm. The command they used is helm install kilifi charts/kilifi.
The application operates correctly with ~210 MB of memory, but 256 MB is recommended.

Swap should remain disabled in the cluster.

Debug and help the developer fix any issue with deployment.

Root (sudo) Access: True

Test: The kilifi application runs properly; it's Service on :3333/healthz returns "kilifi ready to serve".

### Solution

#### 1. inspect the pod manifest

Key information:

* kubernetes.io/limit-ranger: 'LimitRanger plugin set: memory request for init container init-config; memory limit for init container init-config'
* message: '0/1 nodes are available: 1 Insufficient memory. preemption: 0/1 nodes are available: 1 No preemption victims found for incoming pod.'
* Container's resource requests and lmits:
    * kilifi: 512Mi memory
    * init-config: 1Gi memory
* overhead: 1Gi memory
    * this field is added by the runtime class controller

```
admin@ip-172-31-43-69:~$ kubectl get pods -o yaml
apiVersion: v1
items:
- apiVersion: v1
  kind: Pod
  metadata:
    annotations:
      kubernetes.io/limit-ranger: 'LimitRanger plugin set: memory request for init
        container init-config; memory limit for init container init-config'
    creationTimestamp: "2025-10-12T23:00:46Z"
    generateName: kilifi-8f867cbcd-
    labels:
      app.kubernetes.io/instance: kilifi
      app.kubernetes.io/managed-by: Helm
      app.kubernetes.io/name: kilifi
      app.kubernetes.io/version: 0.0.1
      helm.sh/chart: kilifi-0.1.0
      pod-template-hash: 8f867cbcd
    name: kilifi-8f867cbcd-kx6wj
    namespace: default
    ownerReferences:
    - apiVersion: apps/v1
      blockOwnerDeletion: true
      controller: true
      kind: ReplicaSet
      name: kilifi-8f867cbcd
      uid: 9e092914-cfc9-4c85-9907-c5bd1b5a3a4d
    resourceVersion: "551"
    uid: 48109e71-e890-4b19-8992-fbb364efefab
  spec:
    containers:
    - image: localhost:5000/kilifi:v0.0.1
      imagePullPolicy: IfNotPresent
      livenessProbe:
        failureThreshold: 3
        httpGet:
          path: /healthz
          port: http
          scheme: HTTP
        periodSeconds: 5
        successThreshold: 1
        terminationGracePeriodSeconds: 3
        timeoutSeconds: 1
      name: kilifi
      ports:
      - containerPort: 3333
        name: http
        protocol: TCP
      readinessProbe:
        failureThreshold: 3
        httpGet:
          path: /healthz
          port: http
          scheme: HTTP
        periodSeconds: 5
        successThreshold: 1
        timeoutSeconds: 1
      resources:
        limits:
          memory: 512Mi
        requests:
          cpu: 100m
          memory: 512Mi
      terminationMessagePath: /dev/termination-log
      terminationMessagePolicy: File
      volumeMounts:
      - mountPath: /srv/kilifi
        name: config-volume
      - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
        name: kube-api-access-zk9ww
        readOnly: true
    dnsPolicy: ClusterFirst
    enableServiceLinks: true
    initContainers:
    - command:
      - sh
      - -c
      - echo "{}" > /srv/kilifi/config.json
      image: localhost:5000/busybox:1.30
      imagePullPolicy: IfNotPresent
      name: init-config
      resources:
        limits:
          memory: 1Gi
        requests:
          memory: 1Gi
      terminationMessagePath: /dev/termination-log
      terminationMessagePolicy: File
      volumeMounts:
      - mountPath: /srv/kilifi
        name: config-volume
      - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
        name: kube-api-access-zk9ww
        readOnly: true
    overhead:
      memory: 1Gi
    preemptionPolicy: PreemptLowerPriority
    priority: 0
    restartPolicy: Always
    runtimeClassName: default
    schedulerName: default-scheduler
    securityContext: {}
    serviceAccount: default
    serviceAccountName: default
    terminationGracePeriodSeconds: 30
    tolerations:
    - effect: NoExecute
      key: node.kubernetes.io/not-ready
      operator: Exists
      tolerationSeconds: 300
    - effect: NoExecute
      key: node.kubernetes.io/unreachable
      operator: Exists
      tolerationSeconds: 300
    volumes:
    - emptyDir: {}
      name: config-volume
    - name: kube-api-access-zk9ww
      projected:
        defaultMode: 420
        sources:
        - serviceAccountToken:
            expirationSeconds: 3607
            path: token
        - configMap:
            items:
            - key: ca.crt
              path: ca.crt
            name: kube-root-ca.crt
        - downwardAPI:
            items:
            - fieldRef:
                apiVersion: v1
                fieldPath: metadata.namespace
              path: namespace
  status:
    conditions:
    - lastProbeTime: null
      lastTransitionTime: "2025-10-12T23:00:46Z"
      message: '0/1 nodes are available: 1 Insufficient memory. preemption: 0/1 nodes
        are available: 1 No preemption victims found for incoming pod.'
      reason: Unschedulable
      status: "False"
      type: PodScheduled
    phase: Pending
    qosClass: Burstable
kind: List
metadata:
  resourceVersion: ""
```

#### 2. redeploy after setting memory request and limit to 256Mi

Deployment manifest's status tells us `pods "kilifi-7568dfdfc8-vjgpq" is forbidden: minimum memory usage per Container is 512Mi, but request is 256Mi`

```
admin@ip-172-31-43-69:~$ kubectl get deployments.apps kilifi -o yaml
...
  - lastTransitionTime: "2025-11-01T01:17:15Z"
    lastUpdateTime: "2025-11-01T01:17:15Z"
    message: 'pods "kilifi-7568dfdfc8-vjgpq" is forbidden: minimum memory usage per
      Container is 512Mi, but request is 256Mi'
```

#### 3. update limitranges

* min memory from 512Mi to 128Mi
* default from 1Gi to 128Mi

```
admin@ip-172-31-43-69:~$ kubectl get limitranges default -o yaml
apiVersion: v1
kind: LimitRange
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"LimitRange","metadata":{"annotations":{},"name":"default","namespace":"default"},"spec":{"limits":[{"default":{"memory":"1Gi"},"defaultRequest":{"memory":"1Gi"},"max":{"memory":"1Gi"},"min":{"memory":"512Mi"},"type":"Container"}]}}
  creationTimestamp: "2025-10-12T23:00:40Z"
  name: default
  namespace: default
  resourceVersion: "534"
  uid: 4698e73d-a3b8-4a7e-a668-3c4d91dd904e
spec:
  limits:
  - default:
      memory: 1Gi
    defaultRequest:
      memory: 1Gi
    max:
      memory: 1Gi
    min:
      memory: 512Mi
    type: Container
admin@ip-172-31-43-69:~$ kubectl edit limitranges default 
limitrange/default edited
admin@ip-172-31-43-69:~$ kubectl get limitranges default -o yaml
apiVersion: v1
kind: LimitRange
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"LimitRange","metadata":{"annotations":{},"name":"default","namespace":"default"},"spec":{"limits":[{"default":{"memory":"1Gi"},"defaultRequest":{"memory":"1Gi"},"max":{"memory":"1Gi"},"min":{"memory":"512Mi"},"type":"Container"}]}}
  creationTimestamp: "2025-10-12T23:00:40Z"
  name: default
  namespace: default
  resourceVersion: "678"
  uid: 4698e73d-a3b8-4a7e-a668-3c4d91dd904e
spec:
  limits:
  - default:
      memory: 128Mi
    defaultRequest:
      memory: 128Mi
    max:
      memory: 1Gi
    min:
      memory: 128Mi
    type: Container
```

#### 4. check 

```

```
