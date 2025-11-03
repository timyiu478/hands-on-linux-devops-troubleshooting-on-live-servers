## "Tigoni": Patch and Pray!

### Problem

Description: A developer wants to upgrade their stateful application. This application handles their archives/backups.

The application tigoni is deployed on kubernetes in the default namespace.
The application server is deployed by Helm. The command they used is helm install tigoni charts/tigoni.

Upgrade the tigoni application to v2.0.0. The image already exists in the local repository.
Debug and help the developer fix any issue with the upgrade.
Everytime v1.0.0 is launched the archiving code starts on a clean slate.

Root (sudo) Access: True

Test: The tigoni pod with version v2 runs correctly (its endpoint :3000/healthz displays serverVersion:v2)

### Solution

#### 1. Try to upgrade container image from v1 to v2

The container is keep crashing after the upgrade.

```
admin@i-0344c6c51939cc6b3:~$ kubectl edit statefulsets.apps tigoni 
statefulset.apps/tigoni edited
admin@i-0344c6c51939cc6b3:~$ kubectl get statefulsets.apps tigoni -o yaml | grep image
      - image: localhost:5000/tigoni:v2.0.0
        imagePullPolicy: IfNotPresent
admin@i-0344c6c51939cc6b3:~$ kubectl get pods
NAME       READY   STATUS             RESTARTS     AGE
tigoni-0   0/1     CrashLoopBackOff   3 (8s ago)   61s
```

#### 2. check the log

Error Log: `failed to open archive:zip: not a valid zip file`

```
admin@i-0344c6c51939cc6b3:~$ kubectl logs tigoni-0
2025/11/01 01:53:39 tigoni server is ready to serve! 🚀
2025/11/01 01:53:39 ❌ failed to open archive:zip: not a valid zip file
```

#### 3. Try to rollback

It works. 

```
admin@i-0344c6c51939cc6b3:~$ kubectl get pods tigoni-0
NAME       READY   STATUS    RESTARTS   AGE
tigoni-0   1/1     Running   0          9s
admin@i-0344c6c51939cc6b3:~$ kubectl logs tigoni-0
2025/11/01 02:15:56 tigoni server is ready to serve! 🚀
2025/11/01 02:15:56 building archive...
```

#### 4. run a debug pod

```
admin@i-0344c6c51939cc6b3:~$ kubectl apply -f debug-pod.yaml 
pod/alpine created
admin@i-0344c6c51939cc6b3:~$ cat debug-pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: alpine
spec:
  containers:
    - name: debug
      image: localhost:5000/alpine:util
      command: ["sh", "-c", "sleep 3600"]
      volumeMounts:
        - mountPath: /opt/tigoni
          name: archive-server-directory
  volumes:
    - name: archive-server-directory
      persistentVolumeClaim:
        claimName: tigoni
```

#### 5. understand more about the zip file

The size of the zip file is 0.

```
admin@i-0344c6c51939cc6b3:~$ kubectl exec alpine -- find / -name archive.zip
/opt/tigoni/archive.zip
admin@i-0344c6c51939cc6b3:~$ kubectl exec alpine -- ls -alt /opt/tigoni/archive.zip
-rw-r--r--    1 65532    65532            0 Nov  1 02:15 /opt/tigoni/archive.zip
admin@i-0344c6c51939cc6b3:~$ kubectl exec alpine -- zipinfo /opt/tigoni/archive.zip
Archive:  /opt/tigoni/archive.zip
[/opt/tigoni/archive.zip]
  End-of-central-directory signature not found.  Either this file is not
  a zipfile, or it constitutes one disk of a multi-part archive.  In the
  latter case the central directory and zipfile comment will be found on
  the last disk(s) of this archive.
zipinfo:  cannot find zipfile directory in one of /opt/tigoni/archive.zip or
          /opt/tigoni/archive.zip.zip, and cannot find /opt/tigoni/archive.zip.ZIP, period.
command terminated with exit code 9
```

#### 6. try to add raw zip format for an empty archive

It works.

```
admin@i-0344c6c51939cc6b3:~$ kubectl exec alpine -- sh -c "cd /opt/tigoni && printf '\x50\x4b\x05\x06\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00' > archive.zip"
Cadmin@i-0344c6c51939cc6b3:~$ kubectl exec alpine -- zipinfo /opt/tigoni/archive.zip
Archive:  /opt/tigoni/archive.zip
Zip file size: 22 bytes, number of entries: 0
Empty zipfile.
command terminated with exit code 1
admin@i-0344c6c51939cc6b3:~$ kubectl edit statefulsets.apps tigoni 
statefulset.apps/tigoni edited
admin@i-0344c6c51939cc6b3:~$ kubectl get pods
NAME       READY   STATUS        RESTARTS      AGE
alpine     1/1     Running       0             33s
tigoni-0   1/1     Terminating   2 (52s ago)   15d
admin@i-0344c6c51939cc6b3:~$ kubectl get pods -w
NAME       READY   STATUS        RESTARTS      AGE
alpine     1/1     Running       0             36s
tigoni-0   1/1     Terminating   2 (55s ago)   15d
tigoni-0   0/1     Terminating   2 (78s ago)   15d
tigoni-0   0/1     Terminating   2 (78s ago)   15d
tigoni-0   0/1     Terminating   2 (78s ago)   15d
tigoni-0   0/1     Terminating   2 (78s ago)   15d
tigoni-0   0/1     Pending       0             0s
tigoni-0   0/1     Pending       0             0s
tigoni-0   0/1     ContainerCreating   0             0s
tigoni-0   0/1     Running             0             2s
tigoni-0   1/1     Running             0             3s
```
