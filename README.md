# FeedBox volumes and networking starter

FeedBox is a minimal feedback collector with PostgreSQL (`db`), Express (`api`), and an Nginx-served frontend (`web`).

## Project structure

```text
├── server.js
├── Dockerfile
├── package.json
├── package-lock.json
├── docker-compose.yml
├── docker-compose.dev.yml
├── .env.example
└── client/
    ├── Dockerfile
    └── index.html
```

## Configure the environment

Copy the example environment file and replace the placeholder password with a local password:

```bash
cp .env.example .env
```

Do not commit `.env`.

## Run the production-style setup

```bash
docker compose up --build
```

- Frontend: http://localhost:3000
- API: http://localhost:5000/api/messages

The default Compose configuration builds the API and web images. PostgreSQL stores its data in the named `postgres_data` volume, so messages survive container recreation and `docker compose down`.

The custom `backend` bridge network connects only `api` and `db`. PostgreSQL has no published host port, so it is not directly exposed through a host port mapping. The API waits until PostgreSQL passes its health check before starting. The browser talks to the API through its published host port, so the static `web` container does not need access to the database network.

## Run with development file watching

Use the separate development configuration when you want source changes to appear without rebuilding the images:

```bash
docker compose -f docker-compose.yml -f docker-compose.dev.yml up --build
```

The development override mounts the API source into the container, keeps the container's installed `node_modules` in an anonymous volume, and runs Nodemon. It also mounts the client files read-only into Nginx. These development mounts and the file-watching command are intentionally kept out of the default configuration.

## Stop the services

```bash
docker compose down
```

To also delete the persistent PostgreSQL data volume (this permanently removes stored messages):

```bash
docker compose down -v
```

## Validate the Compose configuration

```bash
docker compose config -q
docker compose -f docker-compose.yml -f docker-compose.dev.yml config -q
```

Do not commit `.env` or `node_modules`.
