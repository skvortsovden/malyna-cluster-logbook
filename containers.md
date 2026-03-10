# CONTAINERS

## podman

install podman on the Pi:

```bash
sudo apt update
sudo apt install -y podman
```

what's installed with package:

```bash
dpkg -L podman
```

### deamonless architecture

Podman operates without a central daemon, which means that each container runs as a child process of the Podman command. When you run a container with Podman, it creates a new process for that container. For each container podman runs conmon in the background to monitor the container process. This allows Podman to manage containers without needing a long-running daemon, and it also means that if the Podman command is terminated, the containers will continue to run until they are explicitly stopped.

In contrast, Docker uses a client-server architecture where the Docker daemon (dockerd) manages all container operations. The Docker client communicates with the Docker daemon to create, start, stop, and manage containers. If the Docker daemon is not running, you cannot manage your containers.

### inactive podman systemd service

Podman installs a systemd service called `podman.service` that can be used to manage Podman containers. This service is `inactive`. It is triggered by the `podman.socket` when a request is made to the Podman API. The `podman.socket` listens for incoming requests on a Unix socket, and when a request is received, it starts the `podman.service` to handle the request. This allows Podman to run in a daemonless mode while still providing an API for managing containers.

```
devops@kalyna:~ $ systemctl status podman
○ podman.service - Podman API Service
     Loaded: loaded (/usr/lib/systemd/system/podman.service; disabled; preset: enabled)
     Active: inactive (dead)
TriggeredBy: ○ podman.socket
       Docs: man:podman-system-service(1)
```

`Socket Activation` is a mechanism in systemd that allows services to be started on demand when a request is made to a specific socket. In the case of Podman, the `podman.socket` listens for incoming requests on a Unix socket, and when a request is received, it triggers the `podman.service` to start and handle the request. This allows Podman to run in a daemonless mode while still providing an API for managing containers.


## distroless container images

Distroless images include only what an app needs to run (no shell or package manager), making them smaller and more secure. They’re typically produced via multi-stage builds that copy only runtime artifacts into a minimal base.

```Dockerfile
from docker.io/library/python:3.12-slim AS builder
workdir /app
copy hello.py .

from gcr.io/distroless/python3-debian12
workdir /app

copy --from=builder /app/hello.py .

cmd ["/app/hello.py"]
```

## layers caching order and cache invalidation

If you have a Dockerfile with multiple layers, the caching mechanism will cache each layer separately. When you build the image again, Docker will check if any of the layers have changed. If a layer has changed, Docker will invalidate the cache for that layer and all subsequent layers. This means that if you change a layer that is early in the Dockerfile, it will cause all subsequent layers to be rebuilt, even if they haven't changed. Therefore, it's important to order your Dockerfile layers in a way that minimizes changes to early layers to take advantage of caching effectively.

```Dockerfile
from docker.io/library/python:3.10-alpine
copy get.py get.py
run pip install requests
run python get.py
```

In this example, if you change the `get.py` file, it will invalidate the cache for the `copy get.py get.py` layer and all subsequent layers (`run pip install requests` and `run python get.py`). To optimize caching, you could reorder the layers like this:

```Dockerfile
from docker.io/library/python:3.10-alpine
run pip install requests
copy get.py get.py
run python get.py
```

