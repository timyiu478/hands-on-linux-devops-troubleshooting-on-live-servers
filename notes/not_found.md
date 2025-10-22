## "Atlantis": Not found

### Problem

There is a small "C" application in the /home/admin/app directory. Create the Docker container "app" with a small footprint and minimalistic so you get a hello binary that returns a greeting in Atlantean (Docker multi-stage build). The binary application is automatically called when running docker run app

https://sadservers.com/newserver/atlantis

### Solution

#### 1. read the files

- the build stage base image is debian:13 and the final image is alpine:3.20.
- probably the runtime incompatibility on the C library between debian and alpine.
- https://docs.docker.com/dhi/core-concepts/glibc-musl/

```
admin@i-07f7461da4f6dd338:~/app$ cat -n Dockerfile 
     1  # STAGE 1
     2  FROM    debian:13 AS builder
     3  RUN     apt-get update && apt-get install -y gcc
     4  WORKDIR /src
     5  COPY    hello.c .
     6  RUN     gcc -o hello hello.c
     7
     8  # STAGE 2
     9  FROM    alpine:3.20
    10  COPY    --from=builder /src/hello /usr/local/bin/hello
    11  CMD     ["/usr/local/bin/hello"]
admin@i-07f7461da4f6dd338:~/app$ cat -n hello.c 
     1  #include <stdio.h>
     2
     3  int main() {
     4      printf("SOO-puhk\n");
     5      return 0;
     6  }
```

#### 2. update the dockerfile from debian to alpine

```
admin@i-03d0bf7b85ba6e5ea:~/app$ cat -n Dockerfile 
     1  # STAGE 1
     2  FROM    alpine:3.20 AS builder
     3  RUN     apk update && apk add gcc musl-dev
     4  WORKDIR /src
     5  COPY    hello.c .
     6  RUN     gcc -o hello hello.c
     7
     8  # STAGE 2
     9  FROM    alpine:3.20
    10  COPY    --from=builder /src/hello /usr/local/bin/hello
    11  CMD     ["/usr/local/bin/hello"]
```

#### 3. build and run the container

```
admin@i-03d0bf7b85ba6e5ea:~/app$ docker build . -t app

[+] Building 5.7s (11/11) FINISHED                                                                                                     docker:default
 => [internal] load build definition from Dockerfile                                                                                             0.0s
 => => transferring dockerfile: 281B                                                                                                             0.0s
 => [internal] load metadata for docker.io/library/alpine:3.20                                                                                   0.0s
 => [internal] load .dockerignore                                                                                                                0.0s
 => => transferring context: 2B                                                                                                                  0.0s
 => [internal] load build context                                                                                                                0.0s
 => => transferring context: 28B                                                                                                                 0.0s
 => CACHED [builder 1/5] FROM docker.io/library/alpine:3.20                                                                                      0.0s
 => [builder 2/5] RUN APK update && apk add gcc musl-dev                                                                                         3.9s
 => [builder 3/5] WORKDIR /SRC                                                                                                                   0.1s 
 => [builder 4/5] COPY HELLO.C .                                                                                                                 0.1s 
 => [builder 5/5] RUN GCC -o hello hello.c                                                                                                       1.0s 
 => [stage-1 2/2] COPY --FROM=BUILDER /src/hello /usr/local/bin/hello                                                                            0.1s 
 => exporting to image                                                                                                                           0.1s 
 => => exporting layers                                                                                                                          0.1s 
 => => writing image sha256:a4d03b7160e23fe70e222443e485b653459e314555cbddbf1def38cf62542ca6                                                     0.0s
 => => naming to docker.io/library/app                                                                                                           0.0s
admin@i-03d0bf7b85ba6e5ea:~/app$ docker run app
SOO-puhk
```
