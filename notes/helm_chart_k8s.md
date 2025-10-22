## "Poznań": Helm Chart Issue in Kubernetes

### Problem

NOTE: Prompt may take a few extra seconds to be responsive while the k3s environment gets ready. Root access is not needed for this challenge ("admin" user cannot sudo).

A DevOps engineer created a Helm Chart web_chart with a custom nginx site, however he still gets the default nginx index.html.

You can check for example with `POD_IP=$(kubectl get pods -n default -o jsonpath='{.items[0].status.podIP}')` and `curl -s "${POD_IP}">`.

In addition he should set replicas to 3.

The chart is not working as expected. Fix the configurations so you get the custom HTML page from any nginx pod.

https://sadservers.com/scenario/poznan

### Solution

#### 1. check we can reach the site

We can see the default nginx page.

```bash
admin@i-06d83b64d85efe799:~/web_chart/templates$ kubectl get pods
NAME                              READY   STATUS    RESTARTS        AGE
web-chart-nginx-c867f54dc-rlntp   1/1     Running   1 (7m34s ago)   6d
admin@i-06d83b64d85efe799:~/web_chart/templates$ POD_IP=$(kubectl get pods -n default -o jsonpath='{.items[0].status.podIP}')
admin@i-06d83b64d85efe799:~/web_chart/templates$ curl -s ${POD_IP}
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

#### 2. check the helm chart files

The index file is in the configmap.

```
admin@i-06d83b64d85efe799:~/web_chart/templates$ cat configmap.yaml 
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-cm-index-html
  namespace: default
data:
  index.html: |
    <html>
    <h1>Welcome SadServers</h1>
    </br>
    <h1>Hi! I got deployed successfully</h1>
```

The deployment file is:

```
admin@i-06d83b64d85efe799:~/web_chart/templates$ cat deployment.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-nginx
  labels:
    app: nginx
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
```

#### 3. update the index file by overriding the default nginx index.html

Updated deployment file:

- mount the configmap and override the default nginx index.html.

```
admin@i-06d83b64d85efe799:~/web_chart/templates$ cat deployment.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-nginx
  labels:
    app: nginx
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      replicas: 3
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
          volumeMounts:
                - name: index-volume
                  mountPath: /usr/share/nginx/html/index.html
                  subPath: index.html
      volumes:
        - name: index-volume
          configMap:
            name: {{ .Release.Name }}-cm-index-html
            items:
              - key: index.html
                path: index.html
```

#### 4. helm upgrade

We have to set replicas to 3 first.

```
admin@i-06a1697a435184fc8:~$ cat web_chart/values.yaml 
replicaCount: 3

image:
  repository: localhost:5000/nginx
  tag: "1.17.0"
  pullPolicy: IfNotPresent

service:
  name: nginx-service
  type: ClusterIP
  port: 8080
  targetPort: 9000
```

Then we upgrade the helm chart.

```
admin@i-06a1697a435184fc8:~$ helm upgrade web-chart ~/web_chart/ --namespace default --values ~/web_chart/values.yaml 
Release "web-chart" has been upgraded. Happy Helming!
NAME: web-chart
LAST DEPLOYED: Sun Sep 14 00:34:45 2025
NAMESPACE: default
STATUS: deployed
REVISION: 2
TEST SUITE: None
admin@i-06a1697a435184fc8:~$ helm get values web-chart 
USER-SUPPLIED VALUES:
image:
  pullPolicy: IfNotPresent
  repository: localhost:5000/nginx
  tag: 1.17.0
replicaCount: 3
service:
  name: nginx-service
  port: 8080
  targetPort: 9000
  type: ClusterIP
```
