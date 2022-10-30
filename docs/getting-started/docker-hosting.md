# Hosting with Docker

The recommended way to host this application is through [Docker](https://www.docker.com/).

## Docker Compose

You can base yourself off of this `docker-compose.yml` file:

```yaml
version: '3.9'

services:
  freenalytics:
    image: ghcr.io/freenalytics/freenalytics:latest
    restart: unless-stopped
    depends_on:
      - mongo
      - redis
    ports:
      - '3000:3000'
    environment:
      MONGODB_URI: mongodb://root:password@mongo:27017/freenalytics?authSource=admin
      REDIS_URI: redis://redis:6379
      JWT_SECRET: MY_SUPER_SECRET
      REGISTRATION_OPEN: true

  mongo:
    image: mongo:latest
    restart: unless-stopped
    volumes:
      - ./data-mongo:/data/db
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: password

  redis:
    image: redis:latest
    restart: unless-stopped
    volumes:
      - ./data-redis:/data
    command: redis-server --loglevel warning
```

You can then start the service by running the following in the same folder where the `docker-compose.yml` file is located:

```text
docker-compose up -d
```

This will start the Freenalytics server with all the required services to run. The web dashboard will be available at `http://localhost:3000`.

## Configuration

You can configure the service with the following environment variables:

| Environment Variable | Required | Description                                                                                                                                |
|----------------------|----------|--------------------------------------------------------------------------------------------------------------------------------------------|
| MONGODB_URI          | **Yes**  | The URI of the MongoDB instance to use as database.                                                                                        |
| REDIS_URI            | **Yes**  | The URI of the Redis instance to use as cache.                                                                                             |
| JWT_SECRET           | **Yes**  | A string to use as secret to sign JWT tokens. You can use `openssl rand -hex 32` to generate one for you.                                  |
| JWT_TOKEN_DURATION   | No       | The time (in seconds) that the user JWT tokens will last for. Defaults to `604800` which is 7 days.                                        |
| REGISTRATION_OPEN    | No       | Whether users can register to create an account. By default this is set to `false`. You can enable registration by setting this to `true`. |

User account registration is disabled by default. This makes sure that nobody else can create an account to abuse the service.
It is recommended to keep this option set to `false` once all the necessary user accounts have been created.

## HTTPS

The application does not run through HTTPS by itself, it is recommended to use a reverse proxy such as [nginx](https://hub.docker.com/_/nginx) or use
something like [Cloudflare](https://www.cloudflare.com/) to expose the service through HTTPS.
