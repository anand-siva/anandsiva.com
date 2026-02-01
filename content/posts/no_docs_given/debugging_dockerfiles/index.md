+++
title = "Docker Debugging Playbook: When Builds and Containers Fail (Part I)"
date = 2026-01-31T16:05:21-05:00
author = "Anand Siva"

cover = "docker-debugging-playbook.png"

tags = ["devops", "containers", "docker", "troubleshooting"]
categories = ["devops"]
publications = ["No Docs Given"]

keywords = [
  "dockerfile troubleshooting",
  "docker build failed",
  "OCI runtime create failed",
  "docker run permission denied",
  "alpine vs slim"
]

description = "A practical, real-world approach to debugging Docker builds and containers that fail before they even start."

showFullContent = false
readingTime = true
hideComments = false
+++

I’ve read a lot of articles with the same click-baity tagline of **“DOCKERFILE IS DEAD.”** They usually go on to explain some new tool or utility that supposedly makes containers easier. Unsurprisingly, those posts rack up hundreds of likes and reposts.

I’ll make a promise here: if I’m ever promoting something I built, I’ll tell you up front. Maybe that gets me fewer views — whatever.

The real reason I wanted to write this article is that **people hate troubleshooting Dockerfiles**. Including me. When pipelines break because of Dockerfile failures, no one wants to touch them. And honestly, I don’t blame anyone. When you first look at Docker errors, they look insane, and Dockerfile syntax feels alien, so everyone throws their hands up and waits for “the Docker person” to debug it.

---

## What is a Dockerfile?

Honestly, if you found this article, you already know. But just to ground things:

A Dockerfile is a plain-text recipe that tells Docker exactly how to build a container image — step by step — from a base image to a runnable application, without relying on whatever happens to be installed on your machine.

---

## A simple Dockerfile

Let’s start with something simple — a Dockerfile that most people can read and understand.

```dockerfile
# Start from a very small Linux image
FROM alpine:3.20

# Set a working directory inside the container
WORKDIR /app

# Copy a simple shell script into the image
COPY hello.sh .

# Run the script when the container starts
CMD ["./hello.sh"]
```

hello.sh:

```bash
#!/bin/sh
echo "Hello from an Alpine Linux container!"
```

Build output matters

You might not like Docker’s default animated output. I usually don’t. Let’s rebuild with --progress=plain.

```bash
docker build --progress=plain -t hello-alpine .


#0 building with "desktop-linux" instance using docker driver

#1 [internal] load build definition from Dockerfile
#1 transferring dockerfile: 102B done
#1 DONE 0.0s

#2 [internal] load metadata for docker.io/library/alpine:3.20
#2 DONE 0.4s

#3 [internal] load .dockerignore
#3 transferring context: 48B done
#3 DONE 0.0s

#4 [internal] load build context
#4 transferring context: 29B done
#4 DONE 0.0s

#5 [1/3] FROM docker.io/library/alpine:3.20@sha256:a4f4213abb84c497377b8544c81b3564f313746700372ec4fe84653e4fb03805
#5 resolve docker.io/library/alpine:3.20@sha256:a4f4213abb84c497377b8544c81b3564f313746700372ec4fe84653e4fb03805 done
#5 DONE 0.0s

#6 [2/3] WORKDIR /app
#6 CACHED

#7 [3/3] COPY hello.sh .
#7 CACHED

#8 exporting to image
#8 exporting layers done
#8 exporting manifest sha256:8fd65e6075d2e0aab8cd4e6b5426ffa8e64691fc3a87751653d500ed3ea9fae7 done
#8 exporting config sha256:ef6ed67aea340cd17099b2290f4e40236a3d2d0e29a750518790d346afbd2b68 done
#8 exporting attestation manifest sha256:466a71b353b51e12e568901dee80da71b3d7e48f7a28423ceaa6201303aa7ed1 done
#8 exporting manifest list sha256:0f8c3dfdc832b7ba6124dfdd619571cc4430d74a211af049c67e3af85a879894 done
#8 naming to docker.io/library/hello-alpine:latest done
#8 unpacking to docker.io/library/hello-alpine:latest done
#8 DONE 0.0s
```

With --progress=plain:
* Every RUN command shows full stdout/stderr
* Cached steps are clearly labeled
* Errors aren’t hidden behind animated output
* CI logs become readable and searchable

This one flag alone makes Docker debugging dramatically easier.

---

## First failure: container won’t start

Now let’s run it:

```bash
docker run --rm hello-alpine
exec: "./hello.sh": permission denied
```

This error is annoying, but it teaches something important: sometimes a container is broken in a way that prevents it from starting at all. When the process in `CMD` can’t run, you don’t get a running container to poke at — it just exits immediately.

So how do you debug something that won’t even boot?

---
## Overriding CMD to debug

You don’t have to run the container using the Dockerfile’s CMD. You can override it and start the image with a different command — usually a shell.

For Linux-based images:
* Alpine → /bin/sh
* Debian/Ubuntu → /bin/bash (usually)

Let’s try bash first (this will fail on Alpine):

```bash
docker run --rm -it --entrypoint /bin/bash hello-alpine
exec: "/bin/bash": no such file or directory
```

Correct way:

```bash
docker run --rm -it --entrypoint /bin/sh hello-alpine
```

Inside container: 

```bash
/app # ./hello.sh
/bin/sh: ./hello.sh: Permission denied
```
This error is trivial, but the method is what matters.

Debugging Takeaway

**Production containers should fail fast. Debug containers should fail slow.**

In production, containers should crash immediately so orchestration systems can react.
While debugging, you want the opposite: slow failures, overridden entrypoints, and shells that stay alive long enough for a human to inspect what actually went wrong.

Fix it

The fix is simple: make the script executable.

```dockerfile
FROM alpine:3.20
WORKDIR /app
COPY hello.sh .
RUN chmod +x hello.sh
CMD ["./hello.sh"]
```

```bash
docker build --progress=plain -t hello-alpine .
docker run --rm hello-alpine

Hello from an Alpine Linux container!
```

---

## A more realistic CI/CD failure: `.dockerignore`

This one bites people all the time.

Project layout:

```bash
.
├── Dockerfile
├── config
│   └── app.conf
└── hello.sh
```

```dockerfile
FROM alpine:3.20
WORKDIR /app
COPY config/app.conf /etc/myapp/app.conf
RUN apk add --no-cache ca-certificates
COPY . .
CMD ["sh", "-c", "echo 'container started' && sleep 3600"]
```

Build it:

```bash
docker build --progress=plain -t myapp .

COPY config/app.conf: not found
```
But locally:

```bash
ls config/app.conf
```

Now check hidden files:

```bash
tree -a
.
├── .dockerignore
├── .git
```

.dockerignore:

```bash
config/
```

A `.dockerignore` file tells Docker which files and directories not to send into the build context, making builds faster and images smaller. It’s critical — but it can also break builds when you forget what you excluded.

Imagine a `.dockerignore` with 30+ lines. You’ll spin your wheels wondering why a file you know exists locally never makes it into the build.

---

## Build succeeds, container crashes: nginx example

Last example in Part I: a Dockerfile that builds fine, but crashes on startup.

```dockerfile
FROM nginx:1.27-alpine

# Create a non-root user (common hardening step)
RUN addgroup -S app && adduser -S app -G app

USER app
```

Build it:

```bash
docker build --progress=plain -t nginx-nonroot-broken .
```

Run it:

```bash
docker run nginx-nonroot-broken
```

You’ll see errors like:

```bash
mkdir() "/var/cache/nginx/client_temp" failed (13: Permission denied)
```

What happened?

The nginx image starts fine, but crashes during startup. The build passes because nothing is wrong with the image layers. The failure happens at runtime when nginx tries to write temp files and PID files to directories owned by root.

--- 

## Debugging the nginx startup

Drop into a shell:

```bash
docker run --rm -it --entrypoint /bin/sh nginx-nonroot-broken
```

Now manually run the entrypoint scripts or inspect permissions:

```bash
ls -ld /var/cache/nginx /run /etc/nginx
```

Everything is owned by root. No wonder it failed.

## The easy fix (debug mode)

If your goal is simply to make it work while debugging, don’t fight the base image. The official nginx image is designed to run as root.

```dockerfile
FROM nginx:1.27-alpine
```

Build and run:

```bash
docker build --progress=plain -t nginx-debug-root .
docker run -p 8080:80 nginx-debug-root
```

That’s it.

In real production setups, you’d pre-create and chown nginx’s runtime directories or adjust its PID and temp paths. That’s a deeper topic and outside the scope of this article.

---

## Final takeaway

Every example in this article had the same theme: Docker wasn’t broken. The failures just happened before the container started or so fast that you couldn’t observe them.

The single most useful Docker debugging skill is knowing **how to slow failures down, override startup commands, and inspect the container from the inside**. Once you do that, most “mystery” Dockerfile errors turn into boring Linux problems — permissions, paths, users, or missing files.

In Part II, I’ll dig into failures that only show up in CI and production — where builds pass locally, containers start, and everything still goes sideways.

