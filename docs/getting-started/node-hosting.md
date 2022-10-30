# Manual Hosting with Node.js

The recommended way to host this application is through [Docker](./docker-hosting.md), follow this guide if for some reason you do not want to use Docker.

Keep in mind that this guide assumes you have a [MongoDB](https://www.mongodb.com/) and [Redis](https://redis.io/) instances ready to use.

## Installation

### Pre requirements

First, make sure you're running at least [Node v16.15.1](https://nodejs.org/en/).

Clone this repository:

```text
git clone https://github.com/freenalytics/freenalytics
```

### Inside the `web-dashboard` folder

Install the dependencies:

```text
npm ci
```

And build the application:

```text
npm run build
```

This will create a folder named `build`. You need to move this folder into the `server` folder
and rename it to `client-build`.

```text
mv build ../server/client-build
```

### Inside the `server` folder

Install the dependencies:

```text
npm ci
```

And build the server:

```text
npm run build
```

Create a file named `.env` and add the following configuration:

```text
MONGODB_URI=
REDIS_URI=
JWT_SECRET=
JWT_TOKEN_DURATION=604800
REGISTRATION_OPEN=true
PORT=3000
```

#### Configuration

Here's a description of each of the configuration variables.

| Environment Variable | Required | Description                                                                                                                                |
|----------------------|----------|--------------------------------------------------------------------------------------------------------------------------------------------|
| MONGODB_URI          | **Yes**  | The URI of the MongoDB instance to use as database.                                                                                        |
| REDIS_URI            | **Yes**  | The URI of the Redis instance to use as cache.                                                                                             |
| JWT_SECRET           | **Yes**  | A string to use as secret to sign JWT tokens. You can use `openssl rand -hex 32` to generate one for you.                                  |
| JWT_TOKEN_DURATION   | No       | The time (in seconds) that the user JWT tokens will last for. Defaults to `604800` which is 7 days.                                        |
| REGISTRATION_OPEN    | No       | Whether users can register to create an account. By default this is set to `false`. You can enable registration by setting this to `true`. |

User account registration is disabled by default. This makes sure that nobody else can create an account to abuse the service.
It is recommended to keep this option set to `false` once all the necessary user accounts have been created.

!!! info
        Add the URIs for your MongoDB and Redis instances in their respective variables.

!!! warning "Keep in Mind"
        Make sure you have moved the `web-dashboard/build` folder into `server/client-build`.

You can now start the server with:

```text
npm start
```

The server will be listening to the port you've specified above.

## Auto-starting

If you're running a Linux based operating system, you may use something like `systemd` to create an autostart service for this application.

You can base yourself off the following service file.

```text
[Unit]
Description=Freenalytics Service 
After=network.target

[Service]
Type=simple
User=<USER>
WorkingDirectory=<SERVER_LOCATION>
ExecStart=/usr/bin/npm start
Restart=always

[Install]
WantedBy=multi-user.target
```

!!! note
        Replace `<USER>` with your Unix username and `<SERVER_LOCATION>` with the directory where the server is located (the `server` folder).

You can save this file in:

```text
sudo nano /etc/systemd/system/freenalytics.service
```

And enable it with:

```text
sudo systemctl start freenalytics.service
sudo systemctl enable freenalytics.service
```
