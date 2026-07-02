# Nginx Reverse Proxy with Flask Backend

A small containerized setup built for the DevOps screening task  two
services (Nginx and a Flask backend) running together through Docker
Compose, with Nginx forwarding traffic to the backend.

## Folder Layout

```
docker-reverse-proxy/
├── docker-compose.yml
├── README.md
├── nginx/
│   └── default.conf
└── backend/
    ├── Dockerfile
    └── app.py
```

## What's Running

| Service  | Role                                  | Internal Port |
|----------|----------------------------------------|----------------|
| backend  | Flask app returning a plain response   | 3000           |
| nginx    | Reverse proxy, public entry point      | 80             |

The backend never receives outside traffic directly — only Nginx does.
Every request hitting Nginx gets forwarded to the Flask app behind it.

## How the Two Containers Find Each Other

Docker Compose sets up a private network for every project automatically.
On that network, each service can be reached by its **service name**
instead of an IP address — so inside `nginx/default.conf`, the proxy
target is written as:

```
proxy_pass http://backend:3000;
```

`backend` here isn't a real hostname on the internet — it only resolves
inside this Compose network, pointing to whatever container is running
the `backend` service. This is handled by Docker's internal DNS, so no
manual IP configuration is needed.

Two more details worth noting:
- `backend` uses `expose` (not `ports`), so port 3000 is reachable by
  other containers on the network but not from outside the machine.
- `nginx` uses `ports: 5555:80`, making it the only container reachable
  from the host — visiting `localhost:5555` on your machine reaches port
  80 inside the Nginx container.

## Running It

```bash
cd docker-reverse-proxy
docker compose up --build
```

Once both containers are up, open:

```
http://localhost:5555
```

Expected result in the browser:

```
Hello from Backend
```

To shut everything down:

```bash
docker compose down
```

## Notes

- `depends_on` in the compose file makes sure the backend container
  starts before Nginx, since Nginx has nothing to proxy to otherwise.
- Rebuilding with `--build` ensures Docker picks up any code changes in
  the backend rather than reusing a stale cached image.

## Possible Extensions

- Swap the single backend for multiple replicas with load balancing
- Add a health check endpoint for the backend
- Rebuild the backend in Node.js instead of Flask, for comparison
- Deploy the same setup to a cloud provider (e.g. AWS, Render)