# Docker image resizing

## Introduction

Goal: reduce image size

Tecniques:
- Multi-stage builds
- Scratch builds
- Bazel (https://bazel.build/)
- Distroless (https://github.com/GoogleContainerTools/distroless)
- DockerSlim (https://dockersl.im/)
- Ultimate Packer for eXecutables (UPX) (https://upx.github.io/) 

Some of these might be counter-productive in some scenarios, but might be useful in some particular cases.

---

## Simple code

- hello.c

```c
int main () {
  puts("Hello, world!");
  return 0;
}
```

- hello.go

```golang
package main

import "fmt"

func main () {
  fmt.Println("Hello, world!")
}
```

- Dockerfile

```dockerfile
FROM golang
COPY hello.go .
RUN go build hello.go
CMD ['hello']
```

```
-> docker image ls
REPOSITORY                  TAG                             IMAGE ID            CREATED              SIZE
hello_golang                latest                          5eef86aaf9e7        About a minute ago   811MB
```

For a binary of 2MB we get a 800MB of docker image (x400). What???

---

## Multi-stage builds

Simple idea: don't include the compiler inside the image (security???), but only the binary resulting after compiling.

```dockerfile
# has WORKDIR /go
FROM golang 
COPY hello.go .
RUN go build hello.go

FROM ubuntu:14.04
COPY --from=0 /go/hello .
CMD ["./hello"]
```

*Warning*

`COPY --from` interprets path from the root of previous stage.
But previous stage (golang) has WORKDIR /go
Hint: set explicitly a WORKING in your Dockerfile.

```
-> docker image ls
REPOSITORY                  TAG                             IMAGE ID            CREATED             SIZE
hello_multistage            latest                          31ade6bfec07        4 seconds ago       225MB
hello_golang                latest                          5eef86aaf9e7        4 minutes ago       811MB
```

Image size is now 225MB.
It's better, but we want a smaller image.

---

## Scratch builds

More simplicity: include only the binary.

```dockerfile
FROM golang
WORKDIR /src
COPY hello.go .
RUN go build hello.go

FROM scratch
COPY --from=0 /src/hello .
CMD ["./hello"]
```

```
->docker image ls
REPOSITORY                  TAG                             IMAGE ID            CREATED             SIZE
hello_scratch               latest                          989364b488ed        29 seconds ago      2.06MB
hello_multistage            latest                          31ade6bfec07        16 minutes ago      225MB
hello_golang                latest                          5eef86aaf9e7        21 minutes ago      811MB
```

Now a pretty simple and light container. Only 2MB!

---

## Scratch images: drawbacks

1. No shell: cannot use CMD or RUN instruction with string syntax.

```
...
FROM scratch
COPY --from=0 /go/hello .
CMD ./hello
```

```error
docker: Error response from daemon: OCI runtime create failed: container_linux.go:345: starting container process caused "exec: \"/bin/sh\": stat /bin/sh: no such file or directory": unknown.
```

Use JSON syntax (directly interpreted by docker):

CMD ["./hello"]

---

## Scratch images: drawbacks (2)

2. No tools for debugging or interacting with container (ls, ps, ping, ...) 

Workaround: instead of scratch, use busybox/alpine images. Only few MB of size.


3. No dynamic libraries (C programs are dynamically linked by default)

Workaround 1: build a static binary. Be careful: Alpine has an incompatible standard library.

Workaround 2: add libraries to the image

```
$ ldd hello
	linux-vdso.so.1 (0x00007ffdf8acb000)
	libc.so.6 => /usr/lib/libc.so.6 (0x00007ff897ef6000)
	/lib64/ld-linux-x86-64.so.2 => /usr/lib64/ld-linux-x86-64.so.2 (0x00007ff8980f7000)
```

For big projects, it's difficult to maintain.


4. Use busybox:glibc (GNU C library)

If program uses additional libraries, these libraries will need to be copied as well.

---

## Docker-slim

TODO

---

## Inspect a docker image

- `docker history <image_id>`: show layers

- `docker inspect <image_id>`: display low-level information on Docker objects

- Dive - a tool for exploring each layer in a docker image (https://github.com/wagoodman/dive)

  - show Docker image contents broken down by layer

  - indicate what's changed in each layer

  - estimate "image efficiency"

---

## Recap

- Original image built with gcc: 1.14 GB
- Multi-stage build with gcc and ubuntu: 64.2 MB
- Static glibc binary in alpine: 6.5 MB
- Dynamic binary in alpine: 5.6 MB
- Static binary in scratch: 940 kB
- Static musl binary in scratch: 94 kB

That’s a 12000x size reduction, or 99.99% less disk space.


