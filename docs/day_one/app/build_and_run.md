---
date: "2026-07-14 09:45"
title: "docker build vs docker run: What's the Difference"
description: "docker build and docker run, the gap between a successful build and a working container, and how to pass secrets in without baking them into the image."
---

# Building and Running Your Image

!!! tip "Part of Day One: Containerizing a Single App"
    Second article in [Containerizing a Single App](../overview.md#containerizing-a-single-app). Make sure you've written [Your First Dockerfile](first_dockerfile.md) first.

A Dockerfile is only the plan. Two commands turn it into something running: `docker build` reads the Dockerfile and produces an *image* — the packaged blueprint from [What Is a Container, Really?](../what_is_a_container.md). `docker run` takes that image and starts a *container* from it: an actual running process. They're two separate steps, not one action with two names, and keeping them separate in your head is most of what this article is about.

Here's why that separation earns its keep: a clean `docker build` only tells you the image is correct — nothing about whether the container starts, and nothing about whether the app inside it is actually doing its job. Those are three separate gates, and each one is blind to the next: an image can build perfectly and still fail to start; a container can show `Up` in every status check and still be serving nothing. This article walks through all three — build, run, verify — so a failure at any of them tells you exactly where to look instead of where to guess.

## Building the Image

`docker build` reads your Dockerfile top to bottom and executes each instruction as its own layer — the same layering [Writing Your First Dockerfile](first_dockerfile.md) explained, and the reason instruction order mattered there, is what you're about to watch happen. Run it from the directory containing your Dockerfile:

⚠️ **Caution (Creates an Image):**

``` bash title="Build the Image"
docker build -t my-api:1.0 .
# [+] Building 4.2s (11/11) FINISHED
# => [1/6] FROM docker.io/library/python:3.12-slim
# => [2/6] WORKDIR /app
# => [3/6] COPY requirements.txt .
# => [4/6] RUN pip install --no-cache-dir -r requirements.txt
# => [5/6] COPY . .
# => [6/6] RUN useradd --create-home appuser
# => exporting to image
# => => naming to docker.io/library/my-api:1.0
```

The `-t my-api:1.0` names and tags the image: `my-api` is the name, `1.0` is the version. Skip the tag and Docker defaults to `latest`, which you already know to avoid pinning to in anything but the loosest local experiments. The trailing `.` tells Docker where to find the Dockerfile and the build context: the current directory.

!!! info "In Practice, You Won't Type This by Hand"
    Running `docker build` from your own terminal is how you learn what the command actually does — it's not how a working team ships. In practice, this exact command runs inside a CI pipeline (GitHub Actions, GitLab CI, Jenkins, whatever your team uses), triggered automatically the moment code merges, not by a person sitting there typing it. A CI job is really just this same `docker build`, plus the `docker push` from [Sharing It With Your Team](sharing_with_your_team.md), running on a server instead of your laptop, with the same flags either way. Knowing what the command does by hand is what makes reading — and eventually writing — that CI pipeline make sense, instead of it being a wall of YAML that happens to work.

The terminal reporting success is a good sign, but it's not proof by itself — confirm the image actually landed.

✅ **Safe (Read-Only):**

``` bash title="Confirm the Image Exists"
docker images my-api
# REPOSITORY   TAG    IMAGE ID       CREATED         SIZE
# my-api       1.0    a1b2c3d4e5f6   6 seconds ago   142MB
```

That's the blueprint from [What Is a Container, Really?](../what_is_a_container.md) sitting on disk. Nothing has run yet — which means gate one is cleared, but it hasn't told you anything about gate two.

## Running the Container

Gate two: does the image actually start and turn into a working container? [What Is a Container, Really?](../what_is_a_container.md#image-vs-container-the-blueprint-and-the-building) drew the line between the blueprint sitting on disk and the building it becomes once someone acts on it. `docker run` is the command that does the acting: it takes the image, executes it as a real process, and gives that process a writable layer of its own to work in.

⚠️ **Caution (Starts a Running Process):**

``` bash title="Run the Container"
docker run -d -p 8000:8000 --name my-api-dev my-api:1.0
# 7f8e9d0c1b2a...
```

That long string of characters is the new container's ID. It's Docker confirming a process actually started — not yet a confirmation that it's healthy, or that anything can reach it. Three flags did the real work in that command, and it's worth knowing exactly what each one bought you:

- `-d` runs the container detached — in the background, handing you back your terminal so it doesn't just sit there tied up
- `-p 8000:8000` publishes port 8000 on your machine, forwarding it to port 8000 inside the container (the port your app's `EXPOSE` line documented)
- `--name my-api-dev` gives the container a name you can refer to in later commands, instead of the random one Docker would generate for you

The `-p` flag is the one that trips people up first. Without it, the container's port exists only *inside* the container's own network namespace — nothing on your machine can reach it, even though the app is running and listening just fine from its own point of view. The diagram below is worth pausing on for that exact reason.

```mermaid
flowchart TD
    CURL["curl localhost:8000<br/>(your machine)"]
    subgraph HOST["Host"]
        HP["Host port 8000<br/>(claimed by -p 8000:8000)"]
    end
    subgraph CONT["Container network namespace"]
        CP["Container port 8000"]
        APP["Your app<br/>listening on 0.0.0.0:8000"]
    end
    NOPORT["Container run without -p"]

    CURL --> HP -->|"forwarded by Docker"| CP --> APP
    CURL -.->|"no route in —<br/>port never published"| NOPORT

    style CURL fill:#2d3748,stroke:#cbd5e0,stroke-width:2px,color:#fff
    style HP fill:#d97706,stroke:#cbd5e0,stroke-width:2px,color:#fff
    style CP fill:#2d3748,stroke:#cbd5e0,stroke-width:2px,color:#fff
    style APP fill:#2f855a,stroke:#cbd5e0,stroke-width:2px,color:#fff
    style NOPORT fill:#1a202c,stroke:#cbd5e0,stroke-width:2px,color:#fff
```

### Passing In What the Image Shouldn't Contain

A container that's merely running isn't necessarily a container that's *correctly configured* — gate two isn't finished until the app has what it needs to actually do its job. [Writing Your First Dockerfile](first_dockerfile.md) said secrets don't belong baked into the image; this is where they go instead: passed in at run time, so they exist only in the running container, never in the image itself.

``` bash title="Pass Environment Variables at Run Time"
docker run -d -p 8000:8000 \
  -e DATABASE_URL="postgres://user:pass@db-host:5432/app" \
  -e API_KEY="a-real-key-goes-here" \
  --name my-api-dev my-api:1.0
```

For more than a couple of values, an env file keeps the command readable and keeps the values out of your shell history:

``` text title=".env.local"
DATABASE_URL=postgres://user:pass@db-host:5432/app
API_KEY=a-real-key-goes-here
```

``` bash title="Load Them From a File"
docker run -d -p 8000:8000 --env-file .env.local --name my-api-dev my-api:1.0
```

!!! warning "Don't Commit the Env File"
    `.env.local` holds real credentials, so add it to both `.gitignore` and [`.dockerignore`](first_dockerfile.md#keep-the-build-context-small). This is a local convenience, not a production secrets strategy; the Efficiency tier will cover the real thing.

With the container running, named, and configured, gate two is genuinely cleared. Gate three is the one that actually matters to whoever's waiting on this — not "did a process start," but "does the app work."

## Verifying It Actually Works

This is the gap from the opening: `docker ps` reporting a healthy-looking container was never the same promise as the app inside it responding correctly. Two separate checks close that gap.

✅ **Safe (Read-Only):**

``` bash title="Check the Container Is Running"
docker ps
# CONTAINER ID   IMAGE          COMMAND           STATUS         PORTS                    NAMES
# 7f8e9d0c1b2a   my-api:1.0     "python app.py"   Up 8 seconds   0.0.0.0:8000->8000/tcp   my-api-dev
```

`STATUS: Up` means the process started and hasn't exited. It does **not** mean your app is actually serving requests correctly. That's the next check:

``` bash title="Hit the App"
curl http://localhost:8000/health
# {"status": "ok"}
```

If that comes back empty, hangs, or connection-refuses, `docker ps` was lying to you in the sense that matters — the process is up, but something inside it isn't working. [Debugging a Container That Won't Behave](debugging.md) is exactly this situation.

``` bash title="Watch the Logs"
docker logs my-api-dev
# Starting server on port 8000...
# [INFO] Application startup complete
```

All three gates open, in order: the image built, the container started and stayed up, and the app answered. That's the sequence to run through every time something feels wrong — which gate actually failed is almost always more useful information than "it's not working."

## Watch Out For

- **"Port is already allocated."** Something on your machine (often a previous container you forgot was running) already owns port 8000. `docker ps` to find it, or publish to a different host port: `-p 8001:8000`.
- **Editing code and expecting the running container to pick it up.** It won't. A container runs the image it was built from; a code change means rebuild the image (`docker build`), then replace the container (stop the old one, run a new one from the new image).
- **Expecting anything the app writes to survive the container.** The writable layer from [What Is a Container, Really?](../what_is_a_container.md#image-vs-container-the-blueprint-and-the-building) is deleted along with the container. Logs to stdout are safe (`docker logs` reads those), but files written inside the container are gone when it's removed. If your app produces data that has to outlive the container, that needs deliberate handling — volumes, which Efficiency will cover; for Day One, treat the container's filesystem as disposable.
- **Forgetting the container is still running.** `-d` means it keeps running after your terminal command returns, consuming resources, until you stop it. `docker stop my-api-dev` when you're done with a session.

## Practice Exercises

??? question "Exercise 1: Rebuild and Replace"
    Change one line in your app (anything — a log message, a response string), then get the running container to reflect that change.

    ??? tip "Solution"
        ``` bash title="Rebuild, Stop, Replace"
        docker build -t my-api:1.1 .
        docker stop my-api-dev
        docker rm my-api-dev
        docker run -d -p 8000:8000 --name my-api-dev my-api:1.1
        ```

        Bumping the tag (`1.0` → `1.1`) isn't required, but it makes it obvious which image version is actually running, and it means `docker images` still shows you the old one in case you need to roll back to it.

??? question "Exercise 2: Find the Port Conflict"
    Start a container publishing port 8000. Without stopping it, try to start a second container also publishing port 8000. Read the error, then fix it by publishing the second container on a different host port.

    ??? tip "Solution"
        The second `docker run -p 8000:8000 ...` fails with something like `Bind for 0.0.0.0:8000 failed: port is already allocated`. The host port is a single resource, and Docker won't silently share it. Fix: `docker run -p 8001:8000 ...` on the second container, then reach it at `localhost:8001` instead. The *container's* internal port (`8000`) can be identical across both; only the host side has to be unique.

## Quick Recap

| Command | What It Does |
|---------|---------------|
| `docker build -t name:tag .` | Builds an image from the Dockerfile in the current directory |
| `docker images` | Lists images that exist locally |
| `docker run -d -p host:container --name x image` | Starts a container, detached, with a port published |
| `-e KEY=value` / `--env-file` | Passes configuration and secrets in at run time, not build time |
| `docker ps` | Shows running containers and their status |
| `docker logs <name>` | Shows a container's stdout/stderr output |
| `docker stop <name>` | Stops a running container |

## What's Next

Sometimes one of the three gates doesn't open — the container exits immediately, or it runs but nothing responds. **[Debugging a Container That Won't Behave](debugging.md)** picks up exactly there, using this same build → run → verify sequence to figure out which gate actually failed.

## Further Reading

### Official Documentation

- [`docker build` Reference](https://docs.docker.com/reference/cli/docker/build/) — full flag reference
- [`docker run` Reference](https://docs.docker.com/reference/cli/docker/container/run/) — full flag reference

### Related Articles

- [Writing Your First Dockerfile](first_dockerfile.md) — the image this article builds and runs
- [Day One Overview](../overview.md) — see both Day One pathways
